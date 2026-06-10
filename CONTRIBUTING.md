# Contributing

Thanks for taking a look! `thehive-cli` is a single-file Python 3 tool — bug reports, ideas
and pull requests are all welcome.

## Reporting issues

Open a GitHub issue with:
- what you ran (the command),
- what you expected, and what actually happened,
- relevant output from re-running with `--DEBUG` (please **redact** URLs, API keys and any case data).

## Pull requests

- Keep it a **single file** (`thehive-cli`) and match the surrounding style — **tabs** for indentation.
- Before submitting, make sure it still compiles and is indentation-clean:

  ```bash
  python3 -m py_compile thehive-cli
  python3 -m tabnanny thehive-cli
  ```

- Adding a flag? Put the argument in the right group in `parse_arguments()`, wire up its handler,
  and update **README.md** and **CHANGES.md**.
- Prefer the native [`thehive4py`](https://github.com/TheHive-Project/TheHive4py) methods where they exist,
  and fall back to the raw query API (`api.session.make_request(...)`) only when needed.

## Testing

There's no live TheHive instance in CI, so the easiest things to verify are the **pure-logic helpers**
(parsers, the file sorter, the editor diff/lost-update guards) — extract a function and exercise it, or
run end-to-end against a test TheHive instance.

## Not affiliated

This project is independent and **not associated with, endorsed by, or supported by StrangeBee or TheHive**.
