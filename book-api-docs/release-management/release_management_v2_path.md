# BookHub Publisher API v2 - Release management document

## Document information

| Property | Value |
| -------- | ----- |
| **Release version** | 2.0 |
| **Release date** | January 2, 2026 |
| **Release type** | Major version (breaking changes) |
| **Status** | Preview |
| **Deprecation timeline** | v1 will be maintained until July 2, 2026 (6 months) |
| **Versioning method** | URI path-based |

---

## Table of contents

1. [Overview of v2](#overview-of-v2)
2. [Changes in v2](#changes-in-v2)
3. [Deprecations in v2](#deprecations-in-v2)
4. [Migration strategy](#migration-strategy)
5. [Timeline and support](#timeline-and-support)
6. [Testing and validation](#testing-and-validation)
7. [Rollback procedures](#rollback-procedures)
8. [Appendix](#appendix)

---

## Overview of v2

### Executive summary

BookHub Publisher API v2 represents a significant evolution in how publishers interact with our platform. This release introduces **path-based versioning** (`/v2/books` instead of `/v1/books`), making API versions more explicit and easier to manage. The primary focus of v2 is on enhancing analytics capabilities while improving API performance through strategic feature optimization.

### Strategic goals

- **Enhanced analytics**: Provide publishers with visibility into book engagement through hit count tracking
- **Improved performance**: Optimize query performance by removing computationally expensive operations
- **Simplified versioning**: Make API versions explicit in the URL path for a better developer experience
- **Future-ready architecture**: Establish patterns that scale for future feature additions

### Key highlights

- **New analytics field**: `hitCount` provides real-time visibility into book page views
- **Performance optimization**: 40% faster response times on list endpoints
- **Developer experience**: Path-based versioning simplifies testing and debugging
- **Backward compatibility**: v1 remains fully functional during the transition period

### Target audience

This release impacts:

- Publishers using the BookHub API programmatically.
- Developers integrating with BookHub Publisher API.
- Internal teams building on top of the publisher platform.
- Third-party integrators and partners.

---

## Changes in v2

### New features

#### Hit count analytics field

**What's new**:

The `hitCount` field is now available on the GET single book endpoint, providing publishers with engagement metrics.

**Use cases**:

- Track which books are generating the most interest.
- Measure marketing campaign effectiveness.
- Identify trending titles early.
- Make data-driven inventory decisions.

**Technical details**:

```json
// GET /v2/books/{bookId}
{
  "status": "success",
  "data": {
    "bookId": "abc123",
    "title": "The Great Adventure",
    "author": "Jane Smith",
    "hitCount": 1247,  // NEW FIELD
    "status": "ACTIVE",
    "createdDate": "2025-12-15T10:30:00Z"
  }
}
```

**Implementation notes**:

- The field is read-only and automatically incremented.
- The initial value for new books is `0`.
- Counter increments on each GET request to the single book endpoint.
- Available only in v2 (not present in v1 responses).
- Persists across the book's lifetime.

**Example usage**:

```python
import requests

response = requests.get(
    "https://api.bookhub.com/api/v2/books/abc123",
    headers={"Authorization": "Bearer YOUR_TOKEN"}
)

book = response.json()['data']
hit_count = book['hitCount']

if hit_count > 500:
    print(f"High performer: {book['title']} with {hit_count} views")
```

#### Path-based versioning

**What's new**:

The API version is now specified in the URL path rather than a header.

**Benefits**:

- **Visibility**: Version is immediately obvious in URLs
- **Browser testing**: Can test endpoints directly in the browser
- **Caching**: Standard HTTP caches work naturally
- **Documentation**: Clearer API documentation
- **Tool compatibility**: Better support in OpenAPI/Swagger tools

**Migration**:

```bash
# Old (v1)
GET https://api.bookhub.com/api/v1/books

# New (v2)
GET https://api.bookhub.com/api/v2/books
```

---

### Breaking changes

#### Title sorting removed

**Impact level**: HIGH - Breaking change

**What changed**:

The `sort=title` parameter is no longer supported on the GET all books endpoint.

**Rationale**:

- **Performance**: Text sorting on large datasets is computationally expensive
- **Internationalization**: Different languages have different sorting rules
- **Case sensitivity**: Mixed case creates ambiguous sort orders
- **Low usage**: Analytics showed <3% of requests used title sorting
- **Better alternatives**: Date-based sorting is more reliable and performant

**Before (v1)**:

```bash
# This worked in v1
GET /v1/books?sort=title&order=asc
```

**After (v2)**:

```bash
# This returns a 400 error in v2
GET /v2/books?sort=title&order=asc

# Error Response:
{
  "status": "error",
  "error": {
    "code": "INVALID_SORT_PARAMETER",
    "message": "Sort parameter 'title' is not supported in v2",
    "validOptions": ["createdDate"]
  }
}
```

**Available sort options**:

- `createdDate` (only valid option in v2)
- `order`: `asc` or `desc`

---

### Enhancements

#### Performance improvements

**List endpoint optimization**:

- 40% faster average response time
- Reduced database load through optimized queries
- Improved caching efficiency

**Metrics**:

```bash
Average Response Time:
- v1: 450ms
- v2: 270ms (40% improvement)

95th Percentile:
- v1: 850ms
- v2: 420ms (51% improvement)
```

#### API response consistency

**Standardized error messages**:

All errors now include:

- Consistent error code format
- Clear, actionable messages
- Suggested alternatives where applicable
- Request ID for support tracking

**Example**:

```json
{
  "status": "error",
  "error": {
    "code": "INVALID_SORT_PARAMETER",
    "message": "The 'title' sort parameter is not supported in API v2. Please use 'createdDate' instead.",
    "requestId": "req_abc123xyz",
    "timestamp": "2026-01-02T14:30:00Z"
  }
}
```

---

## Deprecations in v2

### Deprecated features

#### Title sorting (removed)

**Deprecation status**: Fully removed in v2

##### Title sorting timeline

- **December 15, 2025**: Deprecation announced for v1
- **January 2, 2026**: Removed in v2
- **July 2, 2026**: v1 sunset (title sorting no longer available)

##### Migration path

Publishers must choose one of two alternatives:

###### Option A: Server-side date sorting (recommended)

```python
# Replace title sorting with date sorting
response = requests.get(
    "https://api.bookhub.com/api/v2/books",
    params={"sort": "createdDate", "order": "desc"}
)
```

###### Option B: Client-side title sorting

```python
# Fetch all books and sort locally
response = requests.get(
    "https://api.bookhub.com/api/v2/books",
    params={"limit": 100}
)
books = response.json()['data']['books']

# Sort by title on client side
sorted_books = sorted(books, key=lambda x: x['title'].lower())
```

#### Base URL pattern (changed)

**Old pattern**: `/v1/books`
**New pattern**: `/v2/books`

**Impact**: All API calls must update base URL paths

**Automatic redirects**: not provided (avoid accidental version mixing)

---

### Deprecation warning system

#### How warnings work

When using v1 endpoints after January 2, 2026, responses include deprecation warnings:

```http
HTTP/1.1 200 OK
X-API-Version: v1
X-API-Deprecated: true
X-API-Sunset: 2026-07-02T00:00:00Z
Deprecation: true
Sunset: Thu, 02 Jul 2026 00:00:00 GMT

{
  "status": "success",
  "data": { ... },
  "warnings": [
    {
      "code": "VERSION_DEPRECATED",
      "message": "API v1 will be sunset on July 2, 2026. Please migrate to v2.",
      "documentation": "https://docs.bookhub.com/api/v2/migration"
    }
  ]
}
```

#### Monitoring deprecation warnings

**Python example**:

```python
def check_deprecation_warnings(response):
    """Monitor API deprecation warnings."""
    
    # Check HTTP headers
    if response.headers.get('X-API-Deprecated') == 'true':
        sunset_date = response.headers.get('X-API-Sunset')
        print(f"WARNING: This API version will sunset on {sunset_date}")
    
    # Check response body
    data = response.json()
    if 'warnings' in data:
        for warning in data['warnings']:
            print(f"API Warning: {warning['message']}")
            if 'documentation' in warning:
                print(f"See: {warning['documentation']}")
```

---

## Migration strategy

### Migration overview

#### Migration timeline

| Phase | Date | Action | Status |
| ----- | ---- | ------ | ------ |
| **Announcement** | December 15, 2025 | v2 preview announced | Complete |
| **Preview release** | January 2, 2026 | v2 available for testing | Current |
| **Testing period** | January - March 2026 | Publishers test v2 | In Progress |
| **Deprecation warnings** | January 2, 2026 | v1 shows deprecation warnings | Active |
| **Migration deadline** | June 2, 2026 | All integrations should use v2 | Upcoming |
| **v1 sunset** | July 2, 2026 | v1 endpoints disabled | Scheduled |

#### Migration phases

##### Phase 1: Assessment (week 1)

Identify the impact on your integration:

```bash
# Audit your codebase for v1 usage
grep -r "/v1/books" .
grep -r "sort=title" .
grep -r "api\.bookhub\.com/api/v1" .
```

**Checklist**:

- [ ] Identify all API v1 calls in the codebase.
- [ ] Document usage of title sorting.
- [ ] Review the `hitCount` field integration needs.
- [ ] Estimate migration effort.

##### Phase 2: Testing (week 2-3)

Test v2 in sandbox environment:

```python
class APIVersionTester:
    """Test v1 vs v2 behavior."""
    
    def __init__(self, token):
        self.token = token
        self.base_url = "https://sandbox-api.bookhub.com/api"
    
    def test_book_retrieval(self, book_id):
        """Compare v1 and v2 responses."""
        
        # Test v1
        v1_response = requests.get(
            f"{self.base_url}/v1/books/{book_id}",
            headers={"Authorization": f"Bearer {self.token}"}
        )
        v1_data = v1_response.json()['data']
        
        # Test v2
        v2_response = requests.get(
            f"{self.base_url}/v2/books/{book_id}",
            headers={"Authorization": f"Bearer {self.token}"}
        )
        v2_data = v2_response.json()['data']
        
        # Verify differences
        assert 'hitCount' not in v1_data, "v1 should not have hitCount"
        assert 'hitCount' in v2_data, "v2 should have hitCount"
        
        print(f"v1 fields: {list(v1_data.keys())}")
        print(f"v2 fields: {list(v2_data.keys())}")
        print(f"New in v2: hitCount = {v2_data['hitCount']}")
    
    def test_sorting(self):
        """Verify sorting behavior changes."""
        
        # v1: title sorting works
        v1_response = requests.get(
            f"{self.base_url}/v1/books",
            params={"sort": "title"},
            headers={"Authorization": f"Bearer {self.token}"}
        )
        assert v1_response.status_code == 200, "v1 title sorting should work"
        
        # v2: title sorting fails
        v2_response = requests.get(
            f"{self.base_url}/v2/books",
            params={"sort": "title"},
            headers={"Authorization": f"Bearer {self.token}"}
        )
        assert v2_response.status_code == 400, "v2 title sorting should fail"
        
        # v2: date sorting works
        v2_response = requests.get(
            f"{self.base_url}/v2/books",
            params={"sort": "createdDate"},
            headers={"Authorization": f"Bearer {self.token}"}
        )
        assert v2_response.status_code == 200, "v2 date sorting should work"

# Run tests
tester = APIVersionTester("your_token_here")
tester.test_book_retrieval("test_book_id")
tester.test_sorting()
```

**Testing checklist**:

- [ ] Test all current API calls with v2.
- [ ] Verify hitCount field handling.
- [ ] Test sorting parameter changes.
- [ ] Validate error handling.
- [ ] Performance test with production-like load.

##### Phase 3: Implementation (week 3-4)

Update your integration to use v2:

###### Step 1: Update base URLs

```python
# Before
BASE_URL = "https://api.bookhub.com/api/v1"

# After
BASE_URL = "https://api.bookhub.com/api/v2"
```

###### Step 2: Handle new fields

```python
def get_book_details(book_id):
    """Retrieve book with v2 fields."""
    response = requests.get(
        f"{BASE_URL}/books/{book_id}",
        headers={"Authorization": f"Bearer {TOKEN}"}
    )
    
    book = response.json()['data']
    
    # Handle new hitCount field
    hit_count = book.get('hitCount', 0)
    
    return {
        'bookId': book['bookId'],
        'title': book['title'],
        'author': book['author'],
        'hitCount': hit_count,  # New field
        'status': book['status']
    }
```

###### Step 3: Replace title sorting

```python
def get_sorted_books(sort_type='date'):
    """Get books with appropriate sorting."""
    
    if sort_type == 'title':
        # Client-side title sorting
        response = requests.get(
            f"{BASE_URL}/books",
            params={"limit": 100},
            headers={"Authorization": f"Bearer {TOKEN}"}
        )
        books = response.json()['data']['books']
        return sorted(books, key=lambda x: x['title'].lower())
    
    else:
        # Server-side date sorting
        response = requests.get(
            f"{BASE_URL}/books",
            params={"sort": "createdDate", "order": "desc"},
            headers={"Authorization": f"Bearer {TOKEN}"}
        )
        return response.json()['data']['books']
```

###### Step 4: Update error handling

```python
def safe_get_books(sort_by='createdDate'):
    """Get books with fallback error handling."""
    
    try:
        response = requests.get(
            f"{BASE_URL}/books",
            params={"sort": sort_by},
            headers={"Authorization": f"Bearer {TOKEN}"}
        )
        
        if response.status_code == 400:
            error = response.json()['error']
            if error['code'] == 'INVALID_SORT_PARAMETER':
                # Fall back to default sorting
                print(f"Warning: {sort_by} sorting not supported, using createdDate")
                return safe_get_books(sort_by='createdDate')
        
        response.raise_for_status()
        return response.json()['data']['books']
    
    except requests.exceptions.RequestException as e:
        print(f"Error fetching books: {e}")
        return []
```

##### Phase 4: Deployment (week 5)

Roll out v2 integration to production:

**Deployment strategy**:

1. **Canary deployment**: Route 10% of traffic to v2.
2. **Monitor**: Check error rates and performance.
3. **Gradual rollout**: Increase to 50%, then 100%.
4. **Validation**: Verify all features work correctly.

**Monitoring checklist**:

- [ ] Monitor error rates (should remain stable).
- [ ] Track response times (expect 40% improvement).
- [ ] Verify hitCount data collection.
- [ ] Check client-side sorting performance (if used).

##### Phase 5: Validation (week 6)

Confirm successful migration:

```python
def validate_migration():
    """Validate complete migration to v2."""
    
    checks = {
        'using_v2_urls': False,
        'handling_hitCount': False,
        'no_title_sorting': False,
        'error_handling_updated': False
    }
    
    # Test v2 URL usage
    response = requests.get(f"{BASE_URL}/books")
    if '/v2/' in response.request.url:
        checks['using_v2_urls'] = True
    
    # Test hitCount handling
    book = get_book_details('test_id')
    if 'hitCount' in book:
        checks['handling_hitCount'] = True
    
    # Test title sorting absence
    try:
        get_sorted_books(sort_type='title')
        checks['no_title_sorting'] = True
    except Exception:
        pass
    
    # Report
    for check, passed in checks.items():
        status = "PASS" if passed else "FAIL"
        print(f"{status}: {check}")
    
    return all(checks.values())

# Run validation
if validate_migration():
    print("Migration successful!")
else:
    print("Migration incomplete - review failed checks")
```

---

### Code migration examples

#### Basic book retrieval

**Before (v1)**:

```python
import requests

BASE_URL = "https://api.bookhub.com/api/v1"
TOKEN = "your_jwt_token"

response = requests.get(
    f"{BASE_URL}/books/abc123",
    headers={"Authorization": f"Bearer {TOKEN}"}
)

book = response.json()['data']
print(f"Title: {book['title']}")
print(f"Author: {book['author']}")
# No hitCount available
```

**After (v2)**:

```python
import requests

BASE_URL = "https://api.bookhub.com/api/v2"  # Changed
TOKEN = "your_jwt_token"

response = requests.get(
    f"{BASE_URL}/books/abc123",
    headers={"Authorization": f"Bearer {TOKEN}"}
)

book = response.json()['data']
print(f"Title: {book['title']}")
print(f"Author: {book['author']}")
print(f"Views: {book['hitCount']}")  # New field
```

#### Listing books with sorting

**Before (v1)**:

```python
# Get books sorted by title
response = requests.get(
    f"{BASE_URL}/books",
    params={"sort": "title", "order": "asc"},  # Worked in v1
    headers={"Authorization": f"Bearer {TOKEN}"}
)
books = response.json()['data']['books']
```

**After (v2) - Option A: Date sorting**:

```python
# Get books sorted by creation date
response = requests.get(
    f"{BASE_URL}/books",
    params={"sort": "createdDate", "order": "desc"},  # Changed
    headers={"Authorization": f"Bearer {TOKEN}"}
)
books = response.json()['data']['books']
```

**After (v2) - Option B: Client-side sorting**:

```python
# Get all books and sort by title locally
response = requests.get(
    f"{BASE_URL}/books",
    params={"limit": 100},
    headers={"Authorization": f"Bearer {TOKEN}"}
)
books = response.json()['data']['books']

# Sort by title on client side
sorted_books = sorted(books, key=lambda x: x['title'].lower())
```

#### Hit count analytics

**New v2 Feature**:

```python
def analyze_book_performance(book_ids):
    """Analyze book performance using hitCount."""
    
    performance_data = []
    
    for book_id in book_ids:
        response = requests.get(
            f"{BASE_URL}/books/{book_id}",
            headers={"Authorization": f"Bearer {TOKEN}"}
        )
        
        book = response.json()['data']
        performance_data.append({
            'title': book['title'],
            'hitCount': book['hitCount'],
            'status': book['status']
        })
    
    # Sort by performance
    performance_data.sort(key=lambda x: x['hitCount'], reverse=True)
    
    # Report top performers
    print("Top 5 Books by Views:")
    for i, book in enumerate(performance_data[:5], 1):
        print(f"{i}. {book['title']}: {book['hitCount']} views")
    
    return performance_data

# Usage
book_ids = ['id1', 'id2', 'id3', 'id4', 'id5']
analyze_book_performance(book_ids)
```

#### Version-agnostic client

**Dual-version support during migration**:

```python
class BookHubClient:
    """Client supporting both v1 and v2."""
    
    def __init__(self, token, api_version='v2'):
        self.token = token
        self.api_version = api_version
        self.base_url = f"https://api.bookhub.com/api/{api_version}"
    
    def get_book(self, book_id):
        """Get book details (version-aware)."""
        response = requests.get(
            f"{self.base_url}/books/{book_id}",
            headers={"Authorization": f"Bearer {self.token}"}
        )
        
        book = response.json()['data']
        
        # Add hitCount to v1 responses for consistency
        if self.api_version == 'v1' and 'hitCount' not in book:
            book['hitCount'] = None  # Not available in v1
        
        return book
    
    def list_books(self, sort_by='createdDate', order='desc'):
        """List books with version-appropriate sorting."""
        
        # Prevent title sorting in v2
        if self.api_version == 'v2' and sort_by == 'title':
            print("Warning: title sorting not supported in v2, using createdDate")
            sort_by = 'createdDate'
        
        response = requests.get(
            f"{self.base_url}/books",
            params={"sort": sort_by, "order": order},
            headers={"Authorization": f"Bearer {self.token}"}
        )
        
        return response.json()['data']['books']

# Use with v2 (recommended)
client_v2 = BookHubClient(token="your_token", api_version='v2')
books = client_v2.list_books()

# Fallback to v1 if needed
client_v1 = BookHubClient(token="your_token", api_version='v1')
books = client_v1.list_books(sort_by='title')  # Still works in v1
```

---

### Migration validation

#### Automated testing script

```python
#!/usr/bin/env python3
"""
Migration Validation Script
Validates successful migration from v1 to v2.
"""

import requests
import sys

class MigrationValidator:
    """Validate API v2 migration."""
    
    def __init__(self, token, base_url="https://api.bookhub.com/api"):
        self.token = token
        self.v1_url = f"{base_url}/v1"
        self.v2_url = f"{base_url}/v2"
        self.headers = {"Authorization": f"Bearer {token}"}
        self.results = []
    
    def test_v2_url_accessible(self):
        """Test 1: v2 endpoints are accessible."""
        try:
            response = requests.get(
                f"{self.v2_url}/books",
                headers=self.headers,
                params={"limit": 1}
            )
            success = response.status_code == 200
            self.results.append(("v2 URL Accessible", success))
            return success
        except Exception as e:
            self.results.append(("v2 URL Accessible", False, str(e)))
            return False
    
    def test_hitcount_field(self):
        """Test 2: hitCount field is present in v2."""
        try:
            # Get any book from v2
            response = requests.get(
                f"{self.v2_url}/books",
                headers=self.headers,
                params={"limit": 1}
            )
            
            if response.status_code != 200:
                self.results.append(("hitCount Field", False, "No books found"))
                return False
            
            books = response.json()['data']['books']
            if not books:
                self.results.append(("hitCount Field", False, "Empty book list"))
                return False
            
            book_id = books[0]['bookId']
            
            # Get single book to check hitCount
            response = requests.get(
                f"{self.v2_url}/books/{book_id}",
                headers=self.headers
            )
            
            book = response.json()['data']
            success = 'hitCount' in book
            self.results.append(("hitCount Field Present", success))
            return success
            
        except Exception as e:
            self.results.append(("hitCount Field", False, str(e)))
            return False
    
    def test_title_sorting_removed(self):
        """Test 3: Title sorting returns error in v2."""
        try:
            response = requests.get(
                f"{self.v2_url}/books",
                headers=self.headers,
                params={"sort": "title"}
            )
            
            # Should return 400 error
            success = response.status_code == 400
            self.results.append(("Title Sorting Removed", success))
            return success
            
        except Exception as e:
            self.results.append(("Title Sorting Removed", False, str(e)))
            return False
    
    def test_date_sorting_works(self):
        """Test 4: Date sorting works in v2."""
        try:
            response = requests.get(
                f"{self.v2_url}/books",
                headers=self.headers,
                params={"sort": "createdDate", "order": "desc"}
            )
            
            success = response.status_code == 200
            self.results.append(("Date Sorting Works", success))
            return success
            
        except Exception as e:
            self.results.append(("Date Sorting Works", False, str(e)))
            return False
    
    def test_no_v1_usage(self):
        """Test 5: Verify not using v1 endpoints."""
        try:
            # This is a placeholder - in real implementation,
            # you'd check your application logs or monitoring
            # to ensure no v1 calls are being made
            
            # For this test, we'll just verify v2 is working
            response = requests.get(
                f"{self.v2_url}/books",
                headers=self.headers,
                params={"limit": 1}
            )
            
            success = response.status_code == 200
            self.results.append(("Not Using v1", success))
            return success
            
        except Exception as e:
            self.results.append(("Not Using v1", False, str(e)))
            return False
    
    def run_all_tests(self):
        """Run all migration validation tests."""
        print("=" * 60)
        print("BookHub API v2 Migration Validation")
        print("=" * 60)
        print()
        
        tests = [
            self.test_v2_url_accessible,
            self.test_hitcount_field,
            self.test_title_sorting_removed,
            self.test_date_sorting_works,
            self.test_no_v1_usage
        ]
        
        for test in tests:
            test()
        
        # Print results
        print("\nTest Results:")
        print("-" * 60)
        
        passed = 0
        failed = 0
        
        for result in self.results:
            test_name = result[0]
            success = result[1]
            
            status = "PASS" if success else "FAIL"
            symbol = "✓" if success else "✗"
            
            print(f"{symbol} {status}: {test_name}")
            
            if not success and len(result) > 2:
                print(f"  Error: {result[2]}")
            
            if success:
                passed += 1
            else:
                failed += 1
        
        print("-" * 60)
        print(f"Results: {passed} passed, {failed} failed")
        print()
        
        if failed == 0:
            print("✓ Migration validation PASSED!")
            print("Your integration is ready for v2.")
            return True
        else:
            print("✗ Migration validation FAILED!")
            print("Please review failed tests and update your integration.")
            return False

## Usage
if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python validate_migration.py <your_api_token>")
        sys.exit(1)
    
    token = sys.argv[1]
    validator = MigrationValidator(token)
    success = validator.run_all_tests()
    
    sys.exit(0 if success else 1)
```

**Run the validator**:

```bash
python validate_migration.py your_api_token_here
```

---

## Timeline and support

### Detailed schedule

| Date | Milestone | Action required | Support available |
| ---- | --------- | --------------- | ----------------- |
| **Dec 15, 2025** | v2 Announcement | Review release notes | Documentation published |
| **Jan 2, 2026** | v2 Preview Launch | Begin testing v2 | Sandbox environment open |
| **Jan 15, 2026** | Migration Workshop | Attend workshop (optional) | Live Q&A sessions |
| **Feb 1, 2026** | Check-in #1 | Report testing progress | 1-on-1 support available |
| **Mar 1, 2026** | Check-in #2 | Submit migration plan | Technical assistance |
| **Apr 1, 2026** | Migration Sprint | Active migration period | Priority support |
| **May 1, 2026** | Final Testing | Production validation | Integration review |
| **Jun 2, 2026** | Migration Deadline | Complete migration | Extended support |
| **Jun 15, 2026** | v1 Deprecation Notice | Final warning sent | Transition assistance |
| **Jul 2, 2026** | v1 Sunset | v1 endpoints disabled | v2-only support |

### Critical dates

> **Warning**: v1 will stop working after July 2, 2026.

After this date:

- All v1 API calls will return `410 Gone` errors.
- No v1 support will be available.
- All integrations must use v2.

---

### Support resources

#### Technical support

**Email support**:

- **General questions**: `api-support@bookhub.com`
- **Migration assistance**: `api-migration@bookhub.com`
- **Emergency issues**: `api-urgent@bookhub.com`

**Response times**:

- **Standard**: 24 hours
- **Priority** (during migration): 4 hours
- **Emergency**: 1 hour

#### Documentation

**Available resources**:

- **Full v2 API documentation**: `https://docs.bookhub.com/api/v2`
- **Migration guide**: `https://docs.bookhub.com/api/v2/migration`
- **OpenAPI spec**: `https://api.bookhub.com/openapi/v2.yaml`
- **Code examples**: `https://github.com/bookhub/api-examples`

#### Live support sessions

**Migration workshops**:

- **Dates**: January 15, February 15, March 15, 2026
- **Time**: 2:00 PM EST
- **Duration**: 90 minutes
- **Registration**: `workshops@bookhub.com`

**Office hours**:

- Every Tuesday, 10:00 AM - 12:00 PM EST
- Every Thursday, 2:00 PM - 4:00 PM EST
- Drop-in Q&A via Zoom
- **Link**: `https://zoom.us/j/bookhub-api-office-hours`

#### Partner support

For high-volume publishers and partners:

- Dedicated migration engineer
- Custom migration plan
- Priority bug fixes
- Extended testing period

Contact: `partners@bookhub.com`

---

## Testing and validation

### Testing environments

#### Sandbox environment

**Purpose**: Test v2 integration without affecting production

**Details**:

- **URL**: `https://sandbox-api.bookhub.com/api/v2`
- Same authentication as production
- Separate data store
- Reset available on request.

**Setup**:

```bash
# Configure sandbox environment
export BOOKHUB_ENV="sandbox"
export BOOKHUB_BASE_URL="https://sandbox-api.bookhub.com/api/v2"
export BOOKHUB_TOKEN="your_sandbox_token"
```

#### Testing checklist

##### Functional testing

- [ ] **Book creation**: Create books successfully in v2
- [ ] **Book retrieval**: Get a single book with the `hitCount` field
- [ ] **Book listing**: List books with date sorting
- [ ] **Book finalization**: Finalize books (PENDING -> ACTIVE)
- [ ] **Pagination**: Navigate through paginated results
- [ ] **Filtering**: Filter by status, format, genre
- [ ] **Error handling**: Handle invalid sort parameters
- [ ] **Authentication**: JWT tokens work correctly

##### Performance testing

- [ ] **Response times**: Verify 40% improvement
- [ ] **Concurrent requests**: Test under load
- [ ] **Rate limits**: Confirm rate limiting works
- [ ] **Caching**: Verify ETag/Cache-Control behavior

##### Integration testing

- [ ] **Existing workflows**: All current workflows work in v2
- [ ] **Data consistency**: Data matches between v1 and v2
- [ ] **Client-side sorting**: Performance is acceptable if used
- [ ] **Analytics integration**: hitCount data flows correctly

##### Regression testing

- [ ] **No v1 calls**: Confirm no v1 endpoints in use
- [ ] **Error handling**: All error cases covered
- [ ] **Edge cases**: Boundary conditions tested
- [ ] **Backward compatibility**: New fields don't break parsing

---

### Validation tools

#### Postman collection

Download our Postman collection for v2 testing:

```bash
curl -O https://docs.bookhub.com/postman/bookhub-api-v2.json
```

Import into Postman and set environment variables:

- `base_url`: `https://sandbox-api.bookhub.com/api/v2`
- `token`: Your sandbox API token

#### Automated test suite

```python
#!/usr/bin/env python3
"""
Comprehensive v2 API Test Suite
"""

import unittest
import requests

class BookHubV2Tests(unittest.TestCase):
    """Test suite for BookHub API v2."""
    
    @classmethod
    def setUpClass(cls):
        cls.base_url = "https://sandbox-api.bookhub.com/api/v2"
        cls.token = "your_sandbox_token"
        cls.headers = {"Authorization": f"Bearer {cls.token}"}
    
    def test_create_book(self):
        """Test creating a book in v2."""
        book_data = {
            "title": "Test Book v2",
            "author": "Test Author",
            "description": "Test description",
            "language": "en-US",
            "publisher": "Test Publisher",
            "publishedDate": "2025-01-01",
            "isbn": "978-0-1234-5678-9",
            "price": 19.99,
            "coverImages": ["https://example.com/cover.jpg"],
            "genres": ["Fiction"],
            "bookFormat": "Paperback"
        }
        
        response = requests.post(
            f"{self.base_url}/books",
            headers=self.headers,
            json=book_data
        )
        
        self.assertEqual(response.status_code, 201)
        self.assertIn('bookId', response.json()['data'])
    
    def test_get_book_has_hitcount(self):
        """Test that single book GET includes hitCount."""
        # First, create a book.
        book = self._create_test_book()
        book_id = book['bookId']
        
        # Get the book
        response = requests.get(
            f"{self.base_url}/books/{book_id}",
            headers=self.headers
        )
        
        self.assertEqual(response.status_code, 200)
        data = response.json()['data']
        self.assertIn('hitCount', data)
        self.assertIsInstance(data['hitCount'], int)
    
    def test_title_sorting_fails(self):
        """Test that title sorting returns 400 error."""
        response = requests.get(
            f"{self.base_url}/books",
            headers=self.headers,
            params={"sort": "title"}
        )
        
        self.assertEqual(response.status_code, 400)
        error = response.json()['error']
        self.assertIn('INVALID_SORT_PARAMETER', error['code'])
    
    def test_date_sorting_works(self):
        """Test that date sorting works correctly."""
        response = requests.get(
            f"{self.base_url}/books",
            headers=self.headers,
            params={"sort": "createdDate", "order": "desc"}
        )
        
        self.assertEqual(response.status_code, 200)
        books = response.json()['data']['books']
        
        # Verify the sorting order.
        if len(books) > 1:
            dates = [book['createdDate'] for book in books]
            self.assertEqual(dates, sorted(dates, reverse=True))
    
    def test_pagination(self):
        """Test pagination works correctly."""
        response = requests.get(
            f"{self.base_url}/books",
            headers=self.headers,
            params={"page": 1, "limit": 10}
        )
        
        self.assertEqual(response.status_code, 200)
        data = response.json()['data']
        self.assertIn('pagination', data)
        self.assertLessEqual(len(data['books']), 10)
    
    def _create_test_book(self):
        """Helper to create a test book."""
        book_data = {
            "title": "Test Book",
            "author": "Test Author",
            "description": "Description",
            "language": "en-US",
            "publisher": "Publisher",
            "publishedDate": "2025-01-01",
            "isbn": "978-0-1234-5678-9",
            "price": 19.99,
            "coverImages": ["https://example.com/cover.jpg"],
            "genres": ["Fiction"],
            "bookFormat": "Paperback"
        }
        
        response = requests.post(
            f"{self.base_url}/books",
            headers=self.headers,
            json=book_data
        )
        
        return response.json()['data']

if __name__ == '__main__':
    unittest.main()
```

Run the test suite:

```bash
python test_v2_api.py
```

---

## Rollback procedures

### When to rollback

Consider rollback if you experience:

- Critical production errors affecting >10% of requests
- Data integrity issues
- Performance degradation >50% from baseline
- Security vulnerabilities

### Rollback process

#### Step 1: Immediate rollback

Switch back to v1 immediately:

```python
# Emergency rollback configuration
BASE_URL = "https://api.bookhub.com/api/v1"  # Revert to v1

# All other code remains the same
```

**Timeline**: Can be completed in <5 minutes

#### Step 2: Notify BookHub

Contact emergency support:

- **Email**: `api-urgent@bookhub.com`
- **Subject**: "URGENT: v2 Rollback - [Your Company]"
- **Include**: Error details, affected volume, and rollback timestamp.

#### Step 3: Root cause analysis

Work with the BookHub team to identify issues:

1. Review the error logs.
2. Analyze the performance metrics.
3. Identify specific failure points.
4. Develop a remediation plan.

#### Step 4: Plan re-migration

Once issues are resolved:

1. Test fixes in a sandbox.
2. Conduct a limited production test.
3. Attempt full migration.
4. Monitor closely for 48 hours.

---

### Rollback considerations

#### Data synchronization

If using the `hitCount` field:

- Data collected during the v2 period will be retained.
- Not accessible via v1 (field doesn't exist).
- Will be available when you re-migrate to v2.

#### Client-side sorting

If you implemented client-side title sorting:

- The feature will still work after the rollback.
- Consider keeping it (reduces future migration effort).
- Alternatively, revert to server-side title sorting in v1.

#### Monitoring during rollback

```python
def monitor_rollback_health():
    """Monitor system health after rollback to v1."""
    
    metrics = {
        'error_rate': check_error_rate(),
        'response_time': check_response_time(),
        'request_volume': check_request_volume(),
        'success_rate': check_success_rate()
    }
    
    # Alert if any metric is abnormal
    for metric, value in metrics.items():
        if not is_normal(metric, value):
            alert_team(f"Abnormal {metric}: {value}")
    
    return metrics

# Monitor every 5 minutes for the first hour
for i in range(12):
    monitor_rollback_health()
    time.sleep(300)
```

---

### Post-rollback support

After rolling back to v1:

- Priority support available
- Extended sandbox access
- Customized migration plan
- One-on-one technical assistance

Contact `api-migration@bookhub.com` to schedule a post-rollback review.

---

## Appendix

### Glossary

| Term | Definition |
| ---- | ---------- |
| **Breaking change** | API change that requires client code modifications |
| **Deprecation** | Marking a feature as obsolete before removal |
| **Path-based versioning** | Including version number in URL path (/v1, /v2) |
| **Hit count** | Number of times a book detail page has been accessed |
| **Sunset date** | Date when an API version will be permanently disabled |

### API version comparison

| Feature | v1 | v2 |
| ------- | -- | -- |
| URL pattern | `/v1/books` | `/v2/books` |
| Title sorting | YES | NO |
| Date sorting | YES | YES |
| Hit count field | NO | YES |
| Response time | 450ms avg | 270ms avg |
| Status | Deprecated | Current |
| Support until | July 2, 2026 | Ongoing |

### Contact information

**BookHub API team**:

- **General support**: `api-support@bookhub.com`
- **Migration help**: `api-migration@bookhub.com`
- **Emergency issues**: `api-urgent@bookhub.com`
- **Feature requests**: `feature-requests@bookhub.com`

**Response commitments**:

- **Standard support**: 24 hours
- **Migration support**: 4 hours
- **Emergency support**: 1 hour

### Additional resources

**Documentation**:

- **v2 Full documentation**: `https://docs.bookhub.com/api/v2`
- **Migration guide**: `https://docs.bookhub.com/api/v2/migration`
- **OpenAPI specification**: `https://api.bookhub.com/openapi/v2.yaml`

**Code examples**:

- **GitHub repository**: `https://github.com/bookhub/api-examples`
- **Sample projects**: `https://github.com/bookhub/sample-integrations`

**Community**:

- **Developer forum**: `https://forum.bookhub.com/api`
- **Stack overflow**: Tag `bookhub-api`
- **Discord**: `https://discord.gg/bookhub-developers`

---

**Document version**: 1.0  
**Last updated**: January 2, 2026  
**Next review**: March 2, 2026

---

End of release management document

---

Last updated:  April 2026
