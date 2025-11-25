# Bathroom Configurator Master Thesis - LaTeX Setup

This repository contains the LaTeX source files for the Master Thesis on LiDAR-based product recommendation systems for bathroom renovations.

## Prerequisites

- **TeX Live 2025** (or later) installed on Windows. Download from [tug.org/texlive](https://tug.org/texlive/).
- **VS Code** with LaTeX extensions (e.g., LaTeX Workshop) for editing and compiling.
- Ensure `latexmk` and `biber` are available in your PATH (included with TeX Live).

## Project Structure

- `main.tex`: The main LaTeX file that includes all chapters and generates the PDF.
- `latex/00_preamble.tex`: Package imports, custom commands, and formatting settings.
- `latex/01_titlepage.tex`: Title page.
- `latex/01a_confidentiality_clause.tex`: Confidentiality clause.
- `latex/01b_gender_equality.tex`: Gender equality statement.
- `latex/02_executive_summary.tex`: Executive summary.
- `latex/03_chapters/`: Individual chapter files (01_introduction.tex, etc.).
- `latex/04_appendix.tex`: Appendix.
- `latex/05_declaration.tex`: Declaration.
- `references.bib`: Bibliography file (BibLaTeX format).
- Other generated files: `.aux`, `.bbl`, `.log`, etc. (ignore in version control).

## How to Generate the PDF

1. Open a terminal in the project root (`D:\EMC2\Thesis`).
2. Run the following command to compile the thesis:
   ```
   latexmk -pdf main.tex
   ```
   - This will generate `main.pdf` in the root directory.
   - `latexmk` handles multiple runs automatically for cross-references, table of contents, and bibliography.

3. If you encounter errors (e.g., "! Extra }, or forgotten \endgroup"):
   - Clean auxiliary files: `latexmk -c`
   - Recompile: `latexmk -pdf main.tex`

4. For bibliography updates:
   - If you modify `references.bib`, run `latexmk -pdf main.tex` again (it calls `biber` automatically).

## Writing and Updating Content

- **Edit chapters**: Modify `.tex` files in `latex/03_chapters/` (e.g., `01_introduction.tex`).
- **Add references**: Edit `references.bib` and cite in text using `\cite{key}`.
- **Formatting**: Use the styles defined in `00_preamble.tex` (e.g., chapters, sections).
- **Compile after changes**: Run `latexmk -pdf main.tex` to see updates in `main.pdf`.
- **VS Code tips**:
  - Install LaTeX Workshop extension.
  - Set `latex-workshop.chktex.path` to `D:/Apps/texlive/2025/bin/windows/chktex.exe` to avoid chktex errors.
  - Use Ctrl+Alt+B to build the PDF directly in VS Code.

## Common Issues

- **Fancyhdr warnings**: Fixed by setting `\headheight` in `00_preamble.tex`.
- **Bibliography not updating**: Run `biber main` manually if needed, then `latexmk -pdf main.tex`.
- **Overfull hbox**: Adjust text or use `\sloppy` for draft mode.
- **Missing packages**: Install via TeX Live Manager if prompted.

## Tips for Efficiency

- Use `latexmk -pvc main.tex` for continuous compilation (auto-recompiles on save).
- Ignore generated files (`.aux`, `.pdf`, etc.) in `.gitignore`.
- For quick checks, compile individual chapters if needed, but always use `main.tex` for the full document.

For questions, refer to the LaTeX documentation or TeX Stack Exchange.
