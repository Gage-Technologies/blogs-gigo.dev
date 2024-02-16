11-1-2023
## GIGO and VS-code: the Battle With Microsoft

When our team embarked on the journey to create GIGO, we recognized the imperative need for a customized development environment. This meant using VS-code.

While, on the surface, this decision might seem commonplace, anyone who has delved into the intricacies of developing extensions for VS Code knows it’s a challenging endeavor. Yet, the versatility of VS Code, when combined with a skilled developer, allows for profound and game-changing modifications through extensions.

The problem arises when a developer decides to perform a modification that vscode never intended the developer to modify. I am that developer.

Let’s paint the scene of this battle. The goal is to make a full suite extension that allows users to read and edit tutorials for programming projects inside of vscode. The enemy: vscode.

![A lone GIGO developer stands before the Goliath](https://cdn-images-1.medium.com/max/2048/1*woPzqUKfiK3f_v0E4Igz9w.png)

The extension must be able to display tutorials and allow them to be easily editable. This would allow both creators and learners an easy and seamless experience.

## **The Quest to Display Tutorials**

So I began, fool-heartedly attempting to build a system to render markdown from a dedicated tutorial file.

After many trials and tribulations, I couldn’t crack it. It seemed every implementation was either too inefficient and slowed vscode down, or too incomplete to cover a wide range of languages.

Finally the vanguard had arrived. A markdown renderer named [Shiki](https://github.com/shikijs/shiki).

    <script src="https://unpkg.com/shiki"></script>
    <!-- or -->
    <script src="https://cdn.jsdelivr.net/npm/shiki"></script>
    
    <script>
      shiki
        .getHighlighter({
          theme: 'nord',
          langs: ['js'],
        })
        .then(highlighter => {
          const code = highlighter.codeToHtml(`console.log('shiki');`, { lang: 'js' })
          document.getElementById('output').innerHTML = code
        })
    </script>

My implementation:

    //findMDFiles finds all markdown files in the workspace folder
        public async findMDFiles(): Promise<any[]> {
            var mdArr: any[] = [];
            try {
                const fs = require('fs');
                const markdown = require('markdown-it');
                const shiki = require('shiki');
                var md: any;
    
                //use shikit to render markdown syntax and get all markdown files
                await shiki.getHighlighter({
                    theme: 'github-dark'
                }).then((highlighter: { codeToHtml: (arg0: any, arg1: { lang: any; }) => any; }) => {
                    //render markdown with shiki highlighter
                    const md = markdown({
                        html: true,
                        highlight: (code: any, lang: any) => {
                            try {
                                return highlighter.codeToHtml(code, { lang });
                            } catch (err) {
                                console.log(`failed to highlight for "${lang}": ${err}`);
                                lang = "txt";
                                return highlighter.codeToHtml(code, { lang });
                            }
                        }
                    });
                    //get path to tutorial
                    let tuitotialPaths = this.baseWorkspaceUri.fsPath + "/.gigo" + "/.tutorials/";
                    //get all README files from file path and push to markdown array
                    fs.readdir(tuitotialPaths, (err: any, files: any) => {
                        files.forEach((f: any) => {
                            if (f.endsWith(".md") && f.indexOf("tutorial-") !== -1) {
                                var numberPattern = /\d+/g;;
                                let tutorialNum = f.match(numberPattern)[0];
                                if (tutorialNum) {
                                    mdArr[tutorialNum - 1] = md.render(fs.readFileSync(`${tuitotialPaths}${f}`, 'utf-8'));
                                }
                            }
                        });
    
                    });
                });
    
            } catch (err) {
                this.logger.error.appendLine(`Tutorial Failed: Failed to start tutorial, a workspace must be open`);
                console.log(err);
            }
    
    
    
            //return markdown array
            return await mdArr;
        }

It had a whole suite of compatible languages and theme-ing to boot. It was so seamless to implement I decided to get daring and add the ability to link to code inside the tutorial using an invention by the Goliath themselves, Microsoft.

[CodeTour](https://github.com/microsoft/codetour) is a vscode plugin that is managed by Microsoft and allows the ability to create “tours” to link through code by clicking through steps.

With a bit of work and sacrifices at the altar of [*The Oracle](https://code.visualstudio.com/api)* I finally had a rather sharp looking markdown renderer with the ability to link to specific code:

![Markdown renderer on the left and code tour link on the right](https://cdn-images-1.medium.com/max/3680/1*IFbECgYoydEg2YrxQ_THXQ.png)

The tours were now implemented so that when a tutorial is created a user can simply click a button and be taken to that step inside the codebase:

    public getCodeTours(): any {
           const fs = require('fs');
           var ctArr: any[] = [];
           var numberPattern = /\d+/g;;
    
    
           let tourPaths = this.baseWorkspaceUri.fsPath + "/.gigo" + "/.tours/";
           fs.readdir(tourPaths, (err: any, files: any) => {
               files.forEach((f: any) => {
                   if (f.endsWith(".tour") && f.indexOf("tutorial-") !== -1) {
                       let tourNum = f.match(numberPattern)[0];
                       if (tourNum) {
                           let tour = fs.readFileSync(`${tourPaths}${f}`, 'utf-8');
                           let ts = JSON.parse(tour).steps;
                           this.tourSteps[tourNum - 1] = ts.length;
                           ctArr[tourNum - 1] = f;
                       }
                   }
               });
           });
           return ctArr;
       }
    
    
    function getSteps(fileContents: any) {
                   let yamlData = yaml.load(fileContents) as any;
                   for (let step of yamlData.steps) {
                       let buttonHtml = `<br><div class="btn-9" style="background: ${currentTheme}; box-shadow: ${boxTheme}; " id="codeStep${step.step_number}" onclick="startCodeTour(${currentPgNum}, ${step.step_number})" onmousedown="this.style.boxShadow = '${activeBoxTheme}'" onmouseup="this.style.boxShadow = '${boxTheme}'">Step ${step.step_number}</div><br>`;
                       let lineIndex = step.line_number - 1; // Convert line number to zero-based index
                       let lines = markdownContent.split('\n');
                       lines[lineIndex] = buttonHtml; // Replace the line with the button HTML
                       markdownContent = lines.join('\n');
    
                       markdownData = md.render(markdownContent);
                       webview.postMessage({ type: 'updateMarkdown', message: md.render(markdownContent) });
                   }
               }

Only one problem remained, a user might wish to make a tutorial for other users to read. This, dear reader, is when *The Goliath* once again tested me.

## Insanity and It’s Effects on Programming Efficiency

VScode uses the [monaco-editor](https://microsoft.github.io/monaco-editor/) to display all editor screens in vscode including the markdown editor. A simple solution is to use the in built markdown file editor and call it a day.

However, this would not include syntax highlighting and the observant among you may remember I added the CodeTour system to link to code. This meant that a new markdown file editor was required to allow that functionality to be simple for the user. The idea is to add some button to the side of the editor that allows for a simple addition of a CodeTour link to the file that contains relevant code.

Luckily vscode does have a system in place for creating a [Custom Editor](https://code.visualstudio.com/api/extension-guides/custom-editors). However, as many could have guessed, this resource has extremely limited documentation. Without much knowledge of how custom editors work I sought to find my own solution:

    <div class="storage-tray">
       <button class="storage-tray-button" id="storage-tray-button" onclick="addCodeTour(this)">+</button>
       </br>
       </br>
       <button id="trash" class="trash">
        <svg id="trash-icon" class="trash-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
         <path class="trash-icon-path" d="M3 6v18h18v-18h-18zm5 14c0 .552-.448 1-1 1s-1-.448-1-1v-10c0-.552.448-1 1-1s1 .448 1 1v10zm5 0c0 .552-.448 1-1 1s-1-.448-1-1v-10c0-.552.448-1 1-1s1 .448 1 1v10zm5 0c0 .552-.448 1-1 1s-1-.448-1-1v-10c0-.552.448-1 1-1s1 .448 1 1v10zm4-18v2h-20v-2h5.711c.9 0 1.631-1.099 1.631-2h5.315c0 .901.73 2 1.631 2h5.712z"/>
        </svg>
    
       
       </button>
      
      </div>
      <div id="pop-container" class="pop-up-container">
       <div id="add-pop" class="add-pop-up"></div>
       <div id="pop-arrow" class="arrow-left"></div>
      </div>
    
       
    
        <div id="delete-container" class="delete-container">
           <b id="delete-prompt" class="delete-prompt" >Are you sure you want to delete?</b>
           </br>
    
           <div class="code-steps-inner">
          <span  class="step-title"><b>Step 0</b></span>
         </div>
    
           </br>
           <div id="button-container" style="padding-top: 50%; display: flex; justify-content: center">
           <button id="delete-btn" class="delete-btn">Delete</button>
           <button id="cancel-btn" class="cancel-btn" onclick="closeDeleteBox()">Cancel</button>
           </div>
        </div>
      
        <div class="code-steps-box">
          <div id="@@@Step0@@@" draggable="true" ondragstart="dragElement(this)" oncontextmenu="expandStep(event, this)" class="code-steps">
          <img  class="move-icon"  src = "${this.moveSVG}" alt="My Happy SVG">
    
          </img>
    
          <div class="code-steps-inner">
          <span  class="step-title"><b>Step 0</b></span>
          </div>
          <div id="file-path-div">
           <label>File Path*:</label>
           <input id="file-path" class="file-path-box">
           </input>
          </div>
          <div id="line-number-div">
           <label>Line Number*:</label>
           <input id="line-number" class="line-number-box">
           </input>
          </div>
          <div id="description-div">
           <label>Description/Code:</label>
           <textarea id="description-input" class="description-box">
           </textarea>
          </div>
    
         
    
    
          <button id="save-step-button" class="save-step" onclick="saveStep(this)">Save</button>
          <button style="display: none;" id="edit-step-button" class="edit-step" onclick="editStep(this)">Edit</button>
         </div>
        </div>
    
      
        <div class="input-container">
         <code-input id="ci-external" lang="Markdown" style="letter-spacing: inherit;" value="${parsedText}"></code-input>
        </div>
    
       <script  nonce="${nonce}" src="${styleJS}" ></script>
       <script type="module" nonce="${nonce}" src="${scriptUri}"></script>
      </div>

In this implementation I was rendering two versions of the same editor to accomplish a form of syntax highlighting.

The internal editor was simply a textbox rendered over a normal markdown file opened inside of vscode, and the external editor was the same text with whitespace preserved with the Shiki renderer providing syntax highlighting.

![First pass at editor with syntax highlighting and codetour buttons](https://cdn-images-1.medium.com/max/2000/0*JRtM-SXMTHXHmOv-)

There were also toolbar controls to add/remove code tour steps. With the ability to use the relative path to a file link inside of the codebase:

![Adding CodeTour step inside of editor](https://cdn-images-1.medium.com/max/2000/0*m7JVS-T2VAuFAKrP)

If this sounds insane that’s because it is. It had a mountain of bugs and the user experience was terrible. However, at this point I was losing grip with reality and ready to yield on this whole endeavor. Microsoft had broken me and the vscode documentation was their lance plunged deep into my heart.

## The Hero’s Journey And Other Fairy-tales

At this point another developer saw my struggles and tried to ease my pain with a suggestion. “Why not just use the monaco-editor?”. WHY. NOT? Well rather than lecture him with the manifesto I had been writing in my head, I decided to simply show him my work.

He simply installed the [package](https://www.npmjs.com/package/monaco-editor) and wrote the following lines of code:

    require.config({
                   paths: {
                       'vs': 'https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.30.1/min/vs'
                   }
               });
        
               require(['vs/editor/editor.main'], function() {
                   const editor = monaco.editor.create(document.getElementById('container'), {
                       value: \`${parsedTextEscaped}\`,
                       language: 'markdown',
                       theme: '${vsTheme}',
                       scrollBeyondLastLine: false,
                   });
    
    
              
                   const parsedYaml = ${tourDataStr};
                   let numbersArray = []
    
                   if (parsedYaml !== "") {
                       parsedYaml.steps.forEach(step => {
                           const button = document.createElement('button');
                           button.classList.add('tour-button-style')
                           button.style.backgroundColor = '#' + Math.floor(Math.random()*16777215).toString(16);
                           button.id = 'Step ' + step.step_number;
                           button.textContent = 'Step ' + step.step_number;
                           button.addEventListener('click', () => {
                               openDeleteBox(step.step_number);
                               });
                           const lineTop = editor.getTopForLineNumber(step.line_number);
                           button.style.top = lineTop + 'px';
                           editor.onDidScrollChange(() => {
                               const scrollInfo = editor.getScrollTop();
                               button.style.top = (lineTop - scrollInfo) + 'px';
                               });
    
                           numbersArray.push(step.line_number);
                           editor.getDomNode().appendChild(button);
                       })
                   }

And it worked. IT WORKED. I was filled with a mixture of euphoria and rage that someone who had not seen the horrors of this journey could so easily have solved it.

I built it out a bit more and finally it looked great:

![Finished editor using monaco](https://cdn-images-1.medium.com/max/2000/0*GlKjbfOehsHoavB0)

I was even able to preserve the CodeTour step buttons:

![CodeTour step being added](https://cdn-images-1.medium.com/max/2000/0*niCqti8AmnbDLyfv)

I felt a great weight lifted when I looked at how well it had turned out. I tried to keep my feelings from showing to this developer that had just saved me hours and hours of painstaking debugging, but I think he noticed.

## The Hardest Pill To Swallow

The extension was done and I moved on with my life working on greater things. Finally free of Microsoft’s icy grasp, but I feel that my perspective has changed.

The things that vscode allows developers to do are undeniably amazing and there are few other IDEs that can claim that.

While their documentation is a nightmare that visits me often as I continue to improve the GIGO extension, I still feel that their product allows developers like me to bring new features to the programming community.

This is my letter of apology to Microsoft and the VScode team. Thank you for your services, perhaps I treated you too harshly.

![Peace at last](https://cdn-images-1.medium.com/max/2000/0*NZbm1pALWYkn2GzJ)

Don’t forget to check out [GIGO.dev](https://gigo.dev)! For a deeper look into my journey, make an attempt on any project and use the extension.

Also feel free to check out the github for a deeper look into the GIGO extension here: [gigo-vsc-ext](https://github.com/Gage-Technologies/gigo-vsc-ext)

Discord: [https://discord.gg/MdKmqBzRqX
](https://discord.gg/MdKmqBzRqX)Twitter:[ https://twitter.com/gigo_dev
](https://twitter.com/gigo_dev)Reddit: [https://www.reddit.com/r/gigodev/](https://www.reddit.com/r/gigodev/)

Also feel free to check out the github for a deeper look into the GIGO extension here: [gigo-vsc-ext](https://github.com/Gage-Technologies/gigo-vsc-ext)

Discord: [https://discord.gg/MdKmqBzRqX
](https://discord.gg/MdKmqBzRqX)Twitter:[ https://twitter.com/gigo_dev
](https://twitter.com/gigo_dev)Reddit: [https://www.reddit.com/r/gigodev/](https://www.reddit.com/r/gigodev/)
