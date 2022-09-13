
Sample app to illustrate log4j vulnerability detection in RHACS.

# Deploy

Create app in `log4shell` namespace:

```bash
oc apply -k base
```

# Demo

View violation of "Log4Shell: log4j Remote Code Execution vulnerability" in the Violations menu.

Modify the policy response method from "Inform" to "Inform and enforce" during the Build and Deploy lifecycle stages.

![acs-policy-behavior.png](acs-policy-behavior.png)

To avoid an outage, running applications will not be automatically terminated by a Deploy lifecycle policy. 

Remove and redeploy the application.

```bash
oc delete -k base
oc apply -k base
```

Vulnerability will be detected and automatically scaled to zero at deployment time.

Example on a SNO cluster. 
Behavior is not what I expected to see.

```
# log4j policy is set to enforce at build, deploy:
(⎈ |demo-lab-bewley-net/system:admin:log4shell)$ oc apply -k base
namespace/log4shell created
service/log4shell created
deployment.apps/log4shell created
imagestream.image.openshift.io/log4shell created
(⎈ |demo-lab-bewley-net/system:admin:log4shell)$
(⎈ |demo-lab-bewley-net/system:admin:log4shell)$ oc get dc,rs,pod -n log4shell
NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/log4shell-5d477577dc   1         1         1       4m29s
replicaset.apps/log4shell-6599975f67   0         0         0       4m30s

NAME                             READY   STATUS    RESTARTS   AGE
pod/log4shell-5d477577dc-6cx2s   1/1     Running   0          4m29s
(⎈ |demo-lab-bewley-net/system:admin:log4shell)$ oc describe deployment log4shell | tail

  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   log4shell-5d477577dc (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m52s  deployment-controller  Scaled up replica set log4shell-6599975f67 to 1
  Normal  ScalingReplicaSet  4m51s  deployment-controller  Scaled up replica set log4shell-5d477577dc to 1
  Normal  ScalingReplicaSet  4m47s  deployment-controller  Scaled down replica set log4shell-6599975f67 to 0
```
# Background

App was created as follows:

```bash
oc new-app \
    --name log4shell \
    --image quay.io/tjungbau/log4shell-app:latest \
    --dry-run \
    -o yaml > base/application.yaml
```


