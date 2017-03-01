

Pod Creation Lifecycle:
===

* [Start state JSON](pending.json)
* [Running state JSON](running.json)

Example Flow from [OpenStack Kuryr-Kubernetes](https://github.com/openstack/kuryr-kubernetes):

Definitions
---
* User - A user or client interacting with kubernetes requesting the creation of a new pod
* Kuryr Controller - A centralized service which watches for pod create events and sets up network ports on neutron for the pod to use. The controller syncrhronizes kubernetes and neutron state. The controller is not responsible for directly creating the linux interface or manipulating the network namespace.
* Kubernetes API - The interface kubernetes uses for monitoring and manipulating the cluster state
* Kubelet - An agent that runs on a compute node. It is responsible for spawning pods and containers. Will execute a CNI plugin to set up networking
* CNI Plugin - Responsible for creating the linux interface and moving the interface into the correct namespace.

![Pod Creation Workflow Diagram](https://raw.githubusercontent.com/openstack/kuryr-kubernetes/master/doc/images/pod_creation_flow.png)

```sh
$ oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git
ruby-ex-1-deploy   0/1       Pending   0          1s
ruby-ex-1-deploy   0/1       Pending   0         1s
ruby-ex-1-deploy   0/1       ContainerCreating   0         1s
ruby-ex-1-build   0/1       Completed   0         2m
ruby-ex-1-deploy   0/1       ContainerCreating   0         3s
ruby-ex-1-deploy   1/1       Running   0         4s
ruby-ex-1-hhmaj   0/1       Pending   0         0s
ruby-ex-1-hhmaj   0/1       Pending   0         0s
ruby-ex-1-hhmaj   0/1       ContainerCreating   0         0s
ruby-ex-1-hhmaj   0/1       ContainerCreating   0         2s
ruby-ex-1-hhmaj   1/1       Running   0         4s
ruby-ex-1-deploy   0/1       Completed   0         9s
ruby-ex-1-deploy   0/1       Terminating   0         9s
ruby-ex-1-deploy   0/1       Terminating   0         9s
```

```diff
diff pod1.json pod2.json 
10c10
<         "resourceVersion": "1230",
---
>         "resourceVersion": "1232",
82a83
>         "nodeName": "192.168.42.164",
96c97,105
<         "phase": "Pending"
---
>         "phase": "Pending",
>         "conditions": [
>             {
>                 "type": "PodScheduled",
>                 "status": "True",
>                 "lastProbeTime": null,
>                 "lastTransitionTime": "2017-03-01T11:03:49Z"
>             }
>         ]
```
```diff
diff pod2.json pod3.json 
10c10
<         "resourceVersion": "1232",
---
>         "resourceVersion": "1234",
99a100,113
>                 "type": "Initialized",
>                 "status": "True",
>                 "lastProbeTime": null,
>                 "lastTransitionTime": "2017-03-01T11:03:49Z"
>             },
>             {
>                 "type": "Ready",
>                 "status": "False",
>                 "lastProbeTime": null,
>                 "lastTransitionTime": "2017-03-01T11:03:49Z",
>                 "reason": "ContainersNotReady",
>                 "message": "containers with unready status: [ruby-ex]"
>             },
>             {
103a118,135
>             }
>         ],
>         "hostIP": "192.168.42.164",
>         "startTime": "2017-03-01T11:03:49Z",
>         "containerStatuses": [
>             {
>                 "name": "ruby-ex",
>                 "state": {
>                     "waiting": {
>                         "reason": "ContainerCreating"
>                     }
>                 },
>                 "lastState": {},
>                 "ready": false,
>                 "restartCount": 0,
>                 "image":
> "172.30.45.92:5000/default/ruby-ex@sha256:53ab21552d0873265b2e06f00bc29f3b4699ecb7c5a7b77b2d189e44b2c3ce1b",
>                 "imageID": ""
```
```diff
diff pod3.json pod4.json 
10c10
<         "resourceVersion": "1234",
---
>         "resourceVersion": "1240",
120a121
>         "podIP": "172.17.0.2",
```
```diff
diff pod4.json pod5.json 
10c10
<         "resourceVersion": "1240",
---
>         "resourceVersion": "1246",
97c97
<         "phase": "Pending",
---
>         "phase": "Running",
107c107
<                 "status": "False",
---
>                 "status": "True",
109,111c109
<                 "lastTransitionTime": "2017-03-01T11:03:49Z",
<                 "reason": "ContainersNotReady",
<                 "message": "containers with unready status: [ruby-ex]"
---
>                 "lastTransitionTime": "2017-03-01T11:03:53Z"
127,128c125,126
<                     "waiting": {
<                         "reason": "ContainerCreating"
---
>                     "running": {
>                         "startedAt": "2017-03-01T11:03:52Z"
132c130
<                 "ready": false,
---
>                 "ready": true,
136c134,136
<                 "imageID": ""
---
>                 "imageID":
> "docker-pullable://172.30.45.92:5000/default/ruby-ex@sha256:53ab21552d0873265b2e06f00bc29f3b4699ecb7c5a7b77b2d189e44b2c3ce1b",
>                 "containerID": "docker://f1f9e3cd1058a6452e8a5c25192b1f4b1eb1d5f42df8042402eeedc395a63715"
```
