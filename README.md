# Python Software Engineering Project: Web Content Extractor

(Project specification confirmed) (Beginner–Intermediate)

We will build a complete Python application called **BookScout – Web Content Extractor and Catalog Manager**.

The application will visit the Books to Scrape website, retrieve book information, extract and clean the data, organize it into Python dictionaries, and save it in four different storage formats:

1. JSON – structured data exchange.
2. TXT – human-readable reports.
3. CSV – spreadsheet-compatible data.
4. SQLite – relational database storage.

The project covers software engineering concepts including modular design, reusable functions, validation, error handling, testing, HTTP requests, HTML parsing, data cleaning, persistence, and command-line application design.

## Project structure

```text
BookScout/
├── main.py
├── config.py
├── requirements.txt
├── README.md
├── INSTRUCTOR_NOTES.md
├── STUDENT_EXERCISES.md
├── scraper/
│   ├── __init__.py
│   ├── fetcher.py
│   ├── parser.py
│   └── cleaner.py
├── storage/
│   ├── __init__.py
│   └── file_handler.py
├── tests/
│   ├── __init__.py
│   ├── test_cleaner.py
│   └── test_parser.py
└── output/
    ├── books.json
    ├── books.txt
    ├── books.csv
    └── books.db
```

## Learning objectives

Students learn to:

- Explain HTTP communication.
- Understand HTML and web scraping.
- Send HTTP GET requests using Requests.
- Parse HTML using BeautifulSoup.
- Find elements using tags, classes, and attributes.
- Extract and clean web data.
- Work with Python lists and dictionaries.
- Organize applications into modules.
- Save data as JSON, TXT, CSV, and SQLite.
- Handle exceptions and unexpected responses.
- Write unit and integration tests.

## Functional requirements

- Accept the number of pages to extract.
- Retrieve HTML using HTTP GET.
- Extract titles, prices, ratings, availability, URLs, and other available fields.
- Clean and validate data.
- Detect duplicates.
- Save JSON, TXT, CSV, and SQLite outputs.
- Display extraction statistics.

## Development steps

### 1. Environment

Use Python 3 and Visual Studio Code. Create a virtual environment:

```powershell
mkdir BookScout
cd BookScout
py -m venv .venv
.venv\Scripts\Activate.ps1
```

### 2. Dependencies

`requirements.txt`:

```text
requests>=2.31.0
beautifulsoup4>=4.12.0
```

Install:

```bash
python -m pip install -r requirements.txt
```

### 3. Website structure

Target website:

https://books.toscrape.com/

Important HTML elements include:

- `article.product_pod` — individual book.
- `h3 a` — book title and link.
- `title` attribute — full title.
- `p.price_color` — price.
- `p.star-rating` — rating.
- `p.instock.availability` — availability.
- `img` — image URL.

### 4. HTTP fetching

`config.py`:

```python
BASE_URL = "https://books.toscrape.com/"
REQUEST_TIMEOUT = 15
REQUEST_DELAY_SECONDS = 1.0
USER_AGENT = "BookScoutClassProject/1.0 (educational web scraping)"
```

`scraper/fetcher.py`:

```python
import time
import requests

from config import (
    REQUEST_TIMEOUT,
    REQUEST_DELAY_SECONDS,
    USER_AGENT
)

_SESSION = requests.Session()

_SESSION.headers.update({
    "User-Agent": USER_AGENT
})

def fetch_page(url):
    response = _SESSION.get(
        url,
        timeout=REQUEST_TIMEOUT
    )

    response.raise_for_status()
    time.sleep(REQUEST_DELAY_SECONDS)

    return response.text
```

### 5. Parsing

Use BeautifulSoup:

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "html.parser")
books = soup.select("article.product_pod")
```

Example extraction:

```python
first_book = books[0]

title = first_book.select_one("h3 a").get("title")
price = first_book.select_one(".price_color").get_text(strip=True)
rating = first_book.select_one(".star-rating").get("class")
```

### 6. Cleaning

`scraper/cleaner.py`:

```python
import re
from urllib.parse import urljoin

def clean_text(value):
    if value is None:
        return ""
    return re.sub(r"\s+", " ", str(value)).strip()

def clean_price(price_text):
    cleaned = clean_text(price_text)

    numeric = re.sub(r"[^0-9.]", "", cleaned)

    if not numeric:
        raise ValueError(f"Invalid price: {price_text}")

    return round(float(numeric), 2)

def make_absolute_url(base_url, relative_url):
    return urljoin(base_url, relative_url)
```

### 7. Book dictionaries

A book is represented as a dictionary:

```python
book = {
    "title": "A Light in the Attic",
    "price": 51.77,
    "currency": "GBP",
    "rating": 3,
    "rating_label": "Three",
    "availability": "In stock",
    "in_stock": True,
    "product_url": "https://books.toscrape.com/catalogue/example",
    "image_url": "https://books.toscrape.com/media/example.jpg",
    "scraped_at": "2026-11-03T..."
}
```

A list of these dictionaries represents the collection.

### 8. Pagination

The scraper can follow the site's Next link:

```python
def find_next_page(html, current_url):
    soup = BeautifulSoup(html, "html.parser")

    next_link = soup.select_one("li.next a")

    if next_link is None:
        return None

    href = next_link.get("href")

    if not href:
        return None

    return make_absolute_url(current_url, href)
```

### 9. JSON storage

```python
import json

def save_json(books, output_dir):
    path = output_dir / "books.json"

    with path.open("w", encoding="utf-8") as file:
        json.dump(
            books,
            file,
            indent=4,
            ensure_ascii=False
        )

    return path
```

### 10. TXT storage

```python
def save_txt(books, output_dir):
    path = output_dir / "books.txt"

    with path.open("w", encoding="utf-8") as file:
        for number, book in enumerate(books, start=1):
            file.write(f"BOOK {number}\n")
            file.write(f"Title: {book['title']}\n")
            file.write(
                f"Price: {book['currency']} "
                f"{book['price']:.2f}\n"
            )
            file.write(f"Rating: {book['rating_label']}\n")
            file.write(
                f"Availability: "
                f"{book['availability']}\n"
            )
            file.write(f"URL: {book['product_url']}\n")
            file.write("-" * 50 + "\n")

    return path
```

### 11. CSV storage

```python
import csv

FIELDS = [
    "title",
    "price",
    "currency",
    "rating",
    "rating_label",
    "availability",
    "in_stock",
    "product_url",
    "image_url",
    "scraped_at"
]

def save_csv(books, output_dir):
    path = output_dir / "books.csv"

    with path.open(
        "w",
        newline="",
        encoding="utf-8-sig"
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=FIELDS
        )

        writer.writeheader()
        writer.writerows(books)

    return path
```

### 12. SQLite storage

```python
import sqlite3

def save_sqlite(books, output_dir):
    path = output_dir / "books.db"

    with sqlite3.connect(path) as connection:
        connection.execute("""
            CREATE TABLE IF NOT EXISTS books (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                price REAL NOT NULL,
                currency TEXT NOT NULL,
                rating INTEGER,
                rating_label TEXT,
                availability TEXT,
                in_stock INTEGER,
                product_url TEXT UNIQUE NOT NULL,
                image_url TEXT,
                scraped_at TEXT
            )
        """)

        connection.executemany("""
            INSERT INTO books (
                title,
                price,
                currency,
                rating,
                rating_label,
                availability,
                in_stock,
                product_url,
                image_url,
                scraped_at
            )
            VALUES (
                :title,
                :price,
                :currency,
                :rating,
                :rating_label,
                :availability,
                :in_stock,
                :product_url,
                :image_url,
                :scraped_at
            )
            ON CONFLICT(product_url)
            DO UPDATE SET
                title = excluded.title,
                price = excluded.price,
                currency = excluded.currency,
                rating = excluded.rating,
                rating_label = excluded.rating_label,
                availability = excluded.availability,
                in_stock = excluded.in_stock,
                image_url = excluded.image_url,
                scraped_at = excluded.scraped_at
        """, books)

    return path
```

### 13. Save all formats

```python
def save_all(books, output_dir):
    output_dir.mkdir(
        parents=True,
        exist_ok=True
    )

    save_json(books, output_dir)
    save_txt(books, output_dir)
    save_csv(books, output_dir)
    save_sqlite(books, output_dir)

    print("All files saved successfully!")
```

### 14. Main application

The main controller coordinates fetching, parsing, pagination, duplicate detection, and storage.

```python
from pathlib import Path

from scraper.fetcher import fetch_page
from scraper.parser import parse_books, find_next_page
from storage.file_handler import save_all

def scrape_books(start_url, page_limit):
    all_books = []
    visited_urls = set()
    current_url = start_url

    for page_number in range(1, page_limit + 1):
        if not current_url:
            break

        if current_url in visited_urls:
            break

        visited_urls.add(current_url)

        print(f"Scraping page {page_number}")

        html = fetch_page(current_url)
        page_books = parse_books(html, current_url)

        all_books.extend(page_books)

        current_url = find_next_page(
            html,
            current_url
        )

    unique_books = []
    seen = set()

    for book in all_books:
        if book["product_url"] not in seen:
            seen.add(book["product_url"])
            unique_books.append(book)

    return unique_books

def main():
    url = "https://books.toscrape.com/"

    books = scrape_books(url, page_limit=3)

    if books:
        save_all(books, Path("output"))
        print(f"Extracted {len(books)} books.")

if __name__ == "__main__":
    main()
```

The final version can add `argparse`, allowing commands such as:

```bash
python main.py --pages 3
```

or:

```bash
python main.py --pages 5 --output-dir my_books
```

### Testing

Run:

```bash
python -m unittest discover -s tests -v
```

Tests should cover:

- Text cleaning.
- Price conversion.
- HTML parsing.
- Pagination.
- Missing fields.
- Duplicate handling.
- Storage operations.
- Integration of fetching, parsing, and saving.

### Suggested exercises

#### Beginner

1. Print the HTTP status code.
2. Extract the website title.
3. Count books on the first page.
4. Extract five titles.
5. Extract prices.
6. Create one book dictionary.
7. Create a list of dictionaries.
8. Save the list as JSON.

#### Intermediate

1. Extract five pages.
2. Display four- and five-star books.
3. Find the cheapest book.
4. Calculate average price.
5. Detect duplicates.
6. Add a minimum-rating CLI option.
7. Add statistics.
8. Query SQLite for five-star books.

#### Advanced

1. Visit individual book pages.
2. Extract descriptions.
3. Extract categories.
4. Extract UPC codes.
5. Extract stock quantities.
6. Add fields to all storage formats.
7. Build title/rating/price search.

### Suggested teaching schedule

| Session | Topic | Deliverable |
|---|---|---|
| 1 | HTTP, HTML and web scraping | HTML retrieval |
| 2 | Requests and BeautifulSoup | First extracted book |
| 3 | Cleaning and dictionaries | Structured records |
| 4 | Pagination and modular programming | Multi-page scraper |
| 5 | JSON, TXT, CSV and SQLite | Four output formats |
| 6 | Testing, debugging and presentation | Completed project |

### Assessment rubric

| Area | Marks |
|---|---:|
| HTTP requests and response handling | 15 |
| HTML parsing and extraction | 20 |
| Data cleaning | 15 |
| Python dictionaries and lists | 10 |
| File and database storage | 20 |
| Pagination and duplicate handling | 10 |
| Testing and presentation | 10 |
| **Total** | **100** |

### Recommended next steps

After the initial project:

1. Add book-detail extraction for descriptions, categories, and UPCs.
2. Add SQLite search and filtering.
3. Build a Streamlit interface.
4. Add more automated tests.
5. Improve production concerns such as logging, retries, configuration, and use of `Decimal` for financial values.

---

## User
create a dowloadable md file for the entire chat history
