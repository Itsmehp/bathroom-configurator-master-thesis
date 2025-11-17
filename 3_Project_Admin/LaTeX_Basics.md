# A Beginner's Guide to LaTeX for Your Thesis on Overleaf

This guide will help you take the content you've written in Markdown and format it in LaTeX. We'll start with a simple, practical example: creating the Executive Summary page.

## 1. Understanding the Basic Structure of a LaTeX File

A LaTeX document has two main parts:

1.  **The Preamble:** This is everything at the very top of the file, before the text `\begin{document}`. Here, you define the type of document you're creating, load packages for extra features (like images or special symbols), and set up overall document properties.
2.  **The Document Body:** This is everything between `\begin{document}` and `\end{document}`. It's where your actual content—text, chapters, and sections—goes.

Commands in LaTeX almost always start with a backslash (`\`).

---

## 2. Creating the Executive Summary Page

Let's create a standalone page for your Executive Summary. This is a complete, copy-and-paste example.

```latex
% --- PREAMBLE ---
% This command defines the overall document type. 'article' is simple and good for starting.
% 12pt sets the default font size. 'a4paper' sets the paper size.
\documentclass[12pt, a4paper]{article}

% This package is needed to set custom margins.
\usepackage[left=3.0cm, right=2.0cm, top=3.0cm, bottom=3.0cm]{geometry}

% --- DOCUMENT BODY ---
\begin{document}

% The \section* command creates a new section. 
% The asterisk (*) means it will NOT be numbered or appear in the Table of Contents.
\section*{Executive Summary}

% You can write your summary text here. 
% LaTeX automatically handles paragraph breaks when you leave a blank line.
The executive summary provides a short and concise overview of the master thesis. After reading the executive summary, the core substance and results should become evident.

It normally is about 1 page in length and contains three main parts: a short problem statement, a brief description of the overall structure and procedure, and a condensed summary of the findings of the study.

% This command tells LaTeX that the document is finished.
\end{document}
```

### Key Commands Explained:

*   `\documentclass[12pt, a4paper]{article}`: This sets up the document. For a thesis, you might eventually use `{report}` or `{book}` instead of `{article}` as they have better support for chapters, but `article` is perfect for getting started.
*   `\usepackage{...}`: This command loads extra functionality. The `geometry` package is one of the most useful for setting margins.
*   `\begin{document}`: Marks the beginning of your content.
*   `\section*{Executive Summary}`: Creates a big, bold heading for your section. As mentioned, the `*` makes it an unnumbered section. If you wanted a numbered "Chapter 1: Introduction", you would write `\section{Introduction}`.
*   `\end{document}`: Marks the end of the document.

---

## 3. How to Use This in Overleaf

Overleaf is an online LaTeX editor that makes this process much easier.

1.  **Create a Project:** Log in to Overleaf and click **"New Project"**, then select **"Blank Project"**. Give it a name like "My Thesis".
2.  **Create a New File (Optional):** Overleaf starts you with a `main.tex` file. You can either delete the content in that file or create a new one by clicking the "New File" icon. Let's say you create `executive_summary.tex`.
3.  **Copy and Paste:** Copy the complete LaTeX code block from step 2 above and paste it into the file on the left-hand side of the Overleaf editor.
4.  **Recompile:** Click the green **"Recompile"** button (or use the shortcut `Ctrl+Enter` / `Cmd+Enter`).

On the right-hand side of the screen, you will see a PDF preview of your formatted Executive Summary page.

You can now edit the text in the left-hand pane, click "Recompile", and see your changes reflected in the PDF on the right. This immediate feedback loop is what makes Overleaf so powerful for learning.
