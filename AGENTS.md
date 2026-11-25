# lib-json-c

`json-c` — C backend of the `json` codec contract: parse/serialize via
libjson-c (`libjson-c`) through the C FFI (`ffi.call`); falls back to
the pure-handl `json` module when libjson-c is absent.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the full `lib-json` surface as one module — `stringify`, `parse`, and the `json` codec token `{ type: "codec", name: "json", encode, decode }` |
| `src/index.test.hnl` | tests |
| `pkg.hnl` | manifest — dep `json` (`branch:main`), `modules: ["json-c"]` |
| `scripts/hnl-gate.sh` | `.hnl` gate |
| `workflows/gate.yaml` | CI — shared `handl-gates` campaign |

## Contract

- Native-speed counterpart of `lib-json`; same surface, same lenient
  `parse` semantics — the fallback path must stay behavior-identical.
- Pure `.hnl` over `ffi.call`; no Rust crate.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
