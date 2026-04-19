# Copilot Instructions

Make this project with good folder structure, separation of concerns and tests are important for this project to keep it manageable and maximise any learnings.

The work we'll be doing will be defined as user stories inside the `userstories` folder.

Please read the instructions under the "Goal" section and then proceed through each and every instruction under the "Actions" section. 
Please make sure you check ALL items that need to be marked as checked when completing a story. 
User story looks like the following:

```markdown
# User Story #number, e.g. User Story #1 setup environment (filename: userstory-1-setup-environment.md)

## Goal

input the goal of the user story here, e.g. Create a single-page application (SPA) that allows users to extract, edit, translate, and merge subtitles for videos directly in the browser using WebAssembly and transformer.js models. The app should be clear, compact, and suitable for hosting on GitHub Pages.

## Constraints

input any constraints here, e.g. - ffmpeg wasm which can manipulate video and audio, such as extract audio from video, and merge subtitle into video.

## Actions

[ ] action to do #1
[ ] action to do #2
```

After completing a user story, please

- check all items that were completed in the Actions section.

```markdown
## Actions

[X] action to do #1
[X] action to do #2
```

## Rules to follow:
- Update the user story mark file about what was done by adding bullets under the "## Completion Notes" section.
- If you don't know how to proceed with a task, please ask.
- I will ask you to create Architecture Decision Records (ADR). When you prepare those, please ask me how to proceed when implementing a user story based on the ADR.
- Please keep everything we do in the context of the project description.
- Use yarn to manage the dependencies of the project and execute the code.
- Use git to commit the changes after completing a user story.
- Unit tests are used for testing, Please make sure to run them before committing your changes.

## The description of the project:

This is a project that intends to build a web-based application for video subtitle extraction, translation, and merging. 

Feature:

- User can select a video from local disk and extract subtitle of the video
- User can edit the srt content in the web page
- Choose target language to translate the srt or copy the original srt to clipboard, in case of the user want to do translation by himself.
- User can edit the translated srt
- User can save the srt as file
- User can generate a new video with put the translated subtitle into the video image

NFR:

- It is a single webpage app
- The transformer.js models need to be downloaded when needed. progress bar are needed to show downloading status. 
- The model should be cached and allow user to reset(clean)
- The ui need clear and compact, could be based on tailwind css
- It can hosed on github page

---
> Source: [maye-msft/web-video-translator](https://github.com/maye-msft/web-video-translator) — distributed by [TomeVault](https://tomevault.io/claim/maye-msft).
<!-- tomevault:4.0:agents_md:2026-04-18 -->
