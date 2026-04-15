# GitHub Credentials

Repository: https://github.com/clarkman-meta/embedded-knowledge
Account: clarkman-meta
Token (PAT): <GITHUB_PAT>
Expiry: ~90 days from 2026-04-14 (renew around 2026-07-13)

## Authenticated Clone & Push

Use the token-embedded URL for full read/write access in any conversation:

```bash
# Clone with authentication
git clone https://clarkman-meta:<GITHUB_PAT>@github.com/clarkman-meta/embedded-knowledge.git /home/ubuntu/embedded-knowledge

cd /home/ubuntu/embedded-knowledge

# Configure git identity (required for commits)
git config user.email "manus@embedded-knowledge"
git config user.name "Manus"

# After making changes, push back
git add .
git commit -m "Add: <description>"
git push origin main
```

## Token Renewal

When the token expires, generate a new one at:
https://github.com/settings/tokens/new
- Note: manus-embedded-knowledge
- Expiration: 90 days
- Scopes: repo (full control)

Then update this file (both in sandbox `/home/ubuntu/skills/embedded-knowledge/references/github-credentials.md`
and in the manus-skill repo) and re-deliver the skill (Add to My Skills).
