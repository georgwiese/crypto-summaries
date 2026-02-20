# Quartz ZK Notes

Personal knowledge base for zero-knowledge cryptography, as an Obsidian vault.

## Structure

- `Books/` - Book notes and summaries
- `Concepts/` - Core concepts organized by category (PCS, protocols, folding schemes)
- `Papers/` - Paper summaries and notes

## Content Guidelines

- Notes use Obsidian-flavored markdown with `[[wikilinks]]`
- Math uses LaTeX: `$inline$` and `$$block$$`. Block math must have newlines after the opening `$$` and before the closing `$$`
- Keep notes atomic and interlinked
- DON'T DUPLICATE, link to existing notes whenever possible or introduce new notes for new concepts
- When creating a new page, search existing pages for unlinked mentions of the new topic and add `[[wikilinks]]` to the first occurrence per page
- Before doing ANY task, read the ENTIRE vault to have context
- Continue the existing style, formatting, and reuse notation as much as possible

## Research behavior

- When summarizing papers, ALWAYS read the primary source first. If you can't access it, say so immediately and ask for a local copy.
- Don't build explanations from secondary sources (blogs, summaries) without explicitly stating that's what you're doing.
- Default to concise explanations. Expand only when asked.

## Summarizing papers
You'll be asked to read relevant papers. Introduce a markdown file in `Papers/`, but if you need to explain a new concept that might be relevant for any other paper, extract it to `Concepts/` and link to it from the paper notes.

When introducing concepts, include a "source" that points back to the paper or book where you found it.

When reading papers, make sure you fully understand them. Also, do web searches to find additional and related content online. Before starting a summary, ask any questions you have about the paper or related concepts.

A paper summary should always link to the PDF on the web (e.g. ePub link) and mention the authors and year.

If the paper has annotations, focus the summary on highlighted sections. If a section is annotated as "skipped", you can give a high-level summary, but don't go into detail.

The creation process should be iterative:
- Start with a MINIMAL summary that I can read in one minute.
- I'll read it and ask questions or ask you to expand.

When there is a conversation where you want to answer with a lot of math content, write your answer to a new markdown file called `Temp.md` in the root of the vault, and tell me to read it. This way I can see the formatting properly.

## Deploy

When asked to deploy, 
- Do `git status` and `git diff` to make sure there are no unintended changes. Especially, remove any temporary files like `Temp.md`.
- run `npx quartz sync` from `..`. This adds any non-ignored files, commits and pushes, which triggers the deployment.