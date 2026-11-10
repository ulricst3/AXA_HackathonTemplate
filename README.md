# Power Pages Site Scaffolding

Scaffolding for building a **Power Pages** site with plain HTML, CSS, and
JavaScript — no build step, no dependencies — deployed with the **pac CLI**
from GitHub Actions.

This repository contains the conventions and the deploy pipeline. It contains
no site content yet: that comes from a real Power Pages site.

## Power Pages Is Not Static Hosting

Worth understanding before you start, because it shapes everything else: you
cannot drop an `index.html` in a folder and have it served.

Site content lives in Dataverse as **records** — web pages, web templates
(Liquid), web files, content snippets, site settings. The pac CLI serialises
those records into a folder tree so they can be version-controlled, edited, and
uploaded again. Your HTML, CSS, and JS are still plain files, but they sit
*inside* that tree, at paths the CLI decides.

So the `site/` folder is **generated, never hand-authored**.

## Quick Start

1. Clone or fork this repository.
2. Install the pac CLI (needs the .NET SDK):

   ```bash
   dotnet tool install --global Microsoft.PowerApps.CLI.Tool
   ```

3. Create the site once in Power Pages, then authenticate and pull it down:

   ```bash
   pac auth create --environment https://<org>.crm4.dynamics.com
   pac pages list -v
   pac pages download --path site --webSiteId <id> --modelVersion <1|2>
   ```

   `pac pages list -v` tells you the data model version — `1` standard,
   `2` enhanced. You need it for every download and upload.
4. Edit the HTML, CSS, and JS inside `site/`. See [AGENTS.md](AGENTS.md) for
   the conventions.
5. Check your work by uploading to a development environment:

   ```bash
   pac pages upload --path site --modelVersion <1|2>
   ```

6. Commit, open a pull request, merge to `main` — GitHub Actions uploads to the
   target environment.

There is **no local preview**. Liquid, web roles, and Dataverse data only
resolve server-side, so a browser opened on a local file shows you nothing
meaningful. Verify in an environment or in the design studio.

## Required GitHub Configuration

Set these under **Settings → Secrets and variables → Actions**. Deployment uses
a service principal:

| Name | Kind |
|---|---|
| `PP_ENVIRONMENT_URL` | variable |
| `PP_APP_ID` | variable |
| `PP_TENANT_ID` | variable |
| `PP_MODEL_VERSION` | variable (`1` or `2`, defaults to `1`) |
| `PP_CLIENT_SECRET` | secret |

## What Is Already Here

```
README.md                                  this file
AGENTS.md                                  conventions and guardrails
.gitignore
.github/workflows/deploy-powerpages.yml    uploads site/ on push to main
specs/_template/spec.md                    copy per feature
```

## Three Rules That Matter

- **`manifest.yml` must be committed.** It tracks deletions; without it,
  content you delete locally is never removed from the target environment. Run
  `pac pages download` before *and* after deleting something. The other
  manifest, `<org-url>-manifest.yml`, is per-developer state and is git-ignored.
- **No secrets in the repository.** Page HTML and web files are downloadable by
  any visitor who can reach the page — an API key there is a published key.
  Service principal credentials belong in GitHub secrets.
- **HTML and CSS carry the site.** Every page must render, read, and navigate
  with JavaScript disabled. Scripts add operations on top of a page that
  already works without them — they never build it.

## Building With an AI Assistant

Point it at [AGENTS.md](AGENTS.md) — it holds the stack exclusions, the
HTML/CSS/JS/Liquid conventions, the pac commands, and the workflow. Write a
spec from [specs/_template/spec.md](specs/_template/spec.md) first, then
implement two or three acceptance criteria at a time.

Open decisions are marked `TODO: Human clarification needed` in `AGENTS.md` —
prompts for whoever adopts this, not oversights.
