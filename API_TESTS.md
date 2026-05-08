# API tests — curl examples

Base URL: `http://localhost:5000/api`

After seeding, demo accounts are:
- **Admin** — `admin@ebook.dev` / `admin123`
- **User**  — `user@ebook.dev`  / `user1234`

---

## 1. Health check

```bash
curl http://localhost:5000/api/health
```

```json
{ "status": "ok", "time": "2025-01-01T..." }
```

---

## 2. Auth

### Register

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name":  "Jane Reader",
    "email": "jane@example.com",
    "password": "secret123"
  }'
```

Response includes a JWT — save it:

```bash
TOKEN=$(curl -s -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@ebook.dev","password":"admin123"}' | jq -r .token)
echo $TOKEN
```

### Login

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@ebook.dev","password":"admin123"}'
```

### Me

```bash
curl http://localhost:5000/api/auth/me \
  -H "Authorization: Bearer $TOKEN"
```

---

## 3. Ebooks — read

### List all

```bash
curl http://localhost:5000/api/ebooks
```

### Search

```bash
curl "http://localhost:5000/api/ebooks?search=lean"
```

### Filter by category + free only, sort by downloads

```bash
curl "http://localhost:5000/api/ebooks?category=business&type=free&sort=downloads&limit=8"
```

### Sort top-rated

```bash
curl "http://localhost:5000/api/ebooks?sort=rating&limit=5"
```

### Single ebook

```bash
curl http://localhost:5000/api/ebooks/<ID>
```

### Related (4)

```bash
curl http://localhost:5000/api/ebooks/<ID>/related
```

### Download free ebook

```bash
curl -OJ http://localhost:5000/api/ebooks/<ID>/download
```

### Download paid ebook (without auth → 402)

```bash
curl -i http://localhost:5000/api/ebooks/<PAID_ID>/download
# HTTP/1.1 401 — Login required to download paid ebooks
```

### Download paid ebook as admin

```bash
curl -OJ http://localhost:5000/api/ebooks/<PAID_ID>/download \
  -H "Authorization: Bearer $TOKEN"
```

---

## 4. Ebooks — admin

### Create (multipart)

```bash
curl -X POST http://localhost:5000/api/ebooks \
  -H "Authorization: Bearer $TOKEN" \
  -F "title=Quiet Mornings" \
  -F "author=A. Reader" \
  -F "description=Essays on slow productivity." \
  -F "category=business" \
  -F "price=0" \
  -F "pages=128" \
  -F "language=English" \
  -F "file=@/path/to/book.pdf;type=application/pdf" \
  -F "cover=@/path/to/cover.jpg;type=image/jpeg"
```

### Update

```bash
curl -X PUT http://localhost:5000/api/ebooks/<ID> \
  -H "Authorization: Bearer $TOKEN" \
  -F "price=9.99" \
  -F "rating=4.6"
```

(You can include `file` / `cover` fields to replace files.)

### Delete

```bash
curl -X DELETE http://localhost:5000/api/ebooks/<ID> \
  -H "Authorization: Bearer $TOKEN"
```

---

## 5. Categories

### List (with counts)

```bash
curl http://localhost:5000/api/categories
```

### Create (admin)

```bash
curl -X POST http://localhost:5000/api/categories \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"science","label":"Science","icon":"🔬"}'
```

---

## 6. Postman collection (paste-ready)

Save the JSON below as `folio.postman_collection.json` and import.

```json
{
  "info": { "name": "Folio Ebook Marketplace", "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json" },
  "variable": [
    { "key": "baseUrl", "value": "http://localhost:5000/api" },
    { "key": "token", "value": "" }
  ],
  "item": [
    {
      "name": "Auth: Login (admin)",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/auth/login",
        "header": [{ "key": "Content-Type", "value": "application/json" }],
        "body": { "mode": "raw", "raw": "{\"email\":\"admin@ebook.dev\",\"password\":\"admin123\"}" }
      }
    },
    { "name": "Ebooks: List",     "request": { "method": "GET", "url": "{{baseUrl}}/ebooks" } },
    { "name": "Ebooks: Search",   "request": { "method": "GET", "url": "{{baseUrl}}/ebooks?search=design&type=paid&sort=rating" } },
    { "name": "Ebooks: One",      "request": { "method": "GET", "url": "{{baseUrl}}/ebooks/REPLACE_ID" } },
    { "name": "Ebooks: Related",  "request": { "method": "GET", "url": "{{baseUrl}}/ebooks/REPLACE_ID/related" } },
    { "name": "Categories: List", "request": { "method": "GET", "url": "{{baseUrl}}/categories" } },
    {
      "name": "Ebooks: Create (admin)",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/ebooks",
        "header": [{ "key": "Authorization", "value": "Bearer {{token}}" }],
        "body": {
          "mode": "formdata",
          "formdata": [
            { "key": "title",       "value": "My Ebook" },
            { "key": "author",      "value": "Me" },
            { "key": "category",    "value": "business" },
            { "key": "price",       "value": "0" },
            { "key": "description", "value": "Sample" },
            { "key": "file",        "type": "file" },
            { "key": "cover",       "type": "file" }
          ]
        }
      }
    }
  ]
}
```
