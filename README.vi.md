[English](README.md) | [Tiếng Việt](README.vi.md)

# FridgeChef

<img src="public/favicon.svg" alt="Logo mũ đầu bếp FridgeChef" width="48" height="48">

Tick nguyên liệu đang có. FridgeChef xếp món theo mức khớp, liệt kê phần còn thiếu, rồi mở trang nấu với từng bước cụ thể.

Hợp đồng API nằm ở [`API.md`](API.md). Bản đồ chức năng → file nằm ở [`NOTES.md`](NOTES.md).

## Ứng dụng làm gì

| Phần | Chi tiết |
| --- | --- |
| Giao diện Board | Thanh điều hướng, cột nguyên liệu, khu kết quả. Đã chốt; bốn bản nháp UI local không còn. |
| Logo | Mũ đầu bếp trong `public/favicon.svg`, không dùng chữ `FC`. |
| Tủ lạnh | 105 nguyên liệu, 8 nhóm, chỉ checkbox (không ô gõ). |
| Khớp món | Điểm = độ phủ × 100 − số nguyên liệu thiếu × 5. Món 0% bị loại. |
| Random recipes | Chọn một món, tick nguyên liệu bắt buộc, rồi khớp. |
| Recently viewed | 12 món vừa mở trên trình duyệt này (khóa `localStorage` `fridgechef.recentIds`). |
| Bước nấu | Nhiệt, thời gian, cách cắt, dấu hiệu chín trong `recipes.json`. |
| Ảnh | Ảnh HTTPS TheMealDB. |

Spec 4 tuần không gồm database, món yêu thích, danh sách mua, autocomplete và React/Tailwind.

## Yêu cầu

- Node.js 18 trở lên
- Một trình duyệt

Không có database. Công thức và nguyên liệu nằm trong file JSON.

## Chạy trên máy

```bash
npm install
npm start
```

Mở [http://localhost:3000](http://localhost:3000).

| Lệnh | Việc lệnh làm |
| --- | --- |
| `npm start` | Express cổng 3000 |
| `npm run dev` | Giống trên, nodemon reload thư mục `server/` |

Muốn bỏ qua API Express trên trình duyệt, đặt `USE_MOCK = true` ở dòng 1 của `public/js/api.js`. Client lúc đó gọi `GET /mock-data.json`.

## Cách dùng

1. Trong **My Fridge**, tick nguyên liệu, hoặc bấm **Random recipes** để điền đủ một món.
2. Bấm **Find recipes**. Thẻ 100% nghĩa là đã tick hết nguyên liệu bắt buộc.
3. Mở thẻ. Xanh = đã có, cam = còn thiếu, mờ = optional.
4. **Recently viewed recipes** trên thanh trái liệt kê món đã mở trên máy này. Mở `recipe.html?id=…` sẽ đẩy id đó lên đầu (tối đa 12).

Còn khóa `fridgechef.ingredients` (các tick). Trong `sessionStorage`: `fridgechef.featuredId` (món random đang nổi) và `fridgechef.lastRandomId` (tránh trùng món random vừa rồi).

## Trang

| URL | File | Việc trang làm |
| --- | --- | --- |
| `/` | `public/index.html` | Tủ lạnh, khớp món, món vừa xem |
| `/recipe.html?id=r001` | `public/recipe.html` | Một món |
| `/styleguide.html` | `public/styleguide.html` | Nút, thẻ, toast, modal |

Giao diện tiếng Anh. Phần khớp vẫn đọc dấu tiếng Việt (`Cà Chua` → tomato) nhờ `normalize` và `SYNONYMS` trong `matchService.js`.

## Kiến trúc

```text
Trình duyệt (HTML/CSS/JS thuần)
    └── Fetch ──► Express (server/app.js)
                      ├── GET  /api/ingredients
                      ├── POST /api/recipes/match   { ingredients: string[] }
                      ├── GET  /api/recipes/:id
                      └── GET  /api/recipes?cuisine=&maxTime=&difficulty=
```

| Tầng | Đường dẫn |
| --- | --- |
| Giao diện tĩnh | `public/` qua `express.static` |
| Route | `server/routes/recipes.js` |
| Kiểm tra | `server/controllers/recipeController.js` (400 / 404) |
| Khớp | `server/services/matchService.js` |
| Công thức | `server/data/recipes.json` — 36 món, `r001`–`r036` |
| Nguyên liệu | `server/data/ingredients.json` — 105 mục, 8 nhóm |

Ảnh là URL `strMealThumb` của TheMealDB, không phải `/img/*.jpg` local.

## Code nằm ở đâu

### Tủ lạnh và Random recipes

- Nhóm checkbox: `public/js/ingredients.js` (`renderGroups`)
- Nguyên liệu đã tick: `querySelectorAll('.ing-item input:checked')`
- Tìm món: `findRecipes()` → `POST /api/recipes/match`
- Random: `randomFridge()` → `GET /api/recipes`, tick tên bắt buộc, rồi khớp
- Nút: `public/index.html` `#random-fridge`

### Món vừa xem

- Sau khi trang chi tiết tải xong: `public/js/recipe-detail.js` gọi `api.recordRecent(id)`
- Id: `api.getRecentIds()` trong `public/js/api.js`
- Danh sách: `public/js/results.js` (`renderRecent`)
- Nav: `public/index.html` `#recent`, `public/recipe.html` `/#recent`

### Công thức điểm

```
required = nguyên liệu có optional === false
coverage = số đã có / required.length
score    = coverage * 100 - missing.length * 5
```

Server: `matchRecipe` trong `server/services/matchService.js`. Bản mock trên trình duyệt: `matchRecipesLocal` trong `public/js/api.js`.

### Layout Board

- Thanh nav: 12.25rem (trước là 15.5rem)
- Cột nguyên liệu: 16.75rem (trước tối đa 22rem)
- Màu `#0d9488`, Plus Jakarta Sans: `public/css/ui-drafts.css` dưới `html[data-ui="board"]`

## Kiểm tra API

```bash
curl http://localhost:3000/api/ingredients
curl -X POST http://localhost:3000/api/recipes/match -H "Content-Type: application/json" -d "{\"ingredients\":[\"egg\",\"tomato\",\"onion\",\"garlic\"]}"
curl http://localhost:3000/api/recipes/r001
```

Body khớp ở trên phải cho Tomato Fried Eggs **100%**. Id không có → 404. Body match không phải `string[]` → 400.

## Cây thư mục

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
