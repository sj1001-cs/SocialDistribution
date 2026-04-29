[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/bRZK9dqv)


CMPUT404-project-socialdistribution
===================================

See [the web page](https://uofa-cmput404.github.io/general/project.html) for a description of the project.

Make a distributed social network!

# License
- MIT License

# Copyright

The authors claiming copyright, if they wish to be known, can list their names here...

Copyright (c) 2026 Kaylem Nice, Tj Bajaj, Shula Juguilon, Jancis Delfin, Sasieni II Tam

# Group Members

* Tj Bajaj
* Kaylem Nice
* Shula Juguilon
* Jancis Delfin
* Sasieni II Tam

# Development Setup For Group
*Add more as we need

## 1. Clone the Repository

```
 git clone <repo_url>
```

 Always work from the developement branch (only a prod and dev branch according to lab spec):

 ```
 git checkout development
 git pull origin development
 ```
 
## 2. Create & Activate Virtual Env
For Mac / Linux:
```
python3 -m venv venv
source venv/bin/activate
```
## 3. Install Requirements
```
pip install -r requirements.txt
```
## 4. Run Migrations
Create the local SQLite database
```
python manage.py migrate
```
## 5. (Optional) Create Admin user
Don't need it, but helpful insight when coding/debugging
```
python manage.py createsuperuser
```
## 6. Start Dev Server
```
python manage.py runserver
```

## 7. (Optional) Environment Variables

For production (Heroku) or local overrides, create a `.env` file in the project root (never commit it):

```
SECRET_KEY=your-secret-key-here
DEBUG=False
DATABASE_URL=postgres://user:pass@host:5432/dbname
```

- If `DATABASE_URL` is set, the app connects to PostgreSQL (Heroku Postgres). Otherwise it defaults to local `db.sqlite3`.
- `DEBUG` defaults to `True` in development.
- A random `SECRET_KEY` is used as the fallback in development only.

## 8. Some Git Rules to Help Us All Out
1. Verify before each push or commit that you do not have any:
- venv/
- __pycache__/
- *.pyc
- db.sqlite3
- .env
- Or IDE Folders (.vscode, etc.)
2. Pull latest code before coding
3. Commit Frequently (even if changes are small), helps reduce conflicts
4. Don't force push, don't rebase


## Architecture Overview

Distributed objects are identified by Fully Qualified IDs (FQID), which are full URLs. These objects include:
- Authors
- Entries
- Comments
- Likes

Each object has two URL types:
- **API URL** (`id` field): the stable REST API endpoint (e.g. `https://node/api/authors/<uuid>/entries/<uuid>/`)
- **Web URL** (`web` field): the human-facing HTML frontend page (e.g. `https://node/authors/<uuid>/entries/<uuid>/`)

### GitHub Activity Sync

If an author sets their GitHub profile URL, the stream page automatically polls the GitHub Events API and imports recent public activity as `PUBLIC` entries. Supported event types include `PushEvent`, `CreateEvent`, `WatchEvent`, `ForkEvent`, `IssuesEvent`, `PullRequestEvent`, and `PublicEvent`. Each event is only imported once (tracked by event ID).

### Soft Deletes

Entries are never physically removed from the database. A `DELETE` request sets `is_deleted=True` on the entry, which hides it from all API responses and the UI. Node admins can still see deleted entries in the Django admin panel.

### Admin Approval Workflow

New author registrations are pending by default. A node admin must approve an account before the author can log in. The Django admin panel provides list/filter views for Authors, Entries, Comments, and Likes.


# SocialDistribution API Documentation

## Overview

This document describes the RESTful API for **Whole Lotta Posts** (SocialDistribution), a distributed social networking platform. The API follows an inbox-push model where nodes communicate by POSTing objects (entries, likes, comments, follow requests) to the relevant author's inbox on a remote node.

- **Local Base URL:** `http://127.0.0.1:8000/api/`
- **Production Base URL:** `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/`
- **Authentication:**
  - **Local endpoints** (`[local]`): Session authentication (cookies) or HTTP Basic Auth. Used by your own frontend.
  - **Remote endpoints** (`[remote]`): HTTP Basic Auth only — used by other nodes connecting to this one.
- **Content-Type:** All request and response bodies are `application/json` unless noted otherwise.

## Heroku INFO
- **Hostname:** [https://jd-node-b-9c1da1a35b21.herokuapp.com/](https://jd-node-b-9c1da1a35b21.herokuapp.com)  
- **CNAME:** None  
- **IP Addresses:**  
  - 54.224.34.30 
  - 54.243.129.215  
  - 34.201.81.34
  - 54.208.186.182

---

## Raw Heroku Input
```text
PS C:\Users\Tejj> nslookup jd-node-b-9c1da1a35b21.herokuapp.com
Server:  name.ualberta.ca
Address:  129.128.5.233

Non-authoritative answer:
Name:    jd-node-b-9c1da1a35b21.herokuapp.com
Addresses:  54.224.34.30
          54.243.129.215
          34.201.81.34
          54.208.186.182

PS C:\Users\Tejj>
```

---

## Accounts and Auth

HTTP Basic Auth credentials for connecting to this node. Use these when making `[remote]` requests to any endpoint.

| Label | Username | Password |
|---|---|---|
| Super User | `jancis1` | `jancis1` |
| Normal User | `yuan1` | `yuan1` |
---

## User Story Coverage Summary

Each endpoint below is tagged with the user stories it satisfies. The legend is:

- ✅ **Satisfied** — this endpoint directly implements the user story at the API level.
- ⚠️ **Part 3-5** — the user story is partially implemented (Part 3-5 work).
- ❌ **Not applicable** — the user story is satisfied by the UI or Django code, not this endpoint.

User stories that have **no API** (e.g. admin approval flow, database constraints, GitHub activity import) are tested via Django code directly and are not listed under any endpoint.

| User Story | Endpoint(s) |
|---|---|
| Consistent author identity / stable URLs | `GET /api/authors/<uuid>/`, `GET /api/authors/<uuid>/entries/<uuid>/` |
| Host multiple authors | `GET /api/authors/` |
| Public profile page | `GET /api/authors/<uuid>/` |
| Edit profile (name, description, picture, GitHub) | `PUT /api/authors/<uuid>/` |
| Make entries (plain text, markdown, image) | `POST /api/authors/<uuid>/entries/` |
| Edit entries | `PUT /api/authors/<uuid>/entries/<uuid>/` |
| Delete entries (soft delete) | `DELETE /api/authors/<uuid>/entries/<uuid>/` |
| Entry visibility (public / unlisted / friends-only) | `POST` and `GET /api/authors/<uuid>/entries/` |
| Stream shows correct entries by visibility | `GET /api/authors/<uuid>/entries/` |
| Shareable link to public/unlisted entry | `GET /api/entries/<fqid>/` |
| Image entries + use in Markdown | `GET /api/authors/<uuid>/entries/<uuid>/image` |
| Follow local authors | `PUT /api/authors/<uuid>/following/<fqid>/` |
| Approve / deny follow requests | `PUT` and `DELETE /api/authors/<uuid>/followers/<fqid>/` |
| See pending follow requests | `GET /api/authors/<uuid>/follow_requests/` |
| Unfollow an author | `DELETE /api/authors/<uuid>/following/<fqid>/` |
| Mutual follow = friends | Combination of followers + following endpoints |
| Comment on entries | `POST /api/authors/<uuid>/entries/<uuid>/comments/` |
| Like entries | `POST /api/authors/<uuid>/entries/<uuid>/like/` |
| Like comments | `POST /api/authors/<uuid>/entries/<uuid>/comments/<uuid>/like/` |
| See likes on entries | `GET /api/authors/<uuid>/entries/<uuid>/likes/` |
| Inbox receives remote entries/likes/comments/follows | `POST /api/authors/<uuid>/inbox/` |

---

## Table of Contents

1. [Authors](#1-authors)
2. [Entries](#2-entries)
3. [Image Entries](#3-image-entries)
4. [Comments](#4-comments)
5. [Likes](#5-likes)
6. [Followers](#6-followers)
7. [Following](#7-following)
8. [Follow Requests](#8-follow-requests)
9. [Inbox](#9-inbox)
10. [Object Schemas](#10-object-schemas)

---

## 1. Authors

### 1.1 List All Authors

**`GET /api/authors/`** `[local, remote]`

Returns a paginated list of all approved local authors on this node.

**Satisfies User Stories:**
- ✅ *"As a node admin, I want to host multiple authors on my node."* — exposes all approved authors on the node.
- ✅ *"As an author, I should be able to browse the public entries of everyone."* — discovering authors is the first step to browsing all public content.
- ✅ *"As an author, I want a consistent identity per node."* — each author is returned with a stable FQID (`id` field) that will not change.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET` that any client or future Android app can call.

**When to use:** Browse all authors registered on the node. Remote nodes can call this to discover local authors.

**Query Parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number (1-based) |
| `size` | integer | `10` | Number of authors per page |

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/?page=1&size=5
```

**Example Response:** `200 OK`
```json
{
  "type": "authors",
  "authors": [
    {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
      "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
      "displayName": "Alice Smith",
      "github": "https://github.com/alicesmith",
      "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
      "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
    },
    {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
      "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
      "displayName": "Bob Jones",
      "github": "https://github.com/bobjones",
      "profileImage": "https://i.imgur.com/abc123.jpeg",
      "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|---|---|---|
| `type` | string | Always `"authors"` |
| `authors` | array | Array of [Author Objects](#author-object) |

---

### 1.2 Get or Update a Single Author

**`GET /api/authors/<author_uuid>/`** `[local, remote]`
**`PUT /api/authors/<author_uuid>/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want a public page with my profile information."* — `GET` returns the author's full public profile.
- ✅ *"As an author, I want to be able to edit my profile: name, description, picture, and GitHub."* — `PUT` allows updating those fields.
- ✅ *"As an author, I want a consistent identity per node."* — the `id` field is always the stable FQID and never changes.
- ✅ *"As a node admin, I want to be able to add, modify, and delete authors."* — admins can use `PUT` to modify author profiles directly.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET`/`PUT` on a resource URL.
- ❌ *"As an author, I want to be able to use my web browser to manage my profile."* — satisfied by the web UI calling this endpoint, not the endpoint itself.

**When to use:**
- `GET`: Retrieve any approved author's public profile. No authentication required.
- `PUT`: Update your own profile. Must be authenticated as the author matching `author_uuid`.

**URL Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `author_uuid` | UUID | The serial (UUID) of the author |

**PUT Request Body** (all fields optional — only sent fields are updated):

| Field | Type | Description |
|---|---|---|
| `displayName` | string | The author's display name |
| `description` | string | A short bio or description |
| `github` | string | Full URL to the author's GitHub |
| `profileImage` | string | Full URL to the author's profile image |

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/
```

**Example GET Response:** `200 OK`
```json
{
  "type": "author",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
  "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
  "displayName": "Alice Smith",
  "github": "https://github.com/alicesmith",
  "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
  "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
}
```

**Example PUT Request:**
```
PUT https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/
Content-Type: application/json

{
  "displayName": "Alice S.",
  "github": "https://github.com/alicesnewhandle"
}
```

**Example PUT Response:** `200 OK`
```json
{
  "type": "author",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
  "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
  "displayName": "Alice S.",
  "github": "https://github.com/alicesnewhandle",
  "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
  "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated, or trying to edit another author's profile |
| `404` | Author not found or not approved |

---

## 2. Entries

### 2.1 List or Create Entries

**`GET /api/authors/<author_uuid>/entries/`** `[local, remote]`
**`POST /api/authors/<author_uuid>/entries/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to make entries."* — `POST` creates a new entry for the authenticated author.
- ✅ *"As an author, entries I make can be in CommonMark."* — `POST` accepts `contentType: "text/markdown"`.
- ✅ *"As an author, entries I make can be in simple plain text."* — `POST` accepts `contentType: "text/plain"`.
- ✅ *"As an author, entries I create can be images."* — `POST` accepts `contentType: "image/png;base64"` and `"image/jpeg;base64"`.
- ✅ *"As an author, other authors cannot modify my entries."* — `POST` is restricted to the authenticated author only.
- ✅ *"As an author, I want to be able to make my entries public, unlisted, or friends-only."* — the `visibility` field accepts `"PUBLIC"`, `"UNLISTED"`, `"FRIENDS"`.
- ✅ *"As an author, entries I create should always be visible to me until they are deleted."* — authenticated as the author, `GET` returns all non-deleted entries regardless of visibility.
- ✅ *"As an author, I want my stream page to not show me entries that have been deleted."* — `is_deleted=False` filter applied on all `GET` queries.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET`/`POST` on the entries collection URL.

**When to use:**
- `GET`: Fetch all visible entries from a given author. Visibility depends on the requester's relationship to the author.
- `POST`: Create a new entry as the authenticated author.

**Visibility Rules for GET:**

| Requester relationship | Entries returned |
|---|---|
| Not authenticated | `PUBLIC` only |
| Authenticated as author | `PUBLIC`, `UNLISTED`, `FRIENDS` |
| Authenticated as friend | `PUBLIC`, `UNLISTED`, `FRIENDS` |
| Authenticated as follower | `PUBLIC`, `UNLISTED` |

**Query Parameters (GET):**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number (1-based) |
| `size` | integer | `10` | Entries per page |

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/?page=1&size=5
```

**Example GET Response:** `200 OK`
```json
{
  "type": "entries",
  "page_number": 1,
  "size": 5,
  "count": 42,
  "src": [
    {
      "type": "entry",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
      "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/",
      "title": "Hello World",
      "description": "My first post",
      "contentType": "text/plain",
      "content": "This is my first entry on SocialDistribution!",
      "visibility": "PUBLIC",
      "published": "2026-03-01T12:00:00+00:00",
      "author": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
        "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
        "displayName": "Alice Smith",
        "github": "https://github.com/alicesmith",
        "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
        "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
      }
    }
  ]
}
```

**POST Request Body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `title` | string | No | `""` | Title of the entry |
| `description` | string | No | `""` | Short summary of the entry |
| `contentType` | string | Yes | `text/plain` | One of: `text/plain`, `text/markdown`, `image/png;base64`, `image/jpeg;base64` |
| `content` | string | Yes | — | The body of the entry (or base64-encoded image data) |
| `visibility` | string | No | `PUBLIC` | One of: `PUBLIC`, `UNLISTED`, `FRIENDS` |

**Example POST Request:**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/
Content-Type: application/json

{
  "title": "My Markdown Post",
  "description": "A post with formatting",
  "contentType": "text/markdown",
  "content": "# Hello\nThis is **bold** text.",
  "visibility": "PUBLIC"
}
```

**Example POST Response:** `201 Created`
```json
{
  "type": "entry",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/ccccdddd-0000-0000-0000-000000000020",
  "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/ccccdddd-0000-0000-0000-000000000020/",
  "title": "My Markdown Post",
  "description": "A post with formatting",
  "contentType": "text/markdown",
  "content": "# Hello\nThis is **bold** text.",
  "visibility": "PUBLIC",
  "published": "2026-03-02T08:30:00+00:00",
  "author": {
    "type": "author",
    "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
    "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
    "displayName": "Alice Smith",
    "github": "https://github.com/alicesmith",
    "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
    "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
  }
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `400` | Invalid or missing required fields |
| `403` | Not authenticated or not this author |

---

### 2.2 Get, Update, or Delete a Single Entry

**`GET /api/authors/<author_uuid>/entries/<entry_uuid>/`** `[local, remote]`
**`PUT /api/authors/<author_uuid>/entries/<entry_uuid>/`** `[local]`
**`DELETE /api/authors/<author_uuid>/entries/<entry_uuid>/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to edit my entries locally."* — `PUT` updates any subset of entry fields without deleting and re-creating.
- ✅ *"As an author, I want to delete my own entries locally."* — `DELETE` soft-deletes the entry; it stays in the DB but disappears from the API and UI.
- ✅ *"As a node admin, I want deleted entries to stay in the database and only be removed from the UI and API."* — soft delete (`is_deleted=True`) keeps the row but hides it from `GET`.
- ✅ *"As an author, I don't want anyone except the node admin to see my deleted entries."* — deleted entries return `404` on `GET`.
- ✅ *"As an author, other authors cannot modify my entries."* — `PUT` and `DELETE` check that the viewer's FQID matches the author's FQID.
- ✅ *"As an author, I want everyone to be able to see my public and unlisted entries if they have a link."* — unauthenticated `GET` returns `PUBLIC` and `UNLISTED` entries by their direct URL.
- ✅ *"As an author, I don't want anyone who isn't a friend to see my friends-only entries."* — `GET` enforces `can_view_entry()` which checks friendship status.
- ✅ *"As a reader, I can get a link to a public or unlisted entry."* — the stable `id` (FQID) in the response is the permanent shareable link.

**When to use:**
- `GET`: View a single entry. Returns `404` if the requester cannot see it.
- `PUT`: Edit an existing entry. Only the author can do this.
- `DELETE`: Soft-delete an entry. Stays in the database but hidden from all API responses and the UI.

**URL Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `author_uuid` | UUID | The serial (UUID) of the author |
| `entry_uuid` | UUID | The serial (UUID) of the entry |

**PUT Request Body** (all fields optional — only sent fields are updated):

| Field | Type | Description |
|---|---|---|
| `title` | string | New title |
| `description` | string | New description |
| `content` | string | New content body |
| `contentType` | string | New content type |
| `visibility` | string | New visibility level |

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/
```

**Example GET Response:** `200 OK`
```json
{
  "type": "entry",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
  "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/",
  "title": "Hello World",
  "description": "My first post",
  "contentType": "text/plain",
  "content": "This is my first entry on SocialDistribution!",
  "visibility": "PUBLIC",
  "published": "2026-03-01T12:00:00+00:00",
  "author": {
    "type": "author",
    "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
    "displayName": "Alice Smith"
  }
}
```

**Example DELETE Response:** `200 OK`
```json
{
  "success": true
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated or not the author |
| `404` | Entry not found, deleted, or not visible to the requester |

---

### 2.3 Get Entry by FQID

**`GET /api/entries/<path:entry_fqid>/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As a reader, I can get a link to a public or unlisted entry."* — any caller with the full URL can retrieve it directly.
- ✅ *"As an author, I want a consistent identity per node."* — FQIDs are permanent and this endpoint resolves them reliably.
- ✅ *"As an author, I don't want anyone who isn't a friend to see my friends-only entries."* — `can_view_entry()` is still enforced when looking up by FQID.

**When to use:** Retrieve an entry when you have its full URL (FQID) but not its author UUID. Useful when processing inbox items that reference entries by their full URL. The `entry_fqid` must be percent-encoded when embedded in a URL.

**Note:** Uses Django's `<path:>` URL converter because FQIDs contain forward slashes.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/entries/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F3a1b2c3d-0000-0000-0000-000000000001%2Fentries%2Faaaabbbb-0000-0000-0000-000000000010/
```

**Example Response:** `200 OK` — same shape as a single [Entry Object](#entry-object).

**Error Responses:**

| Status | Condition |
|---|---|
| `404` | Entry not found, deleted, or not visible |

---

## 3. Image Entries

Image entries are regular entries whose `contentType` is `image/png;base64` or `image/jpeg;base64`. The `content` field holds raw base64-encoded image data. These endpoints decode that base64 and return the raw binary image, allowing the URL to be used directly in an `<img>` tag or in Markdown.

**Satisfies User Stories (entire Image Entries section):**
- ✅ *"As an author, entries I create can be images."* — image entries are stored as base64 and these endpoints serve them as real binary images.
- ✅ *"As an author, entries I create that are in CommonMark can link to images."* — embed the image URL from this endpoint in Markdown: `![alt](https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/entries/<uuid>/image)`.
- ✅ *"As a node admin, images can be hosted on my node."* — images are stored in the database as entries and served via this endpoint with no external CDN needed.
- ✅ *"As an author, I don't want anyone who isn't a friend to see my friends-only entries and images."* — visibility is enforced; returns `404` if the requester can't see the entry.

### 3.1 Get Image Entry by Serial

**`GET /api/authors/<author_uuid>/entries/<entry_uuid>/image`** `[local, remote]`

**When to use:** Embed an image in HTML or Markdown. Visibility rules still apply.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/eeeeffff-0000-0000-0000-000000000030/image
```

**Example Response:** `200 OK`
```
Content-Type: image/png
<binary image bytes>
```

**Error Responses:**

| Status | Condition |
|---|---|
| `404` | Entry not found, not an image, or not visible to requester |
| `500` | Stored base64 data is corrupted |

---

### 3.2 Get Image Entry by FQID

**`GET /api/entries/<path:entry_fqid>/image`** `[local, remote]`

**When to use:** Same as above but when you only have the FQID of the entry, not the author's UUID. Useful for resolving remote image references.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/entries/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F3a1b2c3d%2Fentries%2Feeeeffff/image
```

**Example Response:** `200 OK` — binary image data with correct `Content-Type` header.

---

## 4. Comments

### 4.1 List or Create Comments on an Entry

**Deployment note:** In all deployed examples, comment object IDs, entry IDs, and author IDs must use the node's real Heroku `service_address`, not localhost URLs.

**This node's deployed comment endpoint patterns:**
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/comments/`
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/likes/`
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/like/`

**`GET /api/authors/<author_uuid>/entries/<entry_uuid>/comments/`** `[local, remote]`
**`POST /api/authors/<author_uuid>/entries/<entry_uuid>/comments/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to comment on entries that I can access."* — `POST` creates a comment if the viewer can see the entry.
- ✅ *"As an author, when someone sends me a public entry I want to see the likes."* — `GET` returns all comments on the entry.
- ✅ *"As an author, comments on my friends-only entries are visible only to my friends and the comment's author."* — `GET` returns `404` if the requester cannot view the entry.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET`/`POST` on the comments collection URL.

**When to use:**
- `GET`: Retrieve all comments on a specific entry. Returns `404` if the requester cannot view the entry.
- `POST`: Post a new comment on an entry as the currently authenticated author. If the entry belongs to a remote author, the comment is also automatically POSTed to that remote author's inbox via HTTP Basic Auth.

**Remote inbox push (POST only):** After saving the comment locally, if the entry's author is on a different node and that node is configured in our `Node` table, our server automatically POSTs the comment object to the remote author's inbox at `{entry_author_fqid}/inbox/`. This push is best-effort — if it fails, the comment is still saved locally and `201` is still returned.

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/comments/
```

**Example GET Response:** `200 OK`
```json
{
  "type": "comments",
  "comments": [
    {
      "type": "comment",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/commented/11112222-0000-0000-0000-000000000040",
      "entry": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
      "author": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
        "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
        "displayName": "Bob Jones",
        "github": "https://github.com/bobjones",
        "profileImage": "https://i.imgur.com/abc123.jpeg",
        "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
      },
      "comment": "Great post, Alice!",
      "contentType": "text/plain",
      "published": "2026-03-01T14:00:00+00:00"
    }
  ]
}
```

**POST Request Body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `comment` | string | Yes | — | The comment text |
| `contentType` | string | No | `text/plain` | One of: `text/plain`, `text/markdown` |

**Example POST Request:**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/comments/
Content-Type: application/json

{
  "comment": "Really enjoyed reading this!",
  "contentType": "text/plain"
}
```

**Example POST Response:** `201 Created`
```json
{
  "type": "comment",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/commented/33334444-0000-0000-0000-000000000050",
  "entry": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
  "author": {
    "type": "author",
    "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
    "displayName": "Bob Jones"
  },
  "comment": "Really enjoyed reading this!",
  "contentType": "text/plain",
  "published": "2026-03-02T09:00:00+00:00"
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `400` | Invalid request body |
| `403` | Not authenticated |
| `404` | Entry not found or not visible |

---

### 4.2 Get Likes on a Comment

**`GET /api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/likes/`** `[local, remote]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to like comments that I can access."* — returns all likes on a comment for UI display.
- ✅ *"As an author, comments on my friends-only entries are visible only to my friends and the comment's author."* — returns `404` if the requester cannot view the parent entry.

**When to use:** Retrieve all likes on a specific comment. Returns `404` if the requester can't view the parent entry.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/comments/11112222-0000-0000-0000-000000000040/likes/
```

**Example Response:** `200 OK`
```json
{
  "type": "likes",
  "likes": [
    {
      "type": "like",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/liked/55556666-0000-0000-0000-000000000060",
      "author": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
        "displayName": "Bob Jones"
      },
      "object": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/commented/11112222-0000-0000-0000-000000000040",
      "published": "2026-03-01T15:00:00+00:00"
    }
  ]
}
```

---

### 4.3 Like or Unlike a Comment

**`POST /api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/like/`** `[local]`
**`DELETE /api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/like/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to like comments that I can access."* — `POST` creates a like on the comment for the authenticated author.
- ✅ *"As an author, comments on my friends-only entries are visible only to my friends and the comment's author."* — liking is gated behind `can_view_entry()`.

**When to use:** Toggle the authenticated author's like on a comment. Both methods are idempotent — `POST` when already liked returns the existing like without creating a duplicate; `DELETE` when not liked returns success without error.

**Remote inbox push:** After creating or deleting the like, if the entry author is on a different node configured in our `Node` table, our server POSTs to the remote author's inbox:
- On `POST`: sends the full like object with `"type": "like"`
- On `DELETE`: sends `{"type": "unlike", "object": "<comment_fqid>", "author": {...}}`

This push is best-effort — failures are silently swallowed and do not affect the local response.

**Example POST Request:**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/comments/11112222-0000-0000-0000-000000000040/like/
```

**Example POST Response:** `201 Created` (new like) or `200 OK` (already liked)
```json
{
  "liked": true,
  "count": 4,
  "like": {
    "type": "like",
    "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/liked/77778888-0000-0000-0000-000000000070",
    "author": {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
      "displayName": "Bob Jones"
    },
    "object": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/commented/11112222-0000-0000-0000-000000000040",
    "published": "2026-03-02T09:30:00+00:00"
  }
}
```

**Example DELETE Request:**
```
DELETE https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/comments/11112222-0000-0000-0000-000000000040/like/
```

**Example DELETE Response:** `200 OK`
```json
{
  "liked": false,
  "count": 3
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated |
| `404` | Entry or comment not visible |

---

## 5. Likes

### 5.1 Get Likes on an Entry

**Deployment note:** In all deployed examples, like object IDs and liked object URLs must use the node's real Heroku `service_address`, not localhost URLs.

**This node's deployed like endpoint patterns:**
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/likes/`
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/like/`
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/entries/<entry_uuid>/comments/<comment_uuid>/like/`

**`GET /api/authors/<author_uuid>/entries/<entry_uuid>/likes/`** `[local, remote]`

**Satisfies User Stories:**
- ✅ *"As an author, when someone sends me a public entry I want to see the likes."* — returns all likes on an entry so the UI can display the count and who liked it.
- ✅ *"As an author, I don't want anyone who isn't a friend to see my friends-only entries and images."* — returns `404` if the requester cannot view the entry.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET` on the likes sub-resource.

**When to use:** Retrieve all likes on a specific entry. Returns `404` if the requester does not have permission to see the entry.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/likes/
```

**Example Response:** `200 OK`
```json
{
  "type": "likes",
  "likes": [
    {
      "type": "like",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/liked/99990000-0000-0000-0000-000000000080",
      "author": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
        "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
        "displayName": "Bob Jones",
        "github": "https://github.com/bobjones",
        "profileImage": "https://i.imgur.com/abc123.jpeg",
        "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
      },
      "object": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
      "published": "2026-03-01T16:00:00+00:00"
    }
  ]
}
```

---

### 5.2 Like or Unlike an Entry

**`POST /api/authors/<author_uuid>/entries/<entry_uuid>/like/`** `[local]`
**`DELETE /api/authors/<author_uuid>/entries/<entry_uuid>/like/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to like entries that I can access."* — `POST` creates a like on the entry for the authenticated author.
- ✅ *"As an author, I don't want anyone who isn't a friend to see my friends-only entries and images."* — liking is gated behind `can_view_entry()`.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `POST`/`DELETE` on a like sub-resource.

**When to use:** Toggle the authenticated author's like on an entry. Both methods are idempotent. The response always includes the current `liked` state and total `count` for easy UI consumption.

**Remote inbox push:** After creating or deleting the like, if the entry author is on a different node configured in our `Node` table, our server POSTs to the remote author's inbox:
- On `POST`: sends the full like object with `"type": "like"`
- On `DELETE`: sends `{"type": "unlike", "object": "<entry_fqid>", "author": {...}}`

This push is best-effort — failures are silently swallowed and do not affect the local response.

**Example POST Request:**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/like/
```

**Example POST Response:** `201 Created` (new like) or `200 OK` (already liked)
```json
{
  "liked": true,
  "count": 12,
  "like": {
    "type": "like",
    "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002/liked/aaaabbbb-0000-0000-0000-000000000090",
    "author": {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
      "displayName": "Bob Jones"
    },
    "object": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
    "published": "2026-03-02T10:00:00+00:00"
  }
}
```

**Example DELETE Request:**
```
DELETE https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010/like/
```

**Example DELETE Response:** `200 OK`
```json
{
  "liked": false,
  "count": 11
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated |
| `404` | Entry not found or not visible |

---

## 6. Followers

### 6.1 List All Followers

**`GET /api/authors/<author_uuid>/followers/`** `[local, remote]`

**Satisfies User Stories:**
- ✅ *"As an author, my node will know about my followers, who I am following, and my friends."* — returns the full list of current followers our node is tracking.
- ✅ *"As an author, if I am following another author, and they are following me, I want us to be considered friends."* — the followers list combined with the following list determines mutual friendship.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET` on the followers collection.

**When to use:** Get a list of all authors currently following `author_uuid`. No authentication required. For remote authors not stored locally, a minimal placeholder object `{"type": "author", "id": "<fqid>"}` is returned.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/followers/
```

**Example Response:** `200 OK`
```json
{
  "type": "followers",
  "followers": [
    {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
      "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
      "displayName": "Bob Jones",
      "github": "https://github.com/bobjones",
      "profileImage": "https://i.imgur.com/abc123.jpeg",
      "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
    }
  ]
}
```

---

### 6.2 Check, Accept, or Remove a Follower

**`GET /api/authors/<author_uuid>/followers/<path:foreign_fqid>/`** `[local, remote]`
**`PUT /api/authors/<author_uuid>/followers/<path:foreign_fqid>/`** `[local]`
**`DELETE /api/authors/<author_uuid>/followers/<path:foreign_fqid>/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to be able to approve or deny other authors following me."* — `PUT` accepts a pending follow request; `DELETE` denies it or removes an existing follower.
- ✅ *"As an author, I want to know if I have follow requests."* — `GET` on a specific FQID tells you whether they are already a follower.
- ✅ *"As an author, I want to unfriend other authors by unfollowing them."* — `DELETE` removes them as a follower, breaking the mutual-follow friendship.
- ✅ *"As an author, if I am following another author, and they are following me, I want us to be considered friends."* — accepting via `PUT` creates the `Follower` row; when both sides accept, both nodes see each other as friends.

**When to use:**
- `GET`: Check whether `foreign_fqid` is currently a follower of `author_uuid`. Returns the author object if they are, `404` if not.
- `PUT`: Accept a pending follow request from `foreign_fqid`. Must be authenticated as `author_uuid`. Returns `404` if no pending request exists.
- `DELETE`: Deny a pending follow request, or remove an existing follower. Must be authenticated as `author_uuid`. Returns `404` if neither a follower nor a pending request exists.

**URL Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `author_uuid` | UUID | The local author being followed |
| `foreign_fqid` | string | The full URL (FQID) of the follower, **percent-encoded** in the URL |

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/followers/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F4b2c3d4e-0000-0000-0000-000000000002/
```

**Example GET Response (follower exists):** `200 OK`
```json
{
  "type": "author",
  "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
  "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
  "displayName": "Bob Jones",
  "github": "https://github.com/bobjones",
  "profileImage": "https://i.imgur.com/abc123.jpeg",
  "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
}
```

**Example PUT Request:**
```
PUT https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/followers/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F4b2c3d4e-0000-0000-0000-000000000002/
```

**Example PUT Response:** `200 OK`
```json
{
  "success": true,
  "message": "Follow request accepted"
}
```

**Example DELETE Request:**
```
DELETE https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/followers/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F4b2c3d4e-0000-0000-0000-000000000002/
```

**Example DELETE Response:** `200 OK`
```json
{
  "success": true,
  "message": "Follower removed"
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated or not `author_uuid` |
| `404` | Follower/request not found |

---

## 7. Following

### 7.1 List Authors Being Followed

**`GET /api/authors/<author_uuid>/following/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, my node will know about my followers, who I am following, and my friends."* — returns the full list of authors this author is currently following.
- ✅ *"As an author, if I am following another author, and they are following me, I want us to be considered friends."* — the following list combined with the followers list determines mutual friendship.
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET` on the following collection.

**When to use:** Get the list of authors that `author_uuid` is currently following. Must be authenticated as `author_uuid`.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/following/
```

**Example Response:** `200 OK`
```json
{
  "type": "following",
  "following": [
    {
      "type": "author",
      "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
      "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
      "displayName": "Bob Jones",
      "github": "https://github.com/bobjones",
      "profileImage": "https://i.imgur.com/abc123.jpeg",
      "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
    }
  ]
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated or not `author_uuid` |

---

### 7.2 Check, Send, or Cancel a Follow

**`GET /api/authors/<author_uuid>/following/<path:foreign_fqid>/`** `[local]`
**`PUT /api/authors/<author_uuid>/following/<path:foreign_fqid>/`** `[local]`
**`DELETE /api/authors/<author_uuid>/following/<path:foreign_fqid>/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to follow local authors."* — `PUT` sends a follow request to `foreign_fqid`.
- ✅ *"As an author, I want to unfollow authors I am following."* — `DELETE` removes the `Follower` row on the target's side.
- ✅ *"As an author, I want to unfriend other authors by unfollowing them."* — unfollowing via `DELETE` breaks the mutual follow, removing friendship status.
- ⚠️ *"As an author, I want to follow remote authors. ⧟ Part 3-5 only."* — `PUT` POSTs the follow request to the remote node's inbox when the target is remote.

**When to use:**
- `GET`: Check if `author_uuid` is currently following `foreign_fqid`. Returns the author object if yes, `404` if no. Must be authenticated as `author_uuid`.
- `PUT`: Send a follow request from `author_uuid` to `foreign_fqid`. Creates a `FollowRequest` if one doesn't already exist. If the target is on a remote node, also POSTs the follow request to that author's inbox.
- `DELETE`: Unfollow `foreign_fqid`. Removes the `Follower` row on the target's side. Must be authenticated as `author_uuid`.

**Note:** `foreign_fqid` must be percent-encoded in the URL since it is a full URL containing slashes and colons.

**Example PUT Request:**
```
PUT https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/following/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F4b2c3d4e-0000-0000-0000-000000000002/
```

**Example PUT Response:** `201 Created` (new request) or `200 OK` (already sent)
```json
{
  "success": true,
  "message": "Follow request sent"
}
```

**Example DELETE Request:**
```
DELETE https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/following/https%3A%2F%2Fjd-node-b-9c1da1a35b21.herokuapp.com%2Fapi%2Fauthors%2F4b2c3d4e-0000-0000-0000-000000000002/
```

**Example DELETE Response:** `200 OK`
```json
{
  "success": true,
  "message": "Unfollowed"
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated or not `author_uuid` |
| `404` | Target author not found (PUT), or not following (DELETE) |

---

## 8. Follow Requests

### 8.1 List Pending Follow Requests

**`GET /api/authors/<author_uuid>/follow_requests/`** `[local]`

**Satisfies User Stories:**
- ✅ *"As an author, I want to know if I have follow requests."* — lists all `PENDING` follow requests addressed to this author.
- ✅ *"As an author, I want to be able to approve or deny other authors following me."* — the UI uses this list to show who is waiting; accepting/denying is done via [6.2 Followers](#62-check-accept-or-remove-a-follower).
- ✅ *"As a node admin, I want a RESTful interface for most operations."* — standard REST `GET` on the follow requests collection.

**When to use:** Retrieve all pending follow requests for `author_uuid`. Must be authenticated as `author_uuid`.

**Example Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/follow_requests/
```

**Example Response:** `200 OK`
```json
{
  "type": "follow_requests",
  "follow_requests": [
    {
      "type": "follow",
      "status": "PENDING",
      "actor": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/4b2c3d4e-0000-0000-0000-000000000002",
        "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
        "displayName": "Bob Jones",
        "github": "https://github.com/bobjones",
        "profileImage": "https://i.imgur.com/abc123.jpeg",
        "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/4b2c3d4e-0000-0000-0000-000000000002"
      },
      "object": {
        "type": "author",
        "id": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
        "host": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/",
        "displayName": "Alice Smith",
        "github": "https://github.com/alicesmith",
        "profileImage": "https://i.imgur.com/k7XVwpB.jpeg",
        "web": "https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/3a1b2c3d-0000-0000-0000-000000000001"
      }
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|---|---|---|
| `type` | string | Always `"follow_requests"` |
| `follow_requests` | array | Array of [Follow Request Objects](#follow-request-object) |

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | Not authenticated or not `author_uuid` |

---

## 9. Inbox

### 9.1 Read, Receive, or Clear Inbox

**`GET /api/authors/<author_uuid>/inbox/`** `[local — owner only]`
**`POST /api/authors/<author_uuid>/inbox/`** `[remote]`
**`DELETE /api/authors/<author_uuid>/inbox/`** `[local — owner only]`

**Satisfies User Stories:**
- ✅ *"As an author, I want a stream which shows all the entries I should know about."* — the inbox is the mechanism by which remote entries, likes, and comments are collected and later surfaced in the stream.
- ✅ *"As an author, I want my stream page to show me all the unlisted and friends-only entries of all the authors I follow."* — those entries arrive via `POST` to inbox from the remote node.
- ✅ *"As an author, I want to know if I have follow requests."* — follow request objects from remote authors are delivered to the inbox via `POST` so we can queue them for approval.
- ✅ *"As an author, when someone sends me a public entry I want to see the likes."* — likes from remote nodes are pushed into the inbox via `POST`.
- ✅ *"As an author, I want to comment on entries that I can access."* — comments from remote authors arrive at the entry-owner's inbox via `POST`.
- ⚠️ *"As an author, I want my node to send my entries to my remote followers and friends. ⧟ Part 3-5 only."* — our node POSTs entries to remote followers' inboxes. The endpoint to receive them is implemented here.
- ⚠️ *"As a node admin, node to node connections can be authenticated with HTTP Basic Auth. ⧟ Part 3-5 only."* — `POST` to inbox enforces HTTP Basic Auth from the remote node.
- ❌ *"As an author, I want to be able to use my web-browser to manage my entries."* — the inbox is not a UI feature; it is server-to-server only.

**Important:** The inbox is **not a UI feature** — it is a server-to-server communication channel. Remote nodes push entries, likes, comments, and follow requests into an author's inbox via `POST`. The author's node stores the payload and processes supported item types into local models. `GET` and `DELETE` are owner-only.

**Deployment note:** For cross-node testing, all inbox delivery examples and remote node configurations must use a real Heroku `service_address`. Any inbox payload referencing comments, likes, entries, or authors should contain absolute FQIDs based on the deployed node URL.

**This node's inbox endpoint:**
- `https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<author_uuid>/inbox/`

**Current federation status:** This inbox currently processes remote follow requests, follow acceptances, entries/posts, and delete/tombstone objects. Cross-node comment and like delivery are supported on the sending side and are stored in the inbox when received; full remote processing for those object types is being finalized.

**When to use:**
- `POST`: Used by remote nodes (authenticated via HTTP Basic Auth) to deliver an activity to this author. The payload is stored as raw JSON with `processed=false`, then immediately processed into local models.
- `GET`: Used by the owner to read their inbox items (newest first). Only the logged-in author whose inbox this is can call this.
- `DELETE`: Used by the owner to clear all inbox items.

**POST Request Body:** Any valid activity object — the `type` field determines how it is handled:

| `type` value | Aliases | Behaviour |
|---|---|---|
| `entry` | `post` | Stores/updates the entry locally so it appears in streams |
| `follow` | — | Creates or resets a pending follow request for the owner to review |
| `follow_accepted` | — | Records that the remote author accepted our follow request; creates a `Follower` row on our side and marks the original `FollowRequest` as `ACCEPTED` |
| `comment` | — | Stored in the inbox; remote processing support is being finalized |
| `like` | — | Stored in the inbox; remote processing support is being finalized |
| `unlike` | — | Stored in the inbox; remote processing support is being finalized |
| `delete` | `tombstone` | Soft-deletes the matching entry locally |

**Note on `follow_accepted`:** This is a non-standard extension. When our node sends a follow request to a remote node and that node accepts, the remote node may optionally POST a `follow_accepted` object back to our inbox. The `actor` field should be the remote author who accepted.

**POST Response Fields:**

| Field | Type | Description |
|---|---|---|
| `detail` | string | Human-readable result message e.g. `"Processed post"` |
| `processed` | boolean | `true` if the payload was successfully processed into local models, `false` if stored but processing failed |
| `item` | object | The created [Inbox Item Object](#inbox-item-object) |

**Example POST Request (remote node sending an entry):**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/inbox/
Authorization: Basic <base64-encoded-credentials>
Content-Type: application/json

{
  "type": "entry",
  "id": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001/entries/bbbb-0000-0000-0000-000000000001",
  "title": "Entry from Remote",
  "contentType": "text/plain",
  "content": "Hello from another node!",
  "visibility": "PUBLIC",
  "published": "2026-03-02T10:00:00+00:00",
  "author": {
    "type": "author",
    "id": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001",
    "host": "https://remotenode.com/api/",
    "displayName": "Remote User"
  }
}
```

**Example POST Response:** `201 Created`
```json
{
  "detail": "Processed post",
  "processed": true,
  "item": {
    "id": "ccccdddd-0000-0000-0000-000000000099",
    "itemType": "entry",
    "recipient_author_fqid": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
    "object_fqid": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001/entries/bbbb-0000-0000-0000-000000000001",
    "raw_json": { "type": "entry", "...": "..." },
    "processed": true,
    "received_at": "2026-03-02T10:00:01+00:00"
  }
}
```

**Example POST Request (follow accepted):**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/inbox/
Authorization: Basic <base64-encoded-credentials>
Content-Type: application/json

{
  "type": "follow_accepted",
  "actor": {
    "type": "author",
    "id": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001",
    "host": "https://remotenode.com/api/",
    "displayName": "Remote User"
  }
}
```

**Example POST Request (unlike):**
```
POST https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/inbox/
Authorization: Basic <base64-encoded-credentials>
Content-Type: application/json

{
  "type": "unlike",
  "object": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/entries/aaaabbbb-0000-0000-0000-000000000010",
  "author": {
    "type": "author",
    "id": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001"
  }
}
```

**Example GET Request:**
```
GET https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/inbox/
```

**Example GET Response:** `200 OK`
```json
{
  "type": "inbox",
  "items": [
    {
      "id": "ccccdddd-0000-0000-0000-000000000099",
      "itemType": "entry",
      "recipient_author_fqid": "https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001",
      "object_fqid": "https://remotenode.com/api/authors/aaaa-0000-0000-0000-000000000001/entries/bbbb-0000-0000-0000-000000000001",
      "raw_json": { "type": "entry", "...": "..." },
      "processed": true,
      "received_at": "2026-03-02T10:00:01+00:00"
    }
  ]
}
```

**Example DELETE Request:**
```
DELETE https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/3a1b2c3d-0000-0000-0000-000000000001/inbox/
```

**Example DELETE Response:** `200 OK`
```json
{
  "success": true
}
```

**Error Responses:**

| Status | Condition |
|---|---|
| `403` | `GET` or `DELETE` attempted by non-owner |

---

## 10. Object Schemas

### Author Object

| Field | Type | Example | Description |
|---|---|---|---|
| `type` | string | `"author"` | Always `"author"` |
| `id` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>"` | The FQID (full URL) of the author |
| `host` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/"` | Base API URL of the author's home node |
| `displayName` | string | `"Alice Smith"` | The author's display name |
| `github` | string | `"https://github.com/alicesmith"` | Full URL to the author's GitHub profile |
| `profileImage` | string | `"https://i.imgur.com/k7XVwpB.jpeg"` | Full URL to profile image (external or entry) |
| `web` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/<uuid>"` | URL to the author's HTML profile page |

---

### Entry Object

| Field | Type | Example | Description |
|---|---|---|---|
| `type` | string | `"entry"` | Always `"entry"` |
| `id` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/entries/<uuid>"` | The FQID (full API URL) of the entry |
| `web` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/authors/<uuid>/entries/<uuid>/"` | The frontend HTML page URL for this entry |
| `title` | string | `"Hello World"` | Title of the entry |
| `description` | string | `"A brief summary"` | Short description |
| `contentType` | string | `"text/plain"` | One of: `text/plain`, `text/markdown`, `image/png;base64`, `image/jpeg;base64` |
| `content` | string | `"This is my post."` | Body text, or base64-encoded image data |
| `visibility` | string | `"PUBLIC"` | One of: `PUBLIC`, `UNLISTED`, `FRIENDS` |
| `published` | string | `"2026-03-01T12:00:00+00:00"` | ISO 8601 timestamp of when entry was created |
| `author` | object | `{ "type": "author", ... }` | Embedded [Author Object](#author-object) |

---

### Comment Object

| Field | Type | Example | Description |
|---|---|---|---|
| `type` | string | `"comment"` | Always `"comment"` |
| `id` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/commented/<uuid>"` | The FQID (full URL) of the comment |
| `entry` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/entries/<uuid>"` | The FQID of the entry this comment belongs to |
| `author` | object | `{ "type": "author", ... }` | Embedded [Author Object](#author-object) |
| `comment` | string | `"Great post!"` | The comment text |
| `contentType` | string | `"text/plain"` | One of: `text/plain`, `text/markdown` |
| `published` | string | `"2026-03-01T14:00:00+00:00"` | ISO 8601 timestamp |

---

### Like Object

| Field | Type | Example | Description |
|---|---|---|---|
| `type` | string | `"like"` | Always `"like"` |
| `id` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/liked/<uuid>"` | The FQID (full URL) of the like |
| `author` | object | `{ "type": "author", ... }` | Embedded [Author Object](#author-object) |
| `object` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/<uuid>/entries/<uuid>"` | FQID of the entry or comment that was liked |
| `published` | string | `"2026-03-01T16:00:00+00:00"` | ISO 8601 timestamp |

---

### Follow Request Object

| Field | Type | Example | Description |
|---|---|---|---|
| `type` | string | `"follow"` | Always `"follow"` |
| `status` | string | `"PENDING"` | One of: `PENDING`, `ACCEPTED`, `REJECTED` |
| `actor` | object | `{ "type": "author", ... }` | The author who sent the follow request |
| `object` | object | `{ "type": "author", ... }` | The author they want to follow |

---

### Inbox Item Object

| Field | Type | Example | Description |
|---|---|---|---|
| `id` | UUID | `"ccccdddd-0000-0000-0000-000000000099"` | Internal UUID of the inbox item |
| `itemType` | string | `"entry"` | One of: `entry`, `follow`, `follow_accepted`, `like`, `unlike`, `comment`, `delete` |
| `recipient_author_fqid` | string | `"https://jd-node-b-9c1da1a35b21.herokuapp.com/api/authors/..."` | FQID of the author this item was sent to |
| `object_fqid` | string | `"https://remotenode.com/api/authors/.../entries/..."` | FQID of the object (entry, comment, like, etc.) |
| `raw_json` | object | `{ "type": "entry", ... }` | The full original payload as received |
| `processed` | boolean | `true` | Whether the item was successfully processed into local models |
| `received_at` | string | `"2026-03-02T10:00:01+00:00"` | ISO 8601 timestamp of when item arrived |


## 11. Cross-Node Demo / Testing Notes

Before the demo, verify that:

- each team member has deployed the same codebase to their own Heroku app
- each deployment uses its own Heroku Postgres database
- each node's `service_address` is set to its real Heroku URL
- each team member has provided a working test account that can log in
- comments, inbox delivery, and likes use absolute deployed URLs
- same-node and cross-node interactions have both been tested

### This node

- **service_address:** `https://jd-node-b-9c1da1a35b21.herokuapp.com`
- **hostname:** `jd-node-b-9c1da1a35b21.herokuapp.com`
- **port:** `443`
- **login account:** `YOUR_TEST_LOGIN`
- **password:** `YOUR_TEST_PASSWORD`