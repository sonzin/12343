# Books Tracking

## Overview
The Notion Life OS includes a database for managing reading progress, ratings, and book metadata. This allows for long-term tracking of knowledge acquisition and personal library management.

## Database Discovery
The Notion Life OS contained multiple databases named "Books". Identifying the active one required comparing entries with the UI.
- **Active Database ID**: `4b1535ea-11cb-43c9-b06e-0dfcbc0b7696` (Confirmed via entry verification).
- **Archived Database ID**: `022d2ef2-5bec-4ae4-9a13-83ff5846f819` (Successfully archived on 2026-01-31).

## Database Schema
The "Books" database properties include:

| Property | Type | Description |
| :--- | :--- | :--- |
| **Name** | title | Title of the book |
| **Author** | rich_text | Author(s) name |
| **Status** | select | options: `Wishlist`, `Want to Read`, `Currently Reading`, `Read` |
| **Genre** | select | options: `Thriller`, `History`, `Philosophy`, `Fiction`, `Self Help`, etc. |
| **Rating** | select | options: `⭐`, `⭐⭐`, `⭐⭐⭐`, `⭐⭐⭐⭐`, `⭐⭐⭐⭐⭐` |
| **Start Date** | date | |
| **End Date** | date | |
| **No. of Pages** | number | |
| **Pages Read** | number | |
| **Progress** | formula | |
| **Book Cover** | files | Property for external/internal file links. |

## Common Workflows

### Marking a Book as Completed
When a book is finished:
1.  **Search for Metadata**: If specific details like page count are missing, query external sources (e.g., Simon & Schuster, Goodreads).
2.  Set **Status** to "Read".
3.  Set **End Date** to the completion date (e.g., `2026-01-25`).
4.  **Sync Progress**: Set **Pages Read** equal to **No. of Pages** (e.g., 320/320) to trigger the 100% green progress bar in Notion.
5.  **Add Rating**: Select a star rating (e.g., for a 3.5-star review, round up to `⭐⭐⭐⭐` if half-stars are unavailable).
6.  Optionally add review notes in the page body.

### Adding Cover Images
To add a high-quality cover image via the API:
- **Page Cover**: Set the `cover` attribute of the page object to `{"type": "external", "external": {"url": "IMAGE_URL"}}`.
- **Book Cover Property**: Add the same URL to the `Book Cover` files property for consistency within database views.

## AI Management Patterns
AI agents can interact with the Books database to:
- Find books by title or author.
- Update reading progress.
- Suggest "To Read" books based on genre preferences.
- Generate reading summaries.

## Case Study: "Broken Country" (Jan 2026)
This book was added to the Life OS to test the completion workflow:
- **Challenge**: Multiple "Books" databases existed.
- **Resolution**: Identified the database with "Building a Second Brain" as primary and archived the redundant one.
- **Completion**: Updated with 320 pages, 100% progress, and a rounded 4-star rating (from 3.5).
- **Visuals**: Cover and page cover synced using external URL from publisher.
