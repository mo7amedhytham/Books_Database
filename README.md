# 📚 Books & Authors Database

A basic **PostgreSQL**-based relational database project built on the **PEN stack** (PostgreSQL, Express, Node.js), designed to power a books-and-authors catalog as a standalone REST API. It models books, authors, genres, and publishers, along with the relationships between them, and provides sample SQL to set up, seed, and query the database.

---

## 🧱 Tech Stack (PEN)

- **P — PostgreSQL:** relational database
- **E — Express:** REST API framework
- **N — Node.js:** runtime
- **DB Driver:** [`pg`](https://node-postgres.com/) (node-postgres)
- **Language:** JavaScript (SQL for DDL/DML)

This project is API-only by design — no frontend framework is bundled. Consume the endpoints from whatever client you like (plain HTML/JS, a mobile app, Postman, curl, etc.).

---

## 📂 Project Structure

```
books-authors-db/
│
├── sql/
│   ├── schema.sql          # Table definitions (DDL)
│   ├── seed.sql            # Sample data (DML)
│   └── queries.sql         # Example/common queries
│
├── src/
│   ├── db.js                # PostgreSQL connection pool
│   ├── app.js                # Express app setup
│   ├── server.js             # Entry point (starts the server)
│   ├── routes/
│   │   ├── books.js          # /books routes
│   │   ├── authors.js        # /authors routes
│   │   └── genres.js         # /genres routes
│   └── controllers/
│       ├── booksController.js
│       ├── authorsController.js
│       └── genresController.js
│
├── docs/
│   └── er-diagram.png      # Entity-Relationship diagram (optional)
│
├── .env.example             # Environment variable template
├── package.json
└── README.md
```

---

## 🗃️ Database Schema

### Tables

**authors**
| Column      | Type         | Description              |
|-------------|--------------|---------------------------|
| author_id   | SERIAL PK    | Unique author identifier  |
| first_name  | VARCHAR(50)  | Author's first name       |
| last_name   | VARCHAR(50)  | Author's last name        |
| birth_date  | DATE         | Date of birth              |
| country     | VARCHAR(50)  | Country of origin         |

**publishers**
| Column         | Type        | Description              |
|----------------|-------------|---------------------------|
| publisher_id   | SERIAL PK   | Unique publisher ID       |
| name           | VARCHAR(100)| Publisher name            |
| founded_year   | INT         | Year founded              |

**genres**
| Column     | Type         | Description         |
|------------|--------------|----------------------|
| genre_id   | SERIAL PK    | Unique genre ID      |
| name       | VARCHAR(50)  | Genre name           |

**books**
| Column         | Type          | Description                    |
|----------------|---------------|----------------------------------|
| book_id        | SERIAL PK     | Unique book identifier          |
| title          | VARCHAR(200)  | Book title                      |
| author_id      | INT FK        | References `authors(author_id)` |
| publisher_id   | INT FK        | References `publishers(publisher_id)` |
| genre_id       | INT FK        | References `genres(genre_id)`   |
| published_date | DATE          | Publication date                |
| isbn           | VARCHAR(20)   | ISBN number                     |
| price          | NUMERIC(8,2)  | Book price                      |

### Relationships

- One **author** → many **books** (1:N)
- One **publisher** → many **books** (1:N)
- One **genre** → many **books** (1:N)

---

## ⚙️ Setup Instructions

### 1. Prerequisites
- Node.js (v18+ recommended) and npm
- PostgreSQL installed (v13+ recommended)
- `psql` CLI or a GUI tool like pgAdmin/DBeaver

### 2. Create the database

```bash
createdb -p 1530 books_authors_db
```

### 3. Run the schema

```bash
psql -p 1530 -d books_authors_db -f sql/schema.sql
```

### 4. Seed sample data

```bash
psql -p 1530 -d books_authors_db -f sql/seed.sql
```

> Replace `1530` with your actual PostgreSQL port if different — check `postgresql.conf` if unsure.

### 5. Install dependencies

```bash
npm install
```

`package.json` dependencies typically include:

```json
{
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.12.0",
    "dotenv": "^16.4.5"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  },
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  }
}
```

### 6. Configure environment variables

Copy `.env.example` to `.env` and fill in your credentials (see [Environment Variables](#-environment-variables)).

### 7. Start the server

```bash
npm run dev    # development (auto-restart)
# or
npm start      # production
```

The API will be available at `http://localhost:3000` (or whichever `PORT` you set).

---

## 🔍 Example Queries

```sql
-- All books by a specific author
SELECT b.title, a.first_name, a.last_name
FROM books b
JOIN authors a ON b.author_id = a.author_id
WHERE a.last_name = 'Rowling';

-- Count of books per genre
SELECT g.name, COUNT(b.book_id) AS total_books
FROM genres g
LEFT JOIN books b ON g.genre_id = b.genre_id
GROUP BY g.name
ORDER BY total_books DESC;

-- Books published after 2015, sorted by price
SELECT title, published_date, price
FROM books
WHERE published_date > '2015-01-01'
ORDER BY price DESC;
```

---

## 🌐 Express API

### Database connection (`src/db.js`)

```js
const { Pool } = require('pg');
require('dotenv').config();

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
});

module.exports = pool;
```

### App entry point (`src/server.js`)

```js
const app = require('./app');
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Express app (`src/app.js`)

```js
const express = require('express');
const booksRouter = require('./routes/books');
const authorsRouter = require('./routes/authors');
const genresRouter = require('./routes/genres');

const app = express();
app.use(express.json());

app.use('/books', booksRouter);
app.use('/authors', authorsRouter);
app.use('/genres', genresRouter);

module.exports = app;
```

### Sample route (`src/routes/books.js`)

```js
const express = require('express');
const router = express.Router();
const pool = require('../db');

// GET /books - list all books with author, genre, publisher
router.get('/', async (req, res) => {
  try {
    const result = await pool.query(`
      SELECT b.book_id, b.title, b.published_date, b.price,
             a.first_name, a.last_name,
             g.name AS genre, p.name AS publisher
      FROM books b
      JOIN authors a ON b.author_id = a.author_id
      JOIN genres g ON b.genre_id = g.genre_id
      JOIN publishers p ON b.publisher_id = p.publisher_id
    `);
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server error' });
  }
});

// GET /books/:id - single book detail
router.get('/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const result = await pool.query('SELECT * FROM books WHERE book_id = $1', [id]);
    if (result.rows.length === 0) return res.status(404).json({ error: 'Book not found' });
    res.json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server error' });
  }
});

module.exports = router;
```

### API Endpoints

| Method | Endpoint         | Description                          |
|--------|------------------|----------------------------------------|
| GET    | `/books`         | List all books (with joins)          |
| GET    | `/books/:id`     | Get a single book by ID              |
| POST   | `/books`         | Add a new book                       |
| PUT    | `/books/:id`     | Update a book                        |
| DELETE | `/books/:id`     | Delete a book                        |
| GET    | `/authors`       | List all authors                     |
| GET    | `/authors/:id`   | Get author details + their books     |
| GET    | `/genres`        | List all genres                      |
| GET    | `/genres/:id`    | Get books filtered by genre          |

`authors.js` and `genres.js` routers follow the same pattern as `books.js` above.

---

## 🧪 Environment Variables

```env
PORT=3000
DB_HOST=localhost
DB_PORT=1530
DB_NAME=books_authors_db
DB_USER=your_username
DB_PASSWORD=your_password
```

> Note: `1530` is a custom PostgreSQL port (the default is `5432`) — make sure it matches whatever `port =` is set to in your `postgresql.conf`.

---

## 📌 Future Improvements

- Add a `reviews` table for user ratings/reviews
- Add a `users` table for accounts and wishlists
- Add full-text search on book titles/descriptions
- Add indexes on frequently queried columns (`author_id`, `genre_id`)

---

## 📄 License

This project is open-source and available for learning/demo purposes.