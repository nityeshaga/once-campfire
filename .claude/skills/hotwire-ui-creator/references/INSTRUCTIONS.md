# Hotwire Decision-Maker References

This directory contains reference material for making informed decisions about Hotwire usage.

## Current Contents

- **hotwire-aha-moments.md** - Comprehensive first-principles learning notes about Hotwire, covering:
  - The fundamental insights behind Hotwire
  - Turbo Drive, Frames, Streams, and Morphing
  - Decision frameworks for choosing the right tool
  - Stimulus patterns and philosophy
  - Real-world examples and anti-patterns

## Recommended Additions

To enhance this skill, add the official Hotwire documentation:

### Turbo Documentation
- Download or copy from: https://turbo.hotwired.dev/handbook/introduction
- Key sections to include:
  - Turbo Drive guide
  - Turbo Frames guide
  - Turbo Streams reference
  - Turbo Native (optional)

### Stimulus Documentation
- Download or copy from: https://stimulus.hotwired.dev/handbook/introduction
- Key sections to include:
  - Introduction and core concepts
  - Controllers, actions, targets, values
  - Lifecycle callbacks
  - Best practices

### Suggested Structure
```
references/
├── INSTRUCTIONS.md (this file)
├── hotwire-aha-moments.md (included)
├── turbo/
│   ├── handbook.md
│   ├── drive.md
│   ├── frames.md
│   ├── streams.md
│   └── native.md (optional)
└── stimulus/
    ├── handbook.md
    ├── controllers.md
    ├── actions.md
    ├── targets.md
    ├── values.md
    └── lifecycle.md
```

## Additional Resources

The Campfire codebase (available in `.claude/skills/dhh-rails-reviewer/references/`) provides real-world examples of:
- Turbo Frames usage in `app/views/`
- Stimulus controllers in `app/javascript/controllers/`
- Turbo Stream templates in `app/views/*/create.turbo_stream.erb`
- Broadcasting patterns in `app/models/`

## Usage

When making Hotwire decisions, Claude will:
1. Reference the decision frameworks in hotwire-aha-moments.md
2. Look up specific syntax/APIs in the official docs (when added)
3. Find real-world pattern examples in the Campfire codebase
4. Provide reasoned recommendations based on the trade-offs
