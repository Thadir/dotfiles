# Working in this repo

`main` is branch-protected: changes are required to go through a pull request,
and the "lint" status check must pass before merging. I (the owner) have a
bypass permission that lets a direct push to `main` go through anyway, but
that bypass is not the intended flow — treat the PR route as the default.

For any change here (including small fixes), the normal flow is:

1. Create a feature branch off `main`.
2. Commit the change there and push the branch.
3. Open a pull request against `main`.
4. Wait for the "lint" check to pass (fix and re-push if it fails).
5. Merge via the PR rather than pushing directly to `main`.

Only push straight to `main` if I explicitly ask for that for a specific
change.
