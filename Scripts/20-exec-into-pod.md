# 10. How do you execute commands inside a running Pod?

```
root@controlplane:~$ k get pod
NAME                    READY   STATUS    RESTARTS   AGE
dep1-77bc6bd484-29p5j   1/1     Running   0          35m
dep1-77bc6bd484-c9gpw   1/1     Running   0          35m
dep1-77bc6bd484-ff8f4   1/1     Running   0          33m
dep1-77bc6bd484-h47ww   1/1     Running   0          35m
dep1-77bc6bd484-jmz26   1/1     Running   0          33m
dep2-7f984fdbcd-ns492   1/1     Running   0          20m
nginx                   1/1     Running   0          11m
root@controlplane:~$ k exec -it nginx -- /bin/bash
root@nginx:/# uname -a
Linux nginx 6.8.0-136-generic #136-Ubuntu SMP PREEMPT_DYNAMIC Wed Jul  1 21:53:05 UTC 2026 x86_64 GNU/Linux
root@nginx:/# date
Mon Jul 27 16:50:31 UTC 2026
root@nginx:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          19G  8.6G  9.9G  47% /
tmpfs            64M     0   64M   0% /dev
/dev/vda1        19G  8.6G  9.9G  47% /etc/hosts
shm              64M     0   64M   0% /dev/shm
tmpfs           1.8G   12K  1.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           952M     0  952M   0% /proc/acpi
tmpfs           952M     0  952M   0% /proc/scsi
tmpfs           952M     0  952M   0% /sys/firmware
root@nginx:/#
```

**Note:** `kubectl exec -it <pod>` targets the first/only container by default. For a multi-container pod, specify which container with `-c <container-name>`.
