# Claude Code config for the ML Platform Framework

Personal setup (per your answer — this is for you only, not packaged as
a team plugin yet).

## Install (one-time)
```bash
# Skills and subagents — available across ALL your Claude Code sessions,
# since your work spans three separate repos.
cp -r user-level/skills/* ~/.claude/skills/
cp -r user-level/agents/* ~/.claude/agents/
```

## Per-repo setup
Drop the matching `CLAUDE.md` into the root of each repo once it exists:
```bash
cp ml-project-template/CLAUDE.md   <path-to-your-clone>/ml-project-template/
cp platform-framework/CLAUDE.md    <path-to-your-clone>/platform-framework/
cp ml-algorithm-playbook/CLAUDE.md <path-to-your-clone>/ml-algorithm-playbook/
```
Commit these — they're small, repo-specific, and help anyone (including
future-you) working in that repo, whether or not they use Claude Code.

## What you get
- `/new-ml-project` — scaffold a new project from the template
- `/add-playbook-entry` — draft a new algorithm entry (runs as a subagent
  via `context: fork`, keeping research out of your main session)
- `/add-pillar-module` — scaffold a new src/ module correctly
- Subagents `playbook-writer`, `ci-cd-engineer`, `standards-reviewer` —
  Claude Code will invoke these automatically when a task matches, or you
  can ask for them explicitly: "use the standards-reviewer subagent on
  this PR"

## Build order recommendation
1. Create the three GitHub repos (empty) in your org.
2. Push `platform-framework` and `ml-project-template` content (already
   scaffolded — see the two zips from this conversation).
3. Push `ml-algorithm-playbook` — use `/add-playbook-entry` for your
   pilot's algorithm first, since that's the one you need immediately.
4. Set the two GitHub Actions secrets sets (Dev doesn't need any; Prod
   needs `PROD_SNOWFLAKE_ACCOUNT`, `PROD_DEPLOY_SERVICE_USER`,
   `PROD_DEPLOY_SERVICE_PASSWORD`) once the Prod Snowflake Git Repository
   object exists.
5. Use `/new-ml-project` for the actual pilot.
