# Skills Collection: Scripting & Antiscrap

A repository containing two skill packages structured as plugins, each with its manifest and skill instructions in `SKILL.md`.

## Workflow & Complementary Tools

These skills are used alongside the following plugins and tools:

* **[Chrome Extension Builder Plugin](https://github.com/khadinakbarlabs/chrome-extension-builder-plugin):** Used for building and structuring Chrome extensions.
* **[ReviewBot](https://github.com/hntw/reviewbot):** Used for automated code review and quality checks.

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
```

## Included Skills

### 1. Scripting (`scripting`)
* **Source:** `scripting-v1.zip`
* **Key Files:** `.claude-plugin/plugin.json` and `skills/scripting/SKILL.md`
* **Description:** [Add a brief summary of what the scripting skill does, when to use it, and any key guidelines]

### 2. Antiscrap (`antiscrap`)
* **Source:** `Antiscrap-v1.zip`
* **Key Files:** `.claude-plugin/plugin.json` and `skills/antiscrap/SKILL.md`
* **Description:** [Add a brief summary of what the antiscrap skill does, when to use it, and any key guidelines]

## Installation & Usage

1. Clone or download this repository to your machine:
   ```bash
   git clone [https://github.com/bagpotato/Claude-skill-s.git](https://github.com/bagpotato/Claude-skills.git)
   ```
2. Copy the folder you need (`scripting` or `antiscrap`) into your target environment's skills or plugins directory following the instructions in its `SKILL.md`.
3. Keep the `.claude-plugin/plugin.json` and `skills/.../SKILL.md` paths intact.
