# Alforqan - Test Cases

Copyright (c) 2025–2026 Omar Al-Imam Al-Maghawry Al-Imam. All rights reserved. See COPYRIGHT.txt.

Manual test definitions for **alforqan**. Run `tc` from this repo root (see Testing-feature `docs/project-onboarding.md`).

## Apply latest Testing-feature (rules + toolchain)

**1. Rebuild the vendor CLI** (when `tc` / `tc ui` behavior changed):

```bash
cd "O:/Company work/Company projects/ExcpTime/Testing-feature"
npm run link:dev
```

**2. Sync cursor rules** to this TestCases repo, linked Frontend/Backend, and the monorepo workspace:

```bash
cd "O:/My work/Alforqan website/TestCases"
tc project sync
# after template updates:
tc project sync --force
tc project doctor --strict
```

**3. Browse and run manual tests:**

```bash
tc ui
# optional: tc ui --open
```

`tc project install-rules` is an alias for `tc project sync --testcases-only` (TestCases rules only).

## Single-tenant mode

Specs under `test-cases/single-tenant-mode/` cover deploying Alforqan as one institute with host UI/API hidden. See `docs/single-tenant-strategy.md` in the app repo.

## Halaqat run order (`Requires`)

The halaqat frontend specs under `test-cases/halaqat/frontend/` are the reference for optional **`Requires`** keys. Recommended manual order: **sessions** (`ALFQ-HAL-SES-*`) → **attendance** → **evaluation** → **reports** / **notification deep link** (`ALFQ-HAL-NOT-004`). **Grading** has no session dependencies. Use `tc next <file>` from this repo root; the CLI skips cases whose prerequisites are not yet `succeeded`.
