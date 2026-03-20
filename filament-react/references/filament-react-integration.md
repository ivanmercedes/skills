# Filament + React Integration Reference (v3, v4, v5)

## Goal

Mount React components inside Filament while keeping a single frontend codebase and a minimal version-specific injection layer.

## Detection checklist

Check the existing project before installing anything.

1. Inspect `package.json` for `react` and `react-dom`.
2. Inspect `vite.config.*` and current `@vite(...)` usage.
3. Inspect `resources/js`, `resources/ts`, and existing `.jsx` or `.tsx` files.
4. Inspect Filament pages, widgets, resources, or panel layout overrides for current asset loading.
5. Inspect the panel provider and theme registration to see whether a Filament custom theme is already in use.
6. Verify whether hot reload already works for panel assets during `npm run dev`.
7. If React is already installed, skip installation and continue with the missing integration steps only.

## v3-v4-v5 matrix

1. v3: use Blade views for pages, widgets, or resources and load `@vite(...)` in the view or through a panel hook, but validate whether the theme flow needs the v3-specific compiled asset fallback.
2. v4: keep the same approach as v3 and verify the correct panel context for assets.
3. v5: keep the same approach and verify the final hook or layout used for panel asset injection.

## Suggested structure

- `resources/js/filament-react.jsx`
- `resources/js/components/ExampleCard.jsx`
- `resources/views/filament/pages/react-demo.blade.php`

## Custom theme and hot reload

Use a Filament custom theme when the panel expects assets to flow through the theme pipeline or when hot reload is unreliable without it.

1. Check whether the panel provider already registers a custom theme.
2. If a custom theme already exists, reuse it and add the React entrypoint there or import React from the existing theme entrypoint.
3. If no custom theme exists, create and register one only when needed for reliable panel asset loading or hot reload.
4. For Filament v4 and v5, prefer the generator plus `->viteTheme(...)`.
5. For Filament v3, inspect the Tailwind version before assuming the Vite theme path is identical to v4/v5.
6. Prefer one theme entrypoint for panel styles and scripts instead of parallel asset pipelines.
7. After setup, validate that `npm run dev` updates the Filament panel without a manual production build, unless the v3 fallback path requires a CLI-built asset.

## Install only when missing

Run this only when `react` or `react-dom` is not already installed:

```bash
npm install react react-dom
```

## If a custom theme is needed

Choose the theme flow by Filament version, then fall back to manual configuration if the generator cannot patch the project files.

1. Detect the Filament major version before generating the theme.
2. Run `php artisan make:filament-theme`.
3. If the project has multiple panels, run `php artisan make:filament-theme admin` with the correct panel ID.
4. If the project uses a different package manager, pass it with `--pm`, for example `php artisan make:filament-theme --pm=bun`.
5. For Filament v4 and v5, expect the command to install Tailwind CSS dependencies, generate `resources/css/filament/{panel}/theme.css`, try to add the generated theme file to the Vite `input` array, and try to register `->viteTheme('resources/css/filament/{panel}/theme.css')` in the panel provider.
6. For Filament v3, follow the generator output carefully because the registration and build path may vary depending on the Tailwind setup in the project.
7. If Filament v3 reports a Tailwind v4 mismatch, use the documented CLI fallback to compile the theme and register it with `->theme(asset('css/filament/{panel}/theme.css'))`.
8. If the command fails to patch the files automatically, add `resources/css/filament/{panel}/theme.css` to the `input` array in `vite.config.js` yourself when the version-specific flow requires it.
9. If the command fails to patch the panel provider automatically, register the correct method manually: usually `->viteTheme(...)` for v4/v5, and the generator-indicated method for v3.
10. Import the React bootstrap from the theme entrypoint or make the theme entrypoint the single React-aware bundle.
11. Keep the hot reload path simple: one Vite dev server, one panel theme entrypoint, one React mounting registry.
12. Build with `npm run build` for production, and validate `npm run dev` for hot reload in local development when the chosen version flow supports it.

## Manual custom theme configuration

When automatic setup does not work because the project structure is non-standard, apply these manual steps.

1. Add `resources/css/filament/{panel}/theme.css` to the Vite `input` array when the chosen Filament version flow expects Vite-managed theme assets.
2. Register `->viteTheme('resources/css/filament/{panel}/theme.css')` in the panel provider for the normal v4/v5 flow.
3. For Filament v3 fallback cases, register `->theme(asset('css/filament/{panel}/theme.css'))` if the generator output requires the compiled public asset path.
4. Keep the generated `theme.css` file under `resources/css/filament/{panel}`.
5. Review the generated `@source` directives and extend them with any directories that contain Tailwind classes used in custom Blade, Livewire, or Filament code.
6. Rebuild the assets with `npm run build` after changing the sources, or use `npm run dev` while developing locally when the selected version flow supports HMR.

## Tailwind sources in the theme

Filament's generated theme includes `@source` directives so Tailwind can scan your code. Extend them when your custom React-in-Filament setup uses more directories.

Typical additions include:

- `resources/views/components/**/*`
- `resources/views/livewire/**/*`
- `app/Livewire/**/*`
- Any custom Blade or Filament directories that contain Tailwind utility classes

## React entrypoint (`resources/js/filament-react.jsx`)

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import ExampleCard from './components/ExampleCard';

const registry = {
  ExampleCard,
};

document.querySelectorAll('[data-react-component]').forEach((node) => {
  const name = node.dataset.reactComponent;
  const props = node.dataset.props ? JSON.parse(node.dataset.props) : {};
  const Component = registry[name];

  if (!Component) {
    console.warn(`React component "${name}" not found in registry.`);
    return;
  }

  createRoot(node).render(<Component {...props} />);
});
```

## Example component (`resources/js/components/ExampleCard.jsx`)

```jsx
import React from 'react';

export default function ExampleCard({ title, value }) {
  return (
    <section style={{ border: '1px solid #ddd', padding: '1rem', borderRadius: '0.75rem' }}>
      <h3 style={{ margin: 0 }}>{title}</h3>
      <p style={{ marginTop: '0.5rem' }}>{value}</p>
    </section>
  );
}
```

## Example Blade view (`resources/views/filament/pages/react-demo.blade.php`)

```blade
<x-filament-panels::page>
    <div
        data-react-component="ExampleCard"
        data-props='@json(["title" => "Metric", "value" => "React inside Filament"])'>
    </div>

    @vite(['resources/js/filament-react.jsx'])
</x-filament-panels::page>
```

## If React is already installed

1. Reuse the existing entrypoint if the project already mounts React elsewhere.
2. Reuse the existing alias, import, and component folder conventions.
3. Reuse the existing custom theme if the panel already uses one.
4. Add only the missing Filament Blade root, props wiring, asset inclusion, and theme registration.
5. Avoid duplicating bootstrap files unless the current structure makes separation necessary.

## Diagnostic checklist

1. Confirm that `@vite` points to the real file.
2. Confirm that the `[data-react-component]` selector exists in the final HTML.
3. Confirm that `data-props` contains valid JSON.
4. Confirm that the component name exists in the `registry`.
5. Confirm that `npm run dev` or `npm run build` completed successfully.
6. If it fails by version, validate the panel hook, layout, or asset injection path for v3, v4, or v5.
7. If hot reload fails, validate the custom theme registration, the theme entrypoint, and the Vite dev server URL used by the panel.
8. If Filament v3 behaves differently, confirm whether the project is on a Tailwind v3-compatible path or the documented Tailwind v4 fallback path.
