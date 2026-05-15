# rust-study Agent Guide

## Scope

This file governs the entire repository.

## Audience

- The learner is a Rust beginner.
- The learner has seen JavaScript, Python, and Java before, but is not deeply experienced.
- Explanations should assume curiosity, not prior Rust knowledge.

## Primary Goal

Build a Markdown-first Rust learning repository that helps the learner move from beginner to mastery with a clear path:

1. Getting started
2. Basic Rust
3. Advanced Rust
4. Mastery

## Output Rules

- Use Simplified Chinese for prose unless the user explicitly requests English.
- Keep code, commands, compiler errors, and library names in their original language.
- Prefer Markdown teaching material over large amounts of code.
- When code is needed, keep examples minimal, runnable, and focused on one concept.

## Teaching Style

- Explain concepts from first principles before introducing jargon.
- For difficult Rust topics, compare them to JavaScript, Python, or Java when that genuinely reduces confusion.
- Do not assume the learner understands ownership, borrowing, lifetimes, traits, iterators, or async.
- When the compiler would likely confuse a beginner, explain what the error usually means in plain language.

## Required Structure For Teaching Docs

Each substantive learning document should usually include:

1. What this chapter is for
2. What you need to know first
3. Core concepts
4. Minimal code examples
5. Comparison with JS, Python, or Java when useful
6. Common mistakes and how to debug them
7. Small exercises
8. One challenge task
9. A short recap

## Repository Conventions

- Keep the learning path ordered and easy to navigate.
- Prefer adding new Markdown files rather than overloading one huge file.
- Prefer Chinese filenames and folder names for learning materials when practical.
- Keep `AGENTS.md` as-is because the filename has a special control purpose.
- Keep numbering stable once materials are linked from the main navigation.
- If runnable Rust examples are added later, include the command needed to run them.

## Content Priorities

- Prioritize understanding over breadth.
- Teach the mental model behind ownership and borrowing early and repeatedly.
- Emphasize practical Rust: Cargo, modules, error handling, collections, traits, iterators, testing, concurrency, async, and performance thinking.
- Include challenge exercises regularly, but keep the main line focused on explanation.

## Update Discipline

- Do not rewrite the whole structure when extending the course; append or refine in place.
- Preserve beginner readability.
- If a new topic is advanced, clearly label it as advanced and explain why it matters.
