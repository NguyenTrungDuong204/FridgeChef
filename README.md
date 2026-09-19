[English](README.md) | [Tiếng Việt](README.vi.md)

# FridgeChef

<img src="public/favicon.svg" alt="FridgeChef chef-hat mark" width="48" height="48">

Tick the ingredients you already have. FridgeChef ranks dishes by how much of the recipe you can cover, lists what is still missing, and opens a cook page with step-by-step instructions.

The API contract is in [`API.md`](API.md). A function-to-file map lives in [`NOTES.md`](NOTES.md).

## What it does

| Piece | Detail |
| --- | --- |
| Board UI | Workbench layout: nav rail, ingredient column, results. Locked in; the four local UI drafts are gone. |
| Logo | Chef-hat mark in `public/favicon.svg`, not the letters `FC`. |
| Fridge | 105 ingredients in 8 groups, checkboxes only (no autocomplete). |
| Match | Score = coverage × 100 − missing × 5. Recipes at 0% are dropped. |
| Random recipes | Picks one dish, ticks its required ingredients, then matches. |
| Recently viewed | Last 12 recipes opened in this browser (`localStorage` key `fridgechef.recentIds`). |
| Steps | Heat, timing, cuts, and doneness cues in `recipes.json`. |
| Photos | TheMealDB HTTPS photos. |

The four-week spec leaves out a database, favorites, a shopping list, autocomplete, and React/Tailwind.

## Requirements

- Node.js 18 or newer
- A browser

There is no database. Recipes and ingredients are JSON files.

## Run it

```bash
npm install
npm start
```

Open [http://localhost:3000](http://localhost:3000).

| Command | What it does |
| --- | --- |
| `npm start` | Express on port 3000 |
| `npm run dev` | Same, with nodemon restarting `server/` |

To skip Express APIs in the browser, set `USE_MOCK = true` on line 1 of `public/js/api.js`. The client then loads `GET /mock-data.json`.

## Using the app

1. On **My Fridge**, tick ingredients, or use **Random recipes** to fill one complete dish.
2. Click **Find recipes**. A 100% card means every required ingredient is ticked.
3. Open a card. Green = you have it, amber = missing, muted = optional.
4. **Recently viewed recipes** in the left nav lists dishes you opened on this device. Opening `recipe.html?id=…` prepends that id (max 12).

Also stored: `fridgechef.ingredients` (ticks). In `sessionStorage`: `fridgechef.featuredId` (random pick highlight) and `fridgechef.lastRandomId` (avoid repeating the last random dish).

## Pages

| URL | File | Role |
| --- | --- | --- |
| `/` | `public/index.html` | Fridge, match, recently viewed |
| `/recipe.html?id=r001` | `public/recipe.html` | One recipe |
| `/styleguide.html` | `public/styleguide.html` | Buttons, cards, toast, modal |

The UI is English. Matching still accepts Vietnamese diacritics (`Cà Chua` → tomato) through `normalize` and `SYNONYMS` in `matchService.js`.

## Architecture

```text
Browser (vanilla HTML/CSS/JS)
    └── Fetch ──► Express (server/app.js)
                      ├── GET  /api/ingredients
                      ├── POST /api/recipes/match   { ingredients: string[] }
                      ├── GET  /api/recipes/:id
                      └── GET  /api/recipes?cuisine=&maxTime=&difficulty=
```

| Layer | Path |
| --- | --- |
| Static UI | `public/` via `express.static` |
| Routes | `server/routes/recipes.js` |
| Validation | `server/controllers/recipeController.js` (400 / 404) |
| Match | `server/services/matchService.js` |
| Recipes | `server/data/recipes.json` — 36 dishes, `r001`–`r036` |
| Ingredients | `server/data/ingredients.json` — 105 items, 8 groups |

Photos are TheMealDB `strMealThumb` URLs, not local `/img/*.jpg`.

## Where the code lives

### Fridge and Random recipes

- Checkbox groups: `public/js/ingredients.js` (`renderGroups`)
- Selected values: `querySelectorAll('.ing-item input:checked')`
- Find: `findRecipes()` → `POST /api/recipes/match`
- Random: `randomFridge()` → `GET /api/recipes`, tick required names, then match
- Button: `public/index.html` `#random-fridge`

### Recently viewed

- After a successful detail load: `public/js/recipe-detail.js` calls `api.recordRecent(id)`
- Ids: `api.getRecentIds()` in `public/js/api.js`
- List: `public/js/results.js` (`renderRecent`)
- Nav: `public/index.html` `#recent`, `public/recipe.html` `/#recent`

### Score

```
required = ingredients where optional === false
coverage = have / required.length
score    = coverage * 100 - missing.length * 5
```

Server: `matchRecipe` in `server/services/matchService.js`. Browser mock: `matchRecipesLocal` in `public/js/api.js`.

### Board layout

- Nav rail: 12.25rem (was 15.5rem)
- Ingredient column: 16.75rem (was up to 22rem)
- Color `#0d9488`, Plus Jakarta Sans: `public/css/ui-drafts.css` under `html[data-ui="board"]`

## Check the API

```bash
curl http://localhost:3000/api/ingredients
curl -X POST http://localhost:3000/api/recipes/match -H "Content-Type: application/json" -d "{\"ingredients\":[\"egg\",\"tomato\",\"onion\",\"garlic\"]}"
curl http://localhost:3000/api/recipes/r001
```

The match body above should put Tomato Fried Eggs at 100%. Unknown id → 404. Match body not `string[]` → 400.

## Tree

```text
FridgeChef/
├── API.md
├── NOTES.md
├── README.md
├── README.vi.md
├── mock-data.json
├── package.json
├── public/
│   ├── index.html
│   ├── recipe.html
│   ├── styleguide.html
│   ├── favicon.svg
│   ├── css/base.css
│   ├── css/components.css
│   ├── css/layout.css
│   ├── css/ui-drafts.css
│   └── js/api.js, ingredients.js, results.js, recipe-detail.js
└── server/
    ├── app.js
    ├── routes/recipes.js
    ├── controllers/recipeController.js
    ├── services/matchService.js
    └── data/recipes.json, ingredients.json
```
