# Wikipedia Chain Finder - Specification

## Overview

A web application that finds the shortest path between two Wikipedia articles using a bidirectional iterative deepening search algorithm.

---

## Architecture

```
┌─────────────┐      HTTP/JSON      ┌─────────────┐      HTTPS      ┌─────────────────┐
│   Frontend  │  ←───────────────→  │   Flask     │  ←───────────→  │  Wikimedia API  │
│  (Browser)  │                     │   Backend   │                 │                 │
└─────────────┘                     └─────────────┘                 └─────────────────┘
```

---

## Frontend

**Technology**: HTML, CSS, vanilla JavaScript

**UI Components**:
- Text input for start article title
- Text input for end article title
- "Find Path" button
- Results area (displays chain or error)
- Loading indicator (spinner or text) shown during search

**Behavior**:
- On submit, POST to backend API with both article titles
- Disable inputs and show loading state during request
- On success, display the chain as a numbered list with clickable Wikipedia links
- On failure, display the error message

---

## Backend API

**Technology**: Python Flask

**Single Endpoint**:

```
POST /api/find-chain
Content-Type: application/json

Request:
{
  "start": "Albert Einstein",
  "end": "Jazz"
}

Response (success):
{
  "success": true,
  "chain": ["Albert Einstein", "Germany", "Music", "Jazz"],
  "length": 4
}

Response (failure):
{
  "success": false,
  "chain": null,
  "error": "No path found within depth limit"
}
```

**Error Cases**:
- `"Article not found: {title}"` - start or end article doesn't exist
- `"No path found within depth limit"` - search exhausted without finding connection
- `"API error: {details}"` - Wikimedia API failure

---

## Search Algorithm

**Bidirectional Iterative Deepening Search**:

1. Maintain two frontiers: forward (from start) and backward (from end)
2. At each iteration, expand the smaller frontier by one level
3. After each expansion, check if frontiers intersect
4. If intersection found, reconstruct and return the path
5. If combined depth exceeds limit, return failure

**Depth Limit**:
- Maximum depth of 3 from each direction
- This allows finding chains up to 7 articles (6 hops): start → 3 hops → middle ← 3 hops ← end
- Total expanded levels capped at 6

**Data Structures**:
- Forward visited: `dict[str, str]` mapping article → parent (for path reconstruction)
- Backward visited: `dict[str, str]` mapping article → child (for path reconstruction)
- Current frontier: `set[str]` of articles to expand next

**Path Reconstruction**:
When article X is found in both frontiers:
1. Walk backward from X through forward-visited parents to reach start
2. Walk forward from X through backward-visited children to reach end
3. Combine into complete path

---

## Wikimedia API Integration

**Base URL**: `https://en.wikipedia.org/w/api.php`

**User-Agent**: `WikipediaChainFinder/1.0 (educational project)`

**Get Outgoing Links** (forward search):
```
GET /w/api.php?action=query&titles={title}&prop=links&pllimit=500&plnamespace=0&format=json
```
- `plnamespace=0` restricts to main article namespace
- `pllimit=500` fetches up to 500 links per request
- Handle `continue` token if more links exist (fetch up to 500 total per article)

**Get Backlinks** (backward search):
```
GET /w/api.php?action=query&list=backlinks&bltitle={title}&bllimit=500&blnamespace=0&format=json
```
- Same namespace and limit considerations

**Check Article Exists**:
The link-fetching queries will return empty/missing if article doesn't exist. Check for this before starting search.

**Rate Limiting**:
- Add small delay between API calls if needed (100-200ms)
- Handle HTTP 429 errors gracefully

---

## Project Structure

```
project/
├── app.py              # Flask application, API endpoint
├── search.py           # Bidirectional search algorithm
├── wikipedia_api.py    # Wikimedia API client
├── static/
│   ├── style.css       # Styling
│   └── script.js       # Frontend logic
├── templates/
│   └── index.html      # Main page
└── requirements.txt    # Flask, requests
```

---

## Edge Cases

| Case | Handling |
|------|----------|
| Start equals end | Return immediately with single-article chain |
| Article doesn't exist | Return error before starting search |
| Direct link exists | Found at depth 1, return 2-article chain |
| No path within limit | Return failure after exhausting search |
| Article title normalization | Let Wikimedia API handle redirects and case |

---

## Out of Scope for V1

- Result caching
- Multi-threading/parallel API calls
- Progress updates during search
- Search history
- Alternative path suggestions
