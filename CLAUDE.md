# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flask web application that displays course information, using a simple MVC-style architecture. No database — all course data is hardcoded in-memory.

## Commands

```bash
# Run the application
python src/app.py
# Available at http://127.0.0.1:5000

# Run all tests
python -m unittest discover -s tests

# Run a specific test case
python -m unittest tests.test_app.AppTestCase.test_index
```

## Architecture

**Entry point**: [src/app.py](src/app.py) — creates the Flask app and registers routes via `add_url_rule()` (not decorators), keeping views decoupled from Flask.

**Views**: [src/views.py](src/views.py) — pure functions, import only `render_template` from Flask. Adding a new route means adding a function here and wiring it in `app.py`.

**Data**: [src/models.py](src/models.py) — `Course` class with `title`, `description`, `instructor`, `duration`. A module-level `courses` list holds all data. No IDs — courses are referenced by list index.

**Templates**: `src/templates/` — `layout.html` is the base. `course.html` uses `{% extends 'layout.html' %}`. `index.html` currently uses `{% include %}` instead of `{% extends %}` — inconsistent, fix if touching that template.

**Tests**: [tests/test_app.py](tests/test_app.py) — `sys.path` is patched at the top of the test file to add `src/` since it is not a package. Tests use Flask's built-in test client.

## Known Issues / Gaps

- `course.html` references `course.topics` and `course.title/description/instructor/duration` as template variables, but the `course` view only passes `course_id` (not a `Course` object). The course detail page will render blank/error for those fields until the view is updated to look up and pass the actual `Course` object.
- `Course` has no `topics` attribute — the template's `{% for topic in course.topics %}` loop requires adding this field to the model.
- Courses have no stable ID — to implement course lookup by URL `course_id`, either assign IDs to `Course` objects or use list index.

## Development Workflow

- Add unit tests for every change and confirm they pass before finishing.
- After implementing any new feature, verify it visually using the Playwright MCP tool:
  1. Start the app (`python src/app.py`)
  2. Connect Playwright to `http://127.0.0.1:5000`
  3. Interact with the feature and take a screenshot
  4. Save the screenshot to `test-output/` with a descriptive name (e.g., `feature-name-YYYY-MM-DD.png`)
