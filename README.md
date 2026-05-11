# AIS-tree — Fellowship Navigator

> **Work in progress.** Expect rough edges and things that may change.

A filterable directory helping newcomers discover AI safety career pathways — courses, fellowships, advising programs, and other opportunities. Aimed at people in the EA community exploring AI safety careers.

**Live site:** [fellowship-navigator.nielsdoehring.workers.dev](https://fellowship-navigator.nielsdoehring.workers.dev)

---

## How the data works

All program entries live in a **Google Sheet** and are fetched automatically when the page loads. To add a program, correct an entry, or update a deadline, just edit the sheet — no code changes needed. The site reflects updates within a few minutes.

The data schema (all columns and what values they accept) is documented in [`CLAUDE.md`](CLAUDE.md#data-schema).

---

## How the app works

Single-file React app with no build step:

- React 18 and Babel loaded from CDN (Babel transforms JSX in-browser)
- Tailwind CSS from CDN
- Data fetched from a published Google Sheets CSV on page load

All code lives in `public/index.html`. Filter logic, component structure, and common tasks are documented in [`CLAUDE.md`](CLAUDE.md).

---

## Running locally

```bash
cd public
python3 -m http.server 8000
# Open http://localhost:8000
```

A local server is required — the Google Sheets fetch won't work over `file://` due to CORS. No npm, no build step.

---

## How to contribute

**For data changes** — edit the Google Sheet directly. No GitHub account or technical setup needed.

**For code changes** — you'll need a GitHub account and to work with the project files. Here's the basic flow:

1. **Fork this repo.** On the GitHub page for this project, click "Fork" in the top right. This creates your own copy of the project under your GitHub account, where you can make changes freely without affecting the live site.

2. **Make your changes.** Edit `public/index.html` in your fork (GitHub lets you do this in the browser, or you can clone it to your computer). Test locally if you can (see above).

3. **Open a Pull Request (PR).** Once your changes look good, click "Contribute" → "Open pull request" on your fork. This sends a proposal to have your changes added to the main project.

### Seeing your changes before they go live

When you open a PR, Cloudflare (the service that hosts this site) automatically builds a temporary version of the site with your changes and posts a link in the PR comments — usually within a minute or two of your push. You don't need to do anything to trigger this.

Click that link to see your changes running on a real deployment, with live data from the Google Sheet. Each time you push a new commit to the PR, the preview updates automatically. The main site at the live URL above is completely unaffected until the PR is merged.

**For contributors:** after opening a PR, wait a minute or two for the Cloudflare bot to post its comment, then click the preview link to check that everything looks right before asking for a review.

**For reviewers:** the preview link is the fastest way to verify a contribution — load the page, try the filters, and check that new entries render correctly.

---

## What's not built yet

- Filter by application status (hide closed programs)
- Sort results (by deadline, alphabetically, etc.)
- Text search
- Shareable URLs with filter state encoded in the link
- Fallback data if the Google Sheets fetch fails
