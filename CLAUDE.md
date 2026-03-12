# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GitHub Action that registers an Amazon ECS task definition and deploys it to ECS scheduled tasks (CloudWatch Events rules). It's a fork of the official `amazon-ecs-deploy-task-definition` action, adapted for scheduled tasks instead of ECS services.

## Commands

```bash
npm test                # Run Jest test suite
npm run test:debug      # Run tests sequentially (--runInBand)
npm run lint            # Run prettier:check + eslint
npm run eslint:fix      # Auto-fix lint issues
npm run prettier:fix    # Auto-format code
npm run package         # Bundle with ncc: ncc build src/index.js -o dist
```

## Architecture

### Entry Point & Flow

`src/index.js` is the main GitHub Action entry point (bundled to `dist/index.js` via `@vercel/ncc`). The `run()` function:

1. Initializes AWS SDK clients (ECS, CloudWatchEvents)
2. Reads/parses the task definition file (JSON or YAML)
3. Cleans null/empty values (`cleanNullKeys()`) and removes AWS-returned attributes that can't be re-registered (`removeIgnoredAttributes()`)
4. Registers the task definition via `ECS.registerTaskDefinition()` and sets the `task-definition-arn` output
5. Lists all CloudWatch Event rules, filters by `rule-prefix`, then for each matching rule calls `processCloudwatchEventRule()` to update targets with the new task def ARN

### Utility Module

`src/ecs-cwe.js` provides two filtering functions used in step 5:
- `filterNonEcsClusterTargets(targets, clusterName)` — keeps only targets in the specified ECS cluster (parses cluster name from ARN)
- `filterUnrelatedTaskDefTargets(targets, newTaskDefArn)` — keeps only targets whose task definition family matches the new one (version-agnostic via `simplifyTaskDefinitionArn()`)

### Action Inputs/Outputs

Defined in `action.yml`:
- **Inputs:** `task-definition` (required, file path), `cluster` (default: `'default'`), `rule-prefix` (default: `''`)
- **Output:** `task-definition-arn`

## Code Conventions

- ESLint enforces: ES6, no `console.log` (only warn/error), max 5 function parameters, sorted object keys
- Conventional commits enforced via commitlint + husky
- All PRs should pass `npm run lint` and `npm test` before merging
- After changing `src/`, run `npm run package` to rebuild `dist/index.js`
