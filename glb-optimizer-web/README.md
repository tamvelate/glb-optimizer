# GLB Optimizer

A browser-only tool that shrinks glTF/GLB 3D model files by deduplicating and
resizing their embedded textures. Mesh geometry, materials and node structure
are never touched, so parts and shape stay exactly intact.

Everything runs client-side — the file is never uploaded anywhere. It works
two ways:

- **As a static web page** — open `index.html` in any modern desktop browser
  (Chrome, Edge, Firefox, Safari), or serve the folder with any static host.
- **As a Claude Artifact** — the same file, published on claude.ai, with
  results delivered through the Artifact `downloads` capability (wrapped in
  a `.zip`, since `.glb` isn't on that capability's allowed-extension list).
  The page detects which context it's running in and adapts automatically.

## How it works

A GLB file is a binary container: a 12-byte header, a `JSON` chunk
describing the scene graph, materials and accessors, and a `BIN` chunk
holding the raw bytes — vertex data *and* textures — that the JSON's
`bufferViews` point into.

This tool only touches the byte ranges `images[].bufferView` points to:

1. **Dedup** — identical textures (common when several materials reuse the
   same image) are written once and shared by every `bufferView` that
   pointed at them.
2. **Resize** — a texture larger than the selected max dimension is decoded,
   scaled down, and losslessly re-encoded as PNG, dropping the alpha channel
   when the texture turns out to be fully opaque (no information lost).
3. **Pass-through** — a texture already at or under the max dimension is
   left completely untouched. Browsers' built-in PNG encoders compress more
   weakly than the native tools desktop apps use (e.g. libvips/sharp), so
   re-encoding a texture that doesn't need resizing can make it *larger*,
   not smaller — the original bytes already win.

Every accessor, mesh, material, node and animation keeps its original index
and original bytes, so geometry is bit-for-bit identical and per-part
materials stay separate.

## Known limitation

Because browsers don't expose a tunable PNG compression level the way
native tools do, this tool generally won't match the compression ratio of a
desktop pipeline built on `sharp`/`libvips` at the same max-texture setting,
especially on files with many very large (4K+) textures. If you need the
smallest possible output, lowering "Max texture size" closes most of the
gap, or use a native desktop tool for the best ratio at full resolution.

## Development

Single self-contained `index.html` — no build step, no dependencies. Open
it directly in a browser to test changes.

## License

Private/internal — all rights reserved.
