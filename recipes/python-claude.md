# CLAUDE.md for Django Projects

## Build & Test Commands
- **Install Dependencies:** `pip install -r requirements.txt`
- **Run Dev Server:** `python manage.py runserver`
- **Run Tests:** `python manage.py test` (or `pytest`)
- **Run Specific Test:** `python manage.py test apps.users.tests.test_models`
- **Linting:** `flake8 .` or `ruff check .`
- **Format:** `black .` or `ruff format .`
- **Migrations:** `python manage.py makemigrations && python manage.py migrate`

## Django Architecture Conventions
- **App Structure:** Follow the "apps" folder pattern. Business logic lives in `services.py` or `selectors.py`, not in `models.py` or `views.py`.
- **Views:** Prefer Class-Based Views (CBVs) for CRUD; use Function-Based Views (FBVs) for simple or highly custom logic.
- **ORM Patterns:** - Use `select_related` for ForeignKey/OneToOne and `prefetch_related` for ManyToMany to avoid N+1 queries.
    - Always use `QuerySet` methods for bulk actions.
- **Naming:** - Models: PascalCase (e.g., `UserProfile`).
    - Views: `ActionNameView` (e.g., `UserUpdateView`).
    - Templates: `app_name/action_name.html`.

## Testing & Environment
- **Factory Boy:** Use factories instead of manual model instantiation in tests.
- **Environment:** Use `python-decouple` or `django-environ` for secrets; never hardcode `settings.py` values.
