# Docsy Versioning Plan

## Goal

Add Docsy version navigation to `zencart-docs` in a way that matches the current repository and deployment model, without restructuring the content tree around release versions.

The key decision is:

- keep `content/dev` and `content/user` as audience-based sections
- version the docs by branch and deployed site
- use Docsy's version menu only as navigation between published doc versions

## Current State

The current repository is already set up as a single Hugo site using Docsy:

- Docsy theme is configured in `config.toml`
- Netlify deploys the built site
- the primary public URL is `https://docs.zen-cart.com`

The config already contains:

```toml
version_menu = "Releases"
```

That means the theme is ready to show a version selector, but the selector will not appear until `[[params.versions]]` entries are added.

## Why This Approach

Docsy's version menu is a link list, not a full version-management system.

It does not:

- automatically map one page to the equivalent page in another version
- manage content divergence inside one repository tree
- provide release-aware content resolution

Because of that, the cleanest model for `zencart-docs` is:

- one docs build per maintained release line
- one URL per published release line
- one shared version menu that links between those published sites

This keeps versioning simple and avoids duplicating `content/dev` and `content/user` into release-specific folders.

## Recommended Version Model

Use release branches and separate published doc sites.

Recommended branches:

- `main` or `current` for the current stable release
- `release/2.2`
- `release/2.1`
- optional `dev` for unreleased documentation

Recommended URLs:

- `https://docs.zen-cart.com/` for current stable
- `https://docs.zen-cart.com/2.2/`
- `https://docs.zen-cart.com/2.1/`
- `https://docs.zen-cart.com/dev/` for unreleased docs if published

## Content Structure Policy

Do not version the docs by creating content trees such as:

```text
content/
  current/
  2.2/
  2.1/
```

Do not repurpose `content/dev` to mean a release version.

Instead:

- keep `content/dev` for developer documentation
- keep `content/user` for user documentation
- let Git branches carry the release-specific differences

This preserves the current information architecture and avoids unnecessary duplication.

## Configuration Changes

Each published branch should define the full version menu in `config.toml`.

Example:

```toml
[params]
version_menu = "Releases"

[[params.versions]]
version = "Current"
url = "https://docs.zen-cart.com/"

[[params.versions]]
version = "2.2"
url = "https://docs.zen-cart.com/2.2/"

[[params.versions]]
version = "2.1"
url = "https://docs.zen-cart.com/2.1/"

[[params.versions]]
version = "Dev"
url = "https://docs.zen-cart.com/dev/"
```

Use absolute URLs. Since each version is a separate Hugo build, relative links in the version selector are the wrong tool.

## Base URL Rules

Each deployed version must build with the correct `baseURL`.

Examples:

- current stable:

```toml
baseURL = "https://docs.zen-cart.com/"
```

- 2.2 branch:

```toml
baseURL = "https://docs.zen-cart.com/2.2/"
```

- 2.1 branch:

```toml
baseURL = "https://docs.zen-cart.com/2.1/"
```

This matters because Hugo-generated links and assets depend on the correct base URL, especially when a site is published under a subpath.

## Netlify Deployment Model

The current repository uses Netlify for deployment.

The simplest production setup is:

- one Netlify site per maintained release line
- each Netlify site tracks one Git branch
- each site publishes to its assigned release URL

Suggested site mapping:

- `zencartdocs-current` -> `main` -> `https://docs.zen-cart.com/`
- `zencartdocs-2-2` -> `release/2.2` -> `https://docs.zen-cart.com/2.2/`
- `zencartdocs-2-1` -> `release/2.1` -> `https://docs.zen-cart.com/2.1/`
- optional `zencartdocs-dev` -> `dev` -> `https://docs.zen-cart.com/dev/`

This is easier to operate than trying to make one Netlify site publish multiple branches into multiple subdirectories.

## netlify.toml Considerations

The current build command is tied to the root URL.

Example current pattern:

```toml
command = "hugo -b https://docs.zen-cart.com"
```

For each release site, the build command or environment should match the actual published path.

Examples:

- current:

```toml
command = "hugo -b https://docs.zen-cart.com/"
```

- 2.2:

```toml
command = "hugo -b https://docs.zen-cart.com/2.2/"
```

- 2.1:

```toml
command = "hugo -b https://docs.zen-cart.com/2.1/"
```

This can be handled either by per-site Netlify configuration or by branch-specific config.

## Branch Policy

Recommended branch policy:

- `main`: current stable documentation
- `release/x.y`: maintained older release documentation
- `dev`: unreleased features only

Recommended content flow:

- update `main` first when content applies to current stable
- backport to `release/x.y` only if the change is still accurate for that release
- write unreleased feature docs only in `dev`

Avoid trying to keep all branches identical. That defeats the point of release versioning.

## Authoring Rules

Document these rules in `CONTRIBUTING.md`:

- contributors should choose the branch that matches the release they are documenting
- current release work goes to `main`
- maintenance corrections for older supported versions go to the relevant `release/x.y` branch
- unreleased features go to `dev`
- only backport content when it is still correct in that release line

This reduces accidental documentation drift and prevents current docs from being polluted with old-version caveats.

## Version Notices in the UI

Each published branch should display a short version-status notice.

Recommended notices:

- current stable: “You are viewing the current stable documentation.”
- maintained older release: “You are viewing documentation for Zen Cart 2.2. Some content may differ from current.”
- dev: “You are viewing unreleased development documentation.”

This can be implemented with a small branch-specific banner or partial.

The purpose is to reduce confusion for users who land from search engines or old links.

## Legacy Content Policy

`old_content/` should not be treated as the primary versioning mechanism.

Instead:

- keep it unpublished or lightly linked if it is strictly archival
- or move truly useful historical material into explicit archived release sites

The main navigation should focus on maintained documentation, not raw legacy content.

## Rollout Plan

### Phase 1: Minimal Viable Versioning

Implement only:

- `main` at `https://docs.zen-cart.com/`
- `release/2.2` at `https://docs.zen-cart.com/2.2/`
- `[[params.versions]]` entries in both

This gives working version navigation with minimal operational change.

### Phase 2: Older Maintained Release

If still needed, add:

- `release/2.1`
- its corresponding Netlify site
- a `2.1` version entry in all version menus

### Phase 3: Dev Docs

Add `dev` only if there is a clear need to publish unreleased developer or user documentation.

If there is no real audience for public prerelease docs, skip this phase.

### Phase 4: Cleanup and Policy

After versioning is working:

- update `CONTRIBUTING.md`
- add version-status banners
- review `old_content/`
- review hardcoded root-relative links that may behave differently under subpaths

## Local Verification Checklist

Before enabling production deployment for a versioned branch, verify locally:

```bash
hugo server
```

For a versioned subpath build, also test with an explicit base URL:

```bash
hugo server --baseURL http://localhost:1313/2.2/
```

Check the following:

- the Docsy version selector appears
- links resolve correctly under a subpath
- CSS and JS assets load correctly
- search still works as expected
- no hardcoded root links break navigation

## Recommended First Implementation

The best first step is intentionally narrow:

- create `release/2.2`
- add `[[params.versions]]` to `config.toml`
- configure a second Netlify deployment for `/2.2/`
- leave `content/dev` and `content/user` unchanged

That gives the project working version navigation without forcing a broader docs reorganization.

## Summary

For `zencart-docs`, the right model is:

- audience-based content sections
- branch-based versioning
- separate deployments per maintained release line
- Docsy version selector as cross-site navigation

This matches how Docsy actually works and fits the current Netlify-based deployment model with minimal disruption.

## Appendix: `content/dev` Reorganization Plan

The versioning plan above intentionally keeps release management separate from information architecture. Versioning should be branch-based, but the `content/dev` tree still needs a clearer internal structure.

### Problem Statement

The current `content/dev` layout mixes several different organizing principles:

- audience areas
- architectural topics
- implementation how-to material
- release and contribution process docs
- historical and version-specific technical notes

The main structural problem is that `content/dev/code/` is acting as a catch-all bucket. It currently contains core architecture, admin topics, PHP guidance, database how-to material, notifier docs, forms, template topics, and module-related notes.

There are also a few more specific issues:

- `admin/` is too thin relative to the amount of admin-specific material living under `code/`
- `database/` and `schema/` are closely related but separated in a way that is not obvious to readers
- `testframework/` is accurate internally but less discoverable than `testing/`
- `release_process/` is process material, not technical reference material
- `plugins/encapsulated_plugins/` is more verbose than needed
- `code/Modules/` is capitalized inconsistently
- language-file docs appear to be split conceptually between `code/` and `languages/`

### Reorganization Principle

Keep the top-level `dev` docs organized by how developers look for information, not by where the legacy docs happened to accumulate.

The target buckets should answer questions like:

- “I need to understand the application internals.”
- “I need to build or extend an admin feature.”
- “I need database guidance.”
- “I need plugin guidance.”
- “I need PHP or framework-specific conventions.”
- “I need release or contribution process instructions.”

### Proposed Target Structure

```text
content/dev/
  _index.md
  getting-started/
  architecture/
  admin/
  storefront/
  plugins/
    legacy/
    encapsulated/
  database/
    schema-history/
  languages/
    legacy/
  frontend/
  php/
  testing/
  contributing/
  release/
```

This structure keeps audience-based grouping at the `dev` level, and then uses topic-based grouping within developer docs.

### What Each Section Should Mean

#### `getting-started/`

Use this for orientation and local setup.

Likely content:

- developer environment setup
- how to navigate the docs set
- possibly high-level “where to start” pages for new contributors

#### `architecture/`

Use this for core Zen Cart internals and system behavior.

This is where developers should go when they want to understand how the platform works rather than how to do one specific task.

Likely moves from `code/`:

- program flow
- init system
- inclusion rules
- extra folders
- constants
- product types
- notifiers
- notifier report and notifier lists
- observer creation example

#### `admin/`

Use this for admin-specific developer topics.

This should absorb admin material currently buried in `code/`.

Likely moves:

- admin templating
- admin sanitization
- admin cron
- creating menu entries
- sorting menu entries
- reports
- widgets
- future view-builder material if that section grows

#### `storefront/`

Use this for storefront-facing implementation patterns.

Likely candidates:

- forms
- template settings
- displaying custom fields
- other page or rendering guidance that is not core architecture

This bucket may remain smaller than others, but it is still clearer than leaving storefront implementation topics inside `code/`.

#### `plugins/`

Keep plugin material together, but split it into clearer subgroups.

Suggested substructure:

```text
plugins/
  _index.md
  basics/
  governance/
  encapsulated/
  legacy/
```

Suggested use:

- `basics/`: adding config, language files, versions, basic plugin mechanics
- `governance/`: rules, adoption, help, GitHub process
- `encapsulated/`: encapsulated plugin architecture and implementation notes
- `legacy/`: older plugin-specific compatibility or migration material if needed

Also rename:

- `plugins/encapsulated_plugins/` -> `plugins/encapsulated/`

#### `database/`

Unify database and schema guidance under one obvious section.

Suggested substructure:

```text
database/
  _index.md
  querying/
  tables-and-fields/
  schema-history/
  examples/
```

Move here:

- database querying guidance
- creating tables
- add/modify field examples
- child table examples
- order status history update examples
- existing `database/*`
- all `schema/schema_*.md`
- schema renaming guidance

The reader should not need to decide between “database” and “schema” before finding the right docs.

#### `languages/`

Keep language-file material here as the single home for language docs.

Also consider renaming:

- `languages/pre_158/` -> `languages/legacy/`

If there are duplicate or overlapping language docs elsewhere, consolidate them here.

#### `frontend/`

Use this for browser-side libraries and UI support docs.

Likely moves from `libraries/`:

- Bootstrap
- jQuery
- Font Awesome

This name is clearer to most readers than `libraries/`, especially when the content is primarily about frontend assets.

#### `php/`

Use this for PHP-language and coding-style guidance rather than Zen Cart architecture.

Likely moves:

- PHP idioms
- PHP updates
- PHP version deprecations
- data validation if it reads more like language/runtime guidance than app behavior

#### `testing/`

Rename `testframework/` to `testing/`.

That term is easier for occasional contributors to discover.

#### `contributing/`

This can remain, but should be framed explicitly as process and contribution guidance.

#### `release/`

Rename `release_process/` to `release/`.

This is shorter, easier to scan, and better aligned with how readers will search the nav.

### Priority Moves

If the reorganization is done incrementally, the highest-value first moves are:

1. Remove or consolidate duplicate language-topic material.
2. Rename obvious directories:
   - `plugins/encapsulated_plugins/` -> `plugins/encapsulated/`
   - `testframework/` -> `testing/`
   - `release_process/` -> `release/`
   - `code/Modules/` -> `code/modules/` if it remains in place temporarily
3. Pull admin-specific docs out of `code/` into `admin/`.
4. Pull database/table/schema how-to docs out of `code/` into `database/`.
5. Split the remaining `code/` material into `architecture/`, `php/`, and `storefront/`.

### Recommended Migration Strategy

Do this in two passes.

#### Pass 1: Structural Cleanup

Focus only on:

- directory renames
- moving obviously misplaced pages
- removing duplicates
- updating section index pages and internal links

Do not rewrite the actual content heavily during this pass.

#### Pass 2: Section Reframing

After the moves are stable:

- rewrite `_index.md` pages so each section explains when to use it
- add cross-links between architecture, admin, plugins, database, and testing
- identify pages that need retitling or consolidation

This reduces risk and avoids mixing IA work with broad content rewriting.

### Relationship to the Versioning Plan

These two plans should be treated as complementary but separate:

- versioning decides how releases are published
- tree reorganization decides how developers navigate the docs within a release

The recommended order is:

1. get basic versioning working
2. then perform the `content/dev` reorganization

That order reduces operational risk. It avoids combining deployment changes with major content moves in the same step.

### Recommended First Reorganization Step

If only one content-tree cleanup is done immediately, the best first step is:

- split `code/` by moving admin docs into `admin/` and database/schema docs into `database/`

That removes the worst overload without forcing a full content migration all at once.
