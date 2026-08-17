---
description: Run end-of-sprint PM Review for EISART teams (Product Rangers or Agents Of Item)
---

# PM Review — EISART End-of-Sprint

You are running the end-of-sprint PM Review for the EISART project on Jira.

**Cloud ID**: `38afde28-a949-4c99-b78d-d48b55c9b216`

---

## Step 0 — Ask which team

Ask the user: **"Which team? Product Rangers or Agents Of Item?"**

Then proceed with the correct values below.

### Product Rangers
- JQL Team name: `F-ITEM-Mgmt-Product Rangers`
- ToBeReleased fix version ID: `70628`
- NonDeployable fix version ID: `71012`

### Agents Of Item
- JQL Team name: `F-ITEM-Mgmt-Agents Of Item`
- ToBeReleased fix version ID: `69805`
- NonDeployable fix version ID: `69806`

---

## Step 1 — Find PM Review items

Use the `searchJiraIssuesUsingJql` tool with:
- `jql`: `project = EISART AND status = "PM Review" AND sprint in openSprints() AND fixVersion in (<ToBeReleased ID>, <NonDeployable ID>)`
  - Product Rangers: `fixVersion in (70628, 71012)`
  - Agents Of Item: `fixVersion in (69805, 69806)`
- `fields`: `["summary", "description", "issuetype", "status", "fixVersions", "customfield_10032", "comment", "attachment", "parent"]`
- `searchResultMode`: `"issues"` ← **required by the updated Jira API; always include this**

> **Why fix version and not assignee group or Team field?** `assignee in membersOf()` misses contributors not in the Jira group. The `"Team[Team]"` custom field (customfield_10001) is not JQL-filterable in this Jira instance. Fix version IDs are the most reliable signal — Product Rangers and Agents Of Item items are explicitly tagged with their team's fix versions.

> `searchResultMode` controls what the search returns: `"issues"` = issue data, `"count"` = count only, `"all"` = both. We always use `"issues"`.

Present the full list to the user with: #, Issue Key, Type, Summary, Fix Version, Story Points.

---

## Step 2 — For each item, present a review summary

> **IMPORTANT — Sequential, one at a time**: Process tickets ONE AT A TIME. Fully analyze one ticket, present all findings, then ask "Close it or send back?" — wait for the user's decision and act on it before moving to the next ticket. Do NOT fetch all tickets' details upfront or present bulk analyses.

### Pre-check: Parent field gate

Before doing any analysis, check if the `parent` field is populated on the issue.

- If **Parent is missing** on a Story, Task, or Spike: **automatically send it back** (transition ID `81`) with the comment: `"This ticket is missing a Parent (Epic link). Please link it to the appropriate Epic before it can be reviewed."` — do NOT do any further analysis, and move on to the next ticket.
- If Parent is present: proceed with the full review below.

For each item, retrieve full issue details and analyze:

1. **Acceptance Criteria** — Extract from description. Are they clearly defined?
2. **Fix Version** — Correct? (ToBeReleased = deployable code; NonDeployable = spikes, test automation, data fixes)
3. **PR Search** — Look in comments for a GitHub PR URL. If not found, run:
   ```
   gh api "search/issues?q=org:krogertechnology+EISART-XXXXX+is:pr+is:merged"
   ```
   Replace XXXXX with the issue key number.
4. **PR Analysis** (for each linked PR):
   - `gh pr view <N> --repo krogertechnology/<repo> --json number,title,state,mergedAt,author,reviews`
   - `gh pr diff <N> --repo krogertechnology/<repo>`
   - Report: PR#, repo, state (MERGED/OPEN), merge date, authors, approvers
   - **The Bug** (for bugs): root cause in plain English
   - **The Fix**: before/after code diff explained in plain English — highlight key files changed, logic added/removed, and why it matters
   - **Test coverage**: what scenarios were added/updated (look for `_test.go`, `*Test.java`, `*.spec.*` files in the diff)
   - **Relevance**: does the PR directly address the AC?
   - **Detailed diff summary**: For each significant file changed, briefly describe what changed and why — e.g., "Added `auditFields` population in `mapper.go` — previously this was skipped for generic events". Keep it plain English, no raw code blocks.
5. **Evidence** — Screenshots, Qmetry tests, feature env URLs in comments or attachments?
6. **Flags** — Note "Needs ROAM" but do not block closure on it

Present findings clearly. Then ask user: **"Close it or send back?"**

---

## Step 3 — Act on user decision

### Close
- **Stories / Spikes / Tasks**: transition ID `31`
- **Bugs**: transition ID `141` + field `customfield_10161: {"id": "10094"}` (Action Taken = "Added to Future Release")
- Add a generic comment (no @mentions) summarizing what was done and why closing

### Send Back (Ready for Review)
- Transition ID `81`
- Correct fix version if wrong
- Add a generic comment explaining what is missing or needs correction

---

## Step 4 — Fix Version Rules

| Work type | Fix version to use |
|---|---|
| Deployable code changes (stories, bugs) | ToBeReleased |
| Spikes, test automation, data fixes | NonDeployable |
| Prior quarter fix version (e.g. Q1) | Update to current quarter equivalent |

If the current fix version is from a prior quarter, update it to the matching current quarter version before closing.

---

## Step 5 — Running tally

After each item, maintain a running tally:
- Items closed
- Items sent back
- Story points (if set)

At the end, present a final summary table.

---

## Notes
- Comments are always generic — no @mentions of any individual
- "Needs ROAM" is common — do not block closure on it
- No attachments is OK if comments + merged PR provide strong evidence
- Unmerged but approved PRs: present facts to user and let them decide
- The `products-domain-workflow-service` repo contains the main Go workflow service
- The `mx-eix-workflow-engine` repo contains Inventory Kupid / Temporal workflow code (Java)
- The `products-domain-ehs-feeder` repo is the Avro-based event pipeline for hierarchy/group events (PD EHS Feeder)
