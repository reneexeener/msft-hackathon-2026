# Organizer setup

Notes for whoever is running the hackathon. Participants don't need to read this.

## What already exists

| Thing | Where |
| --- | --- |
| Ideas & Submissions board | <https://github.com/users/reneexeener/projects/59> |
| Issue forms | `.github/ISSUE_TEMPLATE/` |
| Auto-add workflow | `.github/workflows/add-to-project.yml` |
| Labels | `type: *`, `theme: *`, `status: *`, `needs: triage` |

The board has custom fields **Submission type**, **Theme**, **Team**, **Effort**, **Code repo**, and
**Demo link**, plus a **Status** field customised to the hackathon flow (💡 Idea submitted → ✅ Triaged →
🙋 Claimed by team → 🚧 In progress → 📦 Submitted → 🏆 Judged).

## Before the event

### 1. Decide on repository visibility

The repo is currently **private**. Participants can't open issues in a private repo unless they're
collaborators, so pick one:

- **Make it public** — simplest, works for external students:
  ```bash
  gh repo edit reneexeener/msft-hackathon-2026 --visibility public --accept-visibility-change-consequences
  ```
- **Keep it private** and invite each participant as a collaborator (fine for a small internal cohort):
  ```bash
  gh api -X PUT repos/reneexeener/msft-hackathon-2026/collaborators/USERNAME -f permission=triage
  ```

If you make the repo public, consider making the board public too so participants can see it without
being added:

```bash
gh project edit 59 --owner reneexeener --visibility PUBLIC
```

### 2. Wire up automatic board population

Issue forms carry a `projects:` key, which adds an issue to the board automatically — but **only when
the issue author has write access to the project**. Participants normally won't, so set up the workflow
token as well:

1. Create a fine-grained PAT with **read and write** access to your projects
   (<https://github.com/settings/tokens>).
2. Add it as a repository secret named `ADD_TO_PROJECT_PAT`:
   ```bash
   gh secret set ADD_TO_PROJECT_PAT --repo reneexeener/msft-hackathon-2026
   ```

Until that secret exists the workflow logs a notice and exits cleanly — nothing breaks, you just
triage new issues onto the board by hand.

### 3. Fill in the dates

`README.md` has a schedule table with `_TBD_` placeholders. Replace them, and set the same deadlines
as milestones if you want them to show up on the board:

```bash
gh api repos/reneexeener/msft-hackathon-2026/milestones -f title="Idea submission deadline" -f due_on="2026-03-11T23:59:00Z"
gh api repos/reneexeener/msft-hackathon-2026/milestones -f title="Final project submission" -f due_on="2026-03-14T16:00:00Z"
```

### 4. Seed a Discussion category (optional)

Discussions is already enabled, and `.github/ISSUE_TEMPLATE/config.yml` links to it as the place for
"is this a good idea?" questions. Consider opening a pinned welcome thread so the tab isn't empty when
participants arrive — an empty Discussions tab reads as "nobody is here".

### 5. Tidy up the example issues

Six issues labelled `type: example` are seeded to show participants what a good submission looks like.
Leave them up during the event, then close them at the end:

```bash
gh issue list --repo reneexeener/msft-hackathon-2026 --label "type: example" --json number --jq '.[].number' \
  | xargs -I{} gh issue close {} --repo reneexeener/msft-hackathon-2026 --reason "not planned"
```

## During the event

**Daily triage.** Filter the board by `Status: 💡 Idea submitted`, then for each item: set **Theme**
and **Effort**, move it to `✅ Triaged`, and swap the `needs: triage` label for `status: looking-for-team`
or `status: claimed`.

**Useful views to add to the board** (Board or Table view, then group/filter):

| View | Filter | Group by |
| --- | --- | --- |
| Triage queue | `label:"needs: triage"` | Submission type |
| Ideas up for grabs | `label:"status: looking-for-team"` | Theme |
| Team progress | `-no:Team` | Team |
| Judging | `label:"type: project-submission"` | Theme |

**Handy commands:**

```bash
# Everything still awaiting triage
gh issue list --repo reneexeener/msft-hackathon-2026 --label "needs: triage"

# Idea count per theme
for t in planning-and-tracking agentic-workflows automation insights-and-metrics onboarding-and-docs accessibility-and-inclusion; do
  printf "%-30s %s\n" "$t" "$(gh issue list --repo reneexeener/msft-hackathon-2026 --label "theme: $t" --state all --json number --jq 'length')"
done

# All final submissions with their links
gh issue list --repo reneexeener/msft-hackathon-2026 --label "type: project-submission" --state all \
  --json number,title,url --jq '.[] | "\(.number)\t\(.title)\t\(.url)"'
```
