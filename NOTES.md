# NOTES — FridgeChef (VI / EN)

Ghi chú kỹ thuật dưới dự án: **chức năng nào nằm đúng file nào, dòng nào (`#`)**.  
Project notes: **which feature lives in which file and line (`#`)**.

Contract: `API.md`. Overview: `README.md` (English) and `README.vi.md` (Tiếng Việt).

---

## Stack / Công nghệ

| Tech | File:# | VI | EN |
| --- | --- | --- | --- |
| Express app | `server/app.js` `#1-31` | Tạo server, CORS, JSON, static, listen | Create server, CORS, JSON, static, listen |
| cors | `server/app.js` `#9` | Cho phép gọi API cross-origin | Cross-origin API |
| express.json | `server/app.js` `#10` | Parse body tối đa 32kb | Parse body, 32kb cap |
| express.static | `server/app.js` `#11` | Phục vụ `public/` | Serve `public/` |
| nodemon | `package.json` `#9` | Reload khi sửa server | Reload on server edits |
| Fetch | `public/js/api.js` `#69-81` | Gọi HTTP từ trình duyệt | Browser HTTP |
| localStorage ticks | `public/js/api.js` `#148-159` | Nhớ nguyên liệu đã tick | Persist ticks |
| localStorage recently viewed | `public/js/api.js` `#161-179` | 12 món vừa mở trên máy | Last 12 opened recipes |
| CSS variables | `public/css/base.css` `#25-89` | Token màu / khoảng cách | Color/spacing tokens |
| CSS Grid | `public/css/layout.css` `#124-129`, `#148-152` | Lưới checkbox và card | Checkbox + card grids |
| Flexbox | `public/css/layout.css` `#6-23` | Header / nav | Header / nav |
| Sticky toolbar | `public/css/layout.css` `#83-93` | Nút Find recipes dính khi cuộn | Find recipes stays visible |
| `<template>` | `public/index.html` `#135-158` | Khuôn card + skeleton | Card + skeleton stamps |

---

## Backend map / Backend

| VI | EN | File:# |
| --- | --- | --- |
| Gắn 4 endpoint | Mount 4 endpoints | `server/routes/recipes.js` `#6-9` |
| Trả lưới nguyên liệu | Return ingredient groups | `recipeController.js` `#5-7` |
| Khớp món (POST) | Match recipes (POST) | `recipeController.js` `#9-18` |
| Validate body `string[]` → 400 | Validate body `string[]` → 400 | `recipeController.js` `#12-16` |
| Lấy 1 món / 404 | Get one recipe / 404 | `recipeController.js` `#21-27` |
| Lọc danh sách + 400 query | List + filter + 400 query | `recipeController.js` `#29-48` |
| Bỏ dấu tiếng Việt + lowercase | Strip Vietnamese diacritics + lowercase | `matchService.js` `#1-15` |
| Đồng nghĩa (cà chua, hành lá, chickpeas, mayo, …) | Synonyms | `matchService.js` `#17-47` |
| Công thức điểm | Scoring formula | `matchService.js` `#54-68` |
| Lọc 0% + sort | Drop 0% + sort | `matchService.js` `#70-79` |
| Lọc cuisine / time / difficulty | Filter cuisine / time / difficulty | `matchService.js` `#81-90` |
| 404 API lạ | Unknown API 404 | `server/app.js` `#19-21` |
| Middleware lỗi 500 | Error middleware 500 | `server/app.js` `#23-27` |
| Serve mock JSON | Serve mock JSON | `server/app.js` `#13-15` |

### Công thức khớp / Match formula

```text
required = ingredients where optional === false
coverage = have.length / required.length
matchPercent = round(coverage * 100)
score = coverage * 100 - missing.length * 5
```

`server/services/matchService.js` `#54-68`.

POST không GET: danh sách nguyên liệu có thể dài và có dấu → query string xấu và dễ vỡ giới hạn URL.

---

## Frontend map / Frontend

| VI | EN | File:# |
| --- | --- | --- |
| Cờ mock | Mock flag | `api.js` `#1` |
| Bọc fetch + báo lỗi | Fetch wrapper + errors | `api.js` `#69-81` |
| GET nguyên liệu | GET ingredients | `api.js` `#87-93` |
| POST khớp / mock khớp | POST match / local match | `api.js` `#95-105`, `#44-67` |
| GET 1 món | GET one recipe | `api.js` `#107-115` |
| GET list + query | GET list + query | `api.js` `#117-134` |
| Toast | Toast | `api.js` `#136-146` |
| Đọc/ghi localStorage | Read/write localStorage | `api.js` `#148-159` |
| Export `window.FridgeChef` | Export `window.FridgeChef` | `api.js` `#161-171` |
| Đọc checkbox đã tick | Read checked boxes | `ingredients.js` `#4-7` |
| Đếm + lưu + class checked | Count + save + checked class | `ingredients.js` `#10-18` |
| Vẽ fieldset theo nhóm | Render fieldset groups | `ingredients.js` `#21-63` |
| Nút Find recipes | Find recipes button | `ingredients.js` `#75-102` |
| Random recipes (tick required + match) | Random recipes (tick required + match) | `ingredients.js` `#68-73`, `#104-130` |
| Modal xóa hết | Clear-all modal | `ingredients.js` `#148-161` |
| Lọc client | Client filters | `results.js` `#22-33` |
| Skeleton | Skeleton | `results.js` `#44-53` |
| Render card + `--percent` + surprise pick | Render cards + featured | `results.js` `#55-129` |
| Browse all | Browse all | `results.js` `#138-153` |
| CustomEvent match-* | CustomEvent match-* | `results.js` `#155-179` |
| `URLSearchParams` id | `URLSearchParams` id | `recipe-detail.js` `#54-55` |
| Tô có / thiếu | Color have / missing | `recipe-detail.js` `#10-37` |
| Render bước nấu | Render steps | `recipe-detail.js` `#40-47` |

IIFE: `ingredients.js` `#1`+`#163`, `results.js` `#1`+`#199`, `recipe-detail.js` `#1`+`#92` — tránh hai file khai báo `const api` trên global.

---

## HTML / CSS

| VI | EN | File:# |
| --- | --- | --- |
| Semantic header/main/section | Semantic header/main/section | `index.html` `#24-59` |
| Nút Find recipes | Find recipes button | `index.html` `#66` |
| Nút Random recipes | Random recipes button | `index.html` `#64` |
| Template card | Card template | `index.html` `#135-150` |
| Trang chi tiết 2 cột | Detail 2-column markup | `recipe.html` `#53-63` |
| Token `:root` | `:root` tokens | `base.css` `#25-89` |
| Thanh khớp `var(--percent)` | Match bar `var(--percent)` | `components.css` `#179-193` |
| Grid card 260px | Card grid 260px | `layout.css` `#148-152` |
| Grid nguyên liệu 140px | Ingredient grid 140px | `layout.css` `#124-129` |
| Sticky cột nguyên liệu 1024px | Sticky ingredient column 1024px | `layout.css` `#224-234` |

---

## Data rules / Quy tắc dữ liệu

- Recipe fields (do not rename): `id`, `name`, `image`, `cookTime`, `difficulty` (`easy|medium|hard`), `cuisine`, `servings`, `ingredients[{name,quantity,optional}]`, `steps[]`.
- Checkbox `value` must equal `ingredients[].name` after `normalize()`.
- Each recipe `steps[]` is a cook-along list (heat, time, doneness). Updated 2026-09-19 so steps are not one-line summaries.
- `image` = TheMealDB photograph URL (`https://www.themealdb.com/images/media/meals/...jpg`).
- Extra catalog items (2026-09-19): lamb, turkey, duck, squid, kale, asparagus, green bean, peas, cauliflower, sweet potato, leek, chickpea, dill, oregano, feta, parmesan, quinoa, couscous, mayonnaise, mustard, oyster sauce, chili sauce, lentil, black bean, paprika, cumin, orange, strawberry, coconut.
- New dishes that use those items: Lamb Tagine `r031`, Garlic Thyme Lentils `r032`, Chickpea Fajitas `r033`, Split Pea Soup `r034`, Falafel Pita `r035`, Cauliflower Stir Fry `r036` in `server/data/recipes.json`.

---

## Recipe photos / Ảnh món ăn

Ảnh lấy từ TheMealDB (`strMealThumb`).

| VI | EN | File:# |
| --- | --- | --- |
| URL ảnh trên từng món | Photo URL on each recipe | `server/data/recipes.json` field `image` |
| Gắn `src` trên card | Set card `src` | `public/js/results.js` `#95-98` |
| Gắn `src` trang chi tiết | Set detail `src` | `public/js/recipe-detail.js` `#70-74` |
| Lazy + no-referrer | Lazy + no-referrer | `public/index.html` `#148`, `public/recipe.html` `#52` |
| Từ đồng nghĩa mới | New synonym keys | `matchService.js` `#17-47` (`chickpeas`→`chickpea`, `mayo`→`mayonnaise`, `calamari`→`squid`, …) |

---

## Out of scope / Không làm (spec 4 tuần)

Autocomplete, database, favorites, shopping list, autocannon benchmark.
