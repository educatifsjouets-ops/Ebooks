# 📚 Folio — Ebook Marketplace MVP

A production-ready MVP for an ebook marketplace where users can browse, preview, and download ebooks (free or paid), and admins can manage the catalogue. Built to evolve into a multi-vendor SaaS platform.

> **Stack:** Next.js 14 (App Router) · Tailwind CSS · Node.js · Express · MongoDB · JWT · Multer

---

## ✨ Features

### MVP (shipped)

- 🔐 **Auth** — JWT-based register / login / logout, role system (`user` / `admin` / `seller` reserved)
- 📖 **Ebook CRUD** — title, author, description, category, price, PDF, cover, pages, language, rating, downloads
- 🔎 **Search & filter** — by title/author, category, free vs paid, sort by newest / most-downloaded / top-rated, paginated
- 🧭 **Categories API** — with live ebook counts
- 🤝 **Related engine** — 4 ebooks from same category, sorted by rating + downloads
- ⬇️ **Downloads** — direct download for free ebooks; paid ebooks gated and ready for Stripe in phase 2
- 🛠 **Admin dashboard** — modal form, upload PDF + cover, edit, delete
- 🎨 **Editorial UI** — Fraunces + Plus Jakarta Sans, warm cream/terracotta palette, responsive mobile-first, smooth animations

### Reserved for Phase 2 (architecture in place)

- 💳 Stripe checkout (paid downloads simulate the flow today)
- 🛍 Multi-vendor (Ebook model already has a `seller` field, User has a `seller` role)
- 📊 Seller dashboard / analytics
- 🔁 Subscriptions
- 🛡 DRM / signed URLs

---

## 📁 Project structure

```
ebook-marketplace/
├── backend/                  # Express API
│   ├── config/db.js          # Mongo connection
│   ├── models/               # User, Ebook, Category (Mongoose)
│   ├── middleware/           # auth (JWT, adminOnly), upload (multer), errorHandler
│   ├── controllers/          # auth, ebook, category logic
│   ├── routes/               # /api/auth, /api/ebooks, /api/categories
│   ├── utils/seed.js         # seed admin + 10 ebooks + categories
│   ├── uploads/              # local file storage (pdfs/, covers/)
│   ├── server.js             # entry point
│   └── package.json
│
└── frontend/                 # Next.js 14 App Router
    ├── app/
    │   ├── layout.js         # fonts, AuthProvider, Navbar/Footer
    │   ├── page.js           # home
    │   ├── ebooks/           # listing
    │   ├── ebook/[id]/       # detail page
    │   ├── category/[name]/  # category page
    │   ├── login/, register/, admin/
    │   └── globals.css
    ├── components/           # Navbar, Footer, EbookCard, FilterBar
    ├── context/AuthContext.js
    ├── lib/api.js            # axios client + JWT interceptor
    └── package.json
```

---

## 🚀 Quick start

### Prerequisites

- **Node.js** ≥ 18
- **MongoDB** running locally (`mongodb://localhost:27017`) — or a MongoDB Atlas connection string

### 1. Backend

```bash
cd backend
cp .env.example .env             # then edit .env if needed
npm install
npm run seed                     # creates admin, demo user, categories, 10 ebooks
npm run dev                      # starts API on http://localhost:5000
```

After seeding you'll see:

```
👤 Admin: admin@ebook.dev / admin123
👤 User:  user@ebook.dev / user1234
```

### 2. Frontend

In a **second terminal**:

```bash
cd frontend
cp .env.local.example .env.local
npm install
npm run dev                      # starts UI on http://localhost:3000
```

Open **http://localhost:3000** and log in as the admin to upload ebooks at `/admin`.

---

## 🔧 Environment variables

### `backend/.env`

| Variable         | Example                                      | Purpose                       |
|------------------|----------------------------------------------|-------------------------------|
| `PORT`           | `5000`                                       | API port                      |
| `MONGO_URI`      | `mongodb://localhost:27017/ebook_marketplace` | Mongo connection              |
| `JWT_SECRET`     | `a_long_random_string`                       | JWT signing secret            |
| `JWT_EXPIRES_IN` | `7d`                                         | Token lifetime                |
| `NODE_ENV`       | `development`                                | Toggles morgan + error stacks |
| `CLIENT_URL`     | `http://localhost:3000`                      | CORS origin                   |

### `frontend/.env.local`

| Variable               | Example                  | Purpose                     |
|------------------------|--------------------------|-----------------------------|
| `NEXT_PUBLIC_API_URL`  | `http://localhost:5000`  | Backend base URL (browser)  |

---

## 📡 API reference

Base URL: `http://localhost:5000/api`

| Method | Endpoint                       | Auth          | Description                    |
|--------|--------------------------------|---------------|--------------------------------|
| POST   | `/auth/register`               | —             | Create account                 |
| POST   | `/auth/login`                  | —             | Get JWT                        |
| GET    | `/auth/me`                     | Bearer        | Current user                   |
| GET    | `/ebooks`                      | —             | List + search + filter + sort  |
| GET    | `/ebooks/:id`                  | —             | Single ebook                   |
| GET    | `/ebooks/:id/related`          | —             | 4 related ebooks               |
| GET    | `/ebooks/:id/download`         | Optional      | Download PDF (paid → gated)    |
| POST   | `/ebooks`                      | Admin         | Create (multipart: file+cover) |
| PUT    | `/ebooks/:id`                  | Admin         | Update                         |
| DELETE | `/ebooks/:id`                  | Admin         | Delete                         |
| GET    | `/categories`                  | —             | List with counts               |
| POST   | `/categories`                  | Admin         | Create                         |
| GET    | `/health`                      | —             | Health check                   |

**Query params for `GET /ebooks`:**
`search`, `category`, `type` (`free`|`paid`), `sort` (`newest`|`downloads`|`rating`), `page`, `limit` (max 50).

See **[API_TESTS.md](./API_TESTS.md)** for full curl examples.

---

## 🗄 Data model (Ebook)

```js
{
  title:          String,        // required
  author:         String,        // required
  description:    String,
  category:       String,        // lowercase, indexed
  price:          Number,        // 0 = free
  fileUrl:        String,        // /uploads/pdfs/xxx.pdf
  coverImage:     String,        // /uploads/covers/xxx.jpg
  pages:          Number,
  language:       String,
  rating:         Number,        // 0–5
  downloadsCount: Number,
  seller:         ObjectId,      // reserved for phase 2
  isPublished:    Boolean,
  createdAt:      Date,
  updatedAt:      Date,
}
```

---

## 🔐 Security notes

- **Inputs** validated with `express-validator` on auth routes.
- **Admin routes** protected by `protect` + `adminOnly` middleware.
- **File uploads** restricted by mimetype:
  - `file` field → `application/pdf` only
  - `cover` field → `image/jpeg`, `image/png`, `image/webp` only
  - 50 MB per file limit
- **Passwords** hashed with bcrypt (10 rounds).
- **JWT** carried as `Authorization: Bearer <token>`.
- **First registered user** automatically becomes admin (bootstrap); after that, all registrations default to `user`. Promote others via direct DB edit or by extending the API.

---

## 🚢 Deployment

### Frontend → Vercel

1. Push the `frontend/` folder to a Git repo.
2. Import to Vercel; set env var: `NEXT_PUBLIC_API_URL=https://your-api.example.com`.
3. Deploy.

### Backend → Railway / Render

1. Push the `backend/` folder.
2. Set env vars from `.env.example`. Use a **MongoDB Atlas** URI for `MONGO_URI`.
3. Set start command to `node server.js`.
4. **Important:** local file storage on Railway/Render is **ephemeral**. For production migrate uploads to S3 / Cloudflare R2 / Supabase Storage. Keep multer's API but swap the storage engine — no controller changes needed.

---

## 🧪 Testing the flow manually

1. Run backend + frontend.
2. Visit `/login` → sign in as `admin@ebook.dev / admin123`.
3. Visit `/admin` → click **Add ebook** → upload any PDF + image.
4. Visit `/ebooks` → confirm card appears with cover, price, rating chip.
5. Open the ebook → click **Download free PDF** → file downloads, count increments.
6. Filter by category, free/paid, change sort → URL updates and grid refreshes.

---

## 📜 License

MIT — use it, fork it, ship it.
