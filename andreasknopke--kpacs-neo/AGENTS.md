# K-PACS.neo — Copilot Instructions

## Project

K-PACS.neo is a cross-platform DICOM medical imaging workstation built with C# 13, .NET 10, fo-dicom, and Avalonia UI. It handles Protected Health Information (PHI) and must meet clinical-software-grade quality and security standards.

## Code Quality

- Write production-grade C# following current .NET conventions and idiomatic patterns.
- The build must always be clean — zero errors, zero warnings. Treat every warning as a defect.
- Use nullable reference types correctly; never suppress nullability warnings without a justified comment.
- Favour explicit, readable code over clever abstractions. Minimize allocations in hot paths (rendering, pixel processing, network I/O).

## Security

- Never log, serialize, display in toasts, or include in exception messages any PHI fields (patient name, patient ID, birth date, accession number, or any DICOM tag that can identify a patient).
- Never hard-code secrets, tokens, passwords, or connection strings. Load them from configuration or environment variables.
- Validate and sanitize all external input: file paths, DICOM tag values, network payloads, and user-supplied query strings.
- Do not trust incoming DICOM data — treat it as untrusted input at every parsing boundary.

## Performance

- Keep the UI thread free at all times. Any work that touches the filesystem, network, database, or pixel processing must run on a background thread or via `Task.Run`, posting results back through `Dispatcher.UIThread`.
- Leverage parallelism for CPU-bound operations — pixel rendering, volume reslicing, and bulk DICOM parsing should scale across available cores using `Parallel.For`, PLINQ, or partitioned task pipelines where the workload justifies it.
- Avoid unnecessary allocations in render and scroll paths. Prefer pooled buffers, `Span<T>`, and stack-allocated structures over temporary heap objects in tight loops.
- Cache expensive results (decoded pixel data, spatial metadata, volume slices) rather than recomputing them on every frame or scroll event.
- Use cancellation tokens consistently so that superseded background work (e.g. a new scroll position replacing a pending decode) is cancelled promptly instead of wasting cycles.

## Architecture

- `KPACS.DCMClasses` is the shared DICOM domain library; it must stay UI-free and have no Avalonia dependency.
- `KPACS.Viewer.Avalonia` is the desktop application; UI logic lives here.
- Keep platform-specific code behind runtime checks (`RuntimeInformation.IsOSPlatform`) or behind clearly scoped helper methods.

## Build

```bash
dotnet build KPACS.Viewer.Avalonia/KPACS.Viewer.Avalonia.csproj
```

---
> Source: [andreasknopke/KPACS-neo](https://github.com/andreasknopke/KPACS-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
