# Security Roslyn Analyzers (Planned)

Custom Roslyn analyzers can enforce security rules at compile time. Below are the planned rules and implementation guidance.

## Planned Analyzer Rules

| Rule ID | Description | Detection |
|---------|-------------|-----------|
| ALFQ001 | POST endpoints must have idempotency check or unique constraint handling | Controller action with [HttpPost] without [RequiresIdempotency] or DbUpdateException handling |
| ALFQ002 | PUT/PATCH on entities must use optimistic concurrency | Entity without RowVersion used in update operations |
| ALFQ003 | Token-based operations must check one-time use | RefreshToken/ResetCode flow without IsUsed/ResetCodeUsed check |
| ALFQ004 | Financial operations must use transactions | Methods containing "Payment" or "Charge" without transaction |
| ALFQ005 | Unique constraints must exist for natural keys | Entity with email/code/username without [Index(IsUnique=true)] |

## Implementation Approach

To implement these analyzers:

1. Create a new class library project targeting `netstandard2.0`
2. Add packages: `Microsoft.CodeAnalysis.CSharp` (4.x), `Microsoft.CodeAnalysis.Analyzers`
3. Create analyzer class inheriting `DiagnosticAnalyzer`
4. Register in `Initialize()` with `context.RegisterSyntaxNodeAction()`
5. Create DiagnosticDescriptor for each rule
6. Add project reference to Alforqan.Api with `OutputItemType="Analyzer"`
7. Add to `.editorconfig` to configure severity

## Alternative: EditorConfig + Code Review

Until analyzers are implemented, use:

- **Security checklist** (`security-checklist.md`) for code reviews
- **Cursor rules** (`.cursor/rules/security/`) for AI-assisted compliance
- **Pre-commit hooks** to run security checklist
- **CI pipeline** to fail on TODO/FIXME in security-sensitive files

## Integration with CI/CD

When analyzers are built:

```yaml
# Example GitHub Actions
- name: Run Security Analyzers
  run: dotnet build --no-restore /p:TreatWarningsAsErrors=true
```

Ensure `TreatWarningsAsErrors` includes analyzer diagnostics for ALFQ* rules.
