---
name: "filament-react"
description: "Configure and integrate React inside Laravel projects that use Filament v3, v4, and v5 to render React components in Blade views, custom pages, widgets, or panel resources. Use when the user asks to add React to Filament, mount React components inside Filament pages, pass props or state from PHP to React, support multi-version Filament setups, check whether React is already installed, finish only the missing setup steps, or troubleshoot Vite asset loading in Filament."
---

# Filament React

Integrate React into Filament v3/v4/v5 with Vite and Blade using a repeatable workflow for mounting React components inside the panel.

## First check existing setup

Before installing anything, inspect the project and decide whether React is already present.

1. Check `package.json` for `react` and `react-dom` in `dependencies` or `devDependencies`.
2. Check whether the project already has a Vite-based frontend flow, usually through `vite.config.*`, `resources/js`, and `@vite(...)` usage.
3. Check whether a React entrypoint or React components already exist, such as `resources/js/*.jsx`, `resources/js/components`, or an existing bootstrap file.
4. Check whether the Filament view, page, widget, or resource already loads the frontend assets.
5. If React and the base frontend stack are already installed, do not reinstall them. Only complete the missing setup.

## Version matrix

- Filament v3: use Blade views for pages, widgets, or resources and register assets with `@vite(...)` or panel hooks.
- Filament v4: keep the same Blade + Vite pattern and prefer dedicated page views to isolate React components.
- Filament v5: keep the same base pattern, validate the panel injection points, and keep a single React entrypoint.

## Base workflow

1. Detect the Filament version (v3/v4/v5) and confirm Laravel, Filament, and Vite are working.
2. Detect whether `react` and `react-dom` are already installed.
3. If React is missing, install the missing dependencies only.
4. If React is already installed, reuse the existing setup and avoid replacing working project conventions unless the user asks for a refactor.
5. Create or extend a React entrypoint (`resources/js/filament-react.jsx`) and one or more components (`resources/js/components/*.jsx`).
6. Mount a root node from Blade (`<div data-react-component=...>` or a fixed id).
7. Load assets with `@vite(...)` from the version-specific view, layout, or panel hook.
8. Pass serialized props from PHP or Blade and mount them in React.
9. Build with `npm run build` or serve with `npm run dev`, then validate inside a Filament page.

## Decision rules

- If `react` and `react-dom` are already present, skip `npm install react react-dom`.
- If an existing React entrypoint is already used by the project, prefer extending it instead of introducing a second competing bootstrap file.
- If the project already has a component registration pattern, follow it.
- If the project already has a Filament-specific asset hook or panel layout override, reuse it.
- Only create new files when the current project structure does not already provide the required integration point.

## Recommended implementation

- Prefer a single bootstrap file that finds all React nodes on the page and mounts them dynamically.
- Avoid multiple bundles per component unless there is a clear need; a single Filament entrypoint keeps caching and debugging simpler.
- Keep stable `data-react-component` names and map them in a JavaScript registry.
- When Filament or Livewire state is involved, pass initial data as props and handle follow-up interactions through endpoints or events.
- Keep only a thin version-specific integration layer where asset injection changes, and share the rest of the React code.
- Preserve the repository's existing frontend conventions whenever React is already installed.

## Version-specific integration

1. Filament v3: create a dedicated Blade view and load `@vite(['resources/js/filament-react.jsx'])` or the project's existing React entrypoint.
2. Filament v4: reuse the v3 pattern and verify that the view is rendered inside the correct panel.
3. Filament v5: reuse the v4 pattern and validate the panel hooks or layout used to inject assets.
4. In all versions, use `data-react-component` and `data-props` attributes to mount multiple components.

## Shared Blade integration

1. Create or update a dedicated Blade view for the page, widget, or resource.
2. Add the React root and safe JSON props with `@js($data)` or `@json($data)`.
3. Include `@vite(['resources/js/filament-react.jsx'])` or the existing project entrypoint in that view or in the panel layout or hook.
4. If the page needs multiple components, use `data-react-component` and `data-props` attributes.

## Quick troubleshooting

- If React does not mount, confirm that `@vite` loads the correct entrypoint.
- If render or hydration issues appear, validate the JSON props and the component's expected types.
- If changes do not show up in development, verify that `npm run dev` is running and HMR is reachable from the panel domain.
- If the production build fails, inspect Vite aliases, import paths, and JSX compatibility.
- If it fails only on one Filament version, check that version's asset injection point before changing the React components.
- If setup work was skipped unexpectedly, re-check whether React was detected from `package.json` or existing JSX files.

## References

Read `references/filament-react-integration.md` when you need copy-ready snippets, the v3/v4/v5 matrix, setup detection guidance, a diagnostic checklist, or the suggested file structure.
