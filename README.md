# Digital Garden Patches

Optional patches for an existing Digital Garden website repository. Independent
of upstream pull request approval. No npm module or color groups are installed.

## Graph node colors

`0001-Add-per-note-graph-node-colors-via-graph-color.patch` adds one optional text
property to notes:

```yaml
---
dg-publish: true
graph-color: "#ff6b6b"
---
```

Supported formats: quoted `#RGB` and `#RRGGBB`. Missing or invalid values keep
the existing theme color. Local and global web graphs are supported. Current
node outlines and hover effects are preserved.

There are no color groups, palettes, dependency updates, or deployment changes.
Digital Garden 2.95.1 exports the property under `dg-note-properties`; enabling
`dg-pass-frontmatter` is not necessary with that version.

## Apply to a site that already has automatic deployment

Use the **website repository created by the Vercel setup**, not the Obsidian
vault or the `obsidian-digital-garden` plugin repository.

First clone this patch repository next to the website repository:

```bash
git clone https://github.com/yupirex/digital-garden-patches.git
```

The following commands run from your website repository. The example assumes
the two repositories are sibling directories. Replace the relative path if
your directories are elsewhere.

Check that the working tree is clean and that you have the latest site changes:

```bash
git status --short
git pull --ff-only
```

If `git status` shows changes, commit or otherwise handle them yourself before
continuing. Do not overwrite existing work.

Check compatibility without changing any site files:

```bash
git apply --check "../digital-garden-patches/0001-Add-per-note-graph-node-colors-via-graph-color.patch"
```

If the check succeeds, apply the patch:

```bash
git apply "../digital-garden-patches/0001-Add-per-note-graph-node-colors-via-graph-color.patch"
git diff --check
git diff
```

Test using the site's existing dependencies and lockfile:

```bash
npm ci
npm test
npm run build
```

On Windows, an existing favicon-copy race can produce `EBUSY`. If that is the
only build error, verify locally with serialized filesystem operations instead:

```bash
npm run get-theme
npm run build:sass
npx --no-install cross-env ELEVENTY_ENV=prod UV_THREADPOOL_SIZE=1 NODE_OPTIONS=--max-old-space-size=3072 eleventy
```

Commit **only the three patched files**:

```bash
git add -- src/helpers/linkUtils.js src/helpers/linkUtils.test.js src/site/_includes/components/graphScript.njk
git commit -m "Add per-note graph node colors via graph-color"
git push
```

If your current branch is the deployment branch connected to Vercel, pushing
starts the existing automatic deployment. Otherwise, merge this commit into
the deployment branch via a PR. No webhook needs to be recreated.

Then add `graph-color` to notes in Obsidian and republish them through Digital
Garden. Wait for Vercel to finish building the new site version.

## Alternative: transfer the commit from your template fork

The identical patch is also available in `yupirex/digitalgarden`, branch
`feature/graph-node-colors`. From a clean website repository:

```bash
git fetch https://github.com/yupirex/digitalgarden.git feature/graph-node-colors
git cherry-pick 849b3f6ed252bab71231cc0a72de2fabd1a23026
```

This transfers only the color commit, not the fork's whole history or its
dependency versions. Run the tests/build and push the resulting commit to
your deployment branch. If cherry-pick reports conflicts and you do not want
to resolve them, `git cherry-pick --abort` cancels that operation.

Use **one method only**: apply the patch OR cherry-pick the commit. Do not do
both, and do not apply the patch again if colors are already supported.

## Compatibility and scope

The patch targets the Digital Garden Pixi v7 graph and was prepared on upstream
commit `c08ad97`. It changes only:

- `src/helpers/linkUtils.js`: export validated numeric node colors.
- `src/site/_includes/components/graphScript.njk`: use the optional color.
- `src/helpers/linkUtils.test.js`: regression tests.

It does not touch notes, `.env`, Vercel configuration, package files, workflows,
or existing deployment hooks. It does not fix unrelated build failures.

For unknown or substantially changed templates, `git apply --check` can fail.
Do not force installation or replace entire source files; adapt the patch to
that template first. After a template upgrade, review whether these changes
are retained. Upstream PR: https://github.com/oleeskild/digitalgarden/pull/424.

Tests on the upstream version: 417 passed. The site build also passed on
Windows with serialized filesystem operations. Applying the patch was verified
on a clean copy of the current `yupirex/swt` website without changing its notes
or deployment files.
