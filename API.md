# FridgeChef API contract

Frontend and backend must follow this table. If you change a path, field, or status code, update this file and tell the whole team.

| Method | Path | Input | Output |
| --- | --- | --- | --- |
| `GET` | `/api/ingredients` | — | Ingredient groups used to build the checkbox grid |
| `POST` | `/api/recipes/match` | `{ "ingredients": ["egg", "tomato"] }` | Array of recipes with `matchPercent` and `missing` |
| `GET` | `/api/recipes/:id` | — | One recipe |
| `GET` | `/api/recipes?cuisine=&maxTime=&difficulty=` | Query string | Filtered recipe array |

## Why `POST /api/recipes/match` is not a GET

The ingredient list can be long and can include spaces or accented characters. Putting that list in a query string is hard to read and can hit URL length limits. `POST` with a JSON body is the right fit.

## Status codes

- `400` — request body or query is the wrong shape
- `404` — recipe id does not exist
- `500` — unexpected server error (handled by the shared error middleware)

## Recipe schema

Every recipe uses these field names. Do not rename them.

```json
{
  "id": "r001",
  "name": "Tomato Fried Eggs",
  "image": "https://www.themealdb.com/images/media/meals/rwvw8q1765660071.jpg",
  "cookTime": 15,
  "difficulty": "easy",
  "cuisine": "vietnamese",
  "servings": 2,
  "ingredients": [
    { "name": "egg", "quantity": "3", "optional": false },
    { "name": "tomato", "quantity": "2", "optional": false }
  ],
  "steps": ["Beat the eggs...", "Fry the aromatics..."]
}
```

`difficulty` is one of `easy` | `medium` | `hard`.

`image` is a public HTTPS URL from [TheMealDB](https://www.themealdb.com) (`strMealThumb`). The browser loads the file with `loading="lazy"` and `referrerpolicy="no-referrer"` (`public/js/results.js` `#95-98`, `public/js/recipe-detail.js` `#70-74`).

## `GET /api/ingredients`

```json
{
  "groups": [
    {
      "id": "proteins",
      "label": "Proteins",
      "ingredients": [
        { "value": "egg", "label": "Eggs" }
      ]
    }
  ]
}
```

`value` is what the client sends to `/api/recipes/match`. It must match `recipe.ingredients[].name` after `normalize()`.

## `POST /api/recipes/match`

**Request**

```json
{ "ingredients": ["egg", "tomato"] }
```

`ingredients` must be an array of strings.

**Response**

An array of recipe objects plus:

- `matchPercent` — `Math.round(coverage * 100)` where coverage is matched required ingredients / required ingredients
- `missing` — required ingredient objects the user does not have
- `score` — `coverage * 100 - missing.length * 5` (used to sort)

Recipes with `0%` coverage are omitted. Results are sorted by `score` descending, then `matchPercent` descending.

## `GET /api/recipes/:id`

Returns one recipe object. `404` if the id is unknown.

## `GET /api/recipes`

Optional query params:

- `cuisine` — exact cuisine slug (`vietnamese`, `italian`, …)
- `maxTime` — number; keep recipes with `cookTime <= maxTime`
- `difficulty` — `easy` | `medium` | `hard`

## Mock data

`mock-data.json` at the repo root (also served as `/mock-data.json`) lets the frontend run while `USE_MOCK = true` in `public/js/api.js`.
