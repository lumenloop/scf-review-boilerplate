---
name: fetch-external-doc
description: "Fetch external documents linked in SCF submissions — Google Docs, Google Drive PDFs, GitHub, Notion, IPFS. Use when a submission links to an architecture doc, whitepaper, or technical spec hosted externally. Handles URL transformation for Google services."
---

# Fetch External Document

## Purpose

SCF submissions frequently link to external architecture documents, whitepapers, and technical specs. These often contain 10x more technical detail than the submission text and can significantly affect Technical Depth and Spec Compliance scores. This skill handles fetching them reliably.

## URL Pattern Detection

When you encounter a URL in a submission (especially in the Technical Architecture column), identify the type and follow the corresponding fetch strategy:

### Google Docs

**URL patterns:**
- `docs.google.com/document/d/{DOC_ID}/...`
- `docs.google.com/document/d/{DOC_ID}/edit`
- `docs.google.com/document/d/{DOC_ID}/view`

**How to fetch:**
1. Extract the `{DOC_ID}` from the URL
2. Construct the export URL: `https://docs.google.com/document/d/{DOC_ID}/export?format=txt`
3. Fetch using WebFetch with the export URL
4. If that fails, try: `https://docs.google.com/document/d/{DOC_ID}/pub`

**Example:**
- Input: `https://docs.google.com/document/d/1aBcDeFgHiJkLmNoPqRsTuVwXyZ/edit?usp=sharing`
- Export URL: `https://docs.google.com/document/d/1aBcDeFgHiJkLmNoPqRsTuVwXyZ/export?format=txt`

### Google Drive Files (PDFs, etc.)

**URL patterns:**
- `drive.google.com/file/d/{FILE_ID}/view`
- `drive.google.com/file/d/{FILE_ID}/preview`
- `drive.google.com/open?id={FILE_ID}`
- `drive.google.com/uc?id={FILE_ID}`

**How to fetch:**
1. Extract the `{FILE_ID}` from the URL
2. Construct the direct download URL: `https://drive.google.com/uc?export=download&id={FILE_ID}`
3. Fetch using WebFetch
4. If the file is a PDF, WebFetch may not render it well — note this in the review and extract what you can
5. Alternative: try `https://drive.google.com/uc?id={FILE_ID}&export=download`

**Example:**
- Input: `https://drive.google.com/file/d/1xYzAbCdEfGhIjKlMnOpQrS/view?usp=sharing`
- Download URL: `https://drive.google.com/uc?export=download&id=1xYzAbCdEfGhIjKlMnOpQrS`

### Google Drive Folders

**URL patterns:**
- `drive.google.com/drive/folders/{FOLDER_ID}`

**How to handle:**
- Cannot fetch folder contents directly
- Mark as UNFETCHABLE
- Note in the review: "Google Drive folder link — cannot fetch contents automatically"

### GitHub

**URL patterns:**
- `github.com/{owner}/{repo}/blob/{branch}/{path}` — file view
- `github.com/{owner}/{repo}` — repo root
- `github.com/{owner}/{repo}/tree/{branch}/{path}` — directory

**How to fetch:**
1. For file links: convert to raw URL by replacing `github.com` with `raw.githubusercontent.com` and removing `/blob/`
   - Input: `https://github.com/org/repo/blob/main/ARCHITECTURE.md`
   - Raw: `https://raw.githubusercontent.com/org/repo/main/ARCHITECTURE.md`
2. Fetch using WebFetch with the raw URL
3. For repo roots or directories: use WebFetch on the original URL, or use `gh api` if available
4. For README-type content: try `https://raw.githubusercontent.com/{owner}/{repo}/{branch}/README.md`

### Notion Pages

**URL patterns:**
- `notion.site/{page-slug}-{PAGE_ID}`
- `{workspace}.notion.site/{page-slug}-{PAGE_ID}`
- `notion.so/{PAGE_ID}`

**How to fetch:**
- Notion pages are client-side rendered (JavaScript) — WebFetch returns only the loading shell, not the actual content
- Mark as UNFETCHABLE with note: "Notion page — requires JavaScript rendering, content not accessible via WebFetch"
- If the submission relies heavily on a Notion doc for its architecture, note this as a gap in the review — the reviewer could not access the full technical detail

### IPFS Links

**URL patterns:**
- `ipfs.io/ipfs/{HASH}`
- `gateway.pinata.cloud/ipfs/{HASH}`
- `{any-gateway}/ipfs/{HASH}`
- `ipfs://{HASH}`

**How to fetch:**
- Use WebFetch with a public gateway: `https://ipfs.io/ipfs/{HASH}`
- If the default gateway is slow, try: `https://gateway.pinata.cloud/ipfs/{HASH}`
- Skip after 15 seconds — IPFS gateways can be unreliable

### Unfetchable URLs

These services cannot be reliably fetched — mark as UNFETCHABLE:
- **Notion** (`notion.site/...`, `notion.so/...`) — client-side JS rendering, WebFetch returns empty shell
- **DocSend** (`docsend.com/...`) — requires email/login
- **Excalidraw** (`excalidraw.com/...`) — renders as canvas, no text
- **Figma** (`figma.com/...`) — requires authentication
- **Loom** (`loom.com/...`) — video, no text extraction
- **Miro** (`miro.com/...`) — requires authentication
- **Whimsical** (`whimsical.com/...`) — visual diagrams, no text extraction

## Fetch Strategy

When processing a submission's links:

1. **Identify all URLs** in the technical architecture, description, traction, and products & services fields
2. **Classify each URL** using the patterns above
3. **Fetch in order of priority:**
   - Architecture docs first (highest impact on scoring)
   - GitHub repos second
   - Other links third
4. **For each fetch, record:**
   - Original URL
   - Transformed URL (if applicable)
   - Status: ACCESSIBLE / UNVERIFIED / UNFETCHABLE
   - Brief summary of what was found (or why it failed)
5. **Timeout:** If any fetch takes more than 15 seconds, skip it and mark UNVERIFIED. Do not get stuck.

## Using the `read-gdoc` Skill

If the `read-gdoc` skill is available in your environment (check with the Skill tool), prefer using it for Google Docs and Google Drive files — it handles the URL transformation automatically:

```
Skill: read-gdoc
Args: https://docs.google.com/document/d/{DOC_ID}/edit
```

If `read-gdoc` is not available, use the manual URL transformation described above with WebFetch.

## Common Issues

- **Google "virus scan" warning**: Large Google Drive files trigger a "can't scan for viruses" page. The download URL still works but may return HTML instead of the file. Check the response content.
- **Google sharing permissions**: If a doc returns a login page, it's not publicly shared. Mark as UNVERIFIED and note "requires authentication."
- **GitHub rate limiting**: If you're fetching many GitHub URLs, you may hit rate limits. Space out requests or use `gh api` with authentication.
- **PDF content**: WebFetch can't extract text from PDFs well. Note "PDF — limited text extraction" and work with whatever you get.
- **Redirects**: Some URLs redirect (e.g., shortened links). WebFetch should follow redirects, but if it returns the redirect target URL, make a second fetch to that URL.
