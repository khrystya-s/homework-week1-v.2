# Pull Request Review

**Repository:** khrystya-s/homework-week1  
**PR #2:** Added python file to test skill for pull-request review  
**Branch:** `feature/ai-cli` → `main`  
**Reviewed file:** `model.py` (515 additions, 0 deletions)  
**Commit SHA:** `206099cf35b4bb2cdcd81898ca8021197ea21d0d`

---

## Summary

`model.py` implements the **Model layer** of an MVC-based Calorie Calculator application.  
It defines data classes (`FoodItem`, `MealEntry`, `ActivityEntry`, `WeightEntry`), utility functions for BMI/BMR/TDEE calculations, and a central `AppModel` class that manages persistence (JSON file), CRUD operations, and analytics.

The overall structure is reasonable and the code is functional. However, several issues were identified spanning correctness bugs, architectural concerns, and code style problems.

---

## Reviewed Files

| File | Status | Additions | Deletions |
|---|---|---|---|
| `model.py` | added | 515 | 0 |

---

## HIGH Issues

### H1 — `load()` silently swallows all `KeyError` exceptions — Potential Bug / Data Loss

**File:** `model.py`  
**Lines:** ~220–240 (the `load` method)

```python
except (json.JSONDecodeError, KeyError):
    self._load_defaults()
```

A `KeyError` during `from_dict` (e.g. missing `"food_name"` in a meal record) silently discards **all** persisted user data and replaces it with defaults. This is a data-loss risk: a single corrupted record in `data.json` causes the entire dataset to be overwritten.

**Recommendation:** Use per-record try/except inside the list comprehensions, skip only the malformed record, and log a warning. At a minimum, handle `KeyError` and `json.JSONDecodeError` separately to distinguish between a fully corrupt file and a single bad record.

---

### H2 — `_load_defaults` calls `self.save()` unconditionally inside the constructor — Potential Bug

**File:** `model.py`  
**Lines:** ~247–258 (the `_load_defaults` method)

```python
def _load_defaults(self):
    self.food_items = [...]
    self.save()
```

If `save()` fails (e.g., filesystem read-only, permissions error), the constructor will raise an unhandled exception and the app will crash on first launch. There is no try/except around this call.

**Recommendation:** Wrap `self.save()` in `_load_defaults()` with a try/except (e.g., `OSError`) so that a failure to write defaults does not prevent the application from starting.

---

### H3 — `recalc_water_from_meals` calls `get_food_by_name` twice per entry — Performance / Logic Bug

**File:** `model.py`  
**Lines:** ~305–315 (the `recalc_water_from_meals` method)

```python
total = sum(
    e.grams
    for e in self.meal_log
    if e.date == date_str
    and self.get_food_by_name(e.food_name) is not None
    and getattr(self.get_food_by_name(e.food_name), "category", "") == "Напої"
)
```

`get_food_by_name` performs a linear scan through `food_items` every call. Here it is called **twice** per entry (once for the `is not None` guard and once for `getattr`). This is redundant and O(n·m).

**Recommendation:** Store the result in a local variable:

```python
total = 0.0
for e in self.meal_log:
    if e.date != date_str:
        continue
    food = self.get_food_by_name(e.food_name)
    if food and food.category == "Напої":
        total += e.grams
self.water_log[date_str] = total
self.save()
```

---

## MEDIUM Issues

### M1 — `DATA_FILE` is a hardcoded relative path — Architecture

**File:** `model.py`  
**Line:** ~197

```python
DATA_FILE = "data.json"
```

The data file path is relative to the current working directory. If the app is launched from a different directory, data will be silently lost or a new empty file will be created.

**Recommendation:** Anchor the path relative to the module file:

```python
import pathlib
DATA_FILE = pathlib.Path(__file__).parent / "data.json"
```

---

### M2 — Inline semicolons in CRUD methods — Maintainability

**File:** `model.py`  
**Lines:** ~255–275

```python
def add_food(self, item: FoodItem):
    self.food_items.append(item); self.save()
```

Combining two statements on one line with semicolons is not idiomatic Python (PEP 8 §"Compound Statements") and makes code harder to diff and debug.

**Recommendation:** Split each statement onto its own line.

---

### M3 — `food_items` searched by linear scan; no indexing — Performance / Architecture

**File:** `model.py`  
**Line:** ~261

```python
def get_food_by_name(self, name: str) -> FoodItem | None:
    return next((f for f in self.food_items if f.name == name), None)
```

This method is called from analytics loops, making those O(n·m). As the database grows this degrades noticeably.

**Recommendation:** Maintain a `dict[str, FoodItem]` index keyed by name and update it in `add_food`, `edit_food`, `delete_food`, and `_load_defaults`.

---

### M4 — Analytics methods iterate full logs O(days × log_size) — Performance

**File:** `model.py`  
**Lines:** ~380–415

Methods like `activity_heatmap` (84 days × log), `calories_vs_burned_by_date`, and `calories_by_date` each iterate the full log once per day.

**Recommendation:** Pre-group entries by date using `collections.defaultdict` before iterating days, reducing complexity from O(days × log_size) to O(log_size + days).

---

### M5 — Missing newline at end of file — Code Style

**File:** `model.py`  
**Diff footer:** `\ No newline at end of file`

POSIX standard and PEP 8 require source files to end with a newline. Many tools (linters, git diff) flag this.

**Recommendation:** Add a trailing newline.

---

## LOW Issues

### L1 — Language inconsistency in comments and docstrings — Readability

**File:** `model.py`

The module docstring and most inline comments are in Ukrainian, while some identifiers and parameter descriptions use English (e.g., `sex: 'male' або 'female'`). This is inconsistent.

**Recommendation:** Choose one language for all comments and docstrings and apply it consistently.

---

### L2 — Inline `#` comments instead of `"""docstrings"""` on public methods — Readability

**File:** `model.py`

Many public methods use `# comment` instead of `"""docstring"""`, making them invisible to `help()` and documentation generators.

**Recommendation:** Convert relevant `#` comments to proper docstrings on public methods.

---

### L3 — `WeightEntry.__init__` assigns attributes out of order — Readability

**File:** `model.py`  
**Lines:** ~150–165

```python
def __init__(self, weight, date=None, note=""):
    self.weight = float(weight)
    self.note = note          # ← assigned before self.date
    self.date = (...)
```

`self.note` is assigned before `self.date`, inconsistent with the parameter order and `to_dict` output.

**Recommendation:** Reorder to match the parameter declaration and `to_dict` output order.

---

### L4 — `bmi_category` and `bmi_color` use single-line `if` returns — Readability

**File:** `model.py`  
**Lines:** ~175–185

```python
if bmi < 18.5: return "bmi_underweight"
```

PEP 8 discourages putting compound statements on one line.

**Recommendation:** Use standard multiline `if`/`return` format.

---

## Recommendations

| # | Priority | What | Why | Benefit |
|---|---|---|---|---|
| 1 | HIGH | Handle `KeyError` per-record in `load()` | Single bad record overwrites entire dataset | Data safety |
| 2 | HIGH | Wrap `save()` in `_load_defaults()` with try/except | IO error crashes app on first launch | Stability |
| 3 | HIGH | Fix double `get_food_by_name` call in `recalc_water_from_meals` | Redundant O(n·m) scans | Correctness + Performance |
| 4 | MEDIUM | Anchor `DATA_FILE` with `pathlib.Path(__file__).parent` | Silent data loss when launched from another directory | Reliability |
| 5 | MEDIUM | Add `dict` index for `food_items` lookup | O(n·m) → O(1) lookup in analytics | Performance |
| 6 | MEDIUM | Pre-group log entries by date in analytics methods | O(days × log_size) → O(log_size + days) | Performance |
| 7 | MEDIUM | Remove semicolons from CRUD one-liners (PEP 8) | Readability and debuggability | Code quality |
| 8 | LOW | Add trailing newline at EOF | POSIX compliance, tool compatibility | Code hygiene |
| 9 | LOW | Convert `#` comments to `"""docstrings"""` on public methods | Enables `help()` and documentation generators | Maintainability |
