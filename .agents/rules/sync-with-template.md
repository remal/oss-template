# Sync with Template

Files matching `includes`/`excludes` in `.github/sync-with-template.yml` are synced automatically from the template repository through PRs (label `sync-with-template`, auto-merged) created by the [sync-with-template action](https://github.com/remal-github-actions/sync-with-template). The template is the repository variable `TEMPLATE_REPOSITORY` (an `owner/repo` value, never an org variable), or GitHub's "generated from" relationship when it is unset. The root of all template chains is [remal/oss-template](https://github.com/remal/oss-template). Files that exist only in the current repo are never touched.

**IMPORTANT: treat every repo as a generated one, including the template repo itself.** A synced file's local edits are reverted by the next sync unless the same change is also made upstream. Before committing, read `.github/sync-with-template.yml` and check which changed files are synced. When you change a synced file, make the change in the current project AND its template(s), or use an escape hatch:

- `.github/sync-with-template-local-transformations.yml` (never synced itself): per-file `ignore`/`delete`/replace/`script` transformations, see the [schema](https://github.com/remal-github-actions/sync-with-template/blob/main/local-transformations.schema.json). This is also how repos extend the sync config itself, since the config file is synced too.
- Modifiable sections: lines between `$$$sync-with-template-modifiable: <name> $$$` and `$$$sync-with-template-modifiable-end$$$` markers keep their local content across syncs. Keep the markers. Marker syntax ([modifiableSections.ts](https://github.com/remal-github-actions/sync-with-template/blob/main/src/internal/modifiableSections.ts)):
  - Markers are matched anywhere in a line, so any comment style works. The end marker takes no name.
  - `<name>`: any characters except `$`, spaces allowed, surrounding whitespace trimmed. Names must be unique within a file.
  - The marker pair must exist in the template's copy of the file. A section present only in the downstream repo is removed by the next sync.
  - The template's section body is the default content for repos that do not override the section.

**Verify where a synced file comes from before editing or attributing it, never guess.** Get the current repo's template source with the command below, then confirm the specific file exists there. A file absent upstream is authored in the current repo and syncs downward, so edit it in the current repo. Naming the source repo does not settle a given file's origin; only confirming the file exists upstream does.

```bash
# Template source of the current repo: the TEMPLATE_REPOSITORY variable, else the generated-from relationship
gh variable get TEMPLATE_REPOSITORY 2>/dev/null || gh api repos/{owner}/{repo} --jq '.template_repository.full_name'
```

Notes:

- Closing a sync PR does not reject the change; the PR is recreated.
- Deleting or renaming a synced file in the template does not remove it downstream. Add the old path to the template's `.github/sync-with-template-delete.list`.
- Keep the `# sync-with-template: adjust` comment on `cron:` lines. The action deliberately shifts such crons per repo, so values differ from the template.
- Rule files live in `.agents/rules/` (`.claude/rules` is a symlink to it) and are synced, so add or edit rules in the template.
- When a change spans the template and downstream repositories, commit and push the parent (template) repository first.
