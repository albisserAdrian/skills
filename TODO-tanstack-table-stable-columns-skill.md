# TODO: skill for stable TanStack Table column defs (editable-cell focus loss)

Turn the notes below into a skill. Proposed home: `skills/stack/` (sits next to
`avoid-use-effect` and `frontend-ui-engineering`). Decide during authoring whether
this is its own skill or a section folded into `frontend-ui-engineering`. Lean
toward its own small skill: the trigger is specific and the failure is easy to
misdiagnose as an extension/browser problem.

## Proposed skill metadata

- **name**: `tanstack-table-stable-columns` (or `stable-table-columns`)
- **description** (draft): Keep TanStack Table column definitions referentially
  stable and pass volatile per-row state through `table.options.meta`. Use when an
  editable cell (an `<input>`, `<textarea>`, or `contenteditable`) loses focus on
  every keystroke, when typing in a table cell is "impossible" or drops characters,
  when IME composition or cursor position resets mid-word, or when a browser
  extension (e.g. Vimium) starts intercepting keys inside a table input. Also when
  a table re-renders or feels slow on each keystroke.

## The behaviour to document

### Symptom

An editable cell in a TanStack Table remounts on every keystroke, so the input
loses focus after each character. Downstream effects:

- Can't type normally; characters land in the wrong place or get dropped.
- Cursor jumps to the end; text selection is lost.
- IME composition (CJK, accents) breaks.
- Keyboard extensions like Vimium leave "insert mode" the instant focus is lost and
  resume grabbing keys, so they appear to "trigger constantly."
- The row/column model rebuilds on each keystroke (wasted work).

These are all one root cause. It reads like a browser or extension bug, but it is a
React remount.

### Root cause

Volatile, per-keystroke state was baked into the memo that produces the **column
definitions**. Column defs are meant to be static, the stable "shape" of the table.
When fast-changing state is a dependency of that memo, the `columns` array is
rebuilt on every keystroke, giving each `cell` a **new function identity**. React's
`flexRender(columnDef.cell, ...)` renders that function as a component type; a new
type reference makes React unmount the old subtree and mount a new one, so the
`<input>` is torn down and recreated. New DOM element = lost focus.

```
keystroke -> setState -> derived closure changes
          -> columns useMemo re-runs (closure in deps) -> NEW column defs
          -> useReactTable rebuilds -> new cell fn identity -> <input> remounts
```

### Correct approach

Keep column defs referentially stable (empty or truly-stable deps). Route the
changing per-row data through `table.options.meta`, the channel the library
documents for extra render-time data the cells need. The cell reads the live value
from `meta`; the defs never change, so React reconciles the same `<input>` in place.

```
keystroke -> setState -> only meta.drafts changes
          -> columns stable -> same cell fn identity
          -> cells re-render in place (no remount) -> focus retained
```

### Minimal before / after

Before (buggy): the cell closure depends on the changing state, so the defs churn.

```tsx
const draftOf = useCallback((row) => edits[row.id] ?? row.original.value ?? '', [edits]);
const columns = useMemo(() => [
  { accessorKey: 'value', cell: ({ row }) => (
      <input value={draftOf(row)} onChange={(e) => setDraft(row.id, e.target.value)} />
    ) },
], [draftOf, setDraft]); // <- rebuilds every keystroke
```

After (stable defs + meta): a module-level cell component reads from meta.

```tsx
type Meta = { drafts: Record<string, string>; setDraft: (id: string, v: string) => void };

function ValueCell({ row, table }: { row: Row<T>; table: Table<T> }) {
  const meta = table.options.meta as Meta;
  const value = meta.drafts[row.original.id] ?? row.original.value ?? '';
  return <input value={value} onChange={(e) => meta.setDraft(row.original.id, e.target.value)} />;
}

const columns = useMemo(() => [
  { accessorKey: 'value', cell: ({ row, table }) => <ValueCell row={row} table={table} /> },
], []); // <- stable

const table = useReactTable({
  data, columns,
  meta: { drafts: edits, setDraft } satisfies Meta, // volatile data rides here
  // ...
});
```

## Why it's correctness, not a workaround

- Matches the library contract: static column defs, volatile data via
  `meta` / `data` / `state`. The buggy version fought that split.
- Fixes a whole class of bugs (cursor jump, selection loss, IME breakage, model
  churn), not just the one visible symptom.
- Cheaper: no per-keystroke column/row-model rebuild.

### Second bug: the controlled input lags on a large table

Stabilizing the defs fixes the remount, but a **controlled** input whose value
lives in parent state re-renders the parent on every keystroke, which reconciles
every row in the table. On the ~600-row stations table this was visibly laggy:
letters appeared behind the cursor. So the "caveat" is not theoretical; it bites as
soon as the table has more than a handful of rows.

Fix: keep keystrokes **local to the cell** and lift into the shared draft map only
on blur. Typing then re-renders one cell, not the table. Resync local state from the
external value when it changes (Discard, or a post-Commit refetch), using the
render-time "adjust state on prop change" pattern so it never fires while typing.

```tsx
function ValueCell({ row, table }) {
  const meta = table.options.meta as Meta;
  const external = meta.drafts[row.original.id] ?? row.original.value ?? '';
  const [value, setValue] = React.useState(external);
  const prev = React.useRef(external);
  if (prev.current !== external) { prev.current = external; setValue(external); } // reset only
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}      // local: one cell re-renders
      onBlur={() => meta.setDraft(row.original.id, value)} // lift: parent re-renders once
    />
  );
}
```

So there are **two independent problems** an editable table cell must solve, and the
skill should cover both explicitly:

1. **Identity churn -> remount -> lost focus.** Fixed by stable column defs + `meta`.
2. **Controlled value in parent state -> whole-table re-render per keystroke ->
   lag.** Fixed by cell-local state lifted on blur (or an uncontrolled input, or an
   external store per row).

Both are needed on a non-trivial table; fixing only #1 leaves the lag. Trade-off to
note: lifting on blur means parent-derived UI (a "pending changes" bar, validity
gating) updates on blur rather than per keystroke, which is fine for batch editing
and matches how the debounced search box in the same codebase behaves.

## Detection heuristics (for the skill's "use when")

- A `useMemo`/`useCallback` that returns TanStack `columns` (or a `cell`) and lists
  component state in its deps.
- An editable element rendered inside a `cell` while that cell's identity is not
  stable.
- Reports of "can't type in the table," "Vimium keeps triggering," "cursor jumps,"
  "IME broken in cell."

## Authoring checklist

- [ ] Confirm final home (`skills/stack/`, own skill vs. fold into
      `frontend-ui-engineering`).
- [ ] Write `SKILL.md` with the frontmatter above; sharpen the `description`
      trigger list (that is what fires the skill).
- [ ] Include the before/after minimal example; keep it framework-accurate for the
      installed TanStack Table major version.
- [ ] Cross-link `avoid-use-effect` and `frontend-ui-engineering`.
- [ ] Generalize beyond TanStack: the underlying rule is "don't derive component
      identity from volatile state," which also bites `React.memo` lists, `key`
      churn, and inline component definitions. Decide scope: table-specific vs. the
      general identity-stability rule with the table as the lead example.
- [ ] Consider running `skill-creator` / `write-a-skill` to scaffold and eval it.

## Provenance

Found while building `/admin/stations` in the gtn/platform monorepo (issue 168,
editable `trafficUrl` cell). The editable cell remounted per keystroke because the
`columns` memo depended on the draft state; Vimium made it obvious by grabbing keys
the moment focus dropped. Fixed by stabilizing the column defs and moving drafts
into `table.options.meta`.
