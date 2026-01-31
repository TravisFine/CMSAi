# Wikipedia Chain Finder

## Project Summary
A web app that finds the shortest path of links between two Wikipedia articles using bidirectional search. Flask backend with vanilla JS frontend.

## Core Requirements
- User enters start/end article titles, app finds the shortest link path connecting them
- Maximum search depth: 7 links
- Uses Wikipedia MediaWiki API (no local database)
- Only mainspace articles (namespace 0)—no File:, Wikipedia:, Help:, etc.
- Bidirectional iterative deepening search algorithm

## Architecture
- **Backend**: Python/Flask with background threading for search jobs
- **Frontend**: Vanilla HTML/CSS/JS (no frameworks)
- **Communication**: Polling pattern—POST creates job, GET polls for status every 1-2 seconds

## API Endpoints
- `POST /api/search` — Start search, returns `job_id`
- `GET /api/search/<job_id>` — Poll status, returns progress or results

## Coding Standards
- Keep dependencies minimal (Flask, requests only)
- No classes where simple functions suffice
- Type hints on all function signatures
- Docstrings on public functions
- Handle Wikipedia API pagination (continuation tokens)
- Include reasonable error handling for API failures
- Set a polite User-Agent header for Wikipedia requests