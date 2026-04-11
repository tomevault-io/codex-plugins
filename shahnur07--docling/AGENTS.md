# Docling Document Processing Workspace

This workspace processes documents, audio files, and YouTube content for text extraction and Bangla audio transcription.

## Quick Reference

When working on document/audio extraction:
1. **Use the `/extract-and-transcribe` skill** for step-by-step checklists
2. **Invoke the `Doc Processor` agent** to orchestrate workflows
3. **Update script variables** before running (input paths, URLs, model sizes)

## Project Purpose

- **Document Parsing**: Convert PDF, DOCX, and other formats to Markdown using Docling
- **Audio Transcription**: Extract audio from WebM files and transcribe to Bangla text using Whisper
- **YouTube Transcription**: Download and transcribe YouTube videos to Bangla text

## Core Scripts

| Script | Purpose | Input | Output |
|--------|---------|-------|--------|
| `docling_script.py` | Document → Markdown | Files (PDF, DOCX, etc.) | `output.md` |
| `extract_bangla_audio.py` | Audio extraction & Bangla transcription | WebM/audio files | `transcription_bangla.txt` |
| `youtube_transcript.py` | YouTube download & Bangla transcription | YouTube URL | Console output |

## Setup Requirements

**Before running any script**, install dependencies:

```powershell
pip install docling whisper ffmpeg yt-dlp torch
```

**Optional but recommended**: Verify GPU availability for Whisper speedup:
```powershell
python -c "import torch; print(f'GPU: {torch.cuda.is_available()}')"
```

## Workflow Patterns

### Pattern: Document Processing
1. Place document in workspace
2. Run: `python docling_script.py document.pdf`
3. Output appears in `output.md`
4. Review extracted Markdown text

### Pattern: Bangla Audio Transcription
1. Ensure audio file is accessible (WebM/MP3/WAV)
2. Update `webm_file` variable in `extract_bangla_audio.py`
3. Run: `python extract_bangla_audio.py`
4. Output saved to `transcription_bangla.txt`

### Pattern: YouTube Transcription
1. Have valid YouTube URL ready
2. Update `url` variable in `youtube_transcript.py`
3. Run: `python youtube_transcript.py`
4. Check console for Bangla transcription text

## Code Conventions

- **File organization**: Keep extracted outputs in workspace root or `document/` subfolder
- **Character encoding**: Always use UTF-8 for Bangla text files
- **Model size**: Use `"medium"` Whisper model; upgrade to `"large"` for higher accuracy (slower)
- **Error handling**: Scripts print to stderr and stdout; capture both during batch processing
- **Cleanup**: Remove temporary audio files after transcription (except when debugging)

## Common Tasks

### Extract text from multiple documents
Add loop to `docling_script.py` that iterates over files in a directory.

### Batch transcribe audio files
Modify `extract_bangla_audio.py` to accept command-line arguments and process multiple files.

### Store outputs to database
Replace file writes with database inserts (SQLite, PostgreSQL, etc.) for scalable storage.

### Validate Bangla text quality
Check for Bengali Unicode range (U+0985 – U+09FF) in output files.

## Troubleshooting

See the **`extract-and-transcribe` skill** for:
- Detailed pre-processing checklist
- Quality checks to run post-execution
- Common issues and fixes table

## Related Documentation

- `README.md` - Project overview
- `document/` - Output directory for processed files
- `.github/skills/extract-and-transcribe/SKILL.md` - Full workflow checklist
- `.github/agents/doc-processor.agent.md` - Orchestration agent

## Best Practices

- ✅ Always validate input files exist before running scripts
- ✅ Check GPU availability to understand processing time expectations
- ✅ Verify output encoding (especially for Bangla text)
- ✅ Keep dependencies up-to-date: `pip install --upgrade docling whisper torch`
- ✅ Use the `Doc Processor` agent for complex, multi-step workflows

---

**Quick start**: Type `/extract-and-transcribe` in chat to reference the full workflow skill.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shahnur07)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shahnur07)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
