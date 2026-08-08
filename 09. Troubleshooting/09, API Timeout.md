Cluster Troubleshooting Scenario (Trading Infrastructure)
Scenario

You administer a Kubernetes cluster that runs risk-calculation and market-data-ingestion services for a trading desk. Every day at market open, request latency to internal APIs spikes and kubectl commands start intermittently timing out or returning connection-refused errors — right when traders most need the dashboards and risk checks to be responsive. The pattern is consistent: it's fine pre-open, degrades sharply in the first 15 minutes after open, and recovers by mid-morning. Leadership wants a repeatable investigation process, not a guess, before the next trading day.

Question: Walk through the investigation path you'd follow to diagnose this recurring, time-boxed degradation. How would you determine whether it's control-plane saturation, network instability, or node-level resource pressure — given that market-open traffic is bursty and predictable — and how would you isolate the failing layer without adding more load to an already-stressed cluster?

Answer

Because the degradation is time-boxed and repeatable, first correlate it against a known trigger: market-open traffic burst on the ingestion services. Check API server health (/readyz, /livez) and etcd latency during that window first, since a burst of API calls from newly-scaled ingestion pods is a classic control-plane saturation trigger. In parallel, check node resource pressure (CPU throttling, memory pressure) on the nodes hosting market-data pods — a burst workload spinning up many pods at once can starve the kubelet and API server on the same nodes. Isolate the layer by comparing reachability from outside the cluster, from a node, and from an in-cluster debug pod during the actual incident window (not after), since the symptom disappears within 15 minutes and post-hoc checks won't show it.

Explanation

The generic troubleshooting flow (API health → node conditions → system components → network path) still applies, but a predictable, recurring, time-boxed incident changes the investigation strategy in two important ways:

You can pre-position your diagnostics instead of reacting. Since the incident always happens in the same 15-minute window, the right move is to have watch loops, a debug pod, and kubectl top already running before market open, rather than trying to SSH in and start investigating after symptoms appear — by the time you connect, the window may have already closed.
The likely cause is workload-shape, not infrastructure failure. A market-data-ingestion service that scales out sharply at open (new pods scheduling, new connections opening, a burst of readiness/liveness probes hitting the API server) is a self-inflicted load spike, not a broken component. The investigation should specifically check whether HPA scale-events, pod churn, or probe frequency line up with the exact timestamps of the API timeouts — that's a different signature from a genuine control-plane outage.
Step-by-step
Before the next market open, set up standing observation: a watch on API health, a kubectl top loop, and an already-running debug pod in the cluster (not spun up reactively).
At market open, capture API health continuously: /readyz?verbose and /livez?verbose, timestamped, so you can correlate exact seconds against other signals.
Capture node conditions and resource pressure on nodes hosting market-data-ingestion and risk-calc pods specifically — CPU throttling and memory pressure on those nodes matter more than cluster-wide averages.
Check for HPA scaling events and pod creation timestamps on the ingestion Deployment — does the API timeout window line up with a burst of new pods scheduling?
Check kube-system: CoreDNS latency and kube-proxy/CNI health, since a burst of new pods means a burst of new DNS lookups and iptables/IPVS rule updates.
Compare reachability from three vantage points during the incident: your laptop (external), a node shell, and the pre-positioned in-cluster debug pod — whichever layer shows the timeout pinpoints where the break is.
If it's control-plane saturation: check etcd's WAL fsync and commit latency during the window — a burst of API writes (new pods, new endpoints) is exactly what pushes etcd over its latency threshold.
Apply the least-disruptive fix first — e.g., a PriorityClass for risk-calc pods so they aren't starved by ingestion pod churn, or a smoother scale-up (lower HPA step size, pre-scaling before open) — before considering control-plane resizing.
Code implementation
bash
# 1) Pre-position: start watching API health BEFORE market open
watch -n 2 "kubectl get --raw='/readyz?verbose' | tail -5"

# 2) API health with timestamps, logged for post-incident correlation
while true; do
  echo "$(date -u +%FT%TZ) $(kubectl get --raw='/livez?verbose' > /dev/null 2>&1 && echo OK || echo FAIL)"
  sleep 5
done

# 3) Node pressure on the specific nodes running market-data/risk-calc pods
kubectl get pods -n trading -o wide | grep -E 'market-data|risk-calc'
kubectl describe node <node-name> | egrep -i 'Ready|Pressure|Allocated|Events'

# 4) HPA and pod-churn correlation
kubectl get hpa -n trading -w
kubectl get events -n trading --sort-by=.lastTimestamp | egrep -i 'Scaled|Started|Pulled'

# 5) CoreDNS / kube-proxy health during the burst
kubectl -n kube-system get pods -l k8s-app=kube-dns -o wide
kubectl -n kube-system logs -l k8s-app=kube-dns --tail=100 | grep -i -E 'error|timeout|latency'

# 6) Pre-positioned debug pod (started before open, reused during the window)
kubectl run netcheck --image=nicolaka/netshoot --restart=Never -n trading -- sleep 3600
kubectl exec -n trading netcheck -- sh -lc '
  echo "API via Service:"; time curl -ksS https://kubernetes.default.svc/readyz?verbose | head -n 5;
  echo "DNS:"; time nslookup kubernetes.default.svc.cluster.local;
'

# 7) etcd latency (kubeadm/self-managed control plane, run on a control-plane node)
# crictl logs $(crictl ps --name etcd -q) --tail=200 | grep -i -E 'took too long|slow fsync|apply request took'
Explanation of code
Running the API-health watch/loop before market open, rather than reactively, is the key adaptation for a time-boxed, predictable incident — reactive diagnosis is too slow for a 15-minute window.
Filtering kubectl get pods -o wide and describe node to the nodes hosting market-data/risk-calc pods (rather than checking every node) narrows the search to where the burst actually lands, instead of averaging out the signal across a large cluster.
kubectl get hpa -w and filtering events for Scaled/Started/Pulled is what confirms or rules out the "self-inflicted load spike from scale-out" theory — if pod creation timestamps line up exactly with the API timeouts, that's strong evidence the burst itself is the trigger.
Pre-starting the netcheck debug pod with sleep 3600 and exec-ing into it during the window (rather than kubectl run --rm -it reactively) avoids adding more pod-creation load to the API server at the exact moment it's already stressed.
time curl .../readyz inside the debug pod gives an in-cluster latency number to compare directly against the external/laptop check — if in-cluster is fast but external is slow, that points at the load balancer or ingress path rather than the API server itself.
The etcd log grep for took too long/slow fsync/apply request took is etcd's own way of self-reporting that write latency (driven by the burst of new pod/endpoint objects) is the real bottleneck — this is the smoking gun for "control-plane saturation caused by workload burst" versus a genuine infrastructure fault.
