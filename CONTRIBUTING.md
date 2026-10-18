# Contributing

Changes arrive as pull requests: fork the repository, commit on a branch, open a PR. CI runs
the validator on every PR against the PR's base branch, and renders every drawn map the PR
changes to a PNG (the `map-renders` artifact of the run) so reviewers can see the drawing. A
red run is not merged.

Run the same check locally before pushing:

```sh
uvx --from "git+https://github.com/dennisbakhuis/divelog#subdirectory=packages/server" \
  divelog validate --kind sites --base origin/main .
```

Errors fail the run. Warnings (`path: warning: ...`) are printed for a reviewer to look at and
do not fail it.

## Sign-off

Every commit carries a Developer Certificate of Origin sign-off (`git commit -s`), stating
that you have the right to submit the change under the licences in `LICENSE.md`. There is no
contributor licence agreement.

## What a change looks like

- **New site**: `sites/<iso3>-<slug>.md`, id matching `^[a-z]{3}-[a-z0-9-]+$` and equal to the
  file name, `revision: 1`.
- **Every change to a site is a new revision.** Any change to a site's fields or description
  raises `revision` by exactly 1 and adds a changelog entry for that revision at the top:
  `{revision, date, author, note}`, with a note saying what changed and a date no earlier than
  the previous entry. Older entries stay exactly as they are: history is never rewritten and a
  revision never goes down.
- **Drawn maps** follow the same rule when the drawing changes (nodes, edges, lines, areas,
  origin, north, to-scale). Fixing only the title or notes needs no new revision.
- **Node ids are stable**, because dives in personal logs annotate their routes by them. A node
  you remove goes into the map's `retired_nodes`; a node you rename goes into `renamed_nodes`
  as `old: new` (the new id must exist). Retired ids are never used again, and neither list
  loses entries.
- **Sites are never deleted.** To rename or merge a site, add `old-id: new-id` to
  `redirects.yaml` and remove the old file. A redirect points straight at an existing site,
  not at another redirect.
- **Regions and photos stay referenced.** A region can only go once no site names it; a photo
  leaves `media/manifest.json` only together with every site reference to it.
- **Provenance**: coordinates name their `geo_source`; a description longer than 400
  characters names its `sources`.
- **Sensitive locations** are marked `location_visibility: region_only` and carry a coarse
  point only: `lat`/`lon` rounded to two decimals (about a kilometre). The validator warns
  about more precise coordinates on such a record.

## Writing

Descriptions, region texts, map titles, notes and labels are neutral and in the third person:
they describe the site, not someone's dives. The validator fails first-person words ("I",
"my", "we", "our", "us", ...) and warns about phrases that speak for one diver without a
pronoun ("in the log", "the workbook", "dives here", "favourite", "the owner's"). A Roman
numeral after a name ("Andrea I") is fine.

## Photos

A photo is a file in `media/sites/`, named in the site's `media` list and listed in
`media/manifest.json` with its `author`, `source` (and `page` where there is one), `licence`
and `public: true`.

- WebP (or JPEG), at most 2000 px on the long side and at most 500 KB.
- At most 3 per site.
- A licence from the allowlist: `CC0-1.0`, `CC-BY-<v>`, `CC-BY-SA-<v>` (versions 2.0, 2.5, 3.0,
  4.0) or `PDM`.
- No recognisable people without their consent.
- No photos of access points that would help someone trespass.

CI checks that every file under `media/` has a manifest entry with an allowlisted licence and
`public: true`, and that every photo a site names exists. Reviewers check the rest: format,
pixel and file size, the number per site, that author and source are filled in, and what the
photo shows.

## What may not enter

- Third-party reference maps (`reference:` refs or underlays) or maps traced from copyrighted
  surveys without permission.
- Text copied from guide books or websites. Citing them in `sources` is fine, copying is not.
- First-person prose: personal stories belong in the diver's own log.
- Personal fields: `notes`, `prices`, `reference_maps`, `subsurface_uuid`, and overlay or fork
  fields (`extends`, `forked_from`, `summary_override`, `base_revision`).
- Photos without an allowlisted licence (CC0, CC BY, CC BY-SA, public domain mark), or not
  marked `public: true` in the manifest.
- Personal map ids (`-p<n>`) and exact points of sensitive locations.

## Maintainers

The repository owner merges, for now, after checking that validation passes, the wording is
neutral, every photo's licence holds and no sensitive access details enter. Pull requests are
reviewed within about two weeks; issues opened from the forms are turned into pull requests by
a maintainer when time allows.
