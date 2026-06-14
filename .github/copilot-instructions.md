# GitHub Copilot Instructions

This repository maintains a Docker image and runtime scripts for Redmine and
RedMica. Prefer small, compatibility-focused changes because these scripts run
during container startup, backup, restore, plugin installation, and database
migration.

## General Guidance

- Use English for code comments, PR titles, PR descriptions, and commit
  messages.
- Keep changes backward compatible unless the PR explicitly states otherwise.
- Avoid AI-related wording in generated content.
- Prefer clear operational error messages. Startup and restore failures should
  explain what version, flavor, adapter, or configuration caused the problem.
- Do not introduce new dependencies unless they are necessary for container
  startup or runtime behavior.

## Bash Runtime Scripts

The main runtime logic lives in `assets/runtime/functions`.

- The script runs with `set -e`; review command substitutions and assignments
  carefully. A failing command in `VAR=$(...)` can terminate the script before
  later error handling runs.
- When calling a helper that can return non-zero, use an explicit guard such as
  `if ! value=$(helper ...); then ...; return 1; fi` or an existing wrapper
  helper.
- Quote variable expansions in conditionals and command arguments unless word
  splitting is intentional.
- Quote arguments passed to version comparison helpers such as `vercmp`.
- Prefer `[[ ... ]]` for Bash conditionals.
- Keep cache files under `${REDMINE_DATA_DIR}/tmp` backward compatible with
  older images.
- Do not replace user data, plugins, themes, attachments, or backups without an
  explicit backup or migration path.

## Redmine and RedMica Version Handling

RedMica has its own public version numbers but maps to compatible Redmine
versions. Upgrade, downgrade, backup, restore, and migration checks should use
the compatible Redmine version when comparing across Redmine and RedMica.

- Preserve direct version comparison for same-flavor downgrade checks where
  appropriate.
- Treat older cache or backup metadata that does not include a flavor as
  `redmine`.
- Unsupported Redmine or RedMica versions should fail with a clear message,
  rather than exiting silently because of `set -e`.
- When adding a new RedMica version, update the compatibility mapping and any
  related validation paths together.

## Backup and Restore Behavior

- Backup metadata should remain readable by future versions.
- Restore validation should fail before mutating data when version or database
  adapter checks do not pass.
- Keep database adapter checks explicit.
- When adding metadata, provide a fallback for backups created by older images.

## Plugin and Bundler Behavior

- Plugin installation runs during startup and may migrate plugin databases.
- Avoid changes that force unnecessary bundle installs or plugin migrations.
- Prefer current Bundler configuration commands, but keep runtime behavior
  compatible with the image's supported Bundler version.

## PR Review Focus

When reviewing PRs, prioritize:

- Container startup regressions.
- Backup and restore safety.
- Redmine/RedMica upgrade and downgrade validation.
- `set -e` behavior in Bash.
- Quoting and word-splitting bugs.
- Backward compatibility with existing cache and backup metadata.
- Clear user-facing failure messages.

## Commit Message Rules

When generating git commit messages:

- Use Conventional Commits.
- Subject must be in English.
- Maximum 72 characters.
- Format:

<type>(<scope>): <summary>

Examples:

feat(rke2): add sakuracloud machine template
fix(ipam): prevent duplicate address allocation
docs(readme): update installation guide

Rules:

- Do not use emojis.
- Do not mention AI-generated content.
- Focus on user-visible changes.
- Prefer imperative mood.
