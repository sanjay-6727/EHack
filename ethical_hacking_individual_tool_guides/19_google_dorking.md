# Google Dorking — Advanced Search Operators

## 1. Purpose
Google dorking uses search operators to narrow public search results. It is useful for finding publicly indexed pages and documents, but should not be used to access, download, or exploit private data.

**Memory hook:** Google dorking = **search with precision filters**.

## 2. Inputs
- Search terms and supported operators.
- A domain you own or are authorized to assess.
- A legitimate research question.

## 3. Common operators
```text
site:example.com
site:example.com filetype:pdf
site:example.com intitle:documentation
site:example.com inurl:help
```
Operator support and results change over time. Use a benign domain or your own site for practice.

## 4. Output and interpretation
Search results show indexed pages/snippets, not a live inventory. A PDF result means a matching file was indexed; it does not prove the document contains sensitive information until reviewed appropriately and with permission.

Explain: “I used `site:` to restrict results to a domain and `filetype:` to focus on PDFs. The results reflect what the search engine indexed, not everything hosted on the server.”

## 5. Responsible use
- Do not attempt to access restricted files or use credentials found online.
- If you discover exposed sensitive information on a site you do not own, do not download or redistribute it; report it through the site's security contact or responsible disclosure process.
- For your own website, remove or restrict sensitive files and request recrawling where appropriate.

## 6. Viva
**Q: What does `site:` do?** Restricts results to a domain/site.
**Q: What does `filetype:pdf` do?** Filters toward PDF results.
**Q: Are search results complete and current?** No; they depend on indexing and can be stale.
**Q: Is finding a public result proof of a vulnerability?** Not by itself; context and access policy matter.
