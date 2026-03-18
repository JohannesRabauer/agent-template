# Project Guidelines

Workspace-level instructions for AI coding agents in this repository.
Use this file as the single source of truth. Do not add a second global instruction file such as `.github/copilot-instructions.md`.

## Build And Test
- Use Maven Wrapper, not a global Maven install.
- Windows commands:
	- `mvnw.cmd -B clean package`
	- `mvnw.cmd -B checkstyle:check -P checkstyle -T2C`
	- `mvnw.cmd -B test pmd:aggregate-pmd-no-fork pmd:check -P pmd -DskipTests -T2C`
	- `mvnw.cmd -B pmd:aggregate-cpd pmd:cpd-check -P pmd -DskipTests -T2C`
- Linux/macOS commands:
	- `./mvnw -B clean package`
	- `./mvnw -B checkstyle:check -P checkstyle -T2C`
	- `./mvnw -B test pmd:aggregate-pmd-no-fork pmd:check -P pmd -DskipTests -T2C`
	- `./mvnw -B pmd:aggregate-cpd pmd:cpd-check -P pmd -DskipTests -T2C`
- Before finalizing larger changes, run at least `clean package` and the relevant lint profile.
- Mandatory validation tools (extend this table by adding rows):

| Tool | Config File | Windows Validation Command | Linux/macOS Validation Command |
| --- | --- | --- | --- |
| Checkstyle | `.config/checkstyle/checkstyle.xml` | `mvnw.cmd -B checkstyle:check -P checkstyle -T2C` | `./mvnw -B checkstyle:check -P checkstyle -T2C` |
| PMD | `.config/pmd/java/ruleset.xml` | `mvnw.cmd -B test pmd:aggregate-pmd-no-fork pmd:check -P pmd -DskipTests -T2C` | `./mvnw -B test pmd:aggregate-pmd-no-fork pmd:check -P pmd -DskipTests -T2C` |
| CPD | `.config/pmd/java/ruleset.xml` | `mvnw.cmd -B pmd:aggregate-cpd pmd:cpd-check -P pmd -DskipTests -T2C` | `./mvnw -B pmd:aggregate-cpd pmd:cpd-check -P pmd -DskipTests -T2C` |

## Architecture
- Root project (`pom.xml`) is an aggregator (`template-placeholder-root`) with two modules:
	- `template-placeholder`: core Java library artifact.
	- `template-placeholder-demo`: runnable demo JAR depending on the core library.
- Shared quality rules live in `.config/checkstyle` and `.config/pmd`.
- CI workflow in `.github/workflows/check-build.yml` validates:
	- Build on Java 17, 21, and 25.
	- Checkstyle and PMD/CPD on pull requests (except Renovate PRs).

## Code Style
- Keep code readable over clever. Prefer explicit steps and early returns for complex branching.
- Validate input immediately and throw `IllegalArgumentException` when contracts are violated.
- Favor composition over inheritance and keep classes focused on one responsibility.
- Use descriptive names and domain terms. Avoid names ending in `Helper` or `Util`.
- Always close resources with try-with-resources.
- Never swallow exceptions. Preserve stack traces and add context.
- Do not use `System.out.println` or `printStackTrace`; use structured logging.
- Do not use `Optional.get()`; use `orElseThrow`, `orElse`, `or`, `map`, or `ifPresent`.
- Avoid `StringBuilder` and `StringBuffer` unless there is a clear measured need.

## Project-Specific Conventions
- `license-maven-plugin` formats license headers during `process-sources` for `src/main/java/**` and `src/test/java/**`.
- Compiler is configured with `-proc:none`; avoid introducing code that depends on annotation processing.
- Follow Checkstyle and PMD configuration instead of adding local style exceptions.
- Entries in the mandatory validation tools table are required quality gates and must not be bypassed or replaced.
- Magic numbers are allowed in test sources by configuration, but keep production code explicit with named constants.
- Generated sources under `src/gen/` and `src/generated/` are excluded from quality checks.

## Common Pitfalls
- `clean package` can update files (for example license headers). If that happens, review and include intentional changes.
- On Windows, use `mvnw.cmd`.
- CI fails if working tree changes are produced during build; keep repo clean after running build commands.
- Release profiles require signing and publishing credentials; do not modify release setup casually.

## Quality Boundaries
- Follow complexity limits reflected in PMD rules:
	- Method cyclomatic complexity up to 25.
	- Class cyclomatic complexity up to 150.
	- Nested `if` depth up to 4.
- Keep cleanup strict: remove unused imports, dead code, and empty blocks while touching files.
- Add or update automated tests for every behavior change.

## Key References
- `README.md`
- `CONTRIBUTING.md`
- `pom.xml`
- `template-placeholder/pom.xml`
- `template-placeholder-demo/pom.xml`
- `.config/checkstyle/checkstyle.xml`
- `.config/pmd/java/ruleset.xml`
- `.github/workflows/check-build.yml`
