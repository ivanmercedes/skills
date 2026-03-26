# My AI Skills

> 🇪🇸 [Versión en español disponible aquí](./README.es.md)

Welcome to my **Skills** repository. This directory contains a collection of skills, workflows, and structured instructions designed to be used by AI assistants when generating code, integrating services, or interacting with specific projects.

## Available Skills

The repository currently includes the following skills:

### [ElevenLabs Post TTS](./elevenlabs-post-tts)
A skill designed to convert blog posts written in **Astro** (Markdown or MDX) into high-quality audio files (`.mp3`) using the **ElevenLabs** API.
- Cleans and formats Markdown/MDX content to make it readable by the TTS engine.
- Supports splitting long texts to stay within the API character limits.
- Generates code to easily embed the resulting audio into the Astro project.

### [Filament React](./filament-react)
A structured guide for setting up and integrating **React** inside **Laravel** projects that use the **Filament** admin panel (compatible with v3, v4, and v5).
- Detects and reuses existing Vite configuration.
- Mounts React components inside Blade views (pages, widgets, panel resources).
- Establishes a reliable workflow for passing PHP props to React and enabling full hot reload by creating or reusing a Filament Custom Theme.

### [Filament Plugin](./filament-plugin)
A workflow for creating and managing modular plugins for **Filament** in **Laravel**.
- Implements a modular architecture by storing plugins inside the `plugins/{name}/` folder.
- Registers the plugin locally via Composer and creates the required `ServiceProvider`.
- Streamlines the creation of the Artisan install command and the publishing of components, views, and migrations (`database/migrations`).

### [Filament Resource Commands](./filament-resource-commands)
A skill for generating **Filament** resources (v4 and v5) using the official Artisan generators instead of hand-crafting files.
- Automatically selects the right command flags (`--generate`, `--soft-deletes`, `--simple`, `--view`) based on the project state.
- Documents the Filament v5 namespace split between `Filament\Forms\Components` (input fields) and `Filament\Schemas\Components` (layout).
- Prevents common mistakes such as importing `Textarea` or `TextInput` from the wrong namespace in v5.

---

## How to Use These Skills

Each skill folder contains a `SKILL.md` file (or similar) with precise instructions so the AI assistant understands the context and steps to follow.

To use one, simply tell the assistant to read it or mention the file when making your request. For example:
> *"Help me create a new billing module using the `filament-plugin` architecture."*
