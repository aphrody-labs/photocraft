# PhotoCraft engine library

`photocraft-engine` is the existing headless L5 owner, shared by the desktop UI,
CLI and WebAssembly app. It has no dependency on egui, eframe, winit or rfd.
Consumers should reuse this crate and the lower-level owners rather than copy
editing state or codec implementations into a new facade crate.

## Byte-oriented hosts

```rust,no_run
use photocraft_engine::{Session, file_cmds};
use serde_json::json;

let mut session = Session::new();
session.execute("file.new", json!({"width": 640, "height": 480, "depth": 16}))?;
session.execute("layer.new.layer", json!({"name": "Overlay"}))?;
let output = file_cmds::export_active_bytes(&session, "download.pcraft", &json!({}))?;
// The host persists output.bytes and reports output.warnings.
# Ok::<(), photocraft_engine::EngineError>(())
```

Import with `file_cmds::open_bytes_as`; place embedded assets with
`file_cmds::place_bytes`. Both accept bytes supplied by a Bun or browser host.
`export_active_bytes` uses the same encoding logic, source encoding and File
Handling preferences as the filesystem save commands. It does not write the
destination, rename the document, reset its dirty state or change history.
Linked assets retain the existing native IO resolution rules against the source
document path; this API does not turn native linked assets into a filesystem
sandbox. Browser hosts supply embedded assets instead.

`Session::execute`, `command_specs`, `Session::undo` and `Session::redo` expose the
existing editing and timeline commands. `Session::active` exposes document and
layer state. For direct typed APIs use the existing owners:

| Owner | Capability |
| --- | --- |
| `photocraft-doc` | Documents, layers, masks and timeline data |
| `photocraft-compose` | CPU compositing |
| `photocraft-io` | Format import/export, options and loss warnings |
| `photocraft-codecs` | Image codec registry and compression |
| `photocraft-format` | Native `.pcraft` persistence |

The byte-export tests live in `src/file_cmds/tests.rs` under
`headless_byte_export_`. They cover layered 16-bit native round trips, PNG bytes,
invalid parameters, absent documents, unchanged save identity and restoration of
the document-path scope after an encoder error. Native and wasm32 compilation
and execution must be qualified separately; a native test is not browser proof.
