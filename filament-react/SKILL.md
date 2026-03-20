---
name: "filament-react"
description: "Configure and integrate React inside Laravel projects that use Filament v3, v4, and v5 to render React components in Blade views, custom pages, widgets, or panel resources. Use when the user asks to add React to Filament, mount React components inside Filament pages, pass props or state from PHP to React, support multi-version Filament setups, check whether React is already installed, finish only the missing setup steps, enable hot reload, validate whether a Filament custom theme is already in use, configure a custom theme when needed, or troubleshoot Vite asset loading in Filament."
---

# Filament React

Integrate React into Filament v3/v4/v5 with Vite and Blade using a repeatable workflow for mounting React components inside the panel.

## First check existing setup

Before installing anything, inspect the project and decide whether React is already present.

1. Check `package.json` for `react` and `react-dom` in `dependencies` or `devDependencies`.
2. Check whether the project already has a Vite-based frontend flow, usually through `vite.config.*`, `resources/js`, and `@vite(...)` usage.
3. Check whether a React entrypoint or React components already exist, such as `resources/js/*.jsx`, `resources/js/components`, or an existing bootstrap file.
4. Check whether the Filament view, page, widget, or resource already loads the frontend assets.
5. Check whether the panel already uses a Filament custom theme and whether that theme is built through Vite.
6. Check whether local development hot reload is already working for Filament assets.
7. If React and the base frontend stack are already installed, do not reinstall them. Only complete the missing setup.

## Version matrix

- Filament v3: use Blade views for pages, widgets, or resources and register assets with `@vite(...)`, panel hooks, or a compiled theme asset when the Tailwind v3 theme flow requires it.
- Filament v4: keep the same Blade + Vite pattern, prefer dedicated page views to isolate React components, and use the generated custom theme flow when panel hot reload depends on it.
- Filament v5: keep the same base pattern, validate the panel injection points, and use the official `make:filament-theme` plus `->viteTheme()` flow when a custom theme is needed.

## Base workflow

1. Detect the Filament version (v3/v4/v5) and confirm Laravel, Filament, and Vite are working.
2. Detect whether `react` and `react-dom` are already installed.
3. If React is missing, install the missing dependencies only.
4. If React is already installed, reuse the existing setup and avoid replacing working project conventions unless the user asks for a refactor.
5. Detect whether Filament already uses a custom theme for panel assets.
6. If a custom theme already exists, reuse it for React-enabled assets and hot reload.
7. If no custom theme exists and the current Filament setup requires one for panel asset bundling or hot reload, create and register it using the version-appropriate Filament theme flow.
8. Create or extend a React entrypoint (`resources/js/filament-react.jsx`) and one or more components (`resources/js/components/*.jsx`).
9. Mount a root node from Blade (`<div data-react-component=...>` or a fixed id).
10. Load assets with `@vite(...)` or through the Filament theme pipeline from the version-specific view, layout, or panel hook.
11. Pass serialized props from PHP or Blade and mount them in React.
12. Build with `npm run build` or serve with `npm run dev`, then validate inside a Filament page with hot reload enabled.

## Decision rules

- If `react` and `react-dom` are already present, skip `npm install react react-dom`.
- If an existing React entrypoint is already used by the project, prefer extending it instead of introducing a second competing bootstrap file.
- If the project already has a component registration pattern, follow it.
- If the project already has a Filament-specific asset hook or panel layout override, reuse it.
- If the project already uses a Filament custom theme, extend that theme instead of creating a second one.
- If hot reload depends on a Filament custom theme in the current version or project structure, configure the theme before wiring React into the panel.
- If the project uses Filament v3, check the Tailwind version before assuming the v4/v5 Vite theme flow will work unchanged.
- If the project uses Filament v3 with Tailwind v4 already installed, prefer the documented CLI fallback and the asset-based `->theme(...)` registration when necessary.
- Only create new files when the current project structure does not already provide the required integration point.

## Recommended implementation

- Prefer a single bootstrap file that finds all React nodes on the page and mounts them dynamically.
- Avoid multiple bundles per component unless there is a clear need; a single Filament entrypoint keeps caching and debugging simpler.
- Keep stable `data-react-component` names and map them in a JavaScript registry.
- When Filament or Livewire state is involved, pass initial data as props and handle follow-up interactions through endpoints or events.
- Keep only a thin version-specific integration layer where asset injection changes, and share the rest of the React code.
- Preserve the repository's existing frontend conventions whenever React is already installed.
- Prefer integrating React through the existing Filament theme asset flow when that is what enables reliable panel hot reload.

## Version-specific integration

1. Filament v3: create a dedicated Blade view and load `@vite(['resources/js/filament-react.jsx'])`, the project's existing React entrypoint, the custom theme entrypoint, or the compiled theme asset path if the v3 Tailwind fallback is in use.
2. Filament v4: reuse the v3 pattern and verify that the view is rendered inside the correct panel.
3. Filament v5: reuse the v4 pattern and validate the panel hooks or layout used to inject assets.
4. In all versions, use `data-react-component` and `data-props` attributes to mount multiple components.

## Shared Blade integration

1. Create or update a dedicated Blade view for the page, widget, or resource.
2. Add the React root and safe JSON props with `@js($data)` or `@json($data)`.
3. Include `@vite(['resources/js/filament-react.jsx'])`, the existing project entrypoint, or the registered Filament theme entrypoint in that view or in the panel layout or hook.
4. If the page needs multiple components, use `data-react-component` and `data-props` attributes.

## Hot reload and custom theme

1. Check whether Filament already has a custom theme directory and panel registration for it.
2. If a custom theme exists, wire the React entrypoint into that theme or reuse the theme entrypoint so Vite hot reload works inside the panel.
3. If no custom theme exists, create one when needed for reliable panel asset loading and hot reload.
4. For Filament v4 and v5, prefer `php artisan make:filament-theme`.
5. If the project has multiple panels, generate the theme for the correct panel: `php artisan make:filament-theme admin`.
6. If the project uses a package manager other than NPM, use the matching `--pm` option, for example `php artisan make:filament-theme --pm=bun`.
7. For Filament v4 and v5, expect the command to install Tailwind dependencies, create `resources/css/filament/{panel}/theme.css`, try to add that file to the Vite input array, and try to register `->viteTheme()` in the panel provider.
8. For Filament v3, check whether the project is on Tailwind CSS v3 or v4 before choosing the theme compilation path.
9. For Filament v3 with the normal supported path, follow the generator output and register the generated theme in the panel provider.
10. For Filament v3 with Tailwind CSS v4 already installed, prefer the documented Tailwind CLI fallback and register the compiled asset with `->theme(asset('css/filament/{panel}/theme.css'))` when the command instructs it.
11. If the generator cannot update files automatically, manually add the theme CSS file to `vite.config.js` and register the correct theme method for that Filament version.
12. Review the generated `theme.css` file and keep its `@source` directives aligned with the directories where Filament views, Livewire code, and custom Blade files use Tailwind classes.
13. Validate that `npm run dev` updates Filament panel assets without requiring a production build, except for documented Filament v3 fallback cases where the CLI build path is required.
14. Keep the theme setup minimal and aligned with the repository's current Filament conventions.

## Quick troubleshooting

- If React does not mount, confirm that `@vite` loads the correct entrypoint.
- If render or hydration issues appear, validate the JSON props and the component's expected types.
- If changes do not show up in development, verify that `npm run dev` is running and HMR is reachable from the panel domain.
- If the production build fails, inspect Vite aliases, import paths, and JSX compatibility.
- If it fails only on one Filament version, check that version's asset injection point before changing the React components.
- If setup work was skipped unexpectedly, re-check whether React was detected from `package.json` or existing JSX files.
- If hot reload fails inside Filament, check whether the panel should be using a custom theme and whether that theme is registered correctly.
- If Filament v3 theme generation behaves differently, check the project's Tailwind version and follow the documented v3 fallback instead of assuming `->viteTheme()` alone is sufficient.

## References

Read `references/filament-react-integration.md` when you need copy-ready snippets, the v3/v4/v5 matrix, setup detection guidance, custom theme and hot reload guidance, a diagnostic checklist, or the suggested file structure.
