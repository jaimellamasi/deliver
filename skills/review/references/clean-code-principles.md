# Clean code principles — review checklist

Caveman rules. Check these when a diff adds or changes code, any language.

## Naming

- Name says what it is. No `data`, `temp`, `stuff`, `x2`. Name reveals intent.
- Function name is a verb phrase. Variable name is a noun. Boolean name asks yes/no
  (`isReady`, `hasItems`).
- One word per concept. Don't call it `fetch` here and `get` there for the same action.
- No mental mapping. Don't rename a domain word to a shorter one just to save keystrokes.
- Name length matches scope. Loop index in 3-line loop: `i` fine. Name in wide scope: full word.

## Functions

- Function does one thing. If you need "and" to describe it, split it.
- Small. If it doesn't fit on one screen, it's doing too much.
- Few arguments. 0–2 is good, 3 is a smell, 4+ means pass an object instead.
- No boolean flag arguments. A flag arg means the function does two things — split it.
- One level of abstraction per function. Don't mix high-level orchestration with low-level
  detail in the same body.
- No side effects hidden behind an innocent name. `getUser()` that also logs the user in is a
  lie.
- Command or query, never both. A function either does something or answers something, not
  both.

## Comments

- Comment explains why, never what. Code already says what.
- Delete commented-out code. Git remembers it, the reader doesn't need to.
- No comment to apologize for bad code. Fix the code instead.
- No redundant comment restating the function/variable name.

## Duplication (DRY)

- Same logic in two places = one of them will rot. Extract it.
- Copy-pasted block with one changed value is duplication, not a new case.
- Don't DRY prematurely across things that only look similar today but change for different
  reasons — that's coupling, not reuse.

## Structure / Organization

- Related things stay close. Unrelated things stay apart.
- File/module has one reason to change (Single Responsibility). Grab-bag "utils" file is a
  smell.
- Depend on abstractions, not concrete details, at module boundaries.
- Public surface is small. Don't expose internals the caller doesn't need.
- Consistent structure across similar modules — same shape, same order of sections.

## Error Handling

- Don't swallow errors silently. Empty catch block is a bug waiting to hide.
- Fail fast and loud on programmer errors (bad input to internal code). Handle gracefully at
  system boundaries (user input, network, disk).
- Error handling is not business logic — don't tangle the two in the same block.
- Return/throw one consistent way per codebase. Don't mix error codes and exceptions in the
  same layer.

## Abstraction Levels

- Don't leak implementation detail through an abstraction's name or interface.
- Higher-level code reads like a story; lower-level code is where the detail lives. Don't
  invert this.
- Interface promises behavior, not structure. Don't design an interface around one class's
  internals.

## Testability

- Hard to test = badly designed, not "needs more mocks." Untestable code usually hides a
  missing seam.
- Function with no side effects is trivially testable. Prefer that shape by default.
- Global/hidden state makes tests flaky and order-dependent. Avoid it.

## Complexity

- Nesting more than 2–3 levels deep = extract and name the inner block.
- Cyclomatic complexity climbing = too many branches in one function. Split by case.
- Clever one-liner that needs a comment to explain it: rewrite it plainer instead.
- If a reviewer has to trace three files to understand one function, the function's
  abstraction is wrong.

## Note

These are defaults, not absolutes. If the target repo's own conventions
(`AGENTS.md`/`CLAUDE.md`, neighbouring code) deliberately diverge, follow the repo — flag the
divergence only if it looks accidental or inconsistent within the repo itself.
