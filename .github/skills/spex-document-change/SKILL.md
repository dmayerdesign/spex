---
name: spex-document-change
description: "Document the change you just made in a changelog file"
user-invokable: false
---

1. Create a new markdown file in the `spec/changelog/` directory. Name the file using the format `YYYYMMDD-descriptive-title.md`, where `YYYYMMDD` is the current date and `descriptive-title` is a brief description of the change (use hyphens to separate words).
2. In the new markdown file, document the change you made. Be very concise and only include need-to-know information:

- The requirement or problem that prompted the change
- The root cause of the problem (if applicable)
- The "when/then" scenario that was failing before the change
- Brief rundown of any spec.md, design.md, or test files that were updated to align with the change
