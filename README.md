# lib-json-c

C backend of the `json` codec contract — the native-speed
implementation, written in Handl over the C FFI (`ffi.call`) against
**json-c** (`libjson-c`).

`src/index.hnl` implements the full `lib-json` surface as a single
module `json-c`: `stringify`, `parse`, and the `json` codec token
(`{ type: "codec", name: "json", encode, decode }`).

## Architecture

- **Parse** — `json_tokener_parse` (strict RFC 8259; malformed input
  or trailing garbage returns NULL → `error.raise`, matching
  `json-js`'s `throws` semantics since the contract has no `{err}`
  channel). The `json_object` tree is walked iteratively on an
  explicit job stack: scalars via `json_object_get_{type,boolean,
  int64,double,string}`, arrays via `json_object_array_length`/
  `_get_idx`, objects via the `lh_table` linked list
  (`json_object_get_object` + `ffi.read` of the `lh_entry`
  `{k,v,next}` layout). `json_object_put` releases the tree.
- **Stringify** — an explicit job stack builds the `json_object` tree
  (`json_object_new_*`, `object_add`/`array_add` steal child
  references), then `json_object_to_json_string_ext` with
  `JSON_C_TO_STRING_PLAIN` + `json_object_put`. Integral numbers with
  `|v| < 2^53` marshal as `int64` (`42` → `42`, not `42.0`); strings
  marshal length-prefixed (`json_object_new_string_len` over
  `bytes.utf8_encode`) so embedded NULs/non-ASCII round-trip.
- **Fallback** — when `libjson-c` cannot be loaded, both exports
  delegate to the `json` module (the pure-handl reference in
  lib-json); `jsonc_available`/`jsonc_version` report which path ran.
  Under the fallback `parse` keeps the reference's lenient semantics.

Divergences vs `lib-json`/`lib-json-handl`/`json-js`: strict parsing,
int64/double number split, control-character escaping with literal
UTF-8 passthrough, insertion-order key preservation, last-wins
duplicate keys, non-finite numbers → `null`. Full list in the
`index.hnl` header.

## Layout

```text
src/index.hnl       the .hnl sources (stringify / parse / json)
src/index.test.hnl  contract tests mirroring lib-json-js's suite
pkg.hnl             package manifest — modules, deps, description, license
```

Dependencies: json

System requirement: `libjson-c` (`apt install libjson-c-dev` on
Debian/Ubuntu; the runtime `libjson-c.so.5` suffices — no headers or
linking needed, `ffi.call` resolves symbols at runtime).

## Gate

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

CI skips the gate while `src/` has no `.hnl` sources.

## License

MIT.
