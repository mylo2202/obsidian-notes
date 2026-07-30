# Obsidian Notes

A repository for managing personal notes.

All notes are written in Markdown, and can be converted to TeX and PDF formats.

## Convert Markdown to other formats

Create a `yaml` file to configure `pandoc` parameters.

```yaml
pdf-engine: xelatex

variables:
  mainfont: TeX Gyre Pagella
  mathfont: TeX Gyre Pagella Math
  fontsize: 12pt
  geometry: margin=2cm
```

To convert a Markdown to a TeX document:

```bash
pandoc note.md --defaults=pdf.yaml -o note.tex
```

To convert a Markdown to a PDF document:

```bash
pandoc note.md --defaults=pdf.yaml -o note.pdf
```
