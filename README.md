# Forqan documentation & QA specs

Central GitHub home for Alforqan product documentation and manual test cases.

| Path | Source on disk |
|------|----------------|
| [`docs/`](docs/) | `Alforqan-Backend/docs/` |
| [`test-cases/`](test-cases/) | `TestCases/test-cases/` (see also `test-cases/QA-README.md`) |

## Sync from local workspaces

After editing the canonical folders locally, refresh this repo and push:

```powershell
$root = "D:\Work\My Work\LMS\Forqan\Forqan-docs"
robocopy "D:\Work\My Work\LMS\Forqan\Alforqan-Backend\docs" "$root\docs" /E /MIR
robocopy "D:\Work\My Work\LMS\Forqan\TestCases\test-cases" "$root\test-cases" /E /MIR /XF QA-README.md
Copy-Item "D:\Work\My Work\LMS\Forqan\TestCases\README.md" "$root\test-cases\QA-README.md" -Force
Set-Location $root
git add -A
git status
git commit -m "Sync docs and test cases from local workspaces"
git push
```

Manual QA tooling (`tc`, test run status, `.testing-feature`) stays in the local `TestCases` repo only — not published here.

## Remote

https://github.com/omarelemam49141/Forqan-docs
