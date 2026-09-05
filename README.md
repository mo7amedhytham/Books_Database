# 📚 Books & Authors Database

A basic **PostgreSQL**-based relational database project designed to power a books-and-authors catalog website. It models books, authors, genres, and publishers, along with the relationships between them, and provides sample SQL to set up, seed, and query the database.

---

## 🧱 Tech Stack

- **Database:** PostgreSQL
- **Language:** SQL (DDL/DML)
- *(Optional/extendable)*: Node.js / Python / PHP backend + HTML/CSS/JS frontend for the "website" layer

---

## 📂 Project Structure

```
books-authors-db/
│
├── sql/
│   ├── schema.sql        # Table definitions (DDL)
│   ├── seed.sql          # Sample data (DML)
│   └── queries.sql       # Example/common queries
│
├── docs/
│   └── er-diagram.png    # Entity-Relationship diagram (optional)
│
├── .env.example           # Environment variable template
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
- PostgreSQL installed (v13+ recommended)
- `psql` CLI or a GUI tool like pgAdmin/DBeaver

### 2. Create the database

```bash
createdb books_authors_db
```

### 3. Run the schema

```bash
psql -d books_authors_db -f sql/schema.sql
```

### 4. Seed sample data

```bash
psql -d books_authors_db -f sql/seed.sql
```

### 5. Verify

```bash
psql -d books_authors_db -c "SELECT * FROM books LIMIT 5;"
```

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

## 🌐 Website Integration (Optional)

This database is designed to back a simple books-catalog website. Typical endpoints/pages might include:

- `GET /books` — list all books (with author, genre, publisher)
- `GET /books/:id` — book detail page
- `GET /authors/:id` — author profile with their books
- `GET /genres/:id` — books filtered by genre

Connect using any PostgreSQL client library (e.g., `pg` for Node.js, `psycopg2` for Python).

---

## 🧪 Environment Variables

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=books_authors_db
DB_USER=your_username
DB_PASSWORD=your_password
```

---

## 📌 Future Improvements

- Add a `reviews` table for user ratings/reviews
- Add a `users` table for accounts and wishlists
- Add full-text search on book titles/descriptions
- Add indexes on frequently queried columns (`author_id`, `genre_id`)

---

## 📄 License

This project is open-source and available for learning/demo purposes.
