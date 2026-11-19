# Skills Collection: Scripting & Antiscrap

A repository containing two skill packages structured as plugins, each with its manifest and skill instructions in `SKILL.md`.

## Repository Structure

To prevent conflicts between each package's `plugin.json` file, both are placed in separate top-level directories:

```text
.
├── README.md
├── scripting/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── scripting/
│           └── SKILL.md
└── antiscrap/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── antiscrap/
            └── SKILL.md