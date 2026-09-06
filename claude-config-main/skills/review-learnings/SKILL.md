---
description: "Review, prune, and consolidate the portable knowledge base. Use when the user wants to audit learnings, remove outdated entries, or merge related knowledge. Subcommands: list, prune, consolidate."
---

# /claude-config:review-learnings

You are reviewing the cross-project knowledge base at `knowledge/` in this plugin's root directory. This skill helps maintain knowledge quality over time.

The user ran: `/claude-config:review-learnings $ARGUMENTS`

---

## Routing

Parse `$ARGUMENTS` to determine the operation:

| Argument | Operation |
|----------|-----------|
| *(empty)* or `list` | List all learnings |
| `list --type=X` | List filtered by type (pattern, decision, playbook) |
| `list --tag=X` | List filtered by tag |
| `prune` | Interactive review and pruning |
| `consolidate` | Find and merge related entries |

---

## Operation: List

1. Read `knowledge/KNOWLEDGE.md`.

2. If the index has no entries (only section headers and comments), respond:
   > No learnings captured yet. Use `/claude-config:learn` to add your first one.

3. Otherwise, read every knowledge file referenced in the index. For each, extract the frontmatter fields.

4. **If no filter flags**, present all learnings grouped by type:

   ```
   ## Patterns (N entries)
   | # | Name | Tags | Created | Source |
   |---|------|------|---------|--------|
   | 1 | [name] | tag1, tag2 | 2026-05-01 | project-name |

   ## Decisions (N entries)
   | # | Name | Tags | Created | Source |
   |---|------|------|---------|--------|

   ## Playbooks (N entries)
   | # | Name | Tags | Created | Source |
   |---|------|------|---------|--------|

   **Total: N learnings**
   ```

5. **If `--type=X`**, show only the matching section.

6. **If `--tag=X`**, show only entries whose `tags` array includes the specified tag, across all types.

---

## Operation: Prune

Interactive review of each knowledge entry for currency and accuracy.

1. Read `knowledge/KNOWLEDGE.md`. If empty, respond with the "no learnings" message above.

2. Read every knowledge file referenced in the index.

3. For each entry, present:

   > ### [N/total] — [Name]
   > **Type:** [type] | **Created:** [date] | **Source:** [source] | **Tags:** [tags]
   >
   > [Full body content]
   >
   > **Assessment:** [Your assessment of whether this is still accurate and useful. Consider:
   > - Has the ecosystem changed? (e.g., library deprecated, API changed)
   > - Is this too project-specific to be cross-project?
   > - Is this duplicated by another entry?
   > - Is the reasoning still valid?]
   >
   > **Recommendation:** Keep / Update / Remove
   >
   > What would you like to do? **(keep / update / remove / skip)**

4. Based on user response:
   - **keep**: Move to next entry.
   - **update**: Ask what to change, apply edits to the file, re-present for confirmation, then move on.
   - **remove**: Delete the knowledge file and remove its line from `knowledge/KNOWLEDGE.md`. Confirm: "Removed [name]."
   - **skip**: Move to next entry without action.

5. After all entries reviewed, present summary:

   > ## Prune Summary
   > - **Kept:** N
   > - **Updated:** N
   > - **Removed:** N
   > - **Skipped:** N

---

## Operation: Consolidate

Find clusters of related learnings and propose merging them.

1. Read `knowledge/KNOWLEDGE.md`. If fewer than 3 entries, respond:
   > Not enough learnings to consolidate. Build up the knowledge base first with `/claude-config:learn`.

2. Read every knowledge file referenced in the index. Extract frontmatter and body content.

3. Identify clusters — entries that share 2+ tags or cover the same topic (based on semantic similarity of name and description). Only propose clusters of 2+ entries.

4. If no clusters found:
   > No related entries found that would benefit from consolidation. The knowledge base looks well-organized.

5. For each cluster found, present:

   > ### Cluster: [Topic]
   > These entries cover related ground:
   >
   > 1. **[Name 1]** ([type]) — [description]
   > 2. **[Name 2]** ([type]) — [description]
   >
   > **Proposed merge:** Combine into a single [type] entry:
   >
   > ```markdown
   > [Full proposed merged file content with frontmatter]
   > ```
   >
   > **Merge these entries?** (yes / edit / no)

6. Based on user response:
   - **yes**: Write the merged file, delete the source files, update `knowledge/KNOWLEDGE.md` (remove old entries, add new one). Confirm what was merged.
   - **edit**: Ask what to change, re-present.
   - **no**: Skip this cluster, move to next.

7. After all clusters processed, present summary:

   > ## Consolidation Summary
   > - **Clusters found:** N
   > - **Merged:** N (M entries → N entries)
   > - **Skipped:** N

---

## Reminders

After any operation that modifies files (prune with removals/updates, or consolidate with merges), end with:

> Changes are saved locally. To persist across machines:
> ```bash
> cd /path/to/claude-config && git add knowledge/ && git commit -m "knowledge: [prune/consolidate] review"
> ```
