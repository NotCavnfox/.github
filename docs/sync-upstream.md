# Shared upstream sync

`sync-upstream.yml` prepares a pull request in the caller fork. Keep the daily
schedule and optional stable-tag dispatch in that fork, and call this workflow
at a reviewed full commit SHA. Inputs are `upstream_repo`, `base_branch`, and
optional `upstream_ref` (published stable `vX.Y.Z`; empty means latest stable).

The caller grants contents, pull-requests, issues, and actions write permissions.
The workflow uses the caller repository context and its `GITHUB_TOKEN`; it needs
no inherited secrets, personal access token, server credentials, or new service.
Checkout intentionally targets the caller repository. Inputs are validated before
checkout and passed as environment data rather than interpolated shell code.

The sync merges the immutable upstream release into the captured fork revision.
An existing branch is reusable only with the same tree and both exact ancestors.
There are no force pushes or automatic merges. A closed PR is not reopened;
conflict issues are reused without repeated daily comments. CI is dispatched only
when the published SHA has no successful or pending matching workflow run.
Independent review still checks the exact PR SHA and current CI before merging.

GitHub's default token cannot push changes to workflow files. This limitation
remains visible as a failed push and requires separate reviewed publication with
existing authorized tooling. Moving code here does not add credential scopes or
make upstream workflow changes automatically publishable.

Release dispatch, source-SHA/CI/ancestry checks, publisher identity checks, and
image publication remain in the fork. Do not move them or restore automatic image
publication as part of this extraction. The existing `build-test.yml` is unchanged.

## Verification

Install `PyYAML==6.0.2`, then run `python test/test-fork-sync.py`. The shared CI
matrix also sets `SYNC_TEST_UPSTREAM_REPO=wizarrrr/wizarr` and
`SYNC_TEST_BASE_BRANCH=main` to exercise the same actual shell with both callers.
The tests use only disposable local Git remotes and a fake GitHub CLI.

Caller artifacts are prepared separately with an explicit unfilled SHA marker.
After this shared revision is reviewed and merged, pin both caller `uses` entries
to its actual full SHA and run their small contract checks plus existing release
regressions. Do not publish an unfilled marker or substitute a mutable branch.
