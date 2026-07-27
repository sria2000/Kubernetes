# 6. How do you check how many Pods are running on each node using the wide option?

```
root@controlplane:~$ k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP              NODE           NOMINATED NODE   READINESS GATES
dep1-77bc6bd484-29p5j   1/1     Running   0          5m45s   192.168.1.233   node01         <none>           <none>
dep1-77bc6bd484-c9gpw   1/1     Running   0          5m45s   192.168.0.144   controlplane   <none>           <none>
dep1-77bc6bd484-ff8f4   1/1     Running   0          3m55s   192.168.1.198   node01         <none>           <none>
dep1-77bc6bd484-h47ww   1/1     Running   0          5m45s   192.168.1.13    node01         <none>           <none>
dep1-77bc6bd484-jmz26   1/1     Running   0          3m55s   192.168.0.193   controlplane   <none>           <none>
root@controlplane:~$
```
