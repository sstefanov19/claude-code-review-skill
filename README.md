# /review - Multi-Persona Code Review Skill for Claude Code

A Claude Code skill that reviews your code through the eyes of **four senior engineering specialists** — each bringing their own expertise, professional paranoia, and years of battle scars to your pull requests.

Stop shipping code that "works but..." — get the review you'd get from a team of senior engineers who actually care.

---

## Why This Exists

AI-generated code is everywhere. Vibe-coded apps ship fast. But speed without scrutiny creates:

- **Security holes** that scanners miss (business logic flaws, IDOR, race conditions)
- **Convention drift** that turns a clean codebase into a patchwork of conflicting styles
- **Clever code** that the author understands today and nobody understands next month
- **SOLID violations** that make the next feature twice as hard to build

This skill doesn't just lint your code. It **thinks about your code** the way experienced engineers do — by understanding the patterns your codebase already uses and judging new code against that baseline.

---

## The Four Personas

Every review activates specialist personas based on what's in your code. Each one writes their section of the review in first person, with their own voice and priorities.

### Senior Backend Engineer
> *12+ years building distributed systems in Java and Spring Boot.*

Reviews pattern consistency and SOLID principles. Scans your existing codebase first to learn its conventions, then flags anything that drifts. Treats inconsistency as seriously as bugs — because in a team, it is.

**Checks for:** Single Responsibility violations, Open/Closed principle breaks, interface bloat, dependency inversion failures, naming convention drift, package structure inconsistencies.

### Senior Frontend Engineer
> *10+ years building production UIs.*

Activated when your code includes JS, TS, HTML, CSS, or framework files. Reviews component architecture, state management patterns, accessibility, and performance.

**Checks for:** Prop drilling, leaking abstractions, inconsistent component patterns, accessibility gaps, state management anti-patterns.

### Senior Security Engineer
> *15+ years of penetration testing, threat modeling, and secure code review.*

This persona thinks like an attacker. For every input, they ask *"what happens if I send something unexpected?"* For every data flow, they trace where secrets and PII travel. Findings include **attack scenarios** — not just "this could be bad" but "here's exactly how someone exploits this."

**Checks for:** SQL/Command/LDAP injection, missing input validation, authentication gaps, authorization bypass (IDOR), data exposure in APIs and logs, hardcoded secrets, weak cryptography, dependency CVEs, race conditions, business logic flaws.

### Senior Staff Engineer / Tech Lead
> *Has onboarded dozens of developers onto large codebases.*

Optimizes for the reader, not the writer. Every method is judged by one question: *can a junior developer on their first day follow this without asking questions?*

**Checks for:** God methods, cryptic naming, one-liner heroics, deep nesting (prefers guard clauses), magic values, commented-out code, missing constant extraction, methods longer than ~15 lines.

---

## What Makes It Powerful

### 1. Context-Aware Reviews (Not Generic Linting)
Before judging a single line, the skill **scans 3-5 existing files** in the same package/feature area. It learns your conventions — naming style, error handling approach, DI pattern, test structure — and uses that as the baseline. A review on a Spring Boot monolith reads completely differently from one on a React SPA.

### 2. Multi-Persona Depth
Generic reviews are shallow. A security engineer thinks differently than a backend architect, who thinks differently than someone focused on readability. By having each persona focus on their domain, the review catches issues that a single-pass review would miss.

### 3. Attack Scenarios, Not Just Warnings
The security persona doesn't just say "input not validated." It explains: *"An attacker sends `'; DROP TABLE users; --` in the `name` parameter, which gets concatenated into the query at line 42, resulting in arbitrary SQL execution."* This makes severity real and actionable.

### 4. Structured, Actionable Output
The review produces a table-based report with severity levels (`critical` / `warning` / `info`), exact file locations, specific recommendations, and an ordered action item list. No vague "consider improving this" — every item tells you exactly what to change.

### 5. Automatic Layer Detection
Drop the skill into a full-stack repo and it figures out what's what. Backend code gets the backend + security personas. Frontend code gets the frontend + security personas. A full-stack PR gets all four. You don't configure anything.

---

## Installation

### Option A: Per-Project (Recommended)

Copy the skill into your project's `.claude/` directory:

```bash
# From your project root
mkdir -p .claude/skills/review
curl -o .claude/skills/review/SKILL.md \
  https://raw.githubusercontent.com/sstefanov19/claude-code-review-skill/main/.claude/skills/review/SKILL.md
```

Commit it to your repo so the whole team gets it:

```bash
git add .claude/skills/review/SKILL.md
git commit -m "Add /review skill for multi-persona code reviews"
```

### Option B: Global (All Projects)

Install it in your home directory so it's available everywhere:

```bash
mkdir -p ~/.claude/skills/review
curl -o ~/.claude/skills/review/SKILL.md \
  https://raw.githubusercontent.com/sstefanov19/claude-code-review-skill/main/.claude/skills/review/SKILL.md
```

### Option C: Clone and Copy

```bash
git clone https://github.com/sstefanov19/claude-code-review-skill.git
cp -r claude-code-review-skill/.claude/skills/review /path/to/your/project/.claude/skills/
```

---

## Usage

Open Claude Code in your project and run:

```
/review
```

That's it. The skill will review all uncommitted changes.

### Review Specific Files

```
/review src/main/java/com/example/UserService.java
```

### Review a Directory

```
/review src/main/java/com/example/auth/
```

### Review a Pull Request

```
/review 42
```

---

## Example Output

```markdown
## Review Summary
The authentication module introduces a custom JWT handler that bypasses Spring Security's
built-in token validation. The implementation works but introduces three security concerns
and deviates from the repository's established patterns in error handling and naming.

## Senior Backend Engineer — Pattern Consistency
| Finding | Severity | Location | Recommendation |
|---------|----------|----------|----------------|
| Field injection used instead of constructor injection | warning | AuthService.java:15 | Refactor to constructor injection — every other service in this package uses it |
| Custom exception doesn't extend BaseApiException | warning | TokenExpiredException.java:3 | Extend BaseApiException to work with the global exception handler |

## Senior Security Engineer — Security
| Finding | Severity | Location | Recommendation |
|---------|----------|----------|----------------|
| JWT secret hardcoded in source | critical | JwtConfig.java:12 | Move to environment variable or vault. **Attack scenario:** anyone with repo access has the signing key and can forge tokens for any user |
| Token expiry not validated | critical | JwtValidator.java:28 | Check `exp` claim before accepting. **Attack scenario:** stolen tokens work forever |
| User ID from token used without ownership check | warning | ProfileController.java:34 | Verify requesting user owns the profile. **Attack scenario:** change the `userId` path param to access other users' data (IDOR) |

## Senior Staff Engineer — Clean Code & Readability
| Finding | Severity | Location | Recommendation |
|---------|----------|----------|----------------|
| `proc()` method — 47 lines, does parsing + validation + persistence | warning | AuthService.java:50 | Split into `parseToken()`, `validateClaims()`, and `persistSession()` |
| Magic number `86400` | info | JwtConfig.java:8 | Extract to `TOKEN_EXPIRY_SECONDS` with a comment explaining the 24h choice |

## Positive Highlights
- **Backend Engineer:** Clean DTO mapping using the project's existing MapStruct pattern.
- **Security Engineer:** Proper use of BCrypt for password hashing with appropriate cost factor.

## Action Items
1. [Security] Move JWT secret to environment variable — hardcoded secrets are a critical risk
2. [Security] Add token expiry validation in JwtValidator
3. [Security] Add ownership check in ProfileController before returning user data
4. [Backend] Refactor AuthService to use constructor injection
5. [Backend] Extend BaseApiException for custom exceptions
6. [Readability] Break up `proc()` into three named methods
7. [Readability] Extract magic number to named constant
```

---

## Customization

The skill file is just Markdown. Fork it and make it yours:

- **Add personas** — need a DBA persona for migration reviews? Add a Step 5.
- **Adjust severity rules** — if your team treats all injection risks as `critical`, update the security section.
- **Add framework-specific checks** — add React hook rules, Django ORM patterns, or Rails conventions.
- **Change the output format** — prefer a checklist over tables? Edit the Output Format section.

---

## Requirements

- [Claude Code](https://claude.ai/code) (CLI, desktop app, or IDE extension)
- That's it. No dependencies, no build step, no API keys.

---

## Contributing

Contributions welcome. If you've got a check that would have caught a bug in production, open a PR.

1. Fork this repo
2. Edit `.claude/skills/review/SKILL.md`
3. Test it on real code: run `/review` in Claude Code against a project
4. Open a PR describing what the change catches and why it matters

---

## License

[MIT](LICENSE)
