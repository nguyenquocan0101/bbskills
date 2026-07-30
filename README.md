# bbskills

Reusable agent workflows bundled as one portable skill collection.

Included workflows:

- `brainstorm`, `spec`, `plan`, `cook`, `fix`, `cicd`
- `frontend-mindset`, `design-taste-frontend`, `minimalist-ui`
- `problem-solving`, `skill-creator`

The entry point is [`SKILL.md`](SKILL.md). Individual workflows are under [`skills/`](skills/).

## Use from Git

```bash
git clone https://github.com/nguyenquocan0101/bbskills.git
```

Then point your agent host at the root `SKILL.md`, or load the individual workflow from
`skills/{name}/SKILL.md`.

## Use as an npm package

The repository includes package metadata so it can later be published as `bbskills` or scoped
under an npm account. The package contains the root dispatcher and all bundled workflows.
