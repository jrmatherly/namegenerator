# Documentation Site Structure

This document describes the organization and structure of the namegenerator documentation, including guidelines for hosting, automation, and maintenance.

---

## Table of Contents

- [Documentation Overview](#documentation-overview)
- [File Structure](#file-structure)
- [Documentation CI/CD](#documentation-cicd)
- [Hosting Options](#hosting-options)
- [Search Configuration](#search-configuration)
- [Version Control](#version-control)
- [Maintenance Guidelines](#maintenance-guidelines)

---

## Documentation Overview

The namegenerator project uses a **multi-layered documentation approach**:

1. **Quick Start**: README.md for immediate onboarding
2. **API Reference**: Complete function and type documentation
3. **User Guides**: Step-by-step tutorials and common tasks
4. **Architecture**: Internal design and implementation details
5. **Examples**: Practical code samples across scenarios
6. **Diagrams**: Visual representations of architecture
7. **Index**: Central navigation and search

---

## File Structure

```
namegenerator/
├── README.md                      # Quick start (root level)
├── PROJECT_INDEX.md               # Repository index (auto-generated)
├── PROJECT_INDEX.json             # Machine-readable index
│
├── docs/                          # Documentation directory
│   ├── INDEX.md                   # Documentation hub
│   ├── API.md                     # Complete API reference
│   ├── USER_GUIDE.md              # Step-by-step user guide
│   ├── ARCHITECTURE.md            # Internal design & implementation
│   ├── USAGE_PATTERNS.md          # Advanced integration patterns
│   ├── CODE_EXAMPLES.md           # Practical code samples
│   ├── DIAGRAMS.md                # Architecture diagrams (Mermaid)
│   ├── DOCUMENTATION_SITE.md      # This file - site structure
│   │
│   ├── assets/                    # Static assets (if needed)
│   │   ├── images/
│   │   └── diagrams/
│   │
│   └── .nojekyll                  # Disable Jekyll on GitHub Pages
│
└── .serena/memories/              # AI context (not public docs)
    ├── project_purpose_and_structure.md
    ├── tech_stack_and_dependencies.md
    ├── code_style_and_conventions.md
    ├── suggested_commands.md
    └── task_completion_checklist.md
```

---

## Documentation Layers

### Layer 1: Quick Reference

| File | Audience | Purpose | Size |
| ------ | ---------- | --------- | ------ |
| `README.md` | New users | Installation & quick start | ~1 KB |
| `PROJECT_INDEX.md` | AI assistants | Efficient context loading | ~3 KB |

### Layer 2: User Documentation

| File | Audience | Purpose | Size |
| ------ | ---------- | --------- | ------ |
| `docs/INDEX.md` | All users | Documentation navigation | ~12 KB |
| `docs/USER_GUIDE.md` | Users | Step-by-step tutorials | ~15 KB |
| `docs/CODE_EXAMPLES.md` | Developers | Practical code samples | ~18 KB |

### Layer 3: API Documentation

| File | Audience | Purpose | Size |
| ------ | ---------- | --------- | ------ |
| `docs/API.md` | Developers | Complete API reference | ~18 KB |
| `docs/USAGE_PATTERNS.md` | Advanced users | Integration patterns | ~25 KB |

### Layer 4: Architecture Documentation

| File | Audience | Purpose | Size |
| ------ | ---------- | --------- | ------ |
| `docs/ARCHITECTURE.md` | Contributors | Design decisions | ~20 KB |
| `docs/DIAGRAMS.md` | Technical readers | Visual architecture | ~15 KB |

---

## Documentation CI/CD

### GitHub Actions Workflow

Create `.github/workflows/documentation.yml`:

```yaml
name: Documentation Generation & Deployment

on:
  push:
    branches: [main]
    paths:
      - '**.go'
      - 'docs/**'
      - 'README.md'

jobs:
  generate-docs:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Go
      uses: actions/setup-python@v4
      with:
        go-version: '1.23'

    - name: Generate PROJECT_INDEX
      run: |
        # Run index generation script if available
        # or commit manually maintained index

    - name: Validate Markdown
      run: |
        npm install -g markdownlint-cli
        markdownlint 'docs/**/*.md' 'README.md'

    - name: Check for broken links
      run: |
        npm install -g markdown-link-check
        find . -name "*.md" -exec markdown-link-check {} \;

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./docs
        enable_jekyll: false
```

---

## Hosting Options

### Option 1: GitHub Pages (Recommended)

**Advantages**:

- Free hosting
- Automatic deployment from `main` branch
- Custom domain support
- HTTPS by default

**Setup**:

1. **Enable GitHub Pages**:

   ```
   Repository Settings → Pages → Source: gh-pages branch
   ```

2. **Create `.nojekyll`**:

   ```bash
   touch docs/.nojekyll
   git add docs/.nojekyll
   git commit -m "Disable Jekyll on GitHub Pages"
   ```

3. **Configure index.html** (optional):

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>namegenerator Documentation</title>
       <meta http-equiv="refresh" content="0; url=docs/INDEX.html">
   </head>
   <body>
       <p>Redirecting to <a href="docs/INDEX.html">documentation</a>...</p>
   </body>
   </html>
   ```

**Access**: `https://username.github.io/namegenerator/`

---

### Option 2: pkg.go.dev (Automatic)

**Advantages**:

- Automatic for Go modules
- Standard Go documentation format
- Integrated with Go tooling

**Access**: `https://pkg.go.dev/github.com/obot-platform/namegenerator`

**Note**: Uses godoc comments from source code.

---

### Option 3: Read the Docs

**Advantages**:

- Professional documentation hosting
- Version management
- Search functionality
- Custom themes

**Setup**: Create `docs/conf.py` for Sphinx configuration.

---

### Option 4: GitBook

**Advantages**:

- Beautiful UI
- Collaborative editing
- Version control
- Search and navigation

**Setup**: Connect repository to GitBook.

---

## Search Configuration

### Option 1: Algolia DocSearch (Free for Open Source)

Add to documentation pages:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@docsearch/css@3" />
<script src="https://cdn.jsdelivr.net/npm/@docsearch/js@3"></script>

<script>
  docsearch({
    appId: 'YOUR_APP_ID',
    apiKey: 'YOUR_API_KEY',
    indexName: 'namegenerator',
    container: '#docsearch',
  });
</script>
```

---

### Option 2: Simple Client-Side Search

Create `docs/search.js`:

```javascript
const searchIndex = {
  'API.md': {
    title: 'API Reference',
    content: 'Complete API reference for namegenerator...',
    url: 'API.html'
  },
  'USER_GUIDE.md': {
    title: 'User Guide',
    content: 'Getting started with namegenerator...',
    url: 'USER_GUIDE.html'
  }
  // ... more entries
};

function search(query) {
  const results = [];
  const lowerQuery = query.toLowerCase();

  for (const [file, doc] of Object.entries(searchIndex)) {
    if (doc.title.toLowerCase().includes(lowerQuery) ||
        doc.content.toLowerCase().includes(lowerQuery)) {
      results.push(doc);
    }
  }

  return results;
}
```

---

## Version Control

### Versioning Strategy

**For Major Releases**:

```
docs/
├── v1/
│   ├── INDEX.md
│   ├── API.md
│   └── ...
├── v2/
│   ├── INDEX.md
│   ├── API.md
│   └── ...
└── latest/  (symlink to current version)
```

**Version Badge in Documentation**:

```markdown
**Version**: 1.0.0 | [View other versions](versions.html)
```

---

### Changelog Integration

Link changelog in documentation:

```markdown
## Recent Changes

See [CHANGELOG.md](../CHANGELOG.md) for detailed version history.

### Latest Release (v1.0.0)
- Initial release with core functionality
- 62 adjectives, 58 nouns (3,596 combinations)
- Thread-safety documentation
```

---

## Maintenance Guidelines

### Documentation Review Checklist

Before each release:

- [ ] **Accuracy**: All code examples tested and working
- [ ] **Completeness**: All public APIs documented
- [ ] **Links**: No broken internal/external links
- [ ] **Consistency**: Terminology used consistently
- [ ] **Examples**: Code examples are up-to-date
- [ ] **Diagrams**: Architecture diagrams reflect current design
- [ ] **Versioning**: Version numbers updated
- [ ] **Changelog**: Release notes documented

---

### Automated Checks

**Markdown Linting**:

```bash
markdownlint docs/**/*.md
```

**Link Checking**:

```bash
markdown-link-check docs/**/*.md
```

**Spell Checking**:

```bash
aspell check -l en docs/**/*.md
```

---

### Documentation Coverage Tracking

Create `scripts/doc-coverage.sh`:

```bash
#!/bin/bash

# Count exported symbols in Go code
TOTAL_EXPORTS=$(grep -r "^func\|^type.*struct\|^var\|^const" *.go | grep -v "_test.go" | wc -l)

# Count documented symbols in API.md
DOCUMENTED=$(grep -c "^### \|^#### " docs/API.md)

COVERAGE=$(echo "scale=2; $DOCUMENTED / $TOTAL_EXPORTS * 100" | bc)

echo "Documentation Coverage: ${COVERAGE}%"
echo "Total Exports: $TOTAL_EXPORTS"
echo "Documented: $DOCUMENTED"

if (( $(echo "$COVERAGE < 100" | bc -l) )); then
    echo "⚠️  Warning: Documentation coverage is below 100%"
    exit 1
fi

echo "✅ Documentation coverage is complete"
```

---

### Continuous Improvement

**Regular Reviews**:

- **Quarterly**: Review and update examples
- **Per Release**: Update API documentation
- **Annually**: Comprehensive documentation audit

**User Feedback**:

- GitHub Issues with `documentation` label
- Documentation improvement suggestions
- FAQ updates based on common questions

---

## Navigation Structure

### Primary Navigation

```markdown
📚 Documentation
├── 🚀 Quick Start (README.md)
├── 📖 User Guide (docs/USER_GUIDE.md)
├── 🔧 API Reference (docs/API.md)
├── 💡 Code Examples (docs/CODE_EXAMPLES.md)
├── 🏗️ Architecture (docs/ARCHITECTURE.md)
├── 📊 Diagrams (docs/DIAGRAMS.md)
└── 🔍 Advanced Patterns (docs/USAGE_PATTERNS.md)
```

### Cross-References

Every documentation file should include:

**Top of file** - Navigation breadcrumbs:

```markdown
[Documentation Home](INDEX.md) > [API Reference](API.md)
```

**Bottom of file** - Related documentation:

```markdown
## Related Documentation
- [User Guide](USER_GUIDE.md) - Getting started
- [Examples](CODE_EXAMPLES.md) - Practical code samples
- [Architecture](ARCHITECTURE.md) - Design details
```

---

## Documentation Metrics

Track these metrics:

| Metric | Target | Current |
| -------- | -------- | --------- |
| API Coverage | 100% | 100% |
| Code Examples | 15+ | 16 |
| Diagrams | 10+ | 12 |
| Page Views (monthly) | - | Track in analytics |
| Avg. Session Duration | 3+ min | Track in analytics |
| Bounce Rate | < 50% | Track in analytics |

---

## Documentation Style Guide

### Writing Guidelines

1. **Be Concise**: One idea per sentence
2. **Use Active Voice**: "Generate names" not "Names are generated"
3. **Include Examples**: Every concept needs a code example
4. **Format Code**: Use syntax highlighting for all code blocks
5. **Link Generously**: Cross-reference related documentation

### Code Example Standards

```go
// ✅ Good: Clear, concise, with comments
seed := time.Now().UTC().UnixNano()
gen := namegenerator.NewNameGenerator(seed)
name := gen.Generate() // Returns: "silent-moon"

// ❌ Bad: No context, unclear purpose
gen := namegenerator.NewNameGenerator(12345)
n := gen.Generate()
```

---

## Accessibility

Ensure documentation is accessible:

- **Heading Hierarchy**: Proper H1 → H2 → H3 nesting
- **Alt Text**: All images have descriptive alt text
- **Link Text**: Descriptive ("API Reference" not "click here")
- **Code Contrast**: High contrast syntax highlighting
- **Keyboard Navigation**: Logical tab order

---

## Internationalization (Future)

If translating documentation:

```
docs/
├── en/          # English (primary)
│   ├── INDEX.md
│   └── ...
├── ja/          # Japanese
│   ├── INDEX.md
│   └── ...
└── zh/          # Chinese
    ├── INDEX.md
    └── ...
```

---

## Analytics Integration

### Google Analytics

Add to documentation pages:

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### Track Key Events

- Documentation page views
- Search queries
- External link clicks
- Code example copies

---

## Related Documentation

- [Documentation Index](INDEX.md) - Main documentation hub
- [Project Index](../PROJECT_INDEX.md) - Repository structure
- [Contributing Guide](../CONTRIBUTING.md) - How to contribute

---

**Last Updated**: 2026-01-16
**Maintainer**: Documentation Team
**Version**: 1.0.0
