# Language-Specific Code Review Conventions

Check this reference when reviewing code in the respective language.

---

## 1. Go (Golang)

### Error Handling
- Check `if err != nil` immediately after every call that returns error
- Never discard errors with `_` unless justified with inline comment
- Wrap errors with `fmt.Errorf("context: %w", err)` for traceability

### Concurrency
- Goroutines must have lifetime management (`sync.WaitGroup`, `context.Context`)
- Watch for data races on shared maps/slices (use `sync.Mutex` or channels)
- Check channel close semantics — sender closes, receiver checks `ok`

### Resources
- Use `defer f.Close()` immediately after successful acquisition
- Beware `defer` inside loops — deferred calls accumulate until function exit

### Idioms
- Short concise names (`ctx`, `r`, `w`) for common variables
- Use interfaces for abstractions, concrete types for implementation
- Return error as last return value, always

---

## 2. Python & Django

### Type Safety & Style
- Type annotations required on public methods and API contracts
- Use `Optional[X]` or `X | None` instead of implicit None returns
- Never suppress exceptions bare (`except:`) — catch specific exceptions

### Django ORM & Database
- **N+1 queries**: use `select_related` (FK/OneToOne) and `prefetch_related` (ManyToMany/reverse FK)
- **Query performance**: use `.exists()` and `.count()` instead of `bool(qs)` or `len(qs)`; never load unbounded datasets with `.all()` without pagination or `.iterator()`
- **Bulk operations**: use `bulk_create` / `bulk_update` instead of loop `.save()`
- **Transactions**: wrap multi-model mutating operations in `transaction.atomic()`
- **Migrations**: every model change must include corresponding migration file; avoid risky defaults on nullable fields or non-concurrent index additions on production tables

### Django REST Framework (DRF)
- Enforce `permission_classes` on all API views / ViewSets
- Validate inputs inside Serializer `validate_<field>` / `validate`, never bare in view
- Call `serializer.is_valid(raise_exception=True)`
- Separate business logic into service layer, not fat serializers or fat views

### Celery & Background Tasks
- Pass model IDs / primitives as task arguments, never model instances (stale data / serialization bug)
- Ensure task idempotency and configure explicit retries with backoff

### Async & Security
- Never use synchronous blocking calls (`requests`, sync ORM) inside `async def`
- Parameterized queries only — never f-string raw SQL in `.raw()` or `.extra()`
- No unsafe deserialization (`pickle.loads`, `yaml.load` without SafeLoader)
- Validate and sanitize external webhook / API payloads at trust boundaries

---

## 3. TypeScript / JavaScript

### Typing
- Avoid `any` without explicit justification in comment
- Prefer `unknown` with type guards over `any`

### Async
- Never mix callbacks with `async/await`
- Handle promise rejections — no floating unhandled promises
- Avoid `Promise.all` on unbounded dynamic arrays (concurrency spike risk)

### React / Next.js
- Check hook dependency arrays — missing deps cause stale closures, extra deps cause infinite loops
- Never mutate state directly — always return new object/array
- Watch for hydration mismatches (server vs client rendered content)

---

## 4. Java / Kotlin

### Null Safety
- Kotlin: avoid force unwraps (`!!`) — use `?.let {}` or `?: run {}`
- Java: check `Optional` usage and null validations at DTO boundaries

### Spring
- Proper transaction boundaries (`@Transactional` on service layer, not controller)
- Avoid business logic in Controller layer
- Watch for circular dependency injection

### Performance
- Avoid mutating collections during stream/operation processing
- Be careful with lazy collections holding references to large datasets

---

## 5. PHP

### Security
- Validate all `Request` inputs — never trust client data
- Mass-assignment vulnerability: prefer `$fillable` allowlist over `$guarded` blocklist
- Escape output in templates to prevent XSS

### Eloquent / Doctrine
- Identify N+1 query problems — use eager loading (`with()`)
- Never query database inside loop or blade template

### Types
- Use PHP 8.x strict types (`declare(strict_types=1);`)
- Type all parameters and return values on public methods

---

## 6. Rust

### Safety
- Justify every `unsafe` block with inline comment explaining why
- Avoid `.unwrap()` or `.expect()` in production code — use `?` or `match`

### Ownership
- Avoid excessive `.clone()` to appease borrow checker — prefer references/slices
- Use `Cow<str>` for owned-or-borrowed strings when appropriate

### Error Handling
- Use `thiserror` for library errors, `anyhow` for application errors
- Propagate with `?` operator — never `.unwrap()` after `Result` return

---

## 7. AI Slop & LLM Code Smells (Universal)

Flag these across all languages:
- **Hallucinated APIs & Attributes**: Methods, kwargs, or attributes that look plausible but do not exist in the referenced library/SDK version.
- **Over-engineered Architecture**: Creating single-use factories, wrapper adapters for 2-line functions, unnecessary interfaces, or abstract classes for simple tasks.
- **Narrative / Redundant Comments**: Comments that merely narrate what the next line of code does (`# set status to active`, `# return response`), or typical LLM intro/outro markers left in the code.
- **Phantom Defensiveness**: Layering redundant null checks or try-except blocks around trivial statements that cannot fail or where Python/Go idioms already guarantee safety.
- **Dead Boilerplate**: Copy-pasted helper functions that are never invoked, dummy schemas, or unused type declarations.
- **Subtle Semantic Mismatches**: Code that looks syntactically and idiomatically flawless but subtly flips comparison logic (`<` instead of `<=`), drops timezone awareness (`datetime.now()` without UTC), or misinterprets third-party API return shapes.

---

## 8. UI / Styling / Design Taste (CSS, HTML, Templates, Components)

Apply when diff touches frontend, templates, or styling:
- **Visual Hierarchy**: Clear primary/secondary actions, readable headings, logical emphasis.
- **Spacing & Layout**: Consistent rhythm, alignment, padding/margins; avoid cramped or uneven layouts.
- **Typography**: Legible font sizes/line-height, consistent scale, proper truncation/wrapping.
- **Color & Contrast**: Sufficient contrast (WCAG-minded), meaningful color usage (not color-only signals), consistent palette/tokens.
- **Responsiveness**: Works across breakpoints/viewports; no overflow/cutoff on small screens.
- **Accessibility**: Focus states, semantic markup/labels, keyboard navigability, touch target sizing where relevant.
- **Consistency & Polish**: Matches existing design system/components; avoids one-off styling drift.
- **Tasteful Pragmatism**: Prefer simple, cohesive improvements over ornamental complexity. Treat most visual refinements as 💡 Suggestion unless they break usability/accessibility.
