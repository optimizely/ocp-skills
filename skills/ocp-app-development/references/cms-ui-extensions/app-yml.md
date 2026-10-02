# `app.yml` — the `ui_extensions` block

CMS UI extensions are declared under a top-level `ui_extensions:` key. The block is validated by the `@optimizely/ocp-cms-ui-extensions-sdk` plugin during `ocp app validate`.

## Shape

```yaml
runtime: node22-cms-ext          # required for CMS UI extensions

ui_extensions:
  <injectionPoint>:              # one of UI_EXTENSION_INJECTION_POINTS (sidebar | view | property-editor)
    - name: <unique-name>        # unique across the WHOLE ui_extensions block
      entry_point: <EntryPoint>  # unique across the WHOLE block; matches the entry-file <EntryPoint>
      display_name: <label>      # non-blank; shown to CMS users
      metadata:                  # property-editor only (required there)
        property_type: <type>    # string | richtext | integer | float | boolean
        property_format: <fmt>   # optional; only `string` accepts one (shortstring)
```

Each injection point maps to a **list** of extension items. Each item has these fields:

| Field | Required | Notes |
| --- | --- | --- |
| `name` | yes | Stable identifier for the extension. Unique across all injection points. |
| `entry_point` | yes | Matches the `<EntryPoint>` segment of the entry file (`<EntryPoint>.<injectionPoint>.tsx`) and becomes the CDN bundle filename. Unique across all injection points. |
| `display_name` | yes | Human-readable label; must not be blank. |
| `metadata` | `property-editor` only | String-to-string map passed through to CMS verbatim. Required on `property-editor` entries (see below); `sidebar`/`view` entries do not use it. |

Unknown injection-point keys and unknown item fields are rejected by the schema (the schema is generated from `UI_EXTENSION_INJECTION_POINTS` with `additionalProperties: false`). `metadata` itself accepts arbitrary string keys; only the `property-editor` keys below are validated.

## Property editor metadata

A `property-editor` entry declares which CMS properties it edits:

| Key | Required | Notes |
| --- | --- | --- |
| `property_type` | yes | The property type the editor edits. |
| `property_format` | no | A refinement of the type. Omit it unless the type supports one. |

Supported combinations (the `ALLOWED_PROPERTY_TYPES` map exported from `@optimizely/ocp-cms-ui-extensions-sdk`):

| `property_type` | `property_format` |
| --- | --- |
| `string` | omitted, or `shortstring` |
| `richtext` | omitted |
| `integer` | omitted |
| `float` | omitted |
| `boolean` | omitted |

Values are case-insensitive. Any other combination is a **hard error** in `ocp app validate` — CMS would otherwise silently skip the editor. One entry targets one type; to edit several types, declare several entries.

```yaml
ui_extensions:
  property-editor:
    - name: brand-color-picker
      entry_point: BrandColorPicker
      display_name: Brand Color Picker
      metadata:
        property_type: string
        property_format: shortstring
```

Entry file: `src/cms-ui-extensions/<group>/BrandColorPicker.property-editor.tsx`. Requires `@optimizely/ocp-cms-ui-extensions-sdk` >= 1.1.0-beta.1 — earlier versions reject both the `property-editor` key and `metadata`.

## Full example

```yaml
name: unsplash-viewer
version: 1.0.0
runtime: node22-cms-ext

functions:
  cms_extension:
    entry_point: CmsUiExtension
    description: Backend proxy for the Unsplash extensions.
    accepts: cms_ui_extension          # marks this function as callable from extensions

ui_extensions:
  sidebar:
    - name: unsplash-viewer
      entry_point: UnsplashViewer
      display_name: Unsplash Viewer
  view:
    - name: unsplash-gallery
      entry_point: UnsplashGallery
      display_name: Unsplash Gallery
```

This declares two extensions (one `sidebar`, one `view`) plus a backend function they can call. The entry files would be:
- `src/cms-ui-extensions/unsplash/UnsplashViewer.sidebar.tsx`
- `src/cms-ui-extensions/unsplash/UnsplashGallery.view.tsx`

## The backend function link

A function the extension calls must declare `accepts: cms_ui_extension` in its `functions:` entry. That is what makes it invocable via `context.extension.invokeFunction(...)` from the browser bundle. See backend-proxy.md.
