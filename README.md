# Argo CD multi-instance (root + ApplicationSet)

## 1) Set your repo URL
    grep -rl YOUR_USER . | xargs sed -i 's#https://github.com/YOUR_USER/YOUR_REPO.git#<your repo>#g'
Then push to branch `main` (private repo: add repo credentials yourself).

## 2) Bootstrap (once)
    kubectl apply --server-side -k argocd/crds
    kubectl wait --for condition=established --timeout=60s \
      crd/applications.argoproj.io crd/applicationsets.argoproj.io
    kubectl apply --server-side -k bootstrap

## 3) After that, git only
- Add a team: copy `argocd/instances/team-a` -> `team-c` (rename team-a -> team-c inside) and `apps/team-a` -> `apps/team-c`
- Add an app: create `apps/<team>/<app>/`
- Upgrade Argo CD: change version in `argocd/base` and `argocd/crds` (CRDs must be re-applied by hand)

## Verify
    kubectl get applications -n argocd
    kubectl get applicationsets,applications -n argocd-team-a
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d

## Notes
- `rbac.yaml` gives cluster-admin to each instance's controller (homework simplicity; restrict in real use)
- Root does not manage itself: changes to `bootstrap/` are applied by hand
