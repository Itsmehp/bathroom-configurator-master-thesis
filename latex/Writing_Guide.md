# Writing Guide for Bathroom Configurator Master Thesis

This guide helps you format text, add sections, lists, and other elements in your LaTeX thesis easily. Copy-paste the code snippets and replace with your content.

## Text Alignment

LaTeX justifies text by default (aligns to both left and right margins). To change alignment:

- **Left-aligned (ragged right)**: Use `\raggedright` before the text.
- **Right-aligned**: Use `\raggedleft`.
- **Centered**: Use `\centering`.
- **Justified**: Default, or use `\justifying` (from ragged2e package).

For the whole document, add the command in `00_preamble.tex` after `\onehalfspacing`.

Example for left-aligned text:
```latex
\raggedright
Gender equality is highly expected in this thesis. For readability, the masculine spelling is used throughout the document, but female and other gender identities are explicitly included where necessary.
```

To revert to justified, use `\justifying`.

## Adding Sections and Subsections

LaTeX automatically numbers sections, subsections, etc. Just use the commands; numbering updates automatically.

- **Section**: `\section{Title}`
- **Subsection**: `\subsection{Title}`
- **Subsubsection**: `\subsubsection{Title}`
- **Paragraph**: `\paragraph{Title}` (unnumbered by default)

Example:
```latex
\section{System Requirements and Architecture Designing}
\label{sec:SystemRequirements}

This section covers...

\subsection{Functional Requirements}
\label{sub:FunctionalRequirements}

Details here...

\subsubsection{Detailed Subpoint}
More details...
```

Copy-paste and replace "Title" with your text. Labels are optional for cross-references.

## Adding Lists

Use itemize for bullet lists, enumerate for numbered lists.

- **Bullet list**:
```latex
\begin{itemize}
    \item First item
    \item Second item
    \item Third item
\end{itemize}
```

- **Numbered list**:
```latex
\begin{enumerate}
    \item First item
    \item Second item
    \item Third item
\end{enumerate}
```

- **Nested lists**:
```latex
\begin{itemize}
    \item Main item
    \begin{itemize}
        \item Sub-item
        \item Another sub-item
    \end{itemize}
    \item Next main item
\end{itemize}
```

## Other Common Elements

- **Bold text**: `\textbf{Text}`
- **Italic text**: `\textit{Text}`
- **Code**: `\texttt{Code}`
- **Footnotes**: `Text with footnote\footnote{Footnote content}.`
- **Citations**: `\cite{key}` (add to `references.bib`)
- **Figures/Tables**: See existing examples in chapters.
- **Page break**: `\newpage` or `\cleardoublepage`

## Tips

- Save and compile with `latexmk -pdf main.tex` to see changes.
- Use VS Code's word wrap for readability.
- For math, use `$equation$` inline or `\[equation\]` for display.
- If numbering doesn't update, delete `.aux` files and recompile.

Refer to the main README.md for compilation instructions.