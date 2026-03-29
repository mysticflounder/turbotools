---
name: specforge-design
description: Converts a 1-3 sentence module description into a valid specforge YAML spec through interactive interview. Use when the user says "write a spec for...", "specforge design", "create a spec", "I want to build X" in a specforge context, or describes a module they want to build.
---

# specforge-design

Convert a natural language module description into a valid specforge v0.3 YAML spec through a targeted interview.

## Trigger conditions

Invoke this skill when the user:
- Says "write a spec for X"
- Says "specforge design" or "create a spec"
- Describes a module while working in a specforge project: "I want a utility that..." or "I want to build X"
- Pastes a 1-3 sentence module description while working with specforge

## Process

### Phase 1: Extract from description

**Immediately extract without asking:**
- **Module name** — from noun/verb in description (`sudoku solver` → `sudoku_solver`)
- **Public functions** — from action verbs (`solves puzzles` → `solve(...)`)
- **Types/data structures** — from nouns (`grid` → likely `list[list[int]]`)
- **Return type** — from context (`returns the completed grid` → same type as input)
- **Dependencies** — from domain (math/algorithms → stdlib only; HTTP → `httpx`)

Build a working draft spec in memory before asking any questions.

**Degenerate input:** If the description is too vague to infer at least one function or type unit (e.g., "I want a utility"), ask a single open-ended question before proceeding: "What does this module do — what are the main operations it performs?" Then resume Phase 2 with the answer.

### Phase 2: Targeted interview

Ask only for fields you cannot confidently infer. Ask **one question at a time**.

**Stopping criterion:** every required specforge field is fillable.

Required fields per unit kind:

| Unit kind | Required fields |
|---|---|
| `function` | `name`, `signature` (typed params + return type), `description` |
| `type` | `name`, `bases` or `decorators` if applicable, at least one `member` |
| `constant` | `name`, `type_annotation`, `description` |
| `alias` | `name`, `type_annotation`, `description` |
| `test` | `name`, `signature`, `description` (concrete, verifiable), `requires` |

_Note: `type_annotation` for `constant`/`alias` and the `requires` field for `test` are skill policy — see Conformance rules._

Typical questions (only if not inferable):
1. **Input/output types** — "What's the input format for the grid — `list[list[int]]`, or something else?"
2. **Error handling** — "What should happen with an unsolvable puzzle — raise an exception or return `None`?"
3. **Edge cases** — "Should the function handle an already-solved grid?"
4. **Naming** — "What should the module be called?" (only if ambiguous)
5. **Constants/limits** — "Is there a maximum grid size, or always 9×9?"

Do NOT ask:
- Questions whose answers are obvious from context
- Questions about implementation details (algorithm, approach) — those are for the code-generation step
- More than one question at a time

### Phase 3: Test case generation

Once the public API is clear, generate test units. Each test unit must have:
- A concrete, single-behavior description (not "test that solve works")
- The units it exercises in `requires`

Generate for every module (adapted to context):
- Happy path: valid input produces correct output
- Error cases: each documented exception condition
- Edge cases: boundary inputs (empty, single element, max size)
- Type/value invariants: return type matches declared type

Generate test descriptions without asking the user unless domain-specific behavior is unclear.

### Phase 4: Validation

After building the complete spec in YAML, validate it using specforge's Pydantic models:

```python
import yaml
from specforge.schema import AppSpec, ModuleSpec

module_dict = yaml.safe_load(module_yaml_string)
ModuleSpec.model_validate(module_dict)

app_dict = yaml.safe_load(app_yaml_string)
AppSpec.model_validate(app_dict)

# Cross-reference check: requires must list only existing unit names
unit_names = {u["name"] for u in module_dict["units"]}
for u in module_dict["units"]:
    for ref in u.get("requires", []):
        if ref not in unit_names:
            raise ValueError(f"requires references unknown unit: {ref!r}")
```

If specforge is not importable, write both YAML files and run:
```bash
specforge build spec/app.yaml --output /tmp/specforge-validate-scratch 2>&1
```
and capture any `ValidationError` output.

1. Both `ModuleSpec` and `AppSpec` pass validation → proceed to Phase 5
2. Violations found in either spec → fix the YAML and re-validate (max 3 iterations)
3. Still failing after 3 iterations → surface specific violations to the user; write broken YAML to `spec/<module>.yaml.draft` (not `spec/<module>.yaml`) so the user can repair it manually

### Phase 5: Write output

On successful validation:
- Write `spec/<module>.yaml`
- Write `spec/app.yaml` (manifest referencing the module)
- If `spec/` doesn't exist in the current working directory, create it
- Report: "Spec written to `spec/`. Run `specforge build spec/app.yaml -o out/` to generate code."

---

## Schema reference

```yaml
# app.yaml
name: <module_name>
version: 0.1.0
description: <one sentence>
modules:
- name: <module_name>
  spec: <module_name>.yaml

# <module>.yaml
name: <module_name>
description: <one sentence>
dependencies:
  stdlib: [<list>]
  third_party: [<list>]
  internal: []
units:
- kind: function | type | constant | alias | test
  name: <name>
  signature: "<name>(<params>) -> <return>"   # function/method/test
  description: <concrete description>
  requires: [<unit_names>]                     # test units only
  # type units additionally:
  bases: [<base_classes>]
  decorators: [<decorator_strings>]
  members:
    - kind: field | method
      name: <name>
      signature: "<name>(<params>) -> <return>"  # methods only
      type_annotation: <type>                     # fields only
      decorator: <string>                         # classmethod, staticmethod, property — NOT a list
      description: <description>
```

### Conformance rules

- `type` units must have at least one member
- `test` unit descriptions must describe a single, concrete, verifiable behavior
- `signature` fields must include full type annotations on all parameters and return type
- `requires` must list only unit names that actually exist in the spec
- `description` fields must not be empty or vague ("handles X" not "does stuff")
- `type_annotation` on `constant` and `alias` units: always populate it (not schema-enforced, but required by this skill)
- `decorator` on a `MemberSpec` (member of a `type` unit) is a singular string (`str | None`); `decorators` on a `UnitSpec` (type unit) is a list — do not confuse the two
- `requires` is only meaningful on `test` units; leave it empty or omit it for all other unit kinds

---

## Inference heuristics

| Description pattern | Inferred spec element |
|---|---|
| "raises X if Y" | Test unit: `test_raises_x_when_y` |
| "returns None if not found" | Return type `X \| None`; test for None case |
| "a list of X" | `list[X]` type annotation |
| "optional Y" | function `signature` parameter: `y: Y \| None = None` |
| "grid" without further context | `list[list[int]]` (ask to confirm) |
| "ID" or "identifier" | `int` or `str` (ask to confirm) |
| domain = math/algorithms | stdlib only |
| domain = HTTP/API | suggest `httpx` or `requests` |
| domain = data/CSV | suggest `csv` (stdlib) |
| type with a `value` field or constant-set members (suits, ranks, status codes) | likely enum → `stdlib: [enum]`; ask if `StrEnum` or plain `Enum` |

---

## Example interaction

**User:** "I want a utility that solves sudoku puzzles. It takes a partially-filled 9×9 grid and returns the completed grid."

**Skill (inferred):** Module `sudoku_solver`, function `solve(grid: list[list[int]]) -> list[list[int]]`

**Skill asks:** "What should happen if the puzzle has no solution — raise `ValueError` or return `None`?"

**User:** "Raise ValueError."

**Skill asks:** "Should `solve` validate that the input grid is a legal sudoku puzzle (no duplicate numbers in rows/columns/boxes), or trust the caller?"

**User:** "Validate it."

**Skill generates:**
- `solve(grid: list[list[int]]) -> list[list[int]]` — solves a partially-filled 9×9 sudoku grid
- `validate_grid(grid: list[list[int]]) -> None` — raises `ValueError` if grid is malformed or contains duplicates
- Test units: `test_solve_valid_puzzle`, `test_solve_unsolvable_raises`, `test_validate_rejects_duplicate_in_row`, `test_validate_rejects_wrong_size`, `test_solve_already_complete_grid`

**Skill:** "Does this look right? I have 2 functions and 5 test cases."

**User:** "Yes."

**Skill:** Validates with `ModuleSpec.model_validate`, writes `spec/sudoku_solver.yaml` + `spec/app.yaml`.

---

## Out of scope (v1)

- Multi-module apps (single module only)
- Suggesting implementation strategies
- Running `specforge build` automatically (user does that separately)
- Modifying an existing spec (design → new spec only)

<!-- created_from: 25711f8 -->
