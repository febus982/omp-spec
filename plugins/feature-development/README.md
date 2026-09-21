# Feature Development

Portable OMP workflow for feature work from brainstorming through OpenSpec planning, OMP implementation, verification, and archival.

The plugin includes:

- `skills/feature-development/SKILL.md`;
- `agents/implementation-planner.md`.

Requirements:

- OpenSpec CLI available on `PATH`;
- each target repository initialized with `openspec init --tools oh-my-pi`;
- OMP `@plan` model role configured;
- project-specific planning, repository, and verification policy supplied by the consuming repository's context.

The skill drives the OpenSpec CLI directly and does not require a particular generated OpenSpec workflow profile. The default core profile is sufficient; core plus `verify` is recommended for convenient manual commands.

See the marketplace repository README for installation, configuration, usage, upgrades, and local development.

## License

MIT. See `LICENSE` in this plugin package.
