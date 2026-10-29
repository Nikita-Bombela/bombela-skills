# bombela-skills

Personal Claude Code skills, packaged as a plugin + its own marketplace.

## Install

```
claude plugin marketplace add Nikita-Bombela/bombela-skills
claude plugin install bombela@bombela-skills
```

Skills show up as `/bombela:<name>` in the next session.

## Update

```
claude plugin marketplace update bombela-skills
claude plugin update bombela@bombela-skills
```

## Local development

```
claude --plugin-dir /path/to/bombela-skills
```

Loads the checkout directly for one session, so edits apply on the next launch without reinstalling.
