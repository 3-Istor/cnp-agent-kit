# CNP doctrine

The rules an app must follow to run well on CNP. One source for the guides
(skills), the platform checks and code review.

Format of a rule:

```
### CNP-XX · Title
**MUST|SHOULD|NEVER** the rule.
Why: what goes wrong without it.
Check: the tool, CI job or review that verifies it.
```

- A rule says why. A rule nobody can check is a wish: it gets a check or is
  removed.
- Adding, changing or retiring a rule is a pull request with a line in
  `CHANGELOG.md`.
- Guides link to rules by ID instead of copying them.
