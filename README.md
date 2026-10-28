# bombela-skills

Personal Claude Code skills, packaged as a plugin + its own marketplace.

Add a skill: `skills/<name>/SKILL.md` with `name` and `description` frontmatter.

Try without installing: `claude --plugin-dir ~/Projects/bombela-skills`

Install:
```
/plugin marketplace add ~/Projects/bombela-skills
/plugin install bombela@bombela-skills
```
After edits: `/plugin marketplace update bombela-skills`.
