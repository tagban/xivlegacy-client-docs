# Graphics

The game draws with Direct3D 9: shader model 2 and 3 shaders, fixed-function bits and D3DX helpers.
We implement the D3D9 API ourselves and draw through Metal.

## Shape of the renderer

- **D3D9 front end:** devices, state, state blocks, resources, textures, render targets.
- **Metal back end:**
  - pipelines, depth-stencil and sampler states;
  - a per-frame ring for vertex, index and constant data;
  - render passes and present.
- **Shader translation:** D3D9 shader bytecode is parsed and translated to Metal Shading Language,
  one vertex/pixel shader pair at a time.
- **A CPU reference:** an interpreter for D3D9 vertex shaders checks a draw's translated shader
  against the original bytecode, vertex by vertex. This is how we tell translator bugs from game
  behavior.

## Findings

- **Clears must survive target switches (the cutscene smear).**
  - The Metal back end turns a whole-target Clear into the next pass's load action, which is cheap.
  - When the game switched render targets before drawing anything, the pending clear was dropped.
  - Cutscenes then kept the previous frame's pixels wherever a frame drew nothing: a moving camera
    smeared into radial streaks, and characters doubled.
  - Fix: carry out a pending clear before targets change, before copies, reads and present.
- **Half-pixel offset:** D3D9 pixel centers are half a pixel off Metal's, so positions move by
  (+1/width, −1/height) × w.
- **No negative base vertex:** Metal rejects it, so the base vertex is folded into the indices.
- **Vertex ranges:** for indexed draws, the vertex range comes from the indices actually used.
  Some draws supply less vertex data than their index range claims.
- **Walking stutter came from compiles and the guest lock.**
  - Shader libraries and pipelines are now compiled in the background. A draw whose pipeline isn't
    ready is skipped for a frame instead of stalling.
  - Only the drawing thread gets long lock turns (see [Architecture](architecture.md)).
  - Together these cut hitches while walking from about 15–19 to 1–2 per run, at 50–55 fps.
  - One periodic ~150 ms hitch is still undiagnosed.
- **Instrumentation that found these:** a slow-frame log (frames over 45 ms, broken down into
  other threads' lock time, compiles, pipeline builds, uploads and file reads), and a sampling
  profiler on the drawing thread.

## Debug switches we found useful

- Wireframe mode, gray textures and gray render targets, to tell geometry from texturing problems.
- Skipping or isolating one vertex shader.
- Dumping a draw's inputs, or a texture or render target by id.
- Tracing one frame's targets, clears, copies and draws.
