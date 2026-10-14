# CodeForge

An enterprise code generation and review agent — writes new custom software from a plain-language spec, or reviews existing code for real bugs, security vulnerabilities, and performance issues, then produces a corrected version. Built with the [Claude API](https://docs.claude.com).

## Live demo

**[Try it here](https://claude.ai/artifact/PoSnXUMsyyphygEc5z1ffV)** — note: trying the interactive part requires a free Claude account to sign in with (a platform requirement for any page that calls Claude live, not specific to this project).

## The problem

Two of the most common software requests in an enterprise setting are "build us something custom for this workflow" and "this existing code has problems, can someone look at it." Both normally require a developer's time just to produce a first pass. This project automates a genuinely useful first pass at both — not to replace engineering judgment, but to get a working starting point or a real, specific list of problems much faster than starting from a blank page or a cold read-through.

## What it does

**Build New** — describe a requirement in plain language and choose an output type:
- **Interactive web widget** — generates a complete, self-contained HTML/CSS/JS widget and renders it **live, immediately, in a sandboxed preview** right in the page. It's genuinely functional, not a mockup — an invoice calculator example actually adds/removes line items and computes totals in real time.
- **Code snippet** — for any other language (Python, JavaScript, Java, C#, Go, SQL, TypeScript), writes idiomatic, commented code to the spec.

**Review & Improve** — paste existing code (optionally with context about where it runs), and the agent:
1. Finds real, specific issues — bugs, security vulnerabilities (e.g. injection risks), performance problems, style violations — each tagged with severity and category, not generic nitpicks. It's explicitly instructed to report no issues if the code is genuinely solid, rather than inventing minor complaints to seem thorough.
2. Produces a corrected, improved version of the code
3. Gives a plain-language changelog of what actually changed and why

## A note on the live preview's safety

The generated widget preview runs inside an `<iframe sandbox="allow-scripts">` with no `allow-same-origin` — the standard, deliberate pattern used by tools like CodePen and JSFiddle. This means the generated code can run and render itself, but it's isolated from the parent page and can't access anything outside its own sandboxed context. This was a deliberate security choice, not an afterthought: letting AI-generated code execute live is only safe to offer if it's genuinely contained.

## Example: what the reviewer actually catches

Given this code —
```python
def get_user(username):
    query = "SELECT * FROM users WHERE username = '" + username + "'"
    cursor.execute(query)
    return cursor.fetchone()
```
— the reviewer correctly flags a critical SQL injection vulnerability (string-concatenated user input directly into a query) and rewrites it using parameterized queries, rather than just commenting on style.

## How it works

- One Claude API call per generation or review, using structured JSON output
- For widget generation, the model's HTML output is injected directly into a sandboxed iframe's `srcdoc` — no external dependencies allowed in the generated code, keeping it genuinely self-contained
- For review, issues are sorted client-side by severity (critical → low) before display, not left in arbitrary model output order

## Tech

- HTML / CSS / JavaScript (single file, no build step)
- Claude API, via Anthropic's Artifact runtime
- Sandboxed `<iframe>` execution for live-previewable generated code
- Structured JSON output for both generation and review modes
