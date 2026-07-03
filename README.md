### brace.yazi
Yazi plugin for creating directories using Bash-like brace expansion — pure Lua,
no shell calls, no external dependencies.

---

- [Installation](#Installation)
- [Configuration](#Keymap Configuration)
- [Examples](#Usage Examples)
- [Supported Patterns](#Supported Patterns)
- [Architecture](#Architecture)
- [Other Modules](#Other Modules)
- [Testing](#Testing)
- [Limitations](#Intentional Limitations)
- [License](#License)
- [Note](#Note on Yazi API)

---

### Features
- Comma list: project/{src,include,docs,tests}
- Nested braces: project/{src/{main,test},docs}
- Numeric ranges (with zero-padding & reverse): chapter{1..5}, img{01..03}
- Alphabetic ranges: {A..D}, {a..d}
- Cartesian product: {android,desktop}/{debug,release}
- Recursive mkdir (mkdir -p equivalent) using Yazi's filesystem API
- Preview before execution with Enter/Esc confirmation
- Clear error messages for invalid patterns — no crashes


### Example of brace:

<p align="center">
  <img src="assets/acp_demo.gif" alt="ACP terminal demo: core command, show mode, and security check" width="700">
</p>

<p align="center"><sub>Real terminal output — core command, <code>-s</code> show mode, and <code>--check</code> security scan.</sub></p>

### Installation
Clone into your Yazi plugins directory:
```sh
git clone https://github.com/hastagaming/brace.yazi.git ~/.config/yazi/plugins/brace.yazi
```
or use the official yazi package:
```yazi
ya pkg add hastagaming/brace
```

### Keymap Configuration
Add this to ~/.config/yazi/keymap.toml:
```toml
[[manager.prepend_keymap]]
on   = ["c", "b"]
run  = "plugin brace"
desc = "Create directories with brace expansion"
```
Press c then b inside Yazi to open the brace expansion prompt. Change the
key combination to whatever you prefer.

### Usage Examples
1. Press cb in Yazi.
2. Type a pattern, e.g.:
```code
project/{src,include,docs,tests}
```
3.Plugin displays a preview:
```code
Will create 4 directories
  ✓ project/src
  ✓ project/include
  ✓ project/docs
  ✓ project/tests
```
4. Press Enter to create all directories, or Esc to cancel.

---

### Supported Patterns
|Pattern | Output|
|project/{src,include,docs} | project/src, project/include, project/docs|
|project/{src/{main,test},docs} | project/src/main, project/src/test, project/docs|
|chapter{1..5} | chapter1, chapter2, ..., chapter5|
|img{01..05} | img01, img02, ..., img05 (zero-padded)|
|{A..D} | A, B, C, D|
|{a..d} | a, b, c, d|
|{d..a} | d, c, b, a (reverse)|
|{ios,android}/{arm64,x86} | ios/arm64, ios/x86, android/arm64, android/x86|
Invalid patterns like project/{src,docs (unbalanced braces) are rejected with
a clear error message. The plugin will not crash.

### Architecture
Parser (parser.lua)
The parser is a pure Lua string processor with no knowledge of Yazi, filesystem,
or any I/O. It works in five steps:

1. Top-level brace balance check — if { count ≠ } count, return error immediately.
2. Find matching brace pair — locate the first { and its matching } by tracking depth.
3. Split into three parts — prefix (before {), body (inside), suffix (after }).
4. Interpret body as one of:
  - Numeric range: 1..5, 05..01 (preserves zero-padding)
  - Alphabetic range: a..d, D..A
  - Comma-separated list: split on top-level commas only (ignore commas
  - nested inside inner braces)
  - Literal text: if it's {single_item} with no comma, keep it literal (Bash behavior)
5. Recursive expansion of suffix and each atom from the body. This enables nested braces,
cartesian products, and deeply nested patterns.

---

Because the parser is decoupled, it can be tested directly with plain lua
without running Yazi.

### Other Modules
- fs.lua — The only module touching filesystem. Uses Yazi's built-in fs.create("dir_all", url)
API (equivalent to mkdir -p).
- preview.lua — Displays a list of directories to be created via ya.notify,
then requests confirmation (Enter/Esc) via ya.input.
- main.lua — Orchestrates the full flow: prompt pattern → parse → dedup & sort →
resolve to absolute paths → preview → create directories
→ notify result → refresh Yazi.
- util.lua — Small stateless helpers (trim, is_blank, dedup, sort) to avoid code duplication.

### Testing
All tests are pure Lua and require no Yazi runtime:
```sh
cd brace.yazi/tests
lua parser_test.lua
lua range_test.lua
lua nested_test.lua
lua cartesian_test.lua
```

### Intentional Limitations
This plugin does not include:
Project templates
Pattern history
Variable expansion ($VAR)
Bash execution or shell spawning
External library dependencies
Its sole purpose: brace expansion for directory creation.

### License
MIT — see [LICENSE](./LICENSE) file.

---

## Note on Yazi API:
The plugin uses official Yazi plugin APIs: ya.input, ya.notify,
ya.manager_emit, fs.create, fs.cha, Url, and cx.active.current.cwd.
If your Yazi version differs slightly, check the built-in plugin reference
(e.g., the rename plugin) in your Yazi installation directory to align
function signatures. The parser, workflow, and modular structure remain stable.