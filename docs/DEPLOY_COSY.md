# Deploy DB Converter Auto on CoSy.Bio

This folder is a lightweight static website for DB Converter Auto. It can be hosted as a simple CoSy.Bio app page or through GitHub Pages with a CoSy.Bio custom subdomain.

## Public CoSy.Bio Pattern

The CoSy.Bio software catalog groups tools by research area and links to a mix of external sites, GitHub repositories, and `apps.cosy.bio` pages. The `apps.cosy.bio/StrucTFactor` page is especially close to what DB Converter Auto needs: it has a compact project title, subtitle, GitHub link, contact details, prerequisite notes, installation steps, usage instructions, and citation.

DB Converter Auto should follow that pattern:

- Keep the CoSy.Bio software entry short and research-focused.
- Link the software entry to a dedicated page such as `https://apps.cosy.bio/DB-Conv-Auto/` or `https://db-conv-auto.cosy.bio/`.
- Keep the full technical detail in the GitHub repository and local agent guides.

## Option A: Host Directly on CoSy.Bio

Use this if you have access to the CoSy.Bio server or web CMS.

1. Upload the contents of this `docs/` folder to the chosen web directory.
2. Make sure `index.html`, `styles.css`, and `GithubFig.png` stay in the same folder.
3. Serve the folder from one of these URLs:
   - `https://apps.cosy.bio/DB-Conv-Auto/`
   - `https://www.cosy.bio/db-conv-auto`
4. Add a CoSy.Bio software catalog entry with:
   - Title: `DB Converter Auto`
   - Category: `Systems Medicine`, `Federated Learning`, or another group chosen by the CoSy.Bio maintainers
   - Short description: `Self-supervised neural mapper and inverter creation for converting between two non-overlapping biomedical cohort schemas.`
   - Link: the hosted page URL

## Option B: GitHub Pages plus CoSy.Bio Subdomain

Use this if CoSy.Bio DNS can be edited but the static files should remain deployed from GitHub.

1. Push this repository to GitHub.
2. In the GitHub repository, open `Settings -> Pages`.
3. Set the publishing source to the default branch and the `/docs` folder.
4. Add a custom domain, for example `db-conv-auto.cosy.bio`.
5. In the DNS provider for `cosy.bio`, add:

```text
Type: CNAME
Name: db-conv-auto
Target: <github-owner>.github.io
```

Replace `<github-owner>` with the GitHub user or organization that owns the repository.

Do not use a wildcard DNS record. GitHub recommends adding the custom domain in GitHub Pages settings before pointing DNS at GitHub Pages.

## Files

- `index.html`: public landing page
- `styles.css`: site styling
- `GithubFig.png`: workflow figure reused from the repository documentation
- `readme.md`: full technical documentation
