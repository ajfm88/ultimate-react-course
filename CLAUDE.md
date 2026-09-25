# Ultimate React Course — Notes Workflow

This repo holds code + notes for Jonas Schmedtmann's "The Ultimate React Course." Each numbered folder (`01-pizza-menu`, `02-steps`, ... `06-how-react-works`, `07-usepopcorn-v2`, ...) is one project/section, and most have a `README.md` (or `readme.md`) with lecture notes.

## How we take notes together

The user watches a lecture video and feeds me the content as they go — usually a pasted transcript (the raw spoken transcription of the video) plus one or more screenshots of the slide(s) being discussed. My job is to turn that into a well-organized note appended to that section's README.

**Per message, the workflow is:**

1. Read the transcript + slide image(s) the user sends.
2. Append a new note to the **current section's README** (find the right numbered folder — the user will say if it changed, e.g. "let's continue in folder 07").
3. Keep going section by section, appending as new material comes in — don't rewrite or reorganize earlier notes unless asked.
4. Don't wait for a full lecture before writing — each slide/transcript chunk gets its own note as it arrives, added at the end of the file.

## Style rules for the notes themselves

- **Succinct but rich** — dense with signal, not a transcript dump. Compress the spoken narration into tight bullets; don't narrate "the instructor says...".
- Preserve the slide's own structure and emoji/labels where useful (👉, 🔑, ✅, 🚨, etc.) — the course slides use these consistently and they aid scanning.
- Use `##` for a new lecture/topic, `###` for subsections within it.
- Include code examples from the slides verbatim in fenced code blocks when they're central to the point.
- Bold the key terms and rules, especially anything marked as a rule, warning, or takeaway on the slide.
- Tables are good for comparisons (e.g. event handlers vs. effects, framework vs. library).
- End a section with a `> 🔑` or `> 👉` blockquote takeaway when the slide itself frames one — don't invent one if the source didn't.
- No meta-commentary in the notes ("in this lecture we learned...") — just the content itself, written as reference material for future review.

## Mechanics

- Append via `Edit` (targeting the last line of the file) or a heredoc `Bash` append (`cat >> file <<'EOF' ... EOF`) — whichever is cleaner for the given chunk. Never rewrite the whole file unless fixing something specific.
- If a section's README doesn't exist yet, create it with a top-level `#` title for the section before adding the first note.
- If the user says "don't add this to the notes" (e.g. a tangent, a personal question), just answer normally and skip the README — never write it in.
- If the user pastes something as a correction/addition to a note just written (e.g. "did u put that part? could u also add that part of the code?"), fix the specific section in place rather than appending a duplicate.
