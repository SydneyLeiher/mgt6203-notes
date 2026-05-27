# MGT 6203 — Data Analytics for Business Notes

Class notes for MGT 6203 at Georgia Tech, built with MkDocs + Material theme,
hosted on GitHub Pages.

**Live site:** https://SydneyLeiher.github.io/mgt6203-notes

---

## Setup (one time only)

### 1. Install Python and MkDocs

```bash
pip install -r requirements.txt
```

### 2. Preview locally in your browser

```bash
mkdocs serve
```
Then open http://127.0.0.1:8000 — the site live-reloads as you edit files.

### 3. Push to GitHub to publish

```bash
git add .
git commit -m "Add week 2 notes"
git push
```
GitHub Actions automatically builds and deploys the site. Done.

---

## Adding a New Week

1. Create a new file: `docs/weeks/week02-binary-response.md`
2. Copy the structure from an existing week file
3. Add it to `mkdocs.yml` under `nav:`

```yaml
nav:
  - Home: index.md
  - Week 1:
    - Overview of Data Analytics: weeks/week01-overview.md
    - Linear Models: weeks/week01-linear-models.md
  - Week 2:                                          # ← add this
    - Binary Response Models: weeks/week02-binary-response.md
```

4. Commit and push — the site updates automatically.

---

## File Structure

```
mgt6203-notes/
├── mkdocs.yml              ← site config and navigation
├── requirements.txt        ← Python dependencies
├── .github/
│   └── workflows/
│       └── deploy.yml      ← auto-deploy to GitHub Pages
└── docs/
    ├── index.md            ← home page
    ├── assets/
    │   └── custom.css      ← custom styling
    └── weeks/
        ├── week01-overview.md
        ├── week01-linear-models.md
        └── ...             ← add new weeks here
```

---

## MkDocs Formatting Cheatsheet

### Math formulas
```
Inline: $y = \beta_0 + \beta_1 x$
Block:  $$R^2 = \frac{ESS}{TSS}$$
```

### Callout boxes
```
!!! note "Title"       ← blue info box
!!! tip "Title"        ← green tip box
!!! warning "Title"    ← yellow warning box
!!! danger "Title"     ← red danger box
```

### Tabs
```
=== "Option A"
    Content for A

=== "Option B"
    Content for B
```

### Code blocks
```
    ```r
    model <- lm(Price ~ Age + KM, data = cars)
    summary(model)
    ```
```

### Tables
```
| Column 1 | Column 2 |
|---|---|
| Value 1  | Value 2  |
```
