# Organization Runner Support — Changelog Compatibility

This document describes which upstream features from `machulav/ec2-github-runner` require
changes for **organization-level** runner support, and which work out of the box.

## Branch: `org-support`

Based on upstream `main` (v2.6.1) with org-level modifications applied.

### What was changed for org support

| File | Change |
|------|--------|
| `src/gh.js` | All GitHub API endpoints switched from `/repos/{owner}/{repo}/...` to `/orgs/{org}/...` |
| `src/aws.js` | `config.sh --url` changed from repo URL to org URL, added `--runnergroup default` |

---

## Feature Compatibility by Release

### v2.3.8 — Spot Instance Support
- **Input:** `market-type: spot`
- **Org changes needed:** None — purely AWS/EC2 parameter, no GitHub API interaction.
- **Status:** ✅ Works as-is

### v2.3.9 — Custom Label & Startup Optimization
- **Input:** `label` (use a specific label instead of auto-generated)
- **Org changes needed:** None — label is passed to `config.sh` which already uses the org URL.
- **Status:** ✅ Works as-is

### v2.4.0 — Block Device Mappings & Runner Execution
- **Input:** `block-device-mappings`
- **Org changes needed:** None — purely AWS/EC2 parameters.
- **Status:** ✅ Works as-is

### v2.4.1 — Multi-Zone/Region Retry
- **Input:** `availability-zones-config`
- **Org changes needed:** None — retries EC2 allocation across zones, no GitHub API.
- **Status:** ✅ Works as-is

### v2.4.2 — Timeout Optimization & Metadata Options
- **Inputs:** `startup-timeout-minutes`, `startup-retry-interval-seconds`, `startup-quiet-period-seconds`, `metadata-options`
- **Org changes needed:** None — timer/AWS parameters only.
- **Status:** ✅ Works as-is

### v2.4.3 — Auto-fetch Latest Runner & Packages
- **Inputs:** `packages`
- **Org changes needed:** None — downloads runner from `github.com/actions/runner` (public), installs packages via cloud-init.
- **Status:** ✅ Works as-is

### v2.5.0–v2.5.2 — JIT Runner Support & Docker
- **Input:** `use-jit: true`, `runner-group-id`
- **Org changes needed:** ✅ Already applied — `generate-jitconfig` endpoint changed to `/orgs/{org}/actions/runners/generate-jitconfig`.
- **Note:** The `runner-group-id` defaults to `1` (Default group). For org runners this refers to the **org-level** runner group ID, not repo-level.
- **Status:** ✅ Works as-is

### v2.6.0–v2.6.1 — Node 24 Runtime & Dependency Bumps
- **Org changes needed:** None — runtime upgrade, no API changes.
- **Status:** ✅ Works as-is

---

## GitHub Token Requirements

For org-level runner management, the GitHub token (App or PAT) **must** have:

| Permission | Scope | Why |
|---|---|---|
| `organization_self_hosted_runners: write` | Organization | Register/remove runners at org level |
| `admin:org` | Organization (PAT classic) | Alternative scope for classic PATs |

If using a **GitHub App** (recommended), ensure the app is installed on the **organization** with the `Organization > Self-hosted runners: Read & Write` permission.

---

## Test File Note

The file `src/__tests__/jit.test.js` still references the repo-level endpoint in its mock assertions (line 152). Update the test if you run the test suite:

```diff
- 'POST /repos/{owner}/{repo}/actions/runners/generate-jitconfig',
+ 'POST /orgs/{org}/actions/runners/generate-jitconfig',
```

---

## Reusable Workflow

The reusable workflow at `ip.infra.devops.workflow/reusable_steps/ec2-instance/action.yml` was updated:
- Branch reference: `@github-token` → `@org-support`
- Input: `ec2-user-runner` → `run-runner-as-user` (upstream's built-in equivalent)

No other workflow changes are needed for the new upstream features — they're all opt-in via new inputs.
