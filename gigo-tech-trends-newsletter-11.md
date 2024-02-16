
## **GIGO Tech Trends Newsletter 011**

![](https://cdn-images-1.medium.com/max/3052/1*CAz948ypFNhmHV-k1pxJag.png)

## Text-to-video takes another massive leap forward

Technology is moving fast. Let’s disregard the required advances in technology that set the foundation for artificial intelligence as a whole. Focusing instead on how fast text-to-video generation has grown since its debut.

**Early iterations — The foundation is set**

In 2014, Generative Adversarial Networks (GAN) and Variational Autoencoders (VAE) became two of the most popular approaches used for producing AI-generated content. In general, GAN’s tend to be more widely used for generating multimedia, while VAE’s see more use in signal analysis. These works provided the foundation for a new computer vision task, but were limited to low resolutions and short-range predictions. Early research predominantly used GAN and VAE-based approaches to auto-regressively generate frames given a caption.

The actual mechanics of GANs involve the interplay of two neural networks that work together to generate and then classify data that is representative of reality. GANs generate content using a *generator* neural network that is tested against a second neural network: the *discriminator*, which determines whether the content is passable as human-generated.

VAE’s, at their core, build on neural network autoencoders made up of two neural networks: an *encoder* and a *decoder*. The encoder network optimizes for more efficient ways of representing data, while the decoder network optimizes for more expressive ways of regenerating the original data set.

Recent breakthroughs uniquely approach the problem by utilizing a combination of a diffusion model and a transformer, setting them apart from traditional GAN or VAE approaches. The diffusion model is used to turn random pixels into a picture, the model applies this approach to videos rather than still images. Additionally, the transformer inside processes chunks of video data, allowing it to handle long sequences of video data, a distinct difference from the traditional use of transformers for processing words in language models.

This approach represents a novel direction in text-to-video generation, distinct from the GAN or VAE-based methods commonly used in the past.

**Who is responsible for this breakthrough?**

The King holds their head high with a new crown jewel. Sam Altman’s OpenAI, whose industry lead just last month looked up for grabs due to internal instability, crashes forward with the release of Sora, their new generative text-to-video model.

Sora takes a short text description and turns it into a detailed, high-definition film clip. Limitations of up to 60 seconds may seem restrictive to an outside eye, though looking at the high-quality examples they released it will be hard to have any truly valid complaints given the tremendous leap they’ve taken compared to the 4 second snippets being pushed out by their industry peers. They made progress but surely there is room for improvement, yes?

**Limitations of OpenAI’s Sora model**

**Simulation of Physics and Cause-Effect Understanding**: The model may struggle with accurately simulating the physics of a complex scene and understanding specific instances of cause and effect. For example, it may have difficulty accurately representing the bite mark following a person taking a bite out of a cookie.

**Spatial Details and Consistency**: Sora may confuse spatial details of a prompt, such as mixing up left and right, and its camera imitation may not be consistent over time. Additionally, it may spontaneously spawn new characters in random locations.

**Content Generation Risks**: There are concerns about potential biases, the creation of convincing AI deepfakes, and the tool’s potential impact on various industries, such as film and digital media. OpenAI has stated that it will add provisions to block Sora from generating certain types of content, such as mimicking celebrity likenesses or generating hateful, extremely violent, or sexually explicit content.

Despite these limitations, OpenAI is committed to refining the model, enhancing its capabilities to continue breaking barriers for product expansion.

**Impact and Closing**

OpenAI’s new Sora model is expected to impact a multitude of industries, including Media & Entertainment, Advertising, Education and creators as a whole.

As a positive, the model will likely speed up certain film and TV production timelines allowing for quicker realization of ideas. Those in advertising should be able to leverage the model for high-quality video ad copy generation. Educators will use the tool to create detailed scenes and characters from text bringing history, science and literature epics to life. Creators, better positioned as artists, will run wild with the potential creative opportunities.

On the contrary, there is always a concern for technology replacing the roles of everyday workers. While the concern is always valid — at the end of the day, Sora, like all other software, is a tool. Tools do not drive business unprompted, nor do they replace true human curiosity, creativity and innovation. Tools are used by humans to achieve greater outcomes with less stress, more quickly. While roles will be refined, there is still an inherent need for a human in the loop.

There are other text-to-video entities, such as Pika, that seemed very promising prior to launch but generally fell short once public access was granted to the model.

OpenAI has a history of delivering. While the model is yet to release, Altman and team are already accepting targeted feedback to improve before public access is granted. This release bodes well for the future of text-to-video generation.

To be added to the GIGO Tech Trends weekly Newsletter mailing list please reach out directly to us on this account or email us at **contact@gigo.dev**

Discord: [https://discord.gg/learnprogramming](https://discord.gg/learnprogramming)
Twitter:[ https://twitter.com/gigo_dev
](https://twitter.com/gigo_dev)Reddit: [https://www.reddit.com/r/gigodev/](https://www.reddit.com/r/gigodev/)

GitHub: [https://github.com/Gage-Technologies/gigo.dev](https://github.com/Gage-Technologies/gigo.dev)

Get started on your first project at [gigo.dev](http://gigo.dev)

Sources:
[**A Dive into Text-to-Video Models**
*We're on a journey to advance and democratize artificial intelligence through open source and open science.*huggingface.co](https://huggingface.co/blog/text-to-video)
[**GANs vs. VAEs: What is the best generative AI approach? | TechTarget**
*Read about the differences between GANs vs. VAEs and how the generative AI approaches are used in the tech sector.*www.techtarget.com](https://www.techtarget.com/searchenterpriseai/feature/GANs-vs-VAEs-What-is-the-best-generative-AI-approach)
[**OpenAI Unveils Sora: A Text-to-Video AI Model with Limitations**
*ISP Today*isp.today](https://isp.today/openai-unveils-sora-a-text-to-video-ai-model-with-limitations/)
[**OpenAI Unleashes Realistic Text-to-Video 'Sora' AI**
*OpenAI's Sora tool can generate videos complete with skin textures and high-resolution environments. For now, it's…*www.pcmag.com](https://www.pcmag.com/news/openai-unleashes-realistic-text-to-video-sora-ai)
[**OpenAI launches video generation model Sora**
*OpenAI on Thursday announced Sora, an artificial intelligence (AI) model that can create realistic videos from text…*economictimes.indiatimes.com](https://economictimes.indiatimes.com/tech/technology/openai-launches-video-generation-model-sora/articleshow/107732569.cms)
[**Papers with Code - Paper tables with annotated results for Countering Malicious DeepFakes: Survey…**
*Paper tables with annotated results for Countering Malicious DeepFakes: Survey, Battleground, and Horizon*paperswithcode.com](https://paperswithcode.com/paper/countering-malicious-deepfakes-survey/review/)
[**OpenAI teases an amazing new generative video model called Sora**
*The firm will now share it with a small group of safety testers but the rest of us will have to wait to learn more.*www.technologyreview.com](https://www.technologyreview.com/2024/02/15/1088401/openai-amazing-new-generative-ai-video-model-sora/)
