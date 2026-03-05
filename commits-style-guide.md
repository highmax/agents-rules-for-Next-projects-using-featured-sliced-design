---
trigger: always_on
---

Generate Commit Guidelines

- The commit contains the following structural elements, to communicate intent to the consumers of your library:
- fix: a commit of the type `fix` patches a bug in your codebase (this correlates with PATCH in semantic versioning).
- feat: a commit of the type `feat` introduces a new feature to the codebase (this correlates with MINOR in semantic versioning).
- Others: commit types other than `fix:` and `feat:` are allowed, for example `chore:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, and others.
- A scope may be provided to a commit’s type, to provide additional contextual information and is contained within parenthesis, e.g., `feat(parser): add ability to parse arrays`.
- Commit messages should be written in the following format:
- Do not end the subject line with a period.
- Use the imperative mood in the subject line.
- Use the body to explain what and why you have done something. In most cases, you can leave out details about how a change has been made.
- The commit message should be structured as follows: `<type>[optional scope]: <description>`
