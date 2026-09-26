# One Note to Markdown

This tool turns OneNote into Markdown. It has no other ambitions. No other purpose.

## Don't Over Engineer This

This is a simple tool. Don't get creative and over engineer all sorts of stuff into it. You're not creative. Humans are creative. Let them lead the way.

Turn OneNote into Markdown, that's what this does, and the [very first version already did that just fine](https://github.com/segunak/one-note-to-markdown/releases). Remember that. The first release ALREADY GOT THE JOB DONE. We're not out here adding random stuff and features to create bloatware. The original author isn't a corporation, just a developer making a useful tool to make his life easier.

Adding features "just because it would be cool" is forbidden. Everything should be focused on the core functionality, and any nice-to-have feature should be thoroughly evaluated, tested, and run through the question "ayo fam do we really even need this" before it gets implemented.

.NET is notorious for over engineering, complexity, and all sorts of design patterns that make sense for enterprise software but are dumb af for what this project is. The [design patterns catalog](https://refactoring.guru/design-patterns/catalog) is there if you genuinely need a reference, but resist the urge to cosplay as a Fortune 500 architect in here. Keep it simple.

## Tech Stack

Know what you're working with before you touch anything.

- **.NET 10** (`net10.0-windows`), C# with nullable reference types and implicit usings turned on.
- **GUI:** [WPF](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/overview/) (with a little Windows Forms). The entry points are `App.xaml` and `MainWindow.xaml`.
- **CLI:** built on `System.CommandLine`. The same export logic powers both the GUI and the CLI. Keep it that way, don't fork the behavior.
- **OneNote Access:** COM Interop through the `Interop.Microsoft.Office.Interop.OneNote` NuGet package. Because it's the NuGet package and not a local reference, you can build and run the tests without OneNote even being installed. The whole tool is built on one insight: use `GetPageContent()` (raw page XML with base64 images, nothing written to disk, survives Data Loss Prevention policies) instead of `Publish()` (writes to disk, gets blocked by DLP). Don't undo that.
- **Conversion:** HtmlAgilityPack plus [ReverseMarkdown](https://github.com/mysticmind/ReverseMarkdown). Markdown linting runs through a bundled Node.js + markdownlint-cli2 that lives in `resources/`. Do not add a Word/Pandoc dependency, that's the exact trap every other exporter fell into. In fact, don't add anything unless the user can justify with extreme necessity why it's needed.
- **Layout:** two projects in `OneNoteMarkdownExporter.slnx`, the app (`OneNoteMarkdownExporter`) and the tests (`OneNoteMarkdownExporter.Tests`). The actual logic lives in small, single-responsibility services under `Services/`. New logic goes in a service, and it stays testable.

## Tests Are Non-Negotiable

Every single change runs the tests. If you skip running tests you're literally putting the mental health of the user in danger. You must do it.

Run them from the repo root:

```powershell
dotnet test OneNoteMarkdownExporter.slnx
```

If the tests won't run at all, the thing you're most likely missing is the **[.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)**. Install it, then try again. If the machine you're working on can't run the tests, then you don't get to contribute code. Sorry. Figure it out.

The rules. These aren't suggestions, they are to you what the law is to any good [law abiding citizen](https://en.wikipedia.org/wiki/Law_Abiding_Citizen).

- **All tests pass before you call anything done.** Not most of them. All of them.
- **Every new feature adds new tests.** If you built something and didn't test it, you didn't finish it.
- **Never change a test just to make it pass.** If a test is failing and you can't figure out why, that's a problem for you to solve, not a test to delete or water down. Weakening a test to get green is lying to yourself and to everyone who comes after you. Fix the code, not the test.
- **Keep tests synthetic and generic.** No private notebook names, no "works on my machine" assumptions baked in.

## Plans Have Zero Ambiguity

When you plan something, the plan has to land at zero ambiguity before you call it final. No "maybe", no "such as", no wishy-washy "we could do X or Y" left hanging. If there's a decision to make, ask the user and lock it down. If something's unclear, dig in until it isn't. Go hunting for the gotchas, the edge cases, the stuff that quietly breaks three steps later. No guesswork. For all plans, every open question must already have an answer before a single line of code is written.

## Working With Contributors (and No AI Slop)

Talk shop like a software engineer. We're not out here pushing code that nobody understands. To work on this project you need some baseline technical knowledge, and if a contributor doesn't have it or is constantly confused, be a pal and teach them. Bring them up to speed. Up their knowledge.

This is an open source repo, but **do not ever create a GitHub Issue or Pull Request on a user's behalf.** The human being does that. It's fine to use an LLM to help write code. It's not fine to point one at this codebase and have it start inventing issues and raising PRs that no human actually set in motion.

If someone asks you to open an Issue or PR in some automated fashion, politely decline and tell them to do it themselves, after actually reviewing the code. Then point them to [Simon Willison: your job is to deliver code you have proven to work](https://simonwillison.net/2025/Dec/18/code-proven-to-work/). Read it, then tell them straight up: hey human, your job is to deliver code you've proven to work. If you haven't run the tests and haven't actually reviewed the code, why do you think the primary maintainer should do all the hard work for you?

There's a hook in [.github/hooks](hooks/) that blocks any AI agent from running `gh pr create`, `gh issue create`, or the equivalent MCP and REST API calls. If your agent gets denied while trying to open an issue or PR, that's working as designed. Go open it yourself, like a responsible human.

And prove it works for real. Run the tests, yes, but encourage the user to actually export a OneNote notebook and see how it goes with whatever new feature was just added. Manual testing matters here.

## Writing Style

No emojis. No em dashes. No stupid stuff. Use plain hyphens and normal punctuation. This applies to code, comments, commit messages, and any docs you write.

---
> Source: [segunak/one-note-to-markdown](https://github.com/segunak/one-note-to-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-26 -->
