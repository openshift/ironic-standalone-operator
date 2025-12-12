# Merging Upstream to Downstream

This commands merges changes from upstream
`metal3-io/ironic-standalone-operator` into its downstream fork
`openshift/ironic-standalone-operator`. This operation is necessary to keep the
downstream fork up-to-date with any upstream changes.

### 1. Preparation

Ensure your local repository is up to date by fetching all remote branches from
git.

Check the current branch. Here you need to make sure that:
- There are no local changes you may overwrite
- You are on the right branch that is based on the `main` branch of
  `openshift/ironic-standalone-operator`
- You're on the latest commit in that branch.

Check if the remote branch `main` on `metal3-io/ironic-standalone-operator` has
any outstanding changes compared to the current branch.

Create a new branch for the merge named `merge-upstream-$(date +%Y-%m-%d)`.

### 2. Merge Upstream Changes

Merge the upstream branch into the current one, replacing `upstream` with the
correct name of the remote branch, if necessary:

```bash
git merge --no-ff --no-commit upstream/main
```

### 3. Handle Common Conflicts

**GitHub Workflows**: Downstream typically removes `.github/` directory files. If conflicts occur:
- Keep the downstream version (usually removal)
- Verify with `git diff downstream/main -- .github/`

**Downstream-Specific Changes**: Look for commit messages starting with:
- `DOWNSTREAM:` - Changes specific to OpenShift
- `OCPBUGS-*:` - OpenShift bug fixes

These must be preserved during the merge, unless replaced by an equivalent
upstream commit.

### 4. Downstream specific actions

**OWNERS**: Revert any changes to `OWNERS`, you must keep the downstream
version of this file.

**Go Dependencies**: After merging, update dependencies:
```bash
# Clean up and tidy all go.mod files across all modules
make modules

# Update vendor directories for all modules
make vendor

# These commands handle:
# - Root module (go.mod)
# - api/ module
# - test/ module
# - hack/tools module
```

Include any changes to the `vendor` directories in the future commit.
Note that all dependencies must be vendored in OpenShift. You cannot rely on
downloading modules. Do not use `-mod=mod` with `go build`.

### 5. Verify the Merge

Before creating a PR, ensure:
- There are no more outstanding changes in the upstream branch
- Code compiles: `make build`
- Tests pass: `make test`
- Manifests are up to date: `make manifests`
- Code is properly formatted: `make fmt`
- Linting passes: `make lint`

Hint: all `make` commands must be run from the root directory of the
repository.

### 6. Commit and Push

Add all modified files (including updated vendored dependencies) into the git
staging area.

Generate the changelog to use in the commit message with with
`git log --oneline --no-merges upstream/main ^HEAD`.

Use the following commit command, replacing the upstream git hash
and the changelog:

```bash
git commit -m "Merge upstream metal3-io/ironic-standalone-operator

Merging changes from upstream metal3-io/ironic-standalone-operator
as of <INSERT UPSTREAM HASH HERE>.

<INSERT CHANGELOG HERE>
"
```

Push to the user's fork (replacing `origin` with the name of the user's
personal fork):

```bash
git push -u origin merge-upstream-$(date +%Y-%m-%d)
```

Tell the user how to create the pull request using the base repository
`openshift/ironic-standalone-operator`.
