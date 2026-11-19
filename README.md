# BookScout – Web Content Extractor and Catalog Manager

**BookScout** is a Python software engineering project designed to extract, clean, structure, and store book catalog data from [Books to Scrape](https://books.toscrape.com/). 

Built for beginner-to-intermediate learners, BookScout demonstrates core software engineering principles including modular design, HTTP request handling, DOM parsing, data cleaning, multi-format persistence, command-line interfaces, and unit testing.

---

## Table of Contents

* [Project Architecture](#project-architecture)
* [Features](#features)
* [Learning Objectives](#learning-objectives)
* [Installation & Setup](#installation--setup)
* [Usage](#usage)
* [Data Storage Formats](#data-storage-formats)
* [Testing](#testing)
* [Curriculum & Course Materials](#curriculum--course-materials)
* [License](#license)

---

## Project Architecture

```text
BookScout/
├── main.py                   # Main CLI entry point and orchestration controller
├── config.py                 # Application configuration (URLs, timeouts, headers)
├── requirements.txt          # Third-party dependencies
├── README.md                 # Project documentation and quickstart guide
├── INSTRUCTOR_NOTES.md       # Session schedules, teaching tips, and rubric
├── STUDENT_EXERCISES.md      # Exercises (Beginner, Intermediate, Advanced)
├── scraper/
│   ├── __init__.py
│   ├── fetcher.py            # HTTP request handling and throttling
│   ├── parser.py             # BeautifulSoup HTML parsing and link extraction
│   └── cleaner.py            # Text normalization, regex pricing, and URL resolution
├── storage/
│   ├── __init__.py
│   └── file_handler.py       # Exporters for JSON, TXT, CSV, and SQLite
├── tests/
│   ├── __init__.py
│   ├── test_cleaner.py       # Unit tests for data cleaning logic
│   └── test_parser.py        # Unit tests for HTML parsing logic
└── output/                   # Directory generated for extracted storage files
    ├── books.json
    ├── books.txt
    ├── books.csv
    └── books.db
```

---

## Features

* **Polite Web Scraping:** Uses custom headers, session pooling via `requests`, and configurable rate-limiting delays to respect target servers.
* **Resilient Parsing:** Robust HTML parsing using `BeautifulSoup4` to extract titles, ratings, prices, availability, and image links.
* **Data Cleaning & Normalization:** Strips whitespace, parses currency, standardizes data types, and resolves relative links to absolute URLs.
* **Duplicate Detection:** In-memory tracking ensures unique records using canonical product URLs.
* **Multi-Format Storage Exporter:** Concurrently exports cleaned datasets into four distinct formats:
  * **JSON** – For API and web integrations.
  * **TXT** – Human-readable catalog summaries.
  * **CSV** – UTF-8 BOM encoded files compatible with spreadsheet applications like Excel.
  * **SQLite** – Relational database storage featuring `UPSERT` capabilities for idempotent updates.

---

## Learning Objectives

Students working through the BookScout project learn to:

1. **HTTP Communication:** Work with GET requests, headers, status codes, timeouts, and sessions.
2. **HTML & DOM Navigation:** Target elements using CSS selectors, tags, and attributes.
3. **Data Manipulation:** Clean raw string inputs using regular expressions (`re`) and manipulate core Python data structures (`list`, `dict`, `set`).
4. **Modular Architecture:** Structure Python code into reusable modules and packages.
5. **Data Persistence:** Write to local files (JSON/TXT/CSV) and query relational databases (SQLite).
6. **Software Testing:** Write unit and integration tests using Python’s native `unittest` runner.

---

## Installation & Setup

### Prerequisites

* Python 3.8+ installed on your environment.

### 1. Clone & Set Up Virtual Environment

```bash
mkdir BookScout
cd BookScout
python -m venv .venv
```

Activate the virtual environment:

* **Windows (PowerShell):**
  ```powershell
  .venv\Scripts\Activate.ps1
  ```
* **Linux / macOS:**
  ```bash
  source .venv/bin/activate
  ```

### 2. Install Dependencies

```bash
python -m pip install -r requirements.txt
```

---

## Usage

### Configuration

Modify settings such as delay, user-agent, and timeouts inside `config.py`.

### Running the Application

Execute the main entry point to start the scraping and extraction pipeline:

```bash
python main.py
```

#### Command-Line Arguments (Optional CLI Extension)

```bash
# Scrape specific number of pages
python main.py --pages 5

# Custom output directory
python main.py --pages 3 --output-dir my_books
```

---

## Data Storage Formats

BookScout exports data into the `output/` directory in four structured formats:

### Schema Fields

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `title` | String | Full book title |
| `price` | Float | Numeric price formatted to 2 decimal places |
| `currency` | String | Currency code (e.g., `GBP`) |
| `rating` | Integer | Numeric rating (1–5 scale) |
| `rating_label` | String | Word label of rating (e.g., `Three`) |
| `availability` | String | Stock status string |
| `in_stock` | Boolean / Int | `True` / `1` if available |
| `product_url` | String | Absolute link to product detail page (Unique Key) |
| `image_url` | String | Absolute link to product image |
| `scraped_at` | String | ISO 8601 timestamp of extraction |

---

## Testing

Run the automated test suite using Python's built-in `unittest` discover feature:

```bash
python -m unittest discover -s tests -v
```

The unit test suite covers text cleaning, price parsing, HTML selector resilience, pagination handling, missing element safety, duplicate filtering, and file I/O operations.

---

## Curriculum & Course Materials

* **`STUDENT_EXERCISES.md`**: Contains exercises tiered across Beginner, Intermediate, and Advanced skill levels.
* **`INSTRUCTOR_NOTES.md`**: Contains a 6-session teaching syllabus, lesson plan breakdown, and 100-point assessment rubric.

---

## License

This project is licensed under the **MIT License**.

```text
MIT License

Copyright (c) 2026 BookScout Educational Project

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
