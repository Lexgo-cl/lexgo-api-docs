# LexgoSign API Documentation

This directory contains the complete API documentation for LexgoSign, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Quick Start

### Prerequisites

- Python 3.8 or higher
- pip (Python package manager)

### Installation

1. Install dependencies:

```bash
pip install -r requirements.txt
```

### Local Development

1. Start the development server:

```bash
mkdocs serve
```

2. Open your browser to `http://localhost:8000`

3. Edit markdown files in `docs/` - changes auto-reload

### Building

Build the static site:

```bash
mkdocs build
```

Output will be in the `site/` directory.

## Documentation Structure

```
api/
├── mkdocs.yml              # MkDocs configuration
├── requirements.txt        # Python dependencies
├── docs/                   # Documentation content
│   ├── index.md           # Homepage
│   ├── getting-started/   # Getting started guides
│   │   ├── introduction.md
│   │   ├── authentication.md
│   │   └── quick-start.md
│   ├── api/               # API reference
│   │   ├── overview.md
│   │   ├── envelopes.md
│   │   ├── recipients.md
│   │   ├── validations.md
│   │   ├── settings.md
│   │   ├── webhooks.md
│   │   └── errors.md
│   ├── guides/            # Feature guides
│   │   ├── email-validation.md
│   │   ├── email-templates.md
│   │   ├── webhook-integration.md
│   │   └── evidence-sheet.md
│   ├── examples/          # Code examples
│   │   ├── create-envelope.md
│   │   ├── email-validation-flow.md
│   │   └── webhook-subscriptions.md
│   └── changelog.md       # Version history
└── site/                  # Built site (gitignored)
```

## Writing Documentation

### Markdown Extensions

We use several markdown extensions for enhanced documentation:

#### Code Blocks with Syntax Highlighting

````markdown
```python
def hello_world():
    print("Hello, World!")
```
````

#### Admonitions (Callouts)

```markdown
!!! note "Optional Title"
    This is a note callout

!!! warning
    This is a warning

!!! danger "Critical"
    This is a danger callout

!!! tip
    This is a tip
```

#### Tabs

```markdown
=== "Python"
    ```python
    print("Hello")
    ```

=== "JavaScript"
    ```javascript
    console.log("Hello");
    ```
```

#### Tables

```markdown
| Column 1 | Column 2 |
|----------|----------|
| Value 1  | Value 2  |
```

### Style Guide

- Use **present tense** ("Create an envelope" not "Creates an envelope")
- Use **active voice** ("The API returns..." not "A response is returned...")
- Include **code examples** for all endpoints
- Show **both request and response** examples
- Document all **error cases**
- Link to related documentation
- Use consistent terminology

### File Naming

- Use kebab-case: `email-validation.md`
- Be descriptive: `webhook-integration.md` not `webhooks.md`
- Group by topic in subdirectories

## Deployment

### GitHub Pages

Deploy to GitHub Pages:

```bash
mkdocs gh-deploy
```

### Custom Domain

To deploy to a custom domain:

1. Add `site_url` in `mkdocs.yml`:

```yaml
site_url: https://docs.lexgo.cl
```

2. Configure DNS:

```
CNAME docs.lexgo.cl -> yourusername.github.io
```

3. Deploy:

```bash
mkdocs gh-deploy
```

### Netlify

Deploy to Netlify:

1. Create `netlify.toml`:

```toml
[build]
  command = "mkdocs build"
  publish = "site"

[build.environment]
  PYTHON_VERSION = "3.8"
```

2. Connect repository to Netlify

3. Deploy automatically on push

## Contributing

### Adding New Pages

1. Create markdown file in appropriate `docs/` subdirectory
2. Add entry to `nav` in `mkdocs.yml`:

```yaml
nav:
  - API Reference:
    - Your New Page: api/your-new-page.md
```

3. Test locally with `mkdocs serve`
4. Commit and push

### Updating Content

1. Edit markdown files in `docs/`
2. Test changes locally
3. Commit with descriptive message
4. Push to repository

## CI/CD

### GitHub Actions

Create `.github/workflows/docs.yml`:

```yaml
name: Deploy Documentation

on:
  push:
    branches: [main, master]
    paths:
      - 'api/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: 3.x
      - run: pip install -r api/requirements.txt
      - run: cd api && mkdocs gh-deploy --force
```

## Maintenance

### Updating Dependencies

Keep MkDocs and plugins up to date:

```bash
pip install --upgrade -r requirements.txt
pip freeze > requirements.txt
```

### Link Checking

Check for broken links:

```bash
pip install mkdocs-linkcheck
mkdocs build
```

### Search Index

Search is automatically built and updated on every build.

## Troubleshooting

### Port Already in Use

If port 8000 is busy:

```bash
mkdocs serve -a localhost:8001
```

### Build Errors

Clear site directory and rebuild:

```bash
rm -rf site/
mkdocs build --clean
```

### Plugin Errors

Reinstall dependencies:

```bash
pip install --force-reinstall -r requirements.txt
```

## Resources

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Markdown Guide](https://www.markdownguide.org/)
- [LexgoSign API](https://api.lexgo.cl)

## Support

For documentation issues:
- **GitHub Issues**: Report bugs or suggest improvements
- **Slack**: #documentation channel

## License

Copyright © 2025 Lexgo. All rights reserved.
