At [GIGO](https://gigo.dev), we're all about helping people learn. If there's one thing we're crazy about, it's making education better for everyone. And let's be honest, homework usually isn't that helpful. That's why we made Homework Helper – because sometimes, you just want your homework done for you.

![HH](images/bookhomeowkrhelperPNG.png)

For those of you that don't know, Homework Helper (HH) is an AI software engineer that will do your homework for you. HH isn't your average chatbot. Specializing in math and programming assignments, HH comes equipped with a suite of tools to deliver insightful solutions. A standout feature is the dedicated GIGO DevSpace, enabling HH to run, debug, and iteratively refine code across most common programming languages. Moreover, HH's capability to install third-party packages and access the internet sets it apart from standard chatbots, allowing it to address a wide range of programming challenges with ease.

I'll be honest, Homework Helper is a marketing toy. We hope that users will use it and then choose to learn on our platform when they have free time. Even though this may seem counter-intuitive, we believe there are many people that want to learn and practice cool skills that also hate doing homework. If all you care about is getting an AI bot to do your homework for you, you probably won't care about the rest of this article.

Let's get down to brass tacks – let's look at some of the primary points of the system:
    1. GIGO DevSpace
    2. Code Teacher Integration
    3. User Interface

### GIGO DevSpace.

<Image of editor + terminal>

GIGO's all about making coding as hassle-free as possible. Whenever you run code on our platform – be it Bytes, Challenges, or Journeys – we set you up with a Linux container right in our cloud. This setup's flexible enough to handle pretty much any system dependency or config you need.

Coding's a trial-and-error deal for us humans, so we figured HH should get in on that action too. That's why we've equipped it with its very own GIGO DevSpace. Whenever HH starts banging out code, we ping our main system, and voila – a new container's whipped up in our Kubernetes cluster, all for HH's use.

Once we've got a DevSpace up and running, the first program launched is the GIGO agent. This little program (written in [go](https://go.dev), forked from [coder](https://github.com/coder/coder)) gets the environment ready and offers up a WebSocket API for the fancy stuff. HH hooks up to this via a [zitinet](https://github.com/openziti/ziti) connection, allowing it to run code and stream outputs in real-time.

Right now, the agent can handle a bunch of languages – Python, JS/TS, Go, Rust, C++, and C#, with more on the way. Our endgame? A one-stop API that lets you run almost any language you can think of in the DevSpace. Sure, it's a bit of a challenge with all the different dependencies and tools, but we're working on it.

### Code Teacher Integration

<Image of CT chat in HH>

Let's not forget about the brain behind the operation – Code Teacher (CT). CT's our in-house AI that's designed to offer personalized learning assistance for users on GIGO. But with Homework Helper, CT's got a new gig: completing your homework to in the most human-like fashion possible.

Tweaking an AI system to do your homework without looking like the product of an AI system isn't easy. This involves sophisticated prompting strategies, custom finetuning, and additional systems to ensure the homework reflects the student's personal style. While the Pro version of HH offers enhanced disguise capabilities, we're working towards making these features available to all users within financial constraints.

### User Interface

<Image of complete HH UI>

Finally, the piece de resistance, the main user interface of Homework Helper.

We discussed the HH UI a lot internally. There were strong opinions all around but we believe we picked the best of the variety and have a good starting point.

The main page of the HH provides a QA-like chat window with a code editor on the right-hand side. Our thought is that as we better understand how users utilize HH, we can better adapt the experience for their uses but as of now, the clunky-but-versatile chat interface allows users to show us what they want out of HH.

The dressed up parts of HH include an in-browser code editor with syntax highlighting for most common languages, multi-file support, and the ability to execute the code. In addition to the code editor is a console window to present the output of the code executions directly in the interface. To tie the experience in with the chat experience we have allowed for CT to ask for permission to execute the code and view the output on your behalf. This way CT can (at the permission of the user) enter into an iterative improvement cycle to ensure your homework is completed to satisfaction.

### Closing

Overall, Homework Helper is a new part of the GIGO platform and we are still exploring the experience that we'd like to provide. With that said, the fastest way for us to provide a great experience is to receive feedback on how users make use of it. So head on over to HH on GIGO and let us know what you think!
[Homework Helper](https://www.gigo.dev/homework)
