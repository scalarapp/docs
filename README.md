# docs.getscalar.org

Central documentation hub for all [Scalar](https://getscalar.org)
open-source projects. Lives at https://docs.getscalar.org.

## What this is

A single GitHub Pages site that hosts:

- A landing page listing every published library.
- Each library's reference docs under its own subpath, e.g.
  `docs.getscalar.org/resumekit/`, `docs.getscalar.org/kotlin-foo/`,
  `docs.getscalar.org/go-bar/`, etc.
- (Future) written guides, migration notes, architecture docs —
  anything that isn't tied to a single library.

Libraries live in their own GitHub repositories. This repo only
**aggregates** their docs — it doesn't contain any library source.

## Tooling-agnostic by design

Different languages have different documentation tools:

| Language     | Tool          | `build.type` |
|--------------|---------------|---------------|
| Swift        | DocC          | `swift-docc`  |
| Kotlin       | Dokka         | `kotlin-dokka` *(future)* |
| Go           | `go doc` HTML | `go-doc` *(future)* |
| Rust         | `cargo doc`   | `rust-doc` *(future)* |
| TypeScript   | TypeDoc       | `typedoc` *(future)* |
| Plain guides | MkDocs        | `mkdocs` *(future)* |

Each entry in [`libraries.json`](libraries.json) declares its
`build.type`. The workflow runs a per-tool builder step and drops
the resulting static HTML into `_site/<slug>/`. Adding support for a
new tool means adding a step to `.github/workflows/build.yml`; it
doesn't disturb anything else.

## Adding a library

1. Publish the library as its own GitHub repo (public).
2. Add an entry to [`libraries.json`](libraries.json):

   ```json
   {
     "slug": "yourlib",
     "name": "YourLib",
     "tagline": "One sentence on what it does.",
     "language": "Swift",
     "repo": "scalarapp/YourLib",
     "build": { "type": "swift-docc", "target": "YourLib" },
     "tags": ["networking", "..."]
   }
   ```

3. Commit and push. The site rebuilds within a minute; your library
   appears on the landing page and at `/yourlib/`.

## Triggering a rebuild

- **Automatic**: every push to `main` of this repo.
- **Daily**: `04:17` UTC cron — picks up upstream changes even if a
  library's workflow forgot to dispatch.
- **From a library's CI**: fire a `repository_dispatch` with type
  `library-docs-updated` after its own merge completes. Example:
  [scalarapp/ResumeKit's docs workflow](https://github.com/scalarapp/ResumeKit/blob/main/.github/workflows/docs.yml).
- **Manual**: the `Build docs site` workflow has `workflow_dispatch`
  so you can rerun it from the Actions tab without a commit.

## Hosting

- Source: `gh-pages` branch of this repo.
- Custom domain: `docs.getscalar.org` — CNAME file in the publish
  output + DNS CNAME to `scalarapp.github.io`.
- HTTPS via GitHub Pages / Cloudflare automatically.

No rsync, no custom servers, no secrets. `git push` to `main` →
wait ~60s → new site live.
