# Filament + React Integration Reference (v3, v4, v5)

## Goal

Mount React components inside Filament while keeping a single frontend codebase and a minimal version-specific injection layer.

## Detection checklist

Check the existing project before installing anything.

1. Inspect `package.json` for `react` and `react-dom`.
2. Inspect `vite.config.*` and current `@vite(...)` usage.
3. Inspect `resources/js`, `resources/ts`, and existing `.jsx` or `.tsx` files.
4. Inspect Filament pages, widgets, resources, or panel layout overrides for current asset loading.
5. If React is already installed, skip installation and continue with the missing integration steps only.

## v3-v4-v5 matrix

1. v3: use Blade views for pages, widgets, or resources and load `@vite(...)` in the view or through a panel hook.
2. v4: keep the same approach as v3 and verify the correct panel context for assets.
3. v5: keep the same approach and verify the final hook or layout used for panel asset injection.

## Suggested structure

- `resources/js/filament-react.jsx`
- `resources/js/components/ExampleCard.jsx`
- `resources/views/filament/pages/react-demo.blade.php`

## Install only when missing

Run this only when `react` or `react-dom` is not already installed:

```bash
npm install react react-dom
```

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
3. Add only the missing Filament Blade root, props wiring, and asset inclusion.
4. Avoid duplicating bootstrap files unless the current structure makes separation necessary.

## Diagnostic checklist

1. Confirm that `@vite` points to the real file.
2. Confirm that the `[data-react-component]` selector exists in the final HTML.
3. Confirm that `data-props` contains valid JSON.
4. Confirm that the component name exists in the `registry`.
5. Confirm that `npm run dev` or `npm run build` completed successfully.
6. If it fails by version, validate the panel hook, layout, or asset injection path for v3, v4, or v5.
