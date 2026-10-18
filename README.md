# divelog-sites

A shared, community-maintained database of dive sites: coordinates, type, water, access,
regions, drawn site and cave maps and freely licensed photos. Instances of the
`divelog` module clone this repository read-only and layer a diver's personal log on top.

- `sites/<iso3>-<slug>.md`: one site per file, frontmatter plus a neutral description.
- `regions/<id>.md`: regions the sites belong to.
- `maps/drawn/<id>.json`: drawn maps (`<site>-<n>`).
- `media/sites/`: photos, with source, author and licence in `media/manifest.json`.
- `redirects.yaml`: renamed site ids (old → new); ids are never deleted.

Every site and map carries a `revision` and a newest-first `changelog`.

Validate locally with
`uvx --from "git+https://github.com/dennisbakhuis/divelog#subdirectory=packages/server" divelog validate --kind sites --base origin/main .`.
See `CONTRIBUTING.md` for how to propose changes and `LICENSE.md` for the licence layers.

## Using it in a divelog instance

A divelog instance reads this repository read-only, next to its personal log. The log's
`divelog.yaml` names it as a source and pins the commit it was last reviewed against:

```yaml
format: 1
kind: log
sources:
- kind: commons
  name: divelog-sites
  path: ../divelog-sites   # checkout of this repository, relative to the log
  ref: 0123abc             # commit the log is pinned to
```

The server mounts the checkout (`COMMONS_ROOT`, `/data/commons` in the image) and layers every
site under the log: a site is `commons` (with an optional personal overlay in the log), `log`
(personal only) or `fork` (a personal copy that shadows a commons id).

To follow upstream, run `divelog commons pull --log <log>` (or "pull commons" in the admin
with `COMMONS_GIT_ENABLED=true`): it fast-forwards this checkout, moves `ref` in the log's
`divelog.yaml`, commits the log, and lists the overlays and forks whose site moved to a newer
`revision`. Review each in the site editor and "Mark as reviewed" (or
`POST /api/v1/admin/sites/<id>/pin`); `divelog sites status <log>` ends in `0 stale` when done.

## Proposing a site

From an instance, `divelog sites propose <site-id> --commons <checkout of your fork>` copies a
personal site and its drawn maps into a branch `propose/<site-id>`, validated and signed off.
Push the branch to your fork and open a pull request here; CI validates it against `main` and
renders the maps it changes. Once merged and pulled, `divelog sites adopt <site-id>` turns the
personal record into an overlay on the commons one. Edits to existing sites follow the same
pull-request path; `CONTRIBUTING.md` has the rules (revisions, changelog, neutral wording,
sign-off).

Without an instance, open an issue: the forms for a new site or a correction to an existing one
take a couple of minutes, and a maintainer turns them into a pull request.

Initial content was imported from the maintainer's personal log.
