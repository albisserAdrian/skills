---
name: code-simplification
description: Simplify code for clarity while preserving exact behaviour. Use when refactoring code that works but is harder to read than it should be, when accumulated complexity is slowing changes down, or when a review flags readability issues. Different from architecture deepening: this is about expression, not structure.
---

# Code Simplification

**Simplify means easier to comprehend, not fewer lines.** A 1-line nested ternary is not simpler than a 5-line if/else. The bar is: would a new team member understand this faster than the original?

This skill is about *expression*: making working code clearer. For structural change (deepening modules, moving complexity behind better seams), use `improve-codebase-architecture` instead. They're complementary, not competing.

When working in unfamiliar code, read the project's domain glossary first so renames match the project's vocabulary, and respect ADRs that explain why something looks the way it does.

## When NOT to simplify

- Code is already clean. Don't simplify for the sake of it.
- You don't yet understand what the code does. Comprehend before you change.
- The code is performance-critical and the "simpler" version would be measurably slower.
- The module is about to be rewritten or deleted. Wasted effort.

## The five principles

### 1. Preserve behaviour exactly

Don't change what the code does, only how it expresses it. All inputs, outputs, side effects, error behaviour, and edge cases must remain identical. **All existing tests must still pass without modification.** If a test had to change, you changed behaviour.

```
ASK BEFORE EVERY CHANGE:
→ Same output for every input?
→ Same error behaviour?
→ Same side effects and ordering?
→ All existing tests still pass without modification?
```

### 2. Follow project conventions

Simplification means making code more consistent with the codebase, not imposing external preferences. Read CLAUDE.md / project conventions and study how neighbouring code handles similar patterns *before* simplifying. Match the project's style for imports, function declarations, naming, error handling, type annotations.

Simplification that breaks project consistency is not simplification. It's churn.

### 3. Prefer clarity over cleverness

Explicit code is better than compact code when the compact version requires a mental pause to parse.

```ts
// UNCLEAR: Dense ternary chain
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// CLEAR: Readable mapping
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```ts
// UNCLEAR: Chained reduces with inline logic
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// CLEAR: Named intermediate step
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. Watch for over-simplification

Simplification has a failure mode. Watch for these traps:

- **Inlining too aggressively.** Removing a helper that gave a concept a name makes the call site harder to read.
- **Combining unrelated logic.** Two simple functions merged into one complex function is not simpler.
- **Removing "unnecessary" abstraction.** Some abstractions exist for extensibility or testability, not complexity. Check before removing.
- **Optimising for line count.** Fewer lines is not the goal; easier comprehension is.

### 5. Scope to what changed

Default to simplifying recently modified code. Avoid drive-by refactors of unrelated code unless explicitly asked to broaden scope. Unscoped simplification creates noisy diffs and risks unintended regressions.

## Process

### 1. Understand before touching (Chesterton's Fence)

Before changing or removing anything, understand why it exists. If you see a fence across a road and don't know why, don't tear it down. First understand the reason, then decide if it still applies.

```
BEFORE SIMPLIFYING, ANSWER:
- What is this code's responsibility?
- What calls it? What does it call?
- What are the edge cases and error paths?
- Are there tests that define the expected behaviour?
- Why might it have been written this way? (Performance? Platform constraint? Historical reason?)
- Check git blame: what was the original context?
```

If you can't answer these, you're not ready to simplify. Read more context first.

### 2. Identify opportunities

Scan for these patterns. Each one is a concrete signal, not a vague smell.

**Structural complexity:**

| Pattern | Signal | Simplification |
|---------|--------|----------------|
| Deep nesting (3+ levels) | Hard to follow control flow | Extract conditions into guard clauses or helper functions |
| Long functions (50+ lines) | Multiple responsibilities | Split into focused functions with descriptive names |
| Nested ternaries | Requires mental stack to parse | Replace with if/else, switch, or lookup objects |
| Boolean parameter flags | `doThing(true, false, true)` | Replace with options objects or separate functions |
| Repeated conditionals | Same `if` check in multiple places | Extract to a well-named predicate |

**Naming and readability:**

| Pattern | Signal | Simplification |
|---------|--------|----------------|
| Generic names | `data`, `result`, `temp`, `val`, `item` | Rename to describe content: `userProfile`, `validationErrors` |
| Abbreviated names | `usr`, `cfg`, `btn`, `evt` | Use full words unless universal (`id`, `url`, `api`) |
| Misleading names | Function named `get` that mutates | Rename to reflect actual behaviour |
| Comments explaining "what" | `// increment counter` above `count++` | Delete (code is clear enough) |
| Comments explaining "why" | `// Retry because the API is flaky under load` | Keep (they carry intent the code can't express) |

**Redundancy:**

| Pattern | Signal | Simplification |
|---------|--------|----------------|
| Duplicated logic | Same 5+ lines in multiple places | Extract to a shared function |
| Dead code | Unreachable branches, unused variables, commented-out blocks | Remove (after confirming it's truly dead) |
| Unnecessary abstractions | Wrapper that adds no value | Inline the wrapper, call the underlying function directly |
| Over-engineered patterns | Factory-for-a-factory, strategy-with-one-strategy | Replace with direct approach |
| Redundant type assertions | Casting to a type already inferred | Remove the assertion |

### 3. Apply changes incrementally

One simplification at a time. Run tests after each. **Submit refactoring changes separately from feature or bug-fix changes**. A change that mixes refactor and new behaviour is two changes.

```
FOR EACH SIMPLIFICATION:
1. Make the change
2. Run the test suite
3. Pass → commit (or continue to next)
4. Fail → revert and reconsider
```

Avoid batching multiple simplifications into a single untested change. If something breaks, you need to know which step caused it.

**The Rule of 500:** if a refactor would touch more than 500 lines, invest in automation (codemods, sed scripts, AST transforms) rather than editing by hand. Manual edits at that scale are error-prone and exhausting to review.

### 4. Verify the result

After all simplifications, step back and evaluate the whole:

- Is the simplified version genuinely easier to understand?
- Did you introduce any new patterns inconsistent with the codebase?
- Is the diff clean and reviewable?
- Would a teammate approve this change?

If the "simplified" version is harder to understand or review, revert. Not every attempt succeeds.

## Language examples

### TypeScript / JavaScript

```ts
// SIMPLIFY: Unnecessary async wrapper
// Before
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// After
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// SIMPLIFY: Verbose conditional assignment
// Before
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// After
const displayName = user.nickname || user.fullName;

// SIMPLIFY: Manual array building
// Before
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) activeUsers.push(user);
}
// After
const activeUsers = users.filter((user) => user.isActive);

// SIMPLIFY: Redundant boolean return
// Before
function isValid(input: string): boolean {
  if (input.length > 0 && input.length < 100) return true;
  return false;
}
// After
function isValid(input: string): boolean {
  return input.length > 0 && input.length < 100;
}
```

### Python

```python
# SIMPLIFY: Verbose dict building
# Before
result = {}
for item in items:
    result[item.id] = item.name
# After
result = {item.id: item.name for item in items}

# SIMPLIFY: Nested conditionals → early returns
# Before
def process(data):
    if data is not None:
        if data.is_valid():
            if data.has_permission():
                return do_work(data)
            else:
                raise PermissionError("No permission")
        else:
            raise ValueError("Invalid data")
    else:
        raise TypeError("Data is None")
# After
def process(data):
    if data is None:
        raise TypeError("Data is None")
    if not data.is_valid():
        raise ValueError("Invalid data")
    if not data.has_permission():
        raise PermissionError("No permission")
    return do_work(data)
```

## Honesty

- **No "I'll just quickly simplify this unrelated code too".** Unscoped simplification creates noisy diffs and risks regressions in code you didn't intend to change. Stay focused.
- **No simplifying code you don't fully understand.** Comprehension precedes change.
- **No removing error handling because "it makes the code cleaner".** Quietly weakening error behaviour is a regression dressed as a refactor.
- **No "the original author must have had a reason" as paralysis.** Apply Chesterton's Fence: check `git blame`, then decide. Accumulated complexity often *has* no reason. It's just the residue of iteration under pressure.
- **No mixing refactor with feature work.** Separate them. Mixed changes are harder to review, revert, and read in history.
- **If a "simplification" needs a test change to pass, you changed behaviour.** Revert and reconsider.
