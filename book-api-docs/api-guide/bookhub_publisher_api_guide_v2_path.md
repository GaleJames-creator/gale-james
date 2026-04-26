# BookHub Publisher API - v2 path-based version guide

## Document information

| Property              | Value                            |
|-----------------------|----------------------------------|
| **Version**           | 2.0.                             |
| **Last Updated**      | January 2, 2026                  |
| **API Base URL**      | `https://api.bookhub.com/api`    |
| **v1 Endpoint**       | `https://api.bookhub.com/api/v1` |
| **v2 Endpoint**       | `https://api.bookhub.com/api/v2` |
| **Versioning Method** | URI Path-Based                   |
| **Status**            | Preview                          |

---

## Table of contents

1. [Introduction](#introduction)
2. [What's new in v2](#whats-new-in-v2)
3. [Breaking changes](#breaking-changes)
4. [Migration guide](#migration-guide)
5. [Quick start](#quick-start)
6. [API versioning strategy](#api-versioning-strategy)
7. [Authentication](#authentication)
8. [API endpoints](#api-endpoints)
   - [Create book](#create-book)
   - [Get single book (with `hitCount`)](#get-single-book)
   - [Get all books (no title sorting)](#get-all-books)
   - [Finalize book](#finalize-book)
9. [Data models](#data-models)
10. [Code examples](#code-examples)
11. [Best practices](#best-practices)
12. [Support and resources](#support-and-resources)
13. [Version history](#version-history)

---

## Introduction

### About this version

Version 2 (v2) of the BookHub Publisher API uses **path-based versioning**. Unlike the header-based versioning approach used in other implementations, the version is visible in the URL (`/v2/books`).

### Why path-based versioning?

Path-based versioning offers several advantages for the BookHub Publisher API:

- **Simplicity**: The version is visible in the URL, making it easier to test and debug
- **Caching**: Standard HTTP caches work naturally with different paths
- **Browser testing**: You can test endpoints directly in a browser address bar
- **Documentation clarity**: API documentation is clearer when versions are in the path
- **Tool compatibility**: Works seamlessly with OpenAPI/Swagger tools

---

## What's new in v2

### Hit count analytics (new feature)

The most significant addition in v2 is the `hitCount` field on individual book retrievals. This provides publishers with basic visibility analytics.

#### What is `hitCount`?

The `hitCount` field tracks how many times a specific book's detail page has been accessed via the API. This metric helps publishers:

- **Gauge interest** in specific titles
- **Identify trending books** before they become bestsellers.
- **Measure marketing effectiveness** by post-campaign views.
- **Make data-driven decisions** about inventory and pricing.

#### How it works

```http
GET /v2/books/{bookId}
```

##### Response includes

```json
{
  "status": "success",
  "data": {
    "bookId": "abc123",
    "title": "The Great Adventure",
    "author": "Jane Smith",
    "hitCount": 1247,
    ...
  }
}
```

#### Important notes

- `hitCount` is **read-only** and automatically incremented.
- The count increments each time the GET endpoint is accessed.
- The initial value for new books is `0`.
- The field is **only available in v2** (not in v1).
- The count is per book and persists throughout the book’s lifetime.

---

## Breaking changes

### Title sorting removed (breaking change)

**v2 no longer supports sorting by title** on the GET all books endpoint. This is a **breaking change** that requires migration if you currently use title-based sorting.

#### Why was title sorting removed?

BookHub removed title sorting due to:

- **Performance concerns**: Text-based sorting on large datasets is computationally expensive
- **Internationalization complexity**: Different languages have different sorting rules
- **Case sensitivity issues**: Determining the proper sort order with mixed case is ambiguous
- **Limited usage**: Analytics showed less than 3% of requests used title sorting
- **Better alternatives**: Date-based sorting provides a more stable, performant option

#### What this means

In **v1**, this worked:

```http
GET /v1/books?sort=title&order=asc
```

##### ❌ Invalid request example

In **v2**, this won't work:

```http
GET /v2/books?sort=title&order=asc
```

##### Returns a `400` error

```json
{
  "status": "error",
  "message": "Invalid sort parameter",
  "errors": [{
    "field": "sort",
    "error": "INVALID_VALUE",
    "description": "v2 only supports sorting by createdDate. Use v1 for title sorting."
  }]
}
```

##### ✅ Valid request example

In **v2**, use this instead:

```http
GET /v2/books?sort=createdDate&order=desc
```

#### Available sort options in v2

The `sort` parameter now accepts **only one value**:

- `createdDate` (default: newest first)

The `order` parameter still works:

- `asc`: Ascending (oldest to newest)
- `desc`: Descending (newest to oldest, default)

---

## Migration guide

### Migrating from v1 to v2

#### Step 1: Update base URLs

##### Before (v1)

```http
https://api.bookhub.com/api/v1/books
```

##### After (v2)

```http
https://api.bookhub.com/api/v2/books
```

#### Step 2: Handle the `hitCount` field

If you're retrieving individual books, your code will now receive the `hitCount` field.

##### Python example

```python
# v1 code
response = requests.get(f"{BASE_URL}/v1/books/{book_id}")
book = response.json()['data']
# book has no hitCount field

# v2 code
response = requests.get(f"{BASE_URL}/v2/books/{book_id}")
book = response.json()['data']
hit_count = book.get('hitCount', 0)  # New field available
print(f"This book has been viewed {hit_count} times")
```

#### Step 3: Replace title sorting

If you currently use title sorting, you have two options:

##### Option A: Use date sorting (recommended)

```python
# Before (v1)
response = requests.get(f"{BASE_URL}/v1/books?sort=title&order=asc")

# After (v2) - Sort by creation date instead
response = requests.get(f"{BASE_URL}/v2/books?sort=createdDate&order=desc")
```

##### Option B: Implement client-side sorting

```python
# Fetch all books
response = requests.get(f"{BASE_URL}/v2/books?limit=100")
books = response.json()['data']['books']

# Sort by title on client side
sorted_books = sorted(books, key=lambda x: x['title'].lower())
```

#### Step 4: Update error handling

Be prepared to handle the new error when title sorting is attempted:

```python
import requests

def get_books(sort_by='createdDate', order='desc'):
    params = {'sort': sort_by, 'order': order}
    response = requests.get(f"{BASE_URL}/v2/books", params=params)
    
    if response.status_code == 400:
        error_data = response.json()
        if 'sort' in str(error_data):
            # Invalid sort parameter
            print("Error: v2 only supports sorting by createdDate")
            # Fall back to default
            return get_books(sort_by='createdDate', order=order)
    
    return response.json()
```

---

## Quick start

### Prerequisites

1. BookHub publisher account
2. API credentials (JWT token)
3. HTTPS client (curl, Postman, or programming language HTTP library)

### Basic workflow

```bash
# 1. Create a book (returns PENDING status)
curl -X POST https://api.bookhub.com/api/v2/books \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My New Book",
    "author": "John Doe",
    "description": "A new biography from a well-known author",
    "language": "en-US",
    "publisher": "My Publishing House",
    "publishedDate": "2024-03-15",
    "isbn": "978-0-1234-5678-9",
    "price": 24.99,
    "coverImages": ["https://example.com/cover.jpg"],
    "genres": ["Fiction", "Adventure"],
    "bookFormat": "Hardcover"
  }'

# 2. Get the book to see hitCount (starts at 0)
curl -X GET https://api.bookhub.com/api/v2/books/{bookId} \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# 3. List all books (sorted by creation date only)
curl -X GET https://api.bookhub.com/api/v2/books?sort=createdDate&order=desc \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# 4. Finalize the book (PENDING -> ACTIVE)
curl -X PATCH https://api.bookhub.com/api/v2/books/{bookId}/finalize \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"notes": "Ready for publication."}'
```

---

## API versioning strategy

### Path-based versioning explained

The BookHub Publisher API v2 uses **URI path versioning**, where the version number is part of the URL path:

```http
https://api.bookhub.com/api/{version}/books
```

#### Version paths

- **v1**: `https://api.bookhub.com/api/v1/books` (Legacy)
- **v2**: `https://api.bookhub.com/api/v2/books` (Current)

#### Why is it different from header-based?

Some API implementations use header-based versioning (via the `api-version` header). Path-based versioning was chosen for v2 because:

- **Simplicity**: Version is in the URL, no need to remember custom headers
- **Testability**: Easy to test in browser, curl, and Postman
- **Caching**: Standard HTTP caching works naturally
- **Documentation**: Clearer for developers to understand

#### Version support policy

- **v1**: Supported until December 31, 2026
- **v2**: Current version, recommended for all new integrations
- **Deprecation notice**: 12 months' advance notice before any version is deprecated

---

## Authentication

### JWT bearer token

All API requests require authentication using a JWT (JSON Web Token) bearer token.

#### Obtaining a token

Obtain your JWT credentials by contacting:

- Email: `api-support@bookhub.com`
- Include your publisher account information.

#### Using the token

Include the token in the `Authorization` header:

```http
Authorization: Bearer YOUR_JWT_TOKEN
```

#### Example

```bash
curl -X GET https://api.bookhub.com/api/v2/books \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Token expiration

- Tokens are valid for 24 hours.
- Refresh tokens before expiration.
- 401 Unauthorized response indicates expired token.

---

## API endpoints

### Create book

Creates a new book with `PENDING` status. Books must be finalized before becoming visible to customers.

#### Create book endpoint

```http
POST /v2/books
```

#### Create book request example

```bash
curl -X POST "https://api.bookhub.com/api/v2/books" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000" \
  -H "X-Request-ID: 7f3d9a2c-1b4e-4f8a-9c6d-2e5f1a3b8d4e" \
  -H "X-Publisher-ID: pub_adventure_books" \
  -d '{
    "title": "The Great Adventure",
    "author": "Jane Smith",
    "description": "An epic tale of courage and discovery",
    "language": "en-US",
    "publisher": "Adventure Books Inc",
    "publishedDate": "2024-03-15",
    "isbn": "978-0-1234-5678-9",
    "price": 24.99,
    "coverImages": [
       "https://covers.bookhub.com/great-adventure-front.jpg",
       "https://covers.bookhub.com/great-adventure-back.jpg"
     ],
    "genres": [
       "Fiction",
       "Adventure"
     ],
    "bookFormat": "Hardcover"
  }'
```

> **Note**: `Idempotency-Key` is recommended for all `POST` requests to prevent duplicate book creation. `X-Request-ID` helps track requests in logs. `X-Publisher-ID` scopes the request to a specific publisher account.

#### Create book response (201 Created)

```json
{
  "status": "success",
  "message": "Book created successfully",
  "data": {
    "bookId": "abc123",
    "title": "The Great Adventure",
    "author": "Jane Smith",
    "status": "PENDING",
    "createdDate": "2024-12-25T10:30:00Z",
    ...
  }
}
```

#### Field validation

| Field           | Type.         | Required | Constraints                    |
|-----------------|---------------|----------|--------------------------------|
| `title`         | string        | Yes      | Max 255 characters             |
| `author`        | string        | Yes      | Max 255 characters             |
| `description`   | string        | Yes      | Max 5000 characters            |
| `language`      | string        | Yes      | ISO 639-1 format (e.g., en-US) |
| `publisher`     | string        | Yes      | Max 255 characters             |
| `publishedDate` | string (date) | Yes      | YYYY-MM-DD format              |
| `isbn`          | string        | Yes      | Valid ISBN-10 or ISBN-13       |
| `price`         | number        | Yes      | Minimum 0.01                   |
| `coverImages`   | array         | Yes      | 1-5 valid URLs                 |
| `genres`        | array         | Yes      | 1-5 genre strings              |
| `bookFormat`    | enum          | Yes      | Paperback, Hardcover, or eBook |

---

### Get single book

Retrieves detailed information about a specific book, **including the `hitCount` analytics field** (v2 only).

#### Get single book endpoint

```http
GET /v2/books/{bookId}
```

#### Get single book path parameters

- `bookId` (required): Unique identifier of the book

#### Get single book request example

```bash
curl -X GET "https://api.bookhub.com/api/v2/books/{bookId}" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" 
```

> **Note**: Replace `{bookId}` with the actual book ID returned in the `Create book` response (e.g., `abc123`).

#### Get single book response (200 OK)

```json
{
  "status": "success",
  "data": {
    "bookId": "abc123",
    "title": "The Great Adventure",
    "author": "Jane Smith",
    "description": "An epic tale of courage and discovery",
    "language": "en-US",
    "publisher": "Adventure Books Inc",
    "publishedDate": "2024-03-15",
    "isbn": "978-0-1234-5678-9",
    "price": 24.99,
    "status": "ACTIVE",
    "coverImages": [
      "https://covers.bookhub.com/great-adventure-front.jpg"
    ],
    "genres": [
      "Fiction",
      "Adventure"
    ],
    "bookFormat": "Hardcover",
    "createdDate": "2024-12-25T10:30:00Z",
    "updatedDate": "2024-12-26T15:45:00Z",
    "hitCount": 1247
  }
}
```

#### `hitCount` field explained

The `hitCount` field is **new in v2** and provides basic analytics:

- **Type**: Integer (minimum 0)
- **Purpose**: Tracks how many times the book detail page has been accessed
- **Updates**: Automatically incremented each time this endpoint is called
- **Initial value**: 0 for newly created books
- **Read-only**: Cannot be modified directly via API

#### Use cases for `hitCount`

- **Trending analysis**: Identify which books are getting the most attention
  
   ```python
   # Get the top 10 most-viewed books
   all_books = fetch_all_books()
   trending = sorted(all_books, key=lambda x: x['hitCount'], reverse=True)[:10]
   ```

- **Marketing ROI**: Compare hit counts before and after marketing campaigns
  
   ```python
   before_campaign = get_book('abc123')['hitCount']
   # Run marketing campaign
   after_campaign = get_book('abc123')['hitCount']
   increase = after_campaign - before_campaign
   print(f"Campaign generated {increase} additional views")
   ```

- **Inventory planning**: Prioritize books with high view counts for restocking
  
   ```python
   if book['hitCount'] > 1000 and book['status'] == 'ACTIVE':
       notify_warehouse_to_restock(book['bookId'])
   ```

#### Response headers

```bash
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Cache-Control: max-age=3600, must-revalidate
```

---

### Get all books

Retrieves a paginated list of books.

> **Note**: Title sorting is not supported. Only the `createdDate` sorting is available for the `sort` parameter.

#### Get all books endpoint

```http
GET /v2/books
```

#### Query parameters

| Parameter | Type    | Required | Default     | Description                             |
|-----------|---------|----------|-------------|-----------------------------------------|
| `page`.   | integer | No       | 1           | Page number (minimum 1).                |
| `limit`   | integer | No       | 20          | Items per page (max 100).               |
| `status`  | enum    | No.      | All         | Filter: `PENDING`, `ACTIVE`, `INACTIVE` |
| `sort`    | enum    | No       | createdDate | **v2 only accepts `createdDate`**.      |
| `order`   | enum    | No       | desc        | Sort order: asc or desc                 |

#### ✅ Valid get all books request examples

##### Sort by creation date, newest first (default)

```http
GET /v2/books?sort=createdDate&order=desc
```

##### Sort by creation date, oldest first

```http
GET /v2/books?sort=createdDate&order=asc
```

##### No sort parameter (uses default: createdDate desc)

```http
GET /v2/books
```

#### ❌ Invalid get all books request example

```http
GET /v2/books?sort=title&order=asc
```

Returns a `400` error:

```json
{
  "status": "error",
  "message": "Invalid sort parameter",
  "errors": [{
    "field": "sort",
    "error": "INVALID_VALUE",
    "description": "v2 only supports sorting by createdDate. Use v1 for title sorting."
  }]
}
```

#### Get all books request example

```bash
curl -X GET "https://api.bookhub.com/api/v2/books?page=1&limit=20&status=ACTIVE&sort=createdDate&order=desc" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

#### Get all books response (200 OK)

```json
{
  "status": "success",
  "data": {
    "books": [
      {
        "bookId": "abc123",
        "title": "The Great Adventure",
        "author": "Jane Smith",
        "status": "ACTIVE",
        "price": 24.99,
        "createdDate": "2024-12-25T10:30:00Z",
        ...
      },
      {
        "bookId": "def456",
        "title": "Mystery at Midnight",
        "author": "Bob Johnson",
        "status": "ACTIVE",
        "price": 19.99,
        "createdDate": "2024-12-24T14:20:00Z",
        ...
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalItems": 87,
      "itemsPerPage": 20
    }
  }
}
```

#### Get all books response headers

```bash
X-Total-Count: 87
Link: <https://api.bookhub.com/api/v2/books?page=2>; rel="next",
      <https://api.bookhub.com/api/v2/books?page=5>; rel="last"
```

#### Client-side title sorting (alternative)

If you need books sorted by title, fetch them and sort client-side:

```python
import requests

def get_books_sorted_by_title(ascending=True):
    # Fetch all books (or paginate through all pages)
    response = requests.get(
        "https://api.bookhub.com/api/v2/books",
        params={"limit": 100},
        headers={"Authorization": f"Bearer {token}"}
    )
    books = response.json()['data']['books']
    
    # Sort by title on client side
    return sorted(
        books,
        key=lambda x: x['title'].lower(),
        reverse=not ascending
    )
```

---

### Finalize book

Changes a book's status from `PENDING` to `ACTIVE`, making it visible to customers.

#### Finalize book endpoint

```http
PATCH /v2/books/{bookId}/finalize
```

#### Finalize book path parameters

- `bookId` (required): Unique identifier of the book to finalize

#### Finalize book request example

```bash
curl https://api.bookhub.com/api/v2/books/{bookId}/finalize \
  -X PATCH \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "notes": "Reviewed and approved by editorial team"
  }
'
```

> **Note**: Replace `{bookId}` with the actual book ID returned in the `Create book` response (e.g., `abc123`).

#### Finalize book request parameters

| Parameter | Location | Type   | Required | Description                              |
| --------- | -------- | ------ | -------- | ---------------------------------------- |
| `notes`   | Body     | string | No       | A comment regarding activating the book. |

#### Finalize book response (200 OK)

```json
{
  "status": "success",
  "message": "Book finalized successfully",
  "data": {
    "bookId": "abc123",
    "title": "The Great Adventure",
    "status": "ACTIVE",
    "updatedDate": "2024-12-27T09:15:00Z"
  }
}
```

#### Business rules

- Only books with `PENDING` status can be finalized.
- Finalization is **idempotent** (calling it multiple times has the same effect).
- The book becomes available to customers immediately upon finalization.
- Cannot revert from `ACTIVE` to `PENDING` via API (contact support).

#### Finalize book error response (400 Bad Request)

If the book is not in `PENDING` status:

```json
{
  "status": "error",
  "message": "Cannot finalize book",
  "errors": [{
    "field": "status",
    "error": "INVALID_STATUS_TRANSITION",
    "description": "Book must be in PENDING status to finalize. Current status: ACTIVE"
  }]
}
```

---

## Data models

### BaseBook schema

Standard book object used across all endpoints.

```json
{
  "bookId": "string (read-only)",
  "title": "string (max 255 chars)",
  "author": "string (max 255 chars)",
  "description": "string (max 5000 chars)",
  "language": "string (ISO 639-1, e.g., en-US)",
  "publisher": "string (max 255 chars)",
  "publishedDate": "date (YYYY-MM-DD)",
  "isbn": "string (ISBN-10 or ISBN-13)",
  "price": "number (minimum 0.01)",
  "status": "enum (PENDING, ACTIVE, INACTIVE)",
  "coverImages": ["array of URLs (1-5 items)"],
  "genres": ["array of strings (1-5 items)"],
  "bookFormat": "enum (Paperback, Hardcover, eBook)",
  "createdDate": "datetime (ISO 8601, read-only)",
  "updatedDate": "datetime (ISO 8601, read-only)"
}
```

### BookWithAnalytics schema (v2 only)

Extends BaseBook with and analytics field.

```json
{
  ...BaseBook fields,
  "hitCount": "integer (minimum 0, read-only)"
}
```

The `hitCount` field is **only returned by the GET /v2/books/{bookId} endpoint**.

### Pagination schema

```json
{
  "currentPage": "integer (minimum 1)",
  "totalPages": "integer (minimum 0)",
  "totalItems": "integer (minimum 0)",
  "itemsPerPage": "integer (1-100)"
}
```

### Error response schema

```json
{
  "status": "error",
  "message": "string",
  "errors": [
    {
      "field": "string",
      "error": "string",
      "description": "string",
      "validation_context": {
        "rule": "string",
        "constraint": "object",
        "provided_value": "string"
      }
    }
  ]
}
```

---

## Code examples

### Python example: Create and track book views

```python
import requests
import time

BASE_URL = "https://api.bookhub.com/api/v2"
TOKEN = "your_jwt_token_here"

headers = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# 1. Create a new book
book_data = {
    "title": "Data-Driven Publishing",
    "author": "Sarah Analytics",
    "description": "Understanding metrics in modern publishing",
    "language": "en-US",
    "publisher": "TechBooks Publishing",
    "publishedDate": "2026-01-15",
    "isbn": "978-0-9876-5432-1",
    "price": 29.99,
    "coverImages": ["https://example.com/cover.jpg"],
    "genres": ["Business", "Technology"],
    "bookFormat": "Paperback"
}

create_response = requests.post(
    f"{BASE_URL}/books",
    headers=headers,
    json=book_data
)

if create_response.status_code == 201:
    book_id = create_response.json()['data']['bookId']
    print(f"✓ Book created: {book_id}")
    
    # 2. Check initial hit count
    detail_response = requests.get(
        f"{BASE_URL}/books/{book_id}",
        headers=headers
    )
    book = detail_response.json()['data']
    print(f"Initial hit count: {book['hitCount']}")  # Should be 1 (from this call)
    
    # 3. Finalize the book
    finalize_response = requests.patch(
        f"{BASE_URL}/books/{book_id}/finalize",
        headers=headers,
        json={"notes": "Ready for publication."}
    )
    
    if finalize_response.status_code == 200:
        print("✓ Book finalized and now ACTIVE")
        
        # 4. Simulate some views and track hit count
        for i in range(5):
            time.sleep(1)
            view_response = requests.get(
                f"{BASE_URL}/books/{book_id}",
                headers=headers
            )
            hit_count = view_response.json()['data']['hitCount']
            print(f"View #{i+2}: Hit count is now {hit_count}")

else:
    print(f"Error creating book: {create_response.status_code}")
    print(create_response.json())
```

### JavaScript example: List books with client-side title sort

```javascript
const axios = require('axios');

const BASE_URL = 'https://api.bookhub.com/api/v2';
const TOKEN = 'your_jwt_token_here';

const headers = {
  'Authorization': `Bearer ${TOKEN}`
};

async function getBooksWithTitleSort(ascending = true) {
  try {
    // Fetch books sorted by creation date (only option in v2)
    const response = await axios.get(`${BASE_URL}/books`, {
      params: {
        limit: 100,
        sort: 'createdDate',
        order: 'desc'
      },
      headers: headers
    });
    
    const books = response.data.data.books;
    console.log(`✓ Fetched ${books.length} books`);
    
    // Sort by title on client side
    const sortedBooks = books.sort((a, b) => {
      const titleA = a.title.toLowerCase();
      const titleB = b.title.toLowerCase();
      
      if (ascending) {
        return titleA.localeCompare(titleB);
      } else {
        return titleB.localeCompare(titleA);
      }
    });
    
    console.log('\nBooks sorted by title (client-side):');
    sortedBooks.slice(0, 10).forEach((book, index) => {
      console.log(`${index + 1}. ${book.title} (${book.hitCount} views)`);
    });
    
    return sortedBooks;
    
  } catch (error) {
    if (error.response && error.response.status === 400) {
      console.error('❌ Invalid request:', error.response.data.message);
    } else {
      console.error('❌ Error:', error.message);
    }
  }
}

// Run the example
getBooksWithTitleSort(true);
```

### Java example: Handle removed title sorting

```java
import java.net.http.*;
import java.net.URI;
import com.google.gson.*;

public class BookHubV2Client {
    private static final String BASE_URL = "https://api.bookhub.com/api/v2";
    private static final String TOKEN = "your_jwt_token_here";
    
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        // Build request with only a valid v2 sort parameter
        String url = BASE_URL + "/books?sort=createdDate&order=desc&limit=50";
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Authorization", "Bearer " + TOKEN)
            .GET()
            .build();
        
        HttpResponse<String> response = client.send(
            request,
            HttpResponse.BodyHandlers.ofString()
        );
        
        if (response.statusCode() == 200) {
            Gson gson = new Gson();
            JsonObject jsonResponse = gson.fromJson(
                response.body(),
                JsonObject.class
            );
            
            JsonArray books = jsonResponse
                .getAsJsonObject("data")
                .getAsJsonArray("books");
            
            System.out.println("✓ Fetched " + books.size() + " books");
            System.out.println("Sorted by creation date (newest first):");
            
            for (int i = 0; i < Math.min(5, books.size()); i++) {
                JsonObject book = books.get(i).getAsJsonObject();
                System.out.printf(
                    "%d. %s - Created: %s%n",
                    i + 1,
                    book.get("title").getAsString(),
                    book.get("createdDate").getAsString()
                );
            }
        } else if (response.statusCode() == 400) {
            System.err.println("❌ Bad request - check sort parameter");
            System.err.println("v2 only supports sort=createdDate");
            System.err.println(response.body());
        }
    }
}
```

### PHP example: Track high-performing books

```php
<?php

$base_url = 'https://api.bookhub.com/api/v2';
$token = 'your_jwt_token_here';

$headers = [
    'Authorization: Bearer ' . $token,
    'Content-Type: application/json'
];

function getBookDetails($book_id, $base_url, $headers) {
    $ch = curl_init("$base_url/books/$book_id");
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    
    $response = curl_exec($ch);
    if ($response === false) {
        $error = curl_error($ch);
        curl_close($ch);
        return null;
    }
    $status_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    if ($status_code === 200) {
        return json_decode($response, true)['data'];
    }
    return null;
}

function getAllBooks($base_url, $headers, $status = 'ACTIVE') {
    $ch = curl_init("$base_url/books?status=$status&limit=100&sort=createdDate");
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    
    $response = curl_exec($ch);
    if ($response === false) {
        curl_close($ch);
        return [];
    }
    curl_close($ch);
    
    return json_decode($response, true)['data']['books'];
}

// Get all active books
$books = getAllBooks($base_url, $headers);

// Analyze hit counts
$high_performers = [];
foreach ($books as $book) {
    $details = getBookDetails($book['bookId'], $base_url, $headers);
    if ($details && $details['hitCount'] > 500) {
        $high_performers[] = [
            'title' => $details['title'],
            'hitCount' => $details['hitCount']
        ];
    }
}

// Sort by hit count
usort($high_performers, function($a, $b) {
    return $b['hitCount'] - $a['hitCount'];
});

echo "High-Performing Books (500+ views):\n";
foreach ($high_performers as $index => $book) {
    echo ($index + 1) . ". {$book['title']} - {$book['hitCount']} views\n";
}
?>
```

---

## Best practices

### 1. Use `hitCount` for analytics, not real-time decisions

The `hitCount` field is useful for trend analysis but should not be used for real-time, mission-critical decisions:

✅ **Good uses:**

- Identifying trending books over weeks/months
- Measuring marketing campaign effectiveness
- Generating internal reports

❌ **Avoid:**

- Making instant inventory decisions based on a single view count.
- Using it as the sole metric for success.
- Treating it as perfectly accurate (may have slight delays in updates).

### 2. Migrate away from title sorting properly

If you're migrating from v1 and used title sorting:

**Step 1**: Audit your codebase for `sort=title` parameters

```bash
# Search your codebase
grep -r "sort=title" .
grep -r "sort.*title" .
```

**Step 2**: Replace with date sorting or client-side sorting

```python
# Before
response = get_books(sort='title', order='asc')

# After - Option A: Use date sorting
response = get_books(sort='createdDate', order='desc')

# After - Option B: Client-side sort
all_books = get_books()
sorted_books = sorted(all_books, key=lambda x: x['title'])
```

**Step 3**: Update documentation and inform your team

### 3. Handle pagination correctly

When fetching all books, paginate through results:

```python
def fetch_all_books(base_url, token):
    all_books = []
    page = 1
    
    while True:
        response = requests.get(
            f"{base_url}/books",
            params={'page': page, 'limit': 100},
            headers={'Authorization': f'Bearer {token}'}
        )
        
        data = response.json()['data']
        all_books.extend(data['books'])
        
        if page >= data['pagination']['totalPages']:
            break
        
        page += 1
    
    return all_books
```

### 4. Use the `Idempotency-Key` for create operations

Always use  the `Idempotency-Key` when creating books to prevent duplicates:

```python
import uuid

idempotency_key = str(uuid.uuid4())

response = requests.post(
    f"{BASE_URL}/books",
    headers={
        'Authorization': f'Bearer {token}',
        'Idempotency-Key': idempotency_key
    },
    json=book_data
)
```

### 5. Cache book details appropriately

The API returns `ETag` and `Cache-Control` headers. Use them:

```python
import requests

# First request
response = requests.get(f"{BASE_URL}/books/{book_id}")
etag = response.headers.get('ETag')
book_data = response.json()

# Later request with conditional GET
response = requests.get(
    f"{BASE_URL}/books/{book_id}",
    headers={'If-None-Match': etag}
)

if response.status_code == 304:
    # Use cached data
    print("Data unchanged, using cache")
else:
    # Update cache
    book_data = response.json()
```

### 6. Monitor rate limits

Track rate limit headers to avoid being throttled:

```python
def check_rate_limits(response):
    limit = response.headers.get('X-RateLimit-Limit')
    remaining = response.headers.get('X-RateLimit-Remaining')
    reset = response.headers.get('X-RateLimit-Reset')
    
    print(f"Rate limit: {remaining}/{limit} remaining")
    
    if int(remaining) < 10:
        print(f"⚠️  Low on rate limit! Resets at {reset}")
```

---

## Support and resources

### Getting help

- **API support email**: `api-support@bookhub.com`
- **Developer documentation**: `https://docs.bookhub.com`
- **Status page**: `https://status.bookhub.com`

### Additional resources

- **OpenAPI specification**: Download the v2 YAML spec for use with Swagger/OpenAPI tools
- **Postman collection**: Available upon request
- **SDK libraries**: Python, JavaScript, Java, and PHP SDKs coming soon

### Migration assistance

Need help migrating from v1 to v2? Contact our integration team:

- **Email**: `integrations@bookhub.com`
- **Include**: Current usage patterns, timeline, specific concerns.

### Feedback

We value your feedback on v2 changes:

- **Feature requests**: `features@bookhub.com`
- **Bug reports**: `bugs@bookhub.com`
- **General feedback**: `feedback@bookhub.com`

---

## Version history

| Version | Date            | Changes                                                                                |
|---------|-----------------|----------------------------------------------------------------------------------------|
| 2.0     | January 2, 2026 | Initial v2 release with path-based versioning, `hitCount` field, removed title sorting |

---

Last updated:  April 2026
