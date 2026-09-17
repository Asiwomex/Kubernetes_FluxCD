# Drift-correction demo

This is the actual proof that Flux is doing GitOps, not just running a one-time
`kubectl apply`. It needs a live cluster with Flux bootstrapped (see
[setup.md](setup.md)), so it's written as a script for you to run and record —
it can't be captured from here.

## What you're demonstrating

Git says the `dev` app should run 1 replica. You'll manually scale it to 5
directly against the cluster (bypassing Git entirely), then show Flux noticing
the live state doesn't match the repo and reverting it back to 1 — with no
`kubectl apply` from you.

## Steps

1. **Confirm the starting state matches Git.**

   ```sh
   kubectl get deployment app -n dev
   # DESIRED should read 1
   ```

2. **Introduce drift.** Change something live, without touching Git:

   ```sh
   kubectl scale deployment app -n dev --replicas=5
   kubectl get deployment app -n dev
   # DESIRED now reads 5 — the cluster has drifted from the repo
   ```

3. **Record the drift being caught.** Either wait for the next automatic
   reconcile (`spec.interval: 1m0s` on the `todolist-dev` Kustomization) or force
   it immediately:

   ```sh
   flux reconcile kustomization todolist-dev --with-source
   ```

4. **Confirm it self-healed:**

   ```sh
   kubectl get deployment app -n dev
   # DESIRED is back to 1
   ```

5. **Show the event, not just the before/after.** This is the part that makes the
   demo convincing — capture the controller actually doing the correction:

   ```sh
   flux logs --kind=Kustomization --name=todolist-dev --since=5m
   kubectl get events -n dev --sort-by=.lastTimestamp | tail -20
   ```

## What to capture and where to put it

Record a terminal session covering steps 1–5 in one continuous take (a short
[asciinema](https://asciinema.org/) recording or a screen-capture GIF both work
well for a README). Save it as `docs/screenshots/drift-demo.gif` (or link an
asciinema URL) and reference it from the "Drift correction, live" section of the
main [README](../README.md) — that section currently has a placeholder for it.

At minimum, capture:
- the `kubectl scale ... --replicas=5` command and its confirmation
- `kubectl get deployment app -n dev` showing `5`
- the `flux reconcile` (or the automatic interval firing, via `flux logs`)
- `kubectl get deployment app -n dev` showing it back at `1`

## A second demo, if you have time

The rebuild brief also suggested a rollback demo. The cheapest way to do it here:
push a commit that changes `apps/todolist/base/deployment.yaml` to reference a
tag that doesn't exist (e.g. `asiwomex/django-todolist:does-not-exist`), watch
`flux get kustomizations` and `kubectl get pods -n dev` show the resulting
`ImagePullBackOff`, then `git revert` that commit and show Flux pulling the
revert and restoring healthy pods — same self-healing story, applied to a bad
release instead of manual drift.
