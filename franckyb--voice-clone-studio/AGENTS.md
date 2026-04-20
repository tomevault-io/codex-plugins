# Voice Clone Studio - Development Guidelines

## CRITICAL: Python Environment

**ALWAYS USE THE PROJECT'S VENV - NO EXCEPTIONS!**

- **NEVER use system Python or bare `python` command**
- **ALWAYS use**: `.\venv\Scripts\python.exe` (Windows) or `./venv/bin/python` (Linux/Mac)
- **Before ANY Python operation**: Use venv Python explicitly
- **Examples**:
  - `.\venv\Scripts\python.exe script.py`
  - `.\venv\Scripts\python.exe -c "import module"`
  - `python script.py` (WRONG - uses system Python)
  - `python -c "import module"` (WRONG - uses system Python)

## Code Style

**NO TYPE HINTS**

- Do NOT use type hints (no `def func(x: int) -> str`)
- Do NOT import from `typing` module
- Keep function signatures clean and simple
- Document types in docstrings if needed

## Architecture Overview

Voice Clone Studio v1.0 is fully modular. The main file (`voice_clone_studio.py`) is a ~230 line orchestrator that loads tools dynamically. All features live in `modules/`.

### Project Structure
```
voice_clone_studio.py              # Main orchestrator (~230 lines)
config.json                        # User preferences & enabled tools
modules/
├── core_components/               # OUR core app code
│   ├── __init__.py                # Core exports (modals, emotions, etc.)
│   ├── constants.py               # Central constants (models, languages, speakers)
│   ├── emotion_manager.py         # Emotion system (40+ presets)
│   ├── help_page.py               # Help content functions
│   ├── tool_base.py               # Tool/ToolConfig base classes
│   ├── audio_utils.py             # Audio processing utilities
│   ├── notification.wav           # Completion notification sound
│   │
│   ├── tools/                     # ALL UI TOOLS (tabs)
│   │   ├── __init__.py            # Tool registry, shared state builder, CSS, utilities
│   │   ├── voice_clone.py         # Voice Clone tool
│   │   ├── voice_presets.py       # Voice Presets tool
│   │   ├── conversation.py        # Conversation tool
│   │   ├── voice_design.py        # Voice Design tool
│   │   ├── sound_effects.py       # Sound Effects tool (MMAudio)
│   │   ├── prep_audio.py          # Prep Audio tool (samples + datasets)
│   │   ├── output_history.py      # Output History tool
│   │   ├── train_model.py         # Train Model tool
│   │   └── settings.py            # Settings + Help Guide tool
│   │
│   ├── ai_models/                 # AI model managers
│   │   ├── __init__.py            # get_tts_manager(), get_asr_manager()
│   │   ├── tts_manager.py         # TTS: Qwen3 Base/Custom/Design, VibeVoice, LuxTTS
│   │   ├── asr_manager.py         # ASR: Whisper, Qwen3 ASR, VibeVoice ASR
│   │   ├── foley_manager.py       # Sound Effects: MMAudio
│   │   └── model_utils.py         # Shared model utilities (device, dtype, cache, seed)
│   │
│   ├── ui_components/             # Reusable UI components
│   │   ├── __init__.py            # Component exports
│   │   ├── modals.py              # Confirmation & input modal logic
│   │   ├── confirmation_modal.*   # Confirmation modal (html/css/js)
│   │   ├── input_modal.*          # Input modal (html/css/js)
│   │   └── theme.json             # Gradio theme
│   │
│   └── gradio_filelister/         # Custom Gradio component (v0.4.0)
│
├── deepfilternet/                 # EXTERNAL: Audio denoising
├── mmaudio/                       # EXTERNAL: MMAudio sound effects
├── qwen_finetune/                 # EXTERNAL: Training logic
├── vibevoice_tts/                 # EXTERNAL: VibeVoice TTS
└── vibevoice_asr/                 # EXTERNAL: VibeVoice ASR
```

## Tool System

### How Tools Work

Each tool is a self-contained module in `modules/core_components/tools/` that:
1. Defines a `ToolConfig` with name, description, category, and default enabled state
2. Implements `create_tool(shared_state)` — creates Gradio UI inside a `gr.TabItem`
3. Implements `setup_events(components, shared_state)` — wires up event handlers
4. Returns a component dict for cross-tool communication

### Tool Registry (`tools/__init__.py`)

All tools are registered in `ALL_TOOLS`:
```python
ALL_TOOLS = {
    'voice_clone': (voice_clone, voice_clone.VoiceCloneTool.config),
    'voice_presets': (voice_presets, voice_presets.VoicePresetsTool.config),
    'conversation': (conversation, conversation.ConversationTool.config),
    'voice_design': (voice_design, voice_design.VoiceDesignTool.config),
    'prep_audio': (prep_audio, prep_audio.PrepAudioTool.config),
    'output_history': (output_history, output_history.OutputHistoryTool.config),
    'train_model': (train_model, train_model.TrainModelTool.config),
    'settings': (settings, settings.SettingsTool.config),
}
```

Users can enable/disable tools in Settings > Visible Tools, saved as `enabled_tools` in `config.json`. Settings is always visible.

### Adding a New Tool

1. **Create the module** in `modules/core_components/tools/my_tool.py`:
```python
import gradio as gr
from modules.core_components.tool_base import Tool, ToolConfig

class MyTool(Tool):
    config = ToolConfig(
        name="My Tool",
        module_name="my_tool",
        description="What this tool does",
        enabled=True,
        category="generation"
    )

    @classmethod
    def create_tool(cls, shared_state):
        components = {}
        with gr.TabItem("My Tool"):
            components['input'] = gr.Textbox(label="Input")
            components['btn'] = gr.Button("Go")
            components['output'] = gr.Audio()
        return components

    @classmethod
    def setup_events(cls, components, shared_state):
        save_preference = shared_state.get('save_preference')
        # Wire up events here
        components['btn'].click(...)

get_tool_class = lambda: MyTool
```

2. **Register** in `tools/__init__.py`:
```python
from modules.core_components.tools import my_tool
ALL_TOOLS['my_tool'] = (my_tool, my_tool.MyTool.config)
```

3. **Add to toggleable list** in `settings.py`:
```python
TOGGLEABLE_TOOLS = [
    ...
    ("My Tool", "My Tool"),
]
```

That's it. The main file does not need to be touched.

### Shared State

Tools receive everything they need through `shared_state`, built by `build_shared_state()`:
- `_user_config` / `user_config` — User config dict
- `_active_emotions` — Emotion presets
- `SAMPLES_DIR`, `OUTPUT_DIR`, `DATASETS_DIR`, `TEMP_DIR` — Directories
- `LANGUAGES`, `CUSTOM_VOICE_SPEAKERS`, `MODEL_SIZES_*` — Constants
- `tts_manager`, `asr_manager` — AI model managers
- `save_preference`, `play_completion_beep`, `format_help_html` — Utilities
- `get_sample_choices`, `get_dataset_folders`, etc. — Data helpers
- `confirm_trigger`, `input_trigger` — Modal triggers

### Main File Responsibilities ONLY
- Load config and initialize directories
- Initialize AI model managers
- Call `build_shared_state()`, `create_enabled_tools()`, `setup_tool_events()`
- Wire the Clear VRAM button
- Launch the Gradio app

## Central Constants System

**CRITICAL**: All constants are defined in **ONE place only**: `modules/core_components/constants.py`

**Single source of truth for:**
- Model sizes (MODEL_SIZES, MODEL_SIZES_BASE, MODEL_SIZES_CUSTOM, etc.)
- Languages (LANGUAGES list)
- Custom voice speakers (CUSTOM_VOICE_SPEAKERS)
- Voice clone options (VOICE_CLONE_OPTIONS)
- Generation defaults (QWEN_GENERATION_DEFAULTS, VIBEVOICE_GENERATION_DEFAULTS)
- Supported models (SUPPORTED_MODELS set)
- Audio specifications (SAMPLE_RATE, AUDIO_FORMAT, etc.)

```python
from modules.core_components.constants import (
    LANGUAGES,
    CUSTOM_VOICE_SPEAKERS,
    MODEL_SIZES_CUSTOM,
    QWEN_GENERATION_DEFAULTS
)
```

**NEVER duplicate constants** - always import from `constants.py`

## Configuration System

### User Preferences (`config.json`)
```python
_user_config.get("samples_folder", "samples")
_user_config.get("models_folder", "models")
_user_config.get("trained_models_folder", "models")
```

### Config Keys:
- `samples_folder` - Voice samples
- `output_folder` - Generated audio
- `datasets_folder` - Training data
- `models_folder` - Downloaded models
- `trained_models_folder` - User-trained models
- `browser_notifications` - Audio notification toggle
- `offline_mode` - Use local models only
- `low_cpu_mem_usage` - Memory optimization
- `attention_mechanism` - flash_attention_2/sdpa/eager
- `enabled_tools` - Dict of tool name to bool for visibility

## AI Model Managers

All model loading goes through centralized managers in `modules/core_components/ai_models/`:

### TTS Manager
```python
from modules.core_components.ai_models import get_tts_manager
tts_manager = get_tts_manager(user_config, samples_dir)

model = tts_manager.get_qwen3_base(size)           # 0.6B or 1.7B
model = tts_manager.get_qwen3_custom_voice(size)    # 0.6B or 1.7B
model = tts_manager.get_qwen3_voice_design()        # 1.7B only
model = tts_manager.get_vibevoice_tts(size)          # 1.5B, Large, Large-4bit
tts_manager.unload_all()                             # Free VRAM
```

### ASR Manager
```python
from modules.core_components.ai_models import get_asr_manager
asr_manager = get_asr_manager(user_config)

model = asr_manager.get_whisper()
model = asr_manager.get_vibevoice_asr()
asr_manager.whisper_available  # bool - check if Whisper installed
asr_manager.unload_all()
```

Managers automatically unload the previous model when switching, and respect `offline_mode` and `attention_mechanism` config.

## Emotion System

- **40+ emotions** stored in `config.json` under `"emotions"` key
- **Hardcoded defaults** in `modules/core_components/emotion_manager.py` as `CORE_EMOTIONS`
- **Active emotions** loaded into `_active_emotions` at runtime
- **Intensity**: 0.0-2.0 multiplier
- **Detection**: Regex `\(emotion\)` for Qwen Base conversations
- **Application**: `apply_emotion_preset(emotion, intensity)` adjusts temp/penalty/top_p
- **Management**: Users can save/delete/reset emotions via UI buttons
- **Storage**: Emotions sorted alphabetically (case-insensitive) in config

## Custom Gradio Components

### FileLister (`modules/core_components/gradio_filelister/`)
Custom Gradio component (v0.4.0) for file management. Pre-built wheel in `wheel/`.
- Multi-select for batch file deletion
- Double-click to play audio instantly
- Used by voice_clone, prep_audio, output_history, and voice_presets tools

To rebuild: `python -m build --wheel` in the gradio_filelister directory.

**Creating a FileLister:**
```python
from gradio_filelister import FileLister

components['my_lister'] = FileLister(
    value=get_sample_choices(),  # List of filenames (strings)
    height=250,                  # Pixel height
    show_footer=False,           # Hide footer
    interactive=True,            # Allow selection
)
```

**Reading selections** — value is a dict with `"selected"` key:
```python
def get_selected_filename(lister_value):
    if not lister_value:
        return None
    selected = lister_value.get("selected", [])
    if len(selected) == 1:
        return selected[0]  # Returns filename string (e.g., "sample.wav")
    return None
```

**Updating file list:**
```python
return gr.update(value=get_sample_choices())  # Pass new list of filenames
```

**Double-click to play audio** — wire to a nearby `gr.Audio` preview:
```python
components['my_lister'].double_click(
    fn=None,
    js="() => { setTimeout(() => { const btn = document.querySelector('#my-audio-preview .play-pause-button'); if (btn) btn.click(); }, 150); }"
)
```

## Modals (Confirmation & Input)

Both modals are global components created once in the main UI. Tools access them via `shared_state`:

```python
show_confirmation_modal_js = shared_state['show_confirmation_modal_js']
show_input_modal_js = shared_state['show_input_modal_js']
confirm_trigger = shared_state['confirm_trigger']
input_trigger = shared_state['input_trigger']
```

### Confirmation Modal

Use for destructive actions (delete, clear, reset). Shows a Yes/Cancel dialog.

**Step 1 — Open modal on button click:**
```python
components['delete_btn'].click(
    fn=None,
    inputs=None,
    outputs=None,
    js=show_confirmation_modal_js(
        title="Delete Sample?",
        message="This will permanently delete the selected sample.",
        confirm_button_text="Delete",
        context="my_delete_"          # Unique prefix to identify this action
    )
)
```

**Step 2 — Handle the result via `confirm_trigger`:**
```python
def handle_confirm(confirm_value, other_inputs...):
    # CRITICAL: Filter by context prefix — other tools share this trigger
    if not confirm_value or not confirm_value.startswith("my_delete_"):
        return gr.update()

    # User confirmed — do the action
    # confirm_value = "my_delete_confirm" or "my_delete_cancel"
    if "cancel" in confirm_value:
        return gr.update()

    # Perform the delete...
    return "Deleted successfully"

confirm_trigger.change(
    handle_confirm,
    inputs=[confirm_trigger, ...],
    outputs=[components['status']]
)
```

### Input Modal

Use when you need text input from the user (naming, renaming). Shows a text field with Save/Cancel.

**Step 1 — Open modal on button click:**
```python
save_modal_js = show_input_modal_js(
    title="Save Voice Sample",
    message="Enter a name for this voice sample:",
    placeholder="e.g., MyVoice, Female-Accent",
    context="save_sample_"            # Unique prefix to identify this action
)

# With a pre-populated default value (passed as input):
components['save_btn'].click(
    fn=None,
    inputs=[components['suggested_name']],   # Gradio component with default value
    js=f"""
    (suggestedName) => {{
        const openModal = {save_modal_js};
        openModal(suggestedName);
    }}
    """
)

# Without a default value:
components['save_btn'].click(
    fn=None,
    inputs=None,
    js=show_input_modal_js(
        title="Save Sample",
        message="Enter a name:",
        placeholder="e.g., MyVoice",
        context="save_sample_"
    )
)
```

**Step 2 — Handle the result via `input_trigger`:**
```python
def handle_input_modal(input_value, other_inputs...):
    # CRITICAL: Filter by context prefix — other tools share this trigger
    if not input_value or not input_value.startswith("save_sample_"):
        return gr.update()

    # Check for cancel
    parts = input_value.split("_")
    if len(parts) >= 3 and parts[2] == "cancel":
        return gr.update()

    # Extract user-entered name: "save_sample_<name>_<uuid>"
    raw_name = input_value[len("save_sample_"):]
    name_parts = raw_name.rsplit("_", 1)       # Split off trailing UUID
    chosen_name = name_parts[0] if len(name_parts) > 1 else raw_name

    # Use chosen_name...
    return f"Saved as: {chosen_name}"

input_trigger.change(
    handle_input_modal,
    inputs=[input_trigger, ...],
    outputs=[components['status']]
)
```

**Key rules for modals:**
- **Context prefix must be unique** across all tools (e.g., `"save_sfx_"`, `"save_design_"`, `"delete_output_"`)
- **Always filter by prefix** in your handler — the trigger is shared globally
- **Cancel detection**: Check for `"cancel"` in the value
- **Input value format**: `"{context}{user_text}_{uuid}"` — strip context prefix and trailing UUID
- **Confirm value format**: `"{context}confirm"` or `"{context}cancel"`

## Audio Standards

- **Sample Rate**: 24kHz
- **Format**: WAV, 16-bit PCM, Mono
- **Library**: `soundfile`
- **Validation**: Use `check_audio_format()` from `audio_utils.py`

## User Feedback Patterns

### Long Operations
```python
def long_operation(progress=gr.Progress()):
    progress(0.1, desc="Loading model...")
    # work
    progress(0.5, desc="Processing...")
    # work
    progress(1.0, desc="Done!")
    play_completion_beep()  # Audio notification
    return result, "Success message"
```

### Error Messages
```python
return None, "❌ Error: Clear description of what went wrong and how to fix it"
```

### Status Updates
- Never use emoji,  with ❌ being the only exception, to be used in error messages.
- Provide actionable information
- Include progress indicators

## Platform Compatibility

**Supported platforms:** Windows (CUDA), Linux (CUDA), macOS (MPS/CPU)

### Device Detection
All device logic is centralized in `model_utils.py`:
```python
from modules.core_components.ai_models.model_utils import get_device, get_dtype, empty_device_cache, set_seed

get_device()           # Returns "cuda:0" > "mps" > "cpu" (priority order)
get_dtype()            # CUDA: bfloat16, MPS: float16, CPU: float32
get_dtype("mps")       # Can pass explicit device
empty_device_cache()   # Clears CUDA or MPS cache
set_seed(42)           # torch.manual_seed + cuda.manual_seed_all if available
```

**NEVER hardcode `"cuda:0"` or `torch.bfloat16`** — always use `get_device()` / `get_dtype()`.

### macOS Notes
- Train Model tab is auto-disabled on macOS (runtime override, not persisted)
- Flash Attention 2 is CUDA-only; use `sdpa` on MPS
- `bfloat16` not supported on Apple Silicon — `get_dtype()` returns `float16`

```python
import platform

if platform.system() == "Windows":
    # Windows-specific code
elif platform.system() == "Darwin":
    # macOS-specific code
else:
    # Linux-specific code
```

**Windows Console Encoding:**
- Avoid Unicode symbols in `print()` that go to console
- Use ASCII-safe alternatives: `[OK]` instead of checkmarks
- UI (Gradio) can use any Unicode

## File Path Handling

```python
from pathlib import Path

# Always use Path objects
project_root = Path(__file__).parent
samples_dir = project_root / _user_config.get("samples_folder")

# Convert to string only when needed
str(file_path)
```

## Code Quality Checklist

Before submitting any code change, verify:

- [ ] Is this feature in a tool module? (Not in main file)
- [ ] Does it respect user config settings?
- [ ] Does it provide progress feedback?
- [ ] Does it handle errors gracefully?
- [ ] Does it work cross-platform?
- [ ] Does it play completion notification?
- [ ] Is it documented with docstrings?
- [ ] No type hints used?

## Testing Patterns

**CRITICAL: ALL testing MUST use venv Python!**

1. **ALWAYS use `.\venv\Scripts\python.exe`** for all Python commands
2. **Each tool supports standalone testing:**
   ```python
   python -m modules.core_components.tools.voice_clone
   ```
3. **Test naming**: `test_[feature_name].py`
4. Test with default config
5. Test with custom folder paths
6. Test with offline mode enabled
7. Test on Windows (encoding issues!)
8. Test model loading/unloading
9. Test with notifications disabled

## Common Gotchas

**Model Path Confusion**
- Downloaded models: `models_folder` config
- Trained models: `trained_models_folder` config
- They CAN be different!

**Windows Encoding**
- Console output: ASCII only
- File I/O: UTF-8 with `encoding="utf-8"`
- UI text: Any Unicode

**Path Issues**
- Always use `Path` objects
- Use `/` in path joining (works cross-platform)
- Never hardcode `\\` or `/`

**Circular Imports**
- `tools/__init__.py` imports all tool modules
- Tool modules must NOT import from `tools/__init__.py` at module level
- Use lazy imports inside functions if needed: `from modules.core_components.tools import save_config`

**Gradio 6 CSS**
- Tab DOM structure: `#elem-id > .tab-wrapper > .tab-container[role="tablist"] > button`
- No `.tab-nav` class (that was Gradio 4/5)
- Use `>` direct child selectors to scope CSS to specific tab levels

## Key Functions Reference

**Audio Notification**
```python
play_completion_beep()  # Respects config setting
```

**Model Management**
```python
tts_manager.unload_all()  # Free TTS VRAM
asr_manager.unload_all()  # Free ASR VRAM
```

**Config**
```python
save_preference(key, value)  # Auto-saves to config.json
```

**Help Content**
```python
format_help_html(markdown_text)  # Convert markdown to styled HTML
```

---

## Starting a New Feature?

1. Create a tool module in `modules/core_components/tools/`
2. Follow the Tool pattern (ToolConfig + create_tool + setup_events)
3. Register in `tools/__init__.py`
4. Add to `TOGGLEABLE_TOOLS` in `settings.py`
5. Test standalone: `python -m modules.core_components.tools.my_tool`
6. Document in this file if it introduces a new pattern

**The main file should rarely need changes. Tools are self-contained.**

---
> Source: [FranckyB/Voice-Clone-Studio](https://github.com/FranckyB/Voice-Clone-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
