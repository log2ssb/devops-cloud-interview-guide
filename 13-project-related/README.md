We are actively working on updating the notes for this section.

Question - What branching strategy do you follow, and how do you handle merges to avoid breaking the release branch? If a bug appears in production, what’s your approach to resolving it?
Answer -
**1. Branching Strategy**
I generally follow a GitFlow or a trunk-based with release branch strategy, depending on the team’s needs:

Main (or master) → Always reflects production-ready code.

Develop → Integration branch for features before release.

Feature branches → Created from develop for new features, named like feature/feature-name.

Release branches → Created from develop at the start of a release cycle, e.g., release/1.2. Only bug fixes, documentation, and release-related changes go here.

Hotfix branches → Created from main for urgent production fixes.

This ensures clear separation between ongoing development and stable release code.

**2. Handling Merges to Avoid Breaking the Release Branch**

Pull Requests (PRs) + Mandatory Reviews → All merges to release or main require at least one peer review.

Automated CI Pipelines → Every PR triggers build, linting, unit, and integration tests before merging.

Merge strategy → Avoid fast-forward merges for release branches to keep history clear; use squash or rebase to maintain cleanliness.

No direct pushes → Only merges via PR after validation.

This means that before anything hits the release branch, it’s already been validated by both automation and human review.

**3. Production Bug Handling**
If a bug is detected in production:

Reproduce & Confirm – First, reproduce in a staging or local environment to validate the issue.

Create a Hotfix Branch – Branch from main (e.g., hotfix/1.2.1).

Fix & Test – Apply the fix, run full CI/CD tests, and perform targeted regression testing.

Deploy Hotfix – Merge hotfix into main → deploy to production.

Back-Merge to Develop/Release – Merge the same fix into develop (and current release branch if applicable) to ensure it’s not overwritten in future releases.

This approach ensures:

Minimal downtime.

No divergence between production and ongoing development.

Full traceability in Git history.
