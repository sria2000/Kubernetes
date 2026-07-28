# 2. Workload & Scheduling — Topic Summary

| # | File | Title | Summary |
|---|------|-------|---------|
| 01 | labels-selectors.md | Labels & Selectors | Key-value tags on objects used for grouping/filtering; selectors let other objects (Services, ReplicaSets) query pods by label instead of hardcoded names. |
| 02 | replicaset.md | ReplicaSet | Controller that keeps a fixed number of identical pod replicas running; `selector` must match the pod template's labels or it won't adopt the pods. |
| 03 | deployment.md | Deployment | Manages ReplicaSets, which manage Pods (Deployment → ReplicaSet → Pods). Standard controller for stateless apps; adds rollout/rollback on top of ReplicaSet. |
| 04 | Worker-nodes.md | Worker Nodes | Why clusters run multiple worker nodes — spreads workloads for resilience (no single point of failure) and load distribution. |
| 05 | node-selector.md | Node Selector | Simplest scheduling constraint — hard requirement pinning a pod to nodes with an exact matching label (e.g. `disktype: ssd`). No match = pod stays Pending. |
| 06 | daemonset.md | DaemonSet | Guarantees exactly one pod copy per node (or labeled subset); used for node-level agents (logging, monitoring, CNI). Auto-adds/removes pods as nodes join/leave. |
| 07 | node-affinity.md | Node Affinity | More expressive version of nodeSelector — supports hard rules (`required...`) and soft preferences (`preferred...`) using operators like `In`/`NotIn`/`Exists`. |
| 08 | requests-limits.md | Resource Requests & Limits | `requests` drive scheduling decisions (minimum needed); `limits` cap runtime consumption. Prevents one pod from starving others on the same node. |
| 09 | Priorityclass.md | PriorityClass | Ranks pods by importance; under resource pressure, higher-priority pods can preempt (evict) lower-priority ones to get scheduled. |
| 10 | Sidecar-container.md | Sidecar Container | Helper container in the same pod as the main app (e.g. log shipper, proxy) that extends functionality without touching app code; shares network/volumes. |
| 11 | init_container.md | Init Container | Runs to completion before the main container starts; used for setup tasks (waiting on dependencies, config pull, migrations). Retries on failure. |
| 12 | Deployment_upgrade_rollback.md | Deployment Upgrade & Rollback | Hands-on lab: rolling update via `kubectl set image`, revision history (`rollout history`), rollback (`rollout undo --to-revision`), scaling, and change-cause annotations. |
| 13 | Liveliness-probe.md | Liveness Probe | Checks if a container is still alive/functioning; failure → kubelet **restarts** the container. Covers HTTP and Exec probe types plus timing parameters. |
| 14 | Readiness-probe.md | Readiness Probe | Checks if a container can currently accept traffic; failure → pod **removed from Service endpoints** (no restart). Covers exec probe example with `/tmp/healthy`. |

---

## Probe Quick Reference

| Probe | Question it answers | Failure action |
|---|---|---|
| Startup | Has the app finished starting? | Keep waiting |
| Liveness | Is the app alive? | Restart container |
| Readiness | Can it take traffic right now? | Remove from Service endpoints |
