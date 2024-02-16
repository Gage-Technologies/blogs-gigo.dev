
## How GIGO uses NATS to talk across its cluster

TLDR: We used NATS Jetstream to connect nodes in GIGO for task delegation, DevSpace status updates, chat, and a lot more. I also made a tutorial and playground on GIGO so you can learn how to work with Jetstream or play around with an existing chat application.
Tutorial: [https://gigo.dev/challenge/1716524448963624960](https://gigo.dev/challenge/1716524448963624960)
Chat Playground: [https://www.gigo.dev/challenge/1717005804785106944](https://www.gigo.dev/challenge/1717005804785106944)

![](https://cdn-images-1.medium.com/max/6400/1*v1PyulxXc5_JSf8n5JoXJg.jpeg)

GIGO emerged as a comprehensive platform to tackle the trifecta of challenges often encountered by new developers: the search for meaningful projects, the complexity of setting up development environments, and maintaining motivation during tough hurdles. Created by a team of self-taught developers, GIGO aims to alleviate the same pain points that we once faced.

Within GIGO, users find a range of projects tailored to their preferences. Upon selecting a project, they gain access to DevSpace, a pre-configured environment equipped with a full-fledged web-based VSCode editor. This eliminates the daunting setup process that often discourages newcomers. Beyond that, DevSpace offers an array of smart tools including interactive tutorials and an intelligent assistant named Code Teacher to help navigate the complexities of coding projects.

From the inception of the idea, one principle remained constant: the need for a high-availability (HA) system. Building an HA infrastructure isn’t straightforward when nodes must communicate complex tasks, such as DevSpace management, real-time status updates, and chat functionalities. That’s where NATS Jetstream enters the picture. Jetstream isn’t merely a messaging broker; it’s an integral component of our HA architecture. It manages heavy lifting when it comes to inter-node communications, enabling features like task delegation, real-time status updates, and a seamless chat experience across the platform.

In this article, we’ll dissect Jetstream’s role in GIGO by focusing on:

1. Work Queues: Efficient task distribution to follower nodes

2. Status Updates: Real-time DevSpace statuses delivered to users

3. Chat Messages: Seamless, platform-wide chat experiences

## Work Queues

In Jetstream, a work queue acts like a specialized delivery service, sending a task to only one chosen node. These queues scatter tasks to the cluster, and the speediest node snags the gig. In GIGO, we lean heavily on these work queues, most critically for orchestrating DevSpaces.

When someone on GIGO kick-starts a DevSpace, a message zooms through Jetstream, tapping a worker to roll out the red carpet and create that DevSpace. Since GIGO is open-source we get to show you exactly how it works:

    // format create workspace request and marshall it with gob
    buf := bytes.NewBuffer(nil)
    encoder := gob.NewEncoder(buf)
    err = encoder.Encode(models2.CreateWorkspaceMsg{
      WorkspaceID: workspace.ID,
      OwnerID:     callingUser.ID,
      OwnerEmail:  callingUser.Email,
      OwnerName:   callingUser.UserName,
      Disk:        wsConfig.Resources.Disk,
      CPU:         wsConfig.Resources.CPU,
      Memory:      wsConfig.Resources.Mem,
      Container:   wsConfig.BaseContainer,
      AccessUrl:   accessUrl,
    })
    if err != nil {
      _ = tx.Rollback()
      return nil, fmt.Errorf("failed to encode workspace: %v", err)
    }
    
    // send workspace create message to jetstream so a follower will
    // create the workspace
    _, err = js.PublishAsync(streams.SubjectWorkspaceCreate, buf.Bytes())
    if err != nil {
      _ = tx.Rollback()
      return nil, fmt.Errorf("failed to send workspace create message to jetstream: %v", err)
    }
[**gigo-core/src/gigo/api/external_api/core/workspace.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e ·…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/api/external_api/core/workspace.go#L392C1-L417C3)

In the Jetstream ecosystem, worker nodes stand ready, poised to catch incoming tasks. When a message from Jetstream reaches one of these nodes, it’s an immediate call to action. The worker dives into the task, diligently setting up the user’s DevSpace.

    func asyncCreateWorkspace(nodeId int64, tidb *ti.Database, wsClient *ws.WorkspaceClient, wsStatusUpdater *utils.WorkspaceStatusUpdater,
     js *mq.JetstreamClient, msg *nats.Msg, logger logging.Logger) {
      ctx, span := otel.Tracer("gigo-core").Start(context.TODO(), "async-create-workspace-routine")
      callerName := "asyncCreateWorkspace"
      
      // unmarshall create workspace request message
      var createWsMsg models2.CreateWorkspaceMsg
      decoder := gob.NewDecoder(bytes.NewBuffer(msg.Data))
      err := decoder.Decode(&createWsMsg)
      if err != nil {
        logger.Errorf("(workspace: %d) failed to decode create workspace message: %v", nodeId, err)
        return
      }
      
      // log that we started provisioning
      logger.Infof("(workspace: %d) creating workspace %d", nodeId, createWsMsg.WorkspaceID)
    
      // more code
    }
    
    // more code
    
    // process workspace create stream
    processStream(
      nodeId,
      js,
      workerPool,
      streams.StreamWorkspace,
      streams.SubjectWorkspaceCreate,
      "gigo-core-follower-ws-create",
      time.Minute*10,
      "workspace",
      logger,
      func(msg *nats.Msg) {
        asyncCreateWorkspace(nodeId, tidb, wsClient, wsStatusUpdater, js, msg, logger)
      },
    )
[**gigo-core/src/gigo/subroutines/follower/workspace_management.go at…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/subroutines/follower/workspace_management.go#L834C1-L848C3)
[**gigo-core/src/gigo/subroutines/follower/workspace_management.go at…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/subroutines/follower/workspace_management.go#L188C1-L203C88)

Jetstream brings a lot of value to the workspace management system. Starting a DevSpace takes a little bit of time, and can be a pretty involved process for whatever node is managing the start. When a user clicks the launch button, there’s no guarantee they are talking to the best node for the job. Additionally, we want to reply to their API call quickly. By using Jetstream we can kill both birds with one stone (metaphorically, we don’t kill birds at GIGO).

When the user clicks the launch button we fire the message across the Jetstream which is a near instantaneous process. This gives the UI/UX a snappy feel and it allows us to move the user onto the LaunchPad page faster while they wait for their DevSpace. On the other side, we can load balance the DevSpace creation across nodes. We can confidently assume that if a node is able to receive another job, the node has enough bandwidth to manage the DevSpace creation. So now, instead of queuing the user’s DevSpace behind a bunch of other users, we can trust that a less busy node can get started immediately!

## Status Updates

The life of a GIGO DevSpace is filled with changes. DevSpaces are born into a rapid set of changes, from being provisioned on a K8s cluster, to cloning your Challenge, launching background Docker containers, and so much more! We want the user to see each step of this process so that they can appreciate the hard work we put into GIGO to make the experience seamless. We also want Challenge creators to have insight into how long their DevSpace takes to start so they can optimize slow launches.

To efficiently update the user at each step we chose to use Jetstream for status updates. Whenever a part of the code base updates the state of a DevSpace, the WorkspaceStatusUpdater pushes the new status through the Jetstream to the end user. Here’s an example of when the DevSpace agent updates the startup state of the DevSpace:

    func WorkspaceInitializationStepCompleted(ctx context.Context, tidb *ti.Database, wsStatusUpdater *utils.WorkspaceStatusUpdater, wsId int64,
     step models.WorkspaceInitState) error {
     ctx, span := otel.Tracer("gigo-core").Start(ctx, "workspace-initialization-step-completed-core")
     defer span.End()
     callerName := "WorkspaceInitializationStepCompleted"
    
     // default workspace state to starting but set to Active if this is the VSCodeLaunch init step
     state := models.WorkspaceStarting
     if step == models.WorkspaceInitCompleted {
      state = models.WorkspaceActive
     }
    
     // perform update directly on the workspace row
     _, err := tidb.ExecContext(ctx, &span, &callerName,
      "update workspaces set init_state = ?, state = ?, last_state_update = ? where _id = ?",
      step, state, time.Now(), wsId,
     )
     if err != nil {
      // handle not found
      if err == sql.ErrNoRows {
       return fmt.Errorf("workspace not found")
      }
      // handle error
      return fmt.Errorf("failed to update workspace: %v", err)
     }
    
     // push workspace status update to subscribers
     wsStatusUpdater.PushStatus(ctx, wsId, nil)
    
     return nil
    }
[**gigo-core/src/gigo/api/external_api/core/workspace.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e ·…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/api/external_api/core/workspace.go#L1329)

The agent sends incremental updates about the state of the DevSpace to the WorkspaceInitializationStepCompleted function which then uses the PushStatus method to relay those updates to the user.

    func (u *WorkspaceStatusUpdater) PushStatus(ctx context.Context, id int64, ws *models.Workspace) error {
     // retrieve the workspace if we did not receive one
     if ws == nil {
      var err error
      ws, err = u.getWorkspace(ctx, id)
      if err != nil {
       return err
      }
     }
    
     // increment init state before writing the update to represent the step that we
     // are working on unless we are finished then we return the true code
     if ws.InitState != models.WorkspaceInitCompleted {
      ws.InitState++
     }
    
     // gob encode the status update
     buf := bytes.NewBuffer(nil)
     encoder := gob.NewEncoder(buf)
     err := encoder.Encode(models2.WorkspaceStatusUpdateMsg{
      Workspace: ws.ToFrontend(u.Hostname, u.Tls),
     })
     if err != nil {
      return fmt.Errorf("failed to serialize workspace status update: %v", err)
     }
    
     // send workspace status update message to jetstream so any listening websocket
     // connections can properly update the client for the current state of the workspace
     _, err = u.Js.PublishAsync(fmt.Sprintf(streams.SubjectWorkspaceStatusUpdateDynamic, ws.ID), buf.Bytes())
     if err != nil {
      return fmt.Errorf("failed to send workspace status update message to jetstream: %v", err)
     }
    
     return nil
    }
[**gigo-core/src/gigo/utils/workspace_status_updater.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e ·…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/utils/workspace_status_updater.go#L77C1-L111C2)

On the other side of the Jetstream, we have a WebSocket connected to the GIGO website that is patiently waiting to forward our status updates to the user.

     // format the message and send it to the client
     p.outputChan <- ws.PrepMessage[any](
      msg.SequenceID,
      ws.MessageTypeWorkspaceStatusUpdate,
      status,
     )
    
     // exit if we already have a subscription for this workspace
     p.mu.Lock()
     defer p.mu.Unlock()
     if _, ok := p.wsSubs[workspaceId]; ok {
      return
     }
    
     // create a subscriber to workspace status events
     sub, err := p.s.jetstreamClient.Subscribe(
      fmt.Sprintf(streams.SubjectWorkspaceStatusUpdateDynamic, workspaceId),
      p.workspaceStatusCallback,
     )
     if err != nil {
      p.socket.logger.Errorf("failed to create workspace status subscriber: %v", err)
      // handle internal server error via websocket
      p.outputChan <- ws.PrepMessage[any](
       msg.SequenceID,
       ws.MessageTypeGenericError,
       ws.GenericErrorPayload{
        Code:  ws.ResponseCodeServerError,
        Error: "internal server error occurred",
       },
      )
      return
     }
    
     // save the subscription
     p.wsSubs[workspaceId] = sub
[**gigo-core/src/gigo/api/external_api/workspace_plugin.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/api/external_api/workspace_plugin.go#L145C1-L179C29)

When a user opens the LaunchPad page on GIGO a subscription request is sent over the WebSocket and executes the code chunk above. Status updates like the one we show above are then consumed by the workspaceStatusCallback method and forwarded to the end user.

## Chat Messages

One of the big draws of GIGO is its versatile chat system. It’s not just a global chat we’re talking about here — every coding challenge gets its chat room, and users can start direct messages or even make their private group chats. Building a backend that can smoothly handle all this chat action is pretty complex. That’s where Jetstream comes into play once again.

Jetstream has this neat feature where it can subdivide subjects dynamically if you set it up with a wildcard. This is super handy for our chat setup because we have to isolate the chat streams for every type of chat room, whether it’s tied to a specific challenge or a group chat that users decide to create.

When a user kicks off a new chat, we do the typical things like creating a new database row. But we also add an extra layer by subscribing to a segmented Jetstream subject. This ensures the user receives a real-time stream of messages specific to that chat. On top of that, we use Jetstream to send a broadcast message to all other users in the new chat, so their sessions get the memo to connect to the new chat’s Jetstream subject. We’ll delve deeper into how this works in the sections that follow.

    func (p *WebSocketPluginChat) handleNewChatMessage(msg *ws.Message[any]) {
     // boring validation stuff
    
     // execute core function logic
     chat, event, err := core.CreateChat(p.ctx, p.s.tiDB, p.s.sf, p.socket.user.Load(), createChatParams)
     if err != nil {
      p.socket.logger.Errorf("failed to execute create chat internal: %v", err)
      // handle internal server error via websocket
      p.outputChan <- ws.PrepMessage[any](
       msg.SequenceID,
       ws.MessageTypeGenericError,
       ws.GenericErrorPayload{
        Code:  ws.ResponseCodeServerError,
        Error: "internal server error occurred",
       },
      )
      return
     }
    
     // some code...
    
     // subscribe to the new chat stream
    
     // create an async subscriber to broadcast init messages
     sub, err := p.s.jetstreamClient.Subscribe(
      fmt.Sprintf(streams.SubjectChatMessagesDynamic, chat.ID),
      p.chatMsgHandler,
      nats.AckExplicit(),
      nats.DeliverNew(),
     )
     if err != nil {
      p.socket.logger.Errorf("failed to subscribe to new chat stream: %v", err)
      return
     }
    
     // save the chat and subscription to the map
     p.mu.Lock()
     p.subs[chat.ID] = sub
     p.chats[chat.ID] = chat
     p.mu.Unlock()
    
     // marshall message using gob
     var buf bytes.Buffer
     encoder := gob.NewEncoder(&buf)
     err = encoder.Encode(models2.NewChatMsg{
      Chat: *chat,
     })
     if err != nil {
      p.socket.logger.Errorf("failed to encode message: %v", err)
      return
     }
    
     // save the buffer bytes to a variable
     chatBytes := buf.Bytes()
    
     // forward the message to all the active users so they can subscribe to the chat
     for _, user := range chat.Users {
      // skip if the user is the calling user since me just manually subscribed
      if user == p.socket.user.Load().ID {
       continue
      }
    
      _, err = p.s.jetstreamClient.Publish(
       fmt.Sprintf(streams.SubjectChatNewChatDynamic, user),
       chatBytes,
      )
      if err != nil {
       p.socket.logger.Errorf("failed to publish message to chat stream: %v", err)
      }
     }
    
     // if the event is nil exit
     if event == nil {
      return
     }
    
     // iterate over all the users including and send the event
    
     // marshall message using gob
     var eventBuf bytes.Buffer
     encoder = gob.NewEncoder(&eventBuf)
     err = encoder.Encode(event)
     if err != nil {
      p.socket.logger.Errorf("failed to encode message: %v", err)
      return
     }
    
     // save the buffer bytes to a variable
     eventBytes := eventBuf.Bytes()
    
     // forward the message to all the active users so the new chat is populated in their UI
     for _, user := range chat.Users {
      // skip if the user is the calling user since me just manually subscribed
      if user == p.socket.user.Load().ID {
       continue
      }
    
      _, err = p.s.jetstreamClient.Publish(
       fmt.Sprintf(streams.SubjectChatUpdatedDynamic, user),
       eventBytes,
      )
      if err != nil {
       p.socket.logger.Errorf("failed to publish message to chat stream: %v", err)
      }
     }
    }
[**gigo-core/src/gigo/api/external_api/chat_plugin.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e ·…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/api/external_api/chat_plugin.go#L503)

So, let's dig into what’s going on here. First, we create a new chat and save it to the database, pretty standard stuff. But in the next part, we bring Jetstream into the picture. So once we have a new chat, we have 2 problems.

1. None of the other users in the chat know about the new chat.

2. The other user’s UI hasn’t been updated to show the new chat.

To fix those problems we use Jetstream. Now that we have a chat we forward that chat through the NewChat subject to all of the users that are in the chat. We can do this by using the base NewChat subject and appending the user’s IDs to the subject like this CHAT.NewChat.123456789 . This is the dynamic segmentation of the subjects I mentioned earlier. Jetstream will now create a new segment of the NewChat subject and only users listening to that segment will receive the messages being sent. This happens so fast that the operation can actually be performed in sync with the rest of the function. We also perform a very similar operation on the CHAT.ChatUpdated subject which is used to relay chat updates to the UI. That way we fix problem number 2 by instructing the UI to add the new chat to the chat panel for all of the users in the chat.

Now that we’ve seen what it looks like to ship updates to the other users, let’s take a look at the other side. The receiving end of the NewChat update:

    // create function to handle stream messages for new chats
      newChatHandler := func(m *nats.Msg) {
       // we always ack the message
       _ = m.Ack()
    
       // parse the message
       var newChat models2.NewChatMsg
       decoder := gob.NewDecoder(bytes.NewBuffer(m.Data))
       err := decoder.Decode(&newChat)
       if err != nil {
        s.logger.Errorf("failed to decode new chat message: %v", err)
        return
       }
    
       // skip if we already have a subscription for this chat
       lock.Lock()
       defer lock.Unlock()
       if _, ok := subs[newChat.Chat.ID]; ok {
        return
       }
    
       // create an async subscriber to broadcast init messages
       sub, err := s.jetstreamClient.Subscribe(
        fmt.Sprintf(streams.SubjectChatMessagesDynamic, newChat.Chat.ID),
        chatMessageHandler,
        nats.AckExplicit(),
        nats.DeliverNew(),
       )
       if err != nil {
        s.logger.Errorf("failed to subscribe to new chat stream: %v", err)
        return
       }
    
       subs[newChat.Chat.ID] = sub
       chatMap[newChat.Chat.ID] = &newChat.Chat
    
       // send the message to the client to add the chat
       outputChan <- ws.PrepMessage[any](
        "",
        ws.MessageTypeNewChatBroadcast,
        newChat.Chat.ToFrontend(),
       )
      }
    
      // create an async subscriber to broadcast init messages
      activeChatStream, err = s.jetstreamClient.Subscribe(
       fmt.Sprintf(streams.SubjectChatNewChatDynamic, callingUser.ID),
       newChatHandler,
       nats.AckExplicit(),
       nats.DeliverNew(),
      )
      if err != nil {
       return nil, fmt.Errorf("failed to subscribe to new chat stream: %w", err)
      }
[**gigo-core/src/gigo/api/external_api/chat_plugin.go at 2c914e320f3b0d6db5619558e3a1e5cdb376e42e ·…**
*Core backend system for https://gigo.dev. Contribute to Gage-Technologies/gigo-core development by creating an account…*github.com](https://github.com/Gage-Technologies/gigo-core/blob/2c914e320f3b0d6db5619558e3a1e5cdb376e42e/src/gigo/api/external_api/chat_plugin.go#L163C1-L216C4)

This code block is executed when the user first opens the GIGO platform. A new WebSocket connection is established with the backend and this newChatHandler callback is connected to the same CHAT.NewChat.123456789 subject segment that we used in the prior code chunk. Whenever a new chat message is sent through the segment the newChatHandler is executed and subscribes to the new chat so that the user immediately starts to receive updates from the new chat.

We can see that Jetstream is used at many levels to create a seamless chat experience regardless of which server a user is talking to. The real benefit Jetstream brings over other messaging systems is the dynamic and configurable functionality. Being able to instantaneously create a segment of the message stream for a single chat with no rebalancing cost is a major value add for any low latency system.

## Closing

We’ve had a blast building GIGO and one of the perks of being open-source is the freedom to openly discuss our technology. This paves the way for more deep dives into the challenges we’ve conquered, the solutions we’ve deployed, and the strategies we’ve committed to. Keep an eye out — there’s a lot more where this came from, and we can’t wait to share it with you.

Here are the links to the Jetstream tutorial and the chat application playground on GIGO.
Tutorial: [https://gigo.dev/challenge/1716524448963624960](https://gigo.dev/challenge/1716524448963624960)
Chat Playground: [https://www.gigo.dev/challenge/1717005804785106944](https://www.gigo.dev/challenge/1717005804785106944)
