# API versioning strategy: Decision rationale

## Document information

| Property | Value |
|----------|-------|
| **Decision Date** | January 2, 2026 |
| **Project** | BookHub Publisher API v2 |
| **Author** | Gale James |
| **Status** | Final Decision |

---

## Executive summary

After evaluating header-based and path-based versioning, I recommend path-based versioning for BookHub Publisher API v2.

- **Key decision**: Use URI path versioning (`/v1/books`, `/v2/books`) instead of custom header versioning.

- **Primary rationale**: Clarity, maintainability, and user experience are prioritized over URL simplicity.

---

## Table of contents

1. [Background](#background)
2. [Approaches evaluated](#approaches-evaluated)
3. [Decision criteria](#decision-criteria)
4. [Comparative analysis](#comparative-analysis)
5. [Final recommendation](#final-recommendation)
6. [Trade-offs accepted](#trade-offs-accepted)
7. [Implementation impact](#implementation-impact)
8. [Alternative considered](#alternative-considered)
9. [Conclusion](#conclusion)

---

## Background

### Project context

BookHub Publisher API enables book publishers to programmatically manage their catalog. The target audience includes:

- **Primary users**: Book publishers (varying technical expertise)
- **Integration developers**: Third-party developers building integrations
- **Internal teams**: BookHub's own engineering teams

### The challenge

We need to introduce breaking changes in v2 while maintaining backward compatibility with v1. The versioning approach must support:

- Clear documentation for both versions
- Smooth migration path for existing users
- Long-term maintainability
- Minimal user confusion

---

## Approaches evaluated

### 1. Header-based versioning (date identifiers)

**Format:** Custom `api-version` header with date-based identifiers

```http
GET /api/books HTTP/1.1
Host: api.bookhub.com
Authorization: Bearer <token>
api-version: 2024-12-30
```

**Characteristics:**

- Clean, version-agnostic URLs
- Date-based version identifiers (e.g., `2024-12-30`, `2025-03-15`)
- Supports preview versions (e.g., `2025-03-15-preview`)
- Version specified in the request header

### 2. Path-based versioning (URI versioning)

**Format:** Version embedded in URL path

```http
GET /api/v2/books HTTP/1.1
Host: api.bookhub.com
Authorization: Bearer <token>
```

**Characteristics:**

- Version visible in URL
- Separate URL paths per major version
- Simple integer versioning (v1, v2, v3)
- Standard REST approach

---

## Decision criteria

### Critical factors (must-have)

- **Documentation clarity**: Can users easily find and use the right documentation?
- **Maintainability**: How complex is maintaining documentation for multiple versions?
- **User experience**: How intuitive is the versioning approach for our target audience?
- **Migration path**: How easy is it to transition from v1 to v2?

### Important factors (nice-to-have)

- **URL cleanliness**: Are URLs clean and elegant?
- **Caching behavior**: Does standard HTTP caching work naturally?
- **Industry alignment**: What do leading APIs use?
- **Tooling support**: Do common tools work well with this approach?

---

## Comparative analysis

### 1. Documentation clarity

#### Header-based versioning (documentation clarity)

- **Score**: ❌ Poor
- **Challenge**: Every code example must include a header context

```python
# Must show header in EVERY example
import requests

response = requests.get(
    'https://api.bookhub.com/api/books',
    headers={
        'Authorization': 'Bearer YOUR_TOKEN',
        'api-version': '2024-12-30'  # ← Easy to miss!
    }
)
```

##### Problems

- Users might copy examples without the header.
- Easy to accidentally use the wrong version.
- Must repeat header context throughout documentation.
- Single consolidated doc becomes confusing (which version is this?).

#### Path-based versioning (documentation clarity)

- **Score**: ✅ Excellent
- **Advantage**: Version is immediately visible in every example

```python
# Version is obvious in the URL
import requests

response = requests.get(
    'https://api.bookhub.com/api/v2/books',  # ← Clear!
    headers={'Authorization': 'Bearer YOUR_TOKEN'}
)
```

##### Benefits

- Impossible to miss the version.
- Copy-paste examples are safer.
- Each version has self-contained documentation.
- Clear visual distinction between v1 and v2 examples.

#### Documentation clarity winner

The winner is path-based versioning. In this instance, path-based versioning provided the best benefits.

---

### 2. Documentation maintainability

#### Header-based versioning (documentation maintainability)

- **Score: ❌ Poor**
- **Challenges:**

    1. **Breaking changes are hidden**.

       ```markdown
       ## Create Book Endpoint
       
       `POST /api/books`  ← Same URL for both versions!
       
       ### For api-version: 2024-12-30
       - ISBN format: Can be with or without hyphens
       
       ### For api-version: 2025-03-15
       - ISBN format: MUST include hyphens
       ```

       Confusing for users to track which rules apply at what times.

    2. **Single document complexity**

       - Must annotate every parameter: "Available in 2025-03-15+."
       - Must explain version-specific behavior throughout
       - Risk of missing version-specific details
       - High maintenance burden for documentation updates

    3. **Version matrix required**

       Every feature needs version tracking:

       ```markdown
       | Feature          | 2024-12-30 | 2025-03-15-preview | 2025-03-15 |
       |------------------|------------|--------------------|-----------  |
       | Hit count        | NO         | YES                | YES         |
       | ISBN hyphens     | Optional   | Required           | Required    |
       | Title sorting    | YES        | NO                 | NO          |
       ```

#### Path-based versioning (documentation maintainability)

- **Score**: ✅ Excellent

- **Advantages**:

    1. **Independent documentation**

       ```bash
       docs/
       ├── v1/
       │   └── api-guide.md  ← Complete v1 documentation
       └── v2/
           └── api-guide.md  ← Complete v2 documentation
       ```

    2. **Clean version separation**

       - v1 docs describe v1 behavior (no version annotations needed)
       - v2 docs describe v2 behavior (standalone)
       - Breaking changes are obvious (different docs)
       - Easy to maintain each version independently

    3. **Simpler updates**

       - Update v2 docs without touching v1
       - No risk of breaking v1 documentation
       - Clear deprecation path (remove v1 directory when sunset)

#### Documentation maintainability winner

The winner is path-based versioning.

---

### 3. User experience

#### Header-based versioning (user experience)

- **Score**: ❌ Poor

User confusion scenarios:

1. **Easy to use the wrong version**

   ```bash
   # User copies v2 documentation example...
   curl https://api.bookhub.com/api/books \
     -H "Authorization: Bearer TOKEN"
   # ...but forgets the header! Now using default (v1)
   ```

2. **Browser testing impossible**

   - Can't test GET endpoints in the browser address bar
   - Must use Postman/curl/code for every test
   - Higher barrier for non-technical publishers

4. **Documentation discovery**

   - Users might read v2 docs but use v1 by accident.
   - No visual cue that they're on the wrong version.
   - The header is "invisible" when browsing the documentation.

#### Path-based versioning (user experience)

- **Score**: ✅ Excellent
- **User-friendly features**:

  1. **Visual clarity**

     ```http
     https://api.bookhub.com/api/v1/books  ← "I'm using v1"
     https://api.bookhub.com/api/v2/books  ← "I'm using v2"
     ```

  2. **Browser testing possible**

     ```http
     # Just paste in browser to test (with token in query for GET)
     https://api.bookhub.com/api/v2/books?token=YOUR_TOKEN
     ```

  3. **Documentation alignment**

    - URL structure matches doc structure
    - /v1/ docs naturally correspond to /api/v1/ endpoints
    - Impossible to confuse which version you're using

#### User experience winner

The winner is path-based versioning.

---

### 4. Migration complexity

#### Header-based versioning (migration complexity)

- **Score**: ⚠️ Acceptable
- **Migration steps**:

    1. Update code to add `api-version: 2025-03-15` header.
    2. Update ISBN formats to include hyphens.
    3. Remove title sorting logic.
    4. Add cover image validation.

- **Risk**: Users might forget step #1 (adding the header) and wonder why their code still works with old behavior.

#### Path-based versioning (migration complexity)

- **Score**: ✅ Good
- **Migration steps**:

    1. Change `/v1/books` to `/v2/books` in code.
    2. Update ISBN formats to include hyphens.
    3. Remove title sorting logic.
    4. Add cover image validation.

- **Advantage**: Step #1 is obvious and hard to miss. If you forget to update the URL, you're clearly still on v1.

#### Migration complexity winner

The winner is path-based versioning.

---

### 5. URL cleanliness

#### Header-based versioning (URL cleanliness)

- **Score**: ✅ Excellent

    ```http
    https://api.bookhub.com/api/books
    https://api.bookhub.com/api/books/123
    ```

Clean, version-agnostic URLs. Aesthetically pleasing.

#### Path-based versioning (URL cleanliness)

- **Score**: ⚠️ Acceptable

    ```http
    https://api.bookhub.com/api/v2/books
    https://api.bookhub.com/api/v2/books/123
    ```

URL includes version number. Slightly longer, but still clean.

### URL cleanliness winner

The winner is header-based versioning (a minor aesthetic preference).

---

### 6. HTTP caching

#### Header-based versioning (HTTP caching)

- **Score**: ⚠️ Acceptable

Standard HTTP caches (CDNs, reverse proxies) don't automatically vary by custom headers. Requires:

```http
Vary: api-version
```

Must configure caching infrastructure to recognize the custom header.

#### Path-based versioning (HTTP caching)

- **Score**: ✅ Excellent

Different URLs = different cache entries. Works with all standard HTTP caches out of the box.

```bash
Cache Key: /api/v1/books/123
Cache Key: /api/v2/books/123
```

No special configuration needed.

### HTTP caching winner

The winner is path-based versioning.

---

### 7. Industry alignment

#### Major API providers

| Company | Versioning Approach | Example |
|---------|---------------------|---------|
| **GitHub** | Path-based | `/api/v3/repos` |
| **Stripe** | Header-based | `Stripe-Version: 2023-10-16` |
| **Twilio** | Path-based | `/v1/Messages` |
| **Twitter** | Path-based | `/2/tweets` |
| **Google Cloud** | Path-based | `/v1/projects` |
| **AWS** | Mixed | Varies by service |
| **Shopify** | Path-based | `/admin/api/2024-01/products` |

#### Analysis

- **Path-based**: 5/7 (71%) - Most common
- **Header-based**: 1/7 (14%) - Stripe is a notable exception.
- **Mixed**: 1/7 (14%)

### Industry alignment winner

The winner is path-based versioning (industry standard).

---

### 8. Tooling support

#### Header-based versioning (tooling support)

- **Score**: ⚠️ Acceptable

  - `OpenAPI 3.0`: Supports custom headers ✅
  - Postman: Requires manual header configuration ⚠️
  - Browser testing: Not possible ❌
  - curl: Requires `-H` flag ⚠️
  - SwaggerUI: Works, but the header must be documented ⚠️

#### Path-based versioning (tooling support)

- **Score**: ✅ Excellent

  - `OpenAPI 3.0`: Native support ✅
  - Postman: Works naturally ✅
  - Browser testing: Possible for GET requests ✅
  - curl: Clean syntax ✅
  - SwaggerUI: Perfect visualization ✅

#### Tooling support winner

The winner is path-based versioning.

---

## Final recommendation

Use URI/Path-based versioning for BookHub Publisher API v2:

```
https://api.bookhub.com/api/v1/books  ← Legacy version
https://api.bookhub.com/api/v2/books  ← Current version
```

### Rationale summary

| Criterion | Path-Based | Header-Based | Weight | Winner |
|-----------|------------|--------------|--------|--------|
| Documentation clarity | ✅ Excellent | ❌ Poor | Critical | Path |
| Maintainability | ✅ Excellent | ❌ Poor | Critical | Path |
| User experience | ✅ Excellent | ❌ Poor | Critical | Path |
| Migration path | ✅ Good | ⚠️ Acceptable | Critical | Path |
| URL cleanliness | ⚠️ Acceptable | ✅ Excellent | Nice-to-have | Header |
| HTTP caching | ✅ Excellent | ⚠️ Acceptable | Important | Path |
| Industry alignment | ✅ Excellent | ⚠️ Acceptable | Important | Path |
| Tooling support | ✅ Excellent | ⚠️ Acceptable | Important | Path |

- **Score**: Path-based versioning wins 7/8 criteria

---

## Trade-offs accepted

By choosing path-based versioning, we accept:

### 1. Longer URLs
**Impact:** Minimal
- URLs are slightly longer (`/v2/books` vs `/books`).
- But still clean and readable.
- Users don't typically type API URLs manually.

### 2. URL changes for major versions
**Impact:** Acceptable
- Major versions already require code changes.
- URL change makes migration explicit and obvious.
- Better than the silent header-based version-switching.

### 3. Multiple URL namespaces
**Impact:** Positive
- Forces a clear separation between versions.
- Makes deprecation explicit (remove /v1/ namespace).
- Aligns with documentation structure.

---

## Implementation impact

### Documentation structure

```
docs.bookhub.com/
├── api/
│   ├── v1/
│   │   ├── getting-started.md
│   │   ├── create-book.md
│   │   ├── get-books.md
│   │   └── finalize-book.md
│   └── v2/
│       ├── getting-started.md
│       ├── create-book.md
│       ├── get-books.md
│       └── finalize-book.md
└── migration-guides/
    └── v1-to-v2.md
```

### Code examples clarity

**v1 Documentation:**
```python
# v1 API - ISBN format flexible
response = requests.post(
    'https://api.bookhub.com/api/v1/books',
    json={'isbn': '9780123456789'}  # No hyphens OK
)
```

**v2 Documentation:**
```python
# v2 API - ISBN format strict
response = requests.post(
    'https://api.bookhub.com/api/v2/books',
    json={'isbn': '978-0-12-345678-9'}  # Hyphens required
)
```

### API gateway configuration

```yaml
routes:
  - path: /api/v1/*
    upstream: bookhub-api-v1
    version: 1.0.0
  
  - path: /api/v2/*
    upstream: bookhub-api-v2
    version: 2.0.0
```

Clean routing configuration.

---

## Alternative considered

### Why not header-based versioning?

**Theoretical advantages:**
- ✅ Cleaner URLs (aesthetic preference)
- ✅ Version-agnostic URL structure
- ✅ Supports fine-grained versioning (date-based)

**Practical disadvantages (deal-breakers):**
- ❌ Documentation becomes complex and confusing
- ❌ High maintenance burden (must track version compatibility throughout)
- ❌ Easy for users to accidentally use the wrong version
- ❌ Browser testing impossible
- ❌ Harder to cache effectively
- ❌ Poor tooling support for testing

### When header-based versioning makes sense

Header-based versioning (especially with date identifiers) works well for:

- **Frequent, granular releases** - SaaS products releasing multiple times daily.
- **Sophisticated API consumers** - Highly technical developers with good tooling.
- **Complex compatibility requirements** - Need to support many version combinations.
- **GraphQL APIs** - Where URL-based versioning doesn't fit the paradigm.

**BookHub doesn't fit these criteria:**

- Releases are quarterly, not daily.
- Users include non-technical publishers.
- Simple version requirements (v1 vs v2).
- REST API, not GraphQL.

---

## Conclusion

For the BookHub Publisher API, I chose **path-based versioning** because it prioritizes:

- **Documentation clarity** over URL aesthetics
- **User experience** over theoretical elegance  
- **Maintainability** over flexibility
- **Simplicity** over sophistication

### Key principle

Path-based versioning is:

- Boring ✅
- Battle-tested ✅
- Well-understood ✅
- Widely supported ✅
- Easy to maintain ✅

And, most importantly, it makes our documentation clear and our users successful.

---

**Document version:** 1.1  
**Author:** Gale James  
**Date:** February 24, 2026  
**Status:** Final Decision
