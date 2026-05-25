<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

This is an Electron drawing app project. The main requirements are:
Here’s a clear and structured description you can use to feed into an AI agent to generate the code for your Electron drawing app:

---

### **Electron Drawing App - Functional Requirements**

This is a cross-platform drawing application built with **Electron**, primarily designed for **macOS compatibility**. The application features a **transparent fullscreen canvas** over the desktop that allows users to draw, select, move, and delete drawings using a simple set of keyboard shortcuts.

#### 🖼️ Canvas

* The canvas should be **fullscreen** and **transparent**, allowing the user to draw directly over the desktop or other windows.
* Should always stay on top (optional toggle).

#### 🖊️ Tools & Functionality

1. **Pencil Tool (F1)**

   * Activated by pressing `F1`.
   * Enables **freehand drawing** with the mouse.
   * Uses the currently selected color from the **color picker**.

2. **Selector Tool (F2)**

   * Activated by pressing `F2`.
   * Allows users to **select** drawn elements (e.g., strokes).
   * Selected elements can be:

     * **Moved** with the mouse.
     * **Deleted** using the `BACKSPACE` key.

3. **Clear Canvas (F3)**

   * Pressing `F3` will **clear all drawings** on the canvas.

#### 🎨 Color Picker

* A color picker component should be available in a minimal floating UI or tray menu.
* The selected color affects the pencil tool.

#### ⌨️ Keyboard Shortcuts

* `F1`: Activate **Pencil Tool**.
* `F2`: Activate **Selector Tool**.
* `F3`: **Clear** the entire canvas.
* `BACKSPACE`: Delete selected object(s) (when in Selector mode).

#### 🖥️ Platform Compatibility

* Must run on **macOS**, with proper handling for Electron permissions related to screen overlays and full transparency.
* Should ideally also run on Windows/Linux, but macOS is the primary target.

---

---
> Source: [luizhenriquefbb/my-drawing-app](https://github.com/luizhenriquefbb/my-drawing-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-25 -->
