---
alwaysApply: true
---

Backstage is an open platform for building developer portals. This is a TypeScript monorepo using Yarn workspaces.

## Key Directories

- `/packages`: Core framework packages (prefixed `@backstage/`)
- `/plugins`: Plugin packages (prefixed `@backstage/plugin-*`)
- `/packages/app`: Main example app using the new frontend system
- `/packages/app-legacy`: Example app using the old frontend system
- `/packages/backend`: Example backend for local development
- `/docs`: Documentation files

Packages prefixed with `core-` (e.g., `@backstage/core-plugin-api`) are part of the old frontend system. Packages prefixed with `frontend-` (e.g., `@backstage/frontend-plugin-api`) are part of the new frontend system. Packages prefixed with `backend-` (e.g., `@backstage/backend-plugin-api`) are part of the backend system.

## Writing Standards

Changes to the docs should follow the documentation style guide at `/docs/contribute/doc-style-guide.md`.

## Code Standards

The following files contain guidelines for the project:

- `/CONTRIBUTING.md`: comprehensive contribution guidelines.
- `/STYLE.md`: guidelines for code style.
- `/REVIEWING.md`: guidelines for pull requests and writing changesets.
- `/SECURITY.md`: guidelines for security.
- `/docs/architecture-decisions/`: contains the architecture decisions for the project.

When writing or generating code, always match the existing coding style of each individual package and file. Different packages in the monorepo may have different conventions — consistency within a package is more important than consistency across the repo.

When writing or generating tests, prefer fewer thorough tests with multiple assertions over many small tests. When using React Testing Library, prefer using `screen` and `.findBy*` queries over `waitFor`, and avoid adding test IDs to the implementation.

## Development Flow

Before any of these commands can be run, you need to run `yarn install` in the project root.

- Build: There is no need to build the project during development, and it is verified automatically in the CI pipeline.
- Test: Use `CI=1 yarn test <path>` in the project root to run tests. The path can be either a single file or a directory. Always provide a path, avoid running all tests.
- Type checking: Use `yarn tsc` in the project root to run the type checker. Do not try to run it somewhere else than the project root and do not supply any options.
- Code formatting: Use `yarn prettier --write <...paths>` to format code. Run it explicitly for file paths that you know are changed, not for entire folders - otherwise it may change formatting of unrelated files.
- Lint: Use `yarn lint --fix` in the project root to run the linter.
- API reports: Before submitting a pull request with changes to any package in the workspace, run `yarn build:api-reports` in the project root to generate API reports for all packages.
- Dev server: Use `yarn start` to run the example app locally (frontend on :3000, backend on :7007).
- Create: Use `yarn new` to scaffold new plugins, packages, or modules.

You MUST NOT run builds or create a release by running `yarn build`, `yarn changesets version`, or `yarn release` as part of any changes. Builds and releases are made by separate workflows.

All changes that affect the published version of packages in the `/packages` and `/plugins` directories must be accompanied by a changeset. Only non-private packages require changesets. See the guidelines in `/CONTRIBUTING.md#creating-changesets` for information on how to write good changesets. Changesets are stored in the `/.changeset` directory and should be created by writing changeset files directly — never use the changeset CLI. Breaking changes must be accompanied by a `minor` version bump for packages below version `1.0.0`, or a `major` version bump for packages at version `1.0.0` or higher. For non-breaking changes that introduce new APIs or features, use `minor` for packages at version `1.0.0` or higher, and `patch` for packages below `1.0.0`. Each changeset message should be relevant to the specific package it targets and written for Backstage adopters as the audience — describe user-facing behavior changes in plain language. Never reference internal implementation details such as function names, class names, variable names, or other code symbols that are not part of the public API. If a change spans multiple packages you often need to create separate changesets to make sure they are tailored to each package.

When creating pull requests, use the template at `/.github/PULL_REQUEST_TEMPLATE.md`.

Never update ESLint, Prettier, or TypeScript configuration files unless specifically requested.

Never make changes to the release notes in `/docs/releases` unless explicitly asked. These document past releases and should not be updated based on newer changes.

## Repository Structure

See `/docs/contribute/project-structure.md` for a detailed description of the repository structure.

## Cursor Cloud specific instructions

The environment already has Node (`22 || 24`) and Yarn `4.8.1` (via Corepack) available, and the startup update script runs `yarn install`. Standard commands are documented above in "Development Flow"; the notes below are non-obvious caveats only.

- Running the app: `yarn start` launches both the frontend (`:3000`) and backend (`:7007`) together in a single process — there is no separate backend start step (the root `yarn start-backend` is a no-op that just prints a hint). After it boots, look for `Rspack compiled successfully` and `Listening on :7007` in the logs.
- No external services needed by default: the example app uses an in-memory `better-sqlite3` database and the in-memory Lunr search engine, so Postgres/Redis/OpenSearch/Docker are NOT required to run or test locally. The optional `yarn start:docker` variant wires those up via `docker-compose.deps.yml`.
- Auth: sign in with the built-in **Guest** provider (click "Enter" on the sign-in page) — no credentials or external OAuth setup is required for local development.
- Expected benign startup warnings (not environment problems): catalog warnings for the example `petstore`/`petstore-webhook` placeholders and the `hello-world` API schema, and a `search` error for `explore tools 404` (the explore plugin backend isn't wired into the example backend). The app is fully functional despite these.
- TechDocs (optional): local doc generation needs a Python virtualenv with `mkdocs-techdocs-core` (see `.devcontainer/setup.sh`). This is intentionally NOT part of the update script; the example app runs fine without it. On Ubuntu it also requires the `python3-venv` system package, which is a one-off install.
- Backend API requests require auth — an unauthenticated `curl` to a `:7007` API returns `401`, which indicates the backend is up, not an error.
