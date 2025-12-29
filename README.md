```markdown
# contrib/tui-python

A small, experimental Textual TUI that shells out to the upstream `witr` binary and displays listeners, process details, and parent chains. This is intentionally non-invasive: the Go binary remains the source of truth and the TUI consumes machine-readable output (JSON).

This prototype is intended to be cross-platform (Windows, macOS, Linux) and lightweight — useful for experimenting with UX without changing upstream behavior.

## Goals
- Provide a friendly interactive view of what `witr` reports.
- Rely on the compiled `witr` binary (no rewrite of core scanner).
- Require minimal changes upstream: ideally `witr --json` (or use the included shim).
- Demonstrate Windows-friendly usage and packaging options.

## Requirements
- Python 3.8+
- Recommended (install into a virtualenv):
  pip install -r requirements.txt

Example requirements (contrib/tui-python/requirements.txt):
```
textual
rich
```

(If you add local parsing instead of shelling out, you might need `psutil`.)

## Quick start (assumes `witr --json` exists)
1. Ensure a `witr` binary is available on PATH (or point to it with `--witr-path`).
2. Install dependencies:
   ```
   python -m pip install -r contrib/tui-python/requirements.txt
   ```
3. Run the prototype:
   ```
   python -m contrib.tui_python.main --port 41609 --witr-path ./witr
   ```

If upstream `witr` doesn’t have a `--json` flag, use the provided shim (or build a small wrapper) that converts stdout to the expected JSON schema.

## Expected JSON shape
The TUI expects a stable JSON structure. Minimal example:
```json
{
  "target": "port 41609",
  "process": {
    "pid": 658,
    "name": "cloudflared",
    "user": "wenekar",
    "cmdline": "/usr/bin/cloudflared proxy-dns --port 5053 ...",
    "started": "4-13:22:23",
    "rss_mb": 48
  },
  "parent_chain": [
    {"pid": 1, "name": "systemd"},
    {"pid": 658, "name": "cloudflared"}
  ],
  "listening": ["127.0.0.1:41609"]
}
```
If your `witr` JSON differs, adapt the small parser in `main.py` or adjust the shim.

## Windows notes
- Python works well on Windows 11 (use python.org installer or Microsoft Store).
- Use Windows Terminal or ConHost configured for UTF-8 for best TUI experience.
- To distribute for non-Python users, bundle the TUI with PyInstaller:
  ```
  pyinstaller --onefile --name witr-tui contrib/tui-python/main.py
  ```
  Ship the compiled `witr` binary (Windows build) alongside the bundled TUI for an out-of-the-box experience.

## How this repo differs from upstream
- This is a UX/experimentation fork intended to be a companion to the Go binary, not a rewrite/replacement.
- Focus is on cross-platform usability (Windows-friendly) and a JSON-first contract for frontends.

## Contributing
- Keep changes small and focused: examples include `--json` support, Windows packaging, or incremental UI improvements.
- If something proves useful broadly, we can propose a non-breaking PR upstream.
- Open issues or PRs in this fork for UI improvements or compatibility fixes.

## Attribution
This project is a companion experiment derived from the upstream `witr` project. See the upstream repository for original source and license information.

```
