# The Bananaboat Cookbook

The Bananaboat Cookbook is a web-based recipe manager built with `py4web`, `Vue.js`, and a SQL-backed database via `pydal`. It supports account-based recipe creation, shared ingredient management, recipe search, automatic calorie totals, and API-based data access.

## Stack

- Backend: `py4web`
- Frontend: `Vue.js` + server-rendered templates
- Database: `pydal` models and linking tables
- External data source: [TheMealDB](https://www.themealdb.com/) (`https://www.themealdb.com/api/json/v1/1/search.php?s=`)

## Run the App

1. Start in this project directory:
   - `cd bro`
2. Run py4web apps:
   - `py4web run apps`
3. Open the app:
   - `http://127.0.0.1:8000/recipe_manager`

## Core User Features

- Account sign up and login.
- Shared ingredient catalog:
  - Search ingredients by name.
  - Add ingredients with name, unit, calories-per-unit, and description.
- Recipe management:
  - Create recipes with type, description, instructions, image, and ingredient list.
  - Browse and search recipes by name and type.
  - Edit/delete only recipes authored by the logged-in user.
- Automatic total calorie calculation from recipe ingredients.

## Authorization Rules

- Frontend conditionally renders edit/delete controls only for the author.
- Backend enforces authorization for `PUT` and `DELETE` recipe operations.
- Ingredient `GET` endpoint is public.
- Recipe `GET` endpoint currently requires authentication in code (`@action.uses(db, auth.user)`).

## Database Schema

Defined in `bro/apps/recipe_manager/models.py`:

- `ingredients`
  - `name`, `unit`, `calories_per_unit`, `description`
- `recipes`
  - `name`, `type`, `image`, `servings`, `description`, `instructions`, `author`, `created_on`, `updated_on`
- `recipe_ingredients` (linking table)
  - `recipe_id`, `ingredient_id`, `quantity`

## Calorie Calculation

Recipe calories are computed from linked ingredients:

`total_calories = sum(quantity * calories_per_unit)`

Ingredients with missing calorie values are skipped from the total.

## TheMealDB Import

Implemented in `bro/apps/recipe_manager/private/populate_recipes.py`.

Import flow:

1. Fetch meals from TheMealDB search endpoint.
2. Insert each meal as a recipe.
3. Read up to 20 ingredient/measure fields (`strIngredient1..20`, `strMeasure1..20`).
4. Reuse existing ingredients by name or insert new ones.
5. Insert links in `recipe_ingredients` with parsed quantity values.
6. Commit the transaction.

Quantity parsing strips numeric characters from the measure string and defaults to `1.0` if parsing fails.

Important current behavior: `import_recipes(db)` is called from `models.py` at import time.

## API Endpoints

Routes are defined in `bro/apps/recipe_manager/controllers.py` and served under `/recipe_manager/...`.

### Ingredients

- `GET /recipe_manager/api/ingredients`
  - Optional query: `name`
  - Example: `GET /recipe_manager/api/ingredients?name=avocado`
- `POST /recipe_manager/api/ingredients` (auth required)
  - Adds an ingredient

### Recipes

- `GET /recipe_manager/api/recipes` (auth required)
  - Optional queries: `name`, `type`
  - Example: `GET /recipe_manager/api/recipes?name=avocado&type=Breakfast`
- `POST /recipe_manager/api/recipes` (auth required)
  - Creates a recipe
- `PUT /recipe_manager/api/recipes/<recipe_id>` (auth + author required)
  - Updates a recipe
- `DELETE /recipe_manager/api/recipes/<recipe_id>` (auth + author required)
  - Deletes a recipe
