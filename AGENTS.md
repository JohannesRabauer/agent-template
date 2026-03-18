# Agents.md — Java / Maven Project Coding Standards

Single source of truth for AI coding agents. Merges commit-card standards with Checkstyle and PMD rulesets.

## Principles
- Readable code over clever one-liners. Break complex booleans into step-by-step `if` checks with early returns.
- Extract duplicated logic; don't force premature abstractions. No abstractions for requirements that don't exist yet.
- One class = one responsibility. No hidden side effects. Return `Optional<T>` when a value may be absent.
- Validate inputs immediately. Throw `IllegalArgumentException` rather than silently correcting.
- Never leave messy or dead code behind. Clean up every time you touch code.
- Every opened resource must be closed — use try-with-resources.
- Use interfaces and composition so implementations can be swapped. Every committed line must be explainable. Replace magic numbers with named constants.

## Naming
- Long, descriptive names: `daysUntilExpiration` not `d`. Domain language: `OrderService.processOrder` not `TransactionProcessor.doStuff`.
- No collection-type suffixes on variables (`Map`, `List`, `Set`). No class suffixes `Helper`/`Util`.
- Constants `UPPER_SNAKE`, members `camelCase`, packages `lowercase`, types `PascalCase`.

## Design
- Composition over inheritance. Constructor injection. Code against interfaces, not concrete types.
- Use records for data (`public record User(int id, boolean active) {}`). Use enums (with fields/methods) instead of magic values.
- Always parameterize collections — no raw types. Prefer `List<E>` over arrays. Use bounded wildcards.
- Use `of(…)`/`from(…)` static factory methods with `private` constructors when clearer.
- Fields `private final`. No setters. Use `List.of(…)` / `Collections.unmodifiable*`. Prefer new objects over mutation.
- Complexity limits (PMD): method cyclomatic ≤ 25, class ≤ 150, nested `if` ≤ 4, fields ≤ 50, methods ≤ 100, imports ≤ 200.

## Exceptions
- Never swallow exceptions. Log with context or rethrow wrapped. Preserve stack traces.
- Catch specific types — never `Error`, `Throwable`, or `NullPointerException`. No empty `catch` blocks. No return from `finally`.
- No `System.out.println` / `e.printStackTrace()` — use SLF4J. One logger per class. Use `{}` placeholders.
- Switches must be exhaustive or have `default`; no implicit fall-through.

## Security
- Never commit credentials. Use `@Value("${…}")` or env vars. Hash passwords with `BCryptPasswordEncoder`/`Pbkdf2PasswordEncoder`.
- Sanitize all inputs (including `ZipEntry` names for ZipSlip). No Java object serialization — use JSON.
- Audit `pom.xml` regularly for vulnerabilities. Prefer JDK or existing deps over new ones. Modern Java replaces many legacy libs.

## Testing & Documentation
- JUnit 5 for unit tests, `@SpringBootTest` for integration. Every new feature and every behavioral change must have automated tests. Magic numbers allowed in tests.
- Simplify assertions. Comments explain *why*, not what. Javadoc for public APIs with non-empty `@param`/`@return`/`@throws`.
- `@Override` always present. `@Deprecated` must pair with `@deprecated` Javadoc. No `TODO` in committed code.

## Concurrency
- No unmanaged `Thread.start()` — use `ExecutorService`/`CompletableFuture`. Always provide a dedicated `Executor` to `supplyAsync`/`runAsync`.
- No `parallelStream()` for blocking/remote calls. Always provide timeouts to `Future.get()`, `invokeAll`/`invokeAny`; use `poll` over `take`.
- `DecimalFormat`/`SimpleDateFormat` are thread-unsafe — create locally or use `DateTimeFormatter` (`static final`). No assignment to non-final statics.

## Performance
- Create once as `private static final`: `Pattern`, `DateTimeFormatter`, `HttpClient`, `Gson`, `SecurityProvider`. No inline regex recompilation.
- Specify initial buffer size for `ByteArrayOutputStream`/`StringWriter` (e.g., 4096).
- Enum lookup with >3 constants: use a static `Map`, not streaming `values()`.
- Use `isEmpty()` not `size() == 0`, `StandardCharsets.UTF_8` not `"UTF-8"`, `EnumSet`/`EnumMap` for enum keys, `Set` (not `List`) for JPA `@OneToMany`/`@ManyToMany`. No reflection in `equals`/`hashCode`/`toString`.

## Banned APIs
- No `Optional.get()` → use `orElseThrow()`/`orElse()`/`map()`. No `StringBuilder`/`StringBuffer` → use `+`/`StringJoiner`.
- No `System.setProperty`, `System.gc()`, `@PostConstruct`, `@PreDestroy`, `java.util.Date`/`Calendar` → use `java.time.*`, constructor injection, `AutoCloseable`.
- No double-brace init. Use `"literal".equals(x)`. Prefer method references over lambdas. Try-with-resources for all `AutoCloseable`.

## Cleanup
- Remove all unused imports, fields, methods, parameters, variables, assignments. Remove empty blocks/statements, unnecessary parens/semicolons.
- `default` last in `switch`. Implement `equals` + `hashCode` together. Use `this.` for instance members. Inline variables when possible. Merge identical `catch` branches. Simplify booleans.
- Every class in a package. No `clone()`/`finalize()`. Utility classes get private constructors. Non-extendable classes `final`. Interfaces declare behavior. Parameters `final`. Use `100L` not `100l`.

## Automation & Suppression
- Integrate `maven-checkstyle-plugin` and `maven-pmd-plugin` with `failsOnError=true`. Use auto-formatters before commit. Refactor continuously.
- Checkstyle suppression: `// CHECKSTYLE:OFF RuleName` / `// CHECKSTYLE:ON RuleName`. PMD suppression: `@SuppressWarnings("PMD.RuleName")`. Generated code (`src/gen/`|`src/generated/`) is auto-excluded.
