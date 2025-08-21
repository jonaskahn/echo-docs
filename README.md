# Echo Documentation (submodule)

This directory is a Git submodule pointing to `jonaskahn/echo-docs`. It contains the MkDocs site for the Echo project.

## Submodule usage (in the main repo)

```bash
# add (from repo root)
git submodule add -b main https://github.com/jonaskahn/echo-docs docs

# initialize & update
git submodule update --init --recursive

# pull latest docs later
git submodule update --remote docs
```

## Local development

```bash
cd docs
poetry install --no-root
poetry run mkdocs serve  # http://127.0.0.1:8000
```

## Build

```bash
poetry run mkdocs build --strict
```

## Notes

- MkDocs config: `mkdocs.yml` (project root)
- Content: `docs/` directory (this repo)
- Mermaid diagrams are rendered via `mkdocs-mermaid2`.

