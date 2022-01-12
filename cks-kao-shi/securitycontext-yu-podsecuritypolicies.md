# securityContext与podsecurityPolicies

### 1. 介绍 <a href="#1__10" id="1__10"></a>

S[ecutyContext](https://blog.csdn.net/xixihahalelehehe/article/details/108539153)\
[https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin)

![](https://img-blog.csdnimg.cn/20210514140920852.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/202105141409585.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210514141050142.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210514141138804.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

### 2. Set Container User and Group <a href="#2_set_container_user_and_group_18" id="2_set_container_user_and_group_18"></a>

```
root@master:~/cks/securitytext# k run pod --image=busybox --command -oyaml --dry-run=client > pod.yaml -- sh -c 'sleep 1d'
root@master:~/cks/securitytext# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# k create -f pod.yaml 
root@master:~/cks/security# k get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          83s


root@master:~/cks/securitytext# k exec -ti pod -- sh
/ $ id
uid=1000 gid=3000

/ $ touch test
touch: test: Permission denied
/ $ cd /tmp
/tmp $ touch test
/tmp $ ls -lh
total 0      
-rw-r--r--    1 1000     3000           0 May 15 15:00 test
```

### 3. Force Container Non-Root <a href="#3_force_container_nonroot_67" id="3_force_container_nonroot_67"></a>

```
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}




root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod 
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          68s



#未指定非root用户
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created


oot@master:~/cks/securitytext# k get pod 
NAME   READY   STATUS                       RESTARTS   AGE
pod    0/1     CreateContainerConfigError   0          22s

#报错
root@master:~/cks/securitytext# k describe pod pod 
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  50s                default-scheduler  Successfully assigned default/pod to node2
  Normal   Pulled     33s                kubelet            Successfully pulled image "busybox" in 16.355541429s
  Warning  Failed     29s (x2 over 33s)  kubelet            Error: container has runAsNonRoot and image will run as root (pod: "pod_default(99f9dcab-cebe-4432-ad70-08c86d3ed9be)", container: pod)
  Normal   Pulled     29s                kubelet            Successfully pulled image "busybox" in 2.345508306s
  Normal   Pulling    16s (x3 over 49s)  kubelet            Pulling image "busybox"
```

### 4. Privileged Containers <a href="#4_privileged_containers_163" id="4_privileged_containers_163"></a>

\


![](https://img-blog.csdnimg.cn/20210515231135395.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210515231243387.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

### 5. Create Privileged Containers <a href="#5_create_privileged_containers_166" id="5_create_privileged_containers_166"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515231407815.png)

```
#非root用户测试执行sysctl命令
root@master:~/cks/securitytext# k get pods -w
NAME   READY   STATUS              RESTARTS   AGE
pod    0/1     ContainerCreating   0          3s
pod    1/1     Running             0          23s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ $ sysctl kernel.hostname=attacker
sysctl: error setting key 'kernel.hostname': Read-only file system   #无权限
/ $ exit
command terminated with exit code 1

#默认用户测试执行sysctl命令
root@master:~/cks/securitytext# vim pod.yaml  
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
#    securityContext:
#      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created

root@master:~/cks/securitytext# k get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          32s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # sysctl kernel.hostname=attacker
sysctl: error setting key 'kernel.hostname': Read-only file system  #无权限
/ # id
uid=0(root) gid=0(root) groups=10(wheel)


#添加privileged: true特权
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      privileged: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          36s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # sysctl kernel.hostname=attacker  #命令生效
kernel.hostname = attacker

```

### 6. PrivilegeEscalation <a href="#6_privilegeescalation_269" id="6_privilegeescalation_269"></a>

\


![](https://img-blog.csdnimg.cn/20210515232409858.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210515232446237.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

### 7. Practice - Disable PriviledgeEscalation <a href="#7_practice__disable_priviledgeescalation_273" id="7_practice__disable_priviledgeescalation_273"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515232455714.png)

```
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true 
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted

root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          29s


root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # cat /proc/1/status
Name:	sleep
State:	S (sleeping)
Tgid:	1
Ngid:	0
Pid:	1
PPid:	0
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	64
Groups:	10 
NStgid:	1
NSpid:	1
NSpgid:	1
NSsid:	1
VmPeak:	    1308 kB
VmSize:	    1308 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	       4 kB
VmRSS:	       4 kB
VmData:	      48 kB
VmStk:	     132 kB
VmExe:	     892 kB
VmLib:	18446744073709551612 kB
VmPTE:	      12 kB
VmPMD:	       8 kB
VmSwap:	       0 kB
HugetlbPages:	       0 kB
Threads:	1
SigQ:	0/7778
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000004
SigCgt:	0000000000000000
CapInh:	00000000a80425fb
CapPrm:	00000000a80425fb
CapEff:	00000000a80425fb
CapBnd:	00000000a80425fb
CapAmb:	0000000000000000
Seccomp:	0
Speculation_Store_Bypass:	thread vulnerable
Cpus_allowed:	00000000,00000000,00000000,00000003
Cpus_allowed_list:	0-1
Mems_allowed:	00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	47
nonvoluntary_ctxt_switches:	10
```

\
\


![](https://img-blog.csdnimg.cn/2021051523354412.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/2021051523360734.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210515233653646.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

### 9. Create and enable PodSecurityPolicy <a href="#9_create_and_enable_podsecuritypolicy_374" id="9_create_and_enable_podsecuritypolicy_374"></a>

[pod-security-policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)\
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515233718292.png)

```
root@master:~/cks/securitytext# vim /etc/kubernetes/manifests/kube-apiserver.yaml
---
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
---


root@master:~/cks/securitytext# cat psp.yaml 
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: default
spec:
  allowPrivilegeEscalation: false
  privileged: false  # Don't allow privileged pods!
  # The rest fills in some required fields.
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'


root@master:~/cks/securitytext# k create -f psp.yaml 
podsecuritypolicy.policy/default created


root@master:~/cks/securitytext# k create deploy nginx --image=nginx
deployment.apps/nginx created
root@master:~/cks/securitytext# k get deploy nginx -w
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     0            0           22s
^Croot@master:~/cks/securitytext#  k run nginx --image=nginx
pod/nginx created
root@master:~/cks/securitytext# k get pod nginx
NAME    READY   STATUS              RESTARTS   AGE
nginx   1/1     Running             0          44s

root@master:~/cks/securitytext#  k create role psp-access --verb=use --resource=podsecuritypolicies
role.rbac.authorization.k8s.io/psp-access created
root@master:~/cks/securitytext# k create rolebinding psp-access --role=psp-access --serviceaccount=default:default
rolebinding.rbac.authorization.k8s.io/psp-access created
root@master:~/cks/securitytext# k get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     0            0           3m26s
root@master:~/cks/securitytext# k delete deploy nginx
deployment.apps "nginx" deleted
root@master:~/cks/securitytext# k create deploy nginx --image=nginx
deployment.apps/nginx created
^Croot@master:~/cks/securitytext# k get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           20s
```

allowPrivilegeEscalation设置为rue

```
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/securitytext# k -f pod.yaml create
Error from server (Forbidden): error when creating "pod.yaml": pods "pod" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.allowPrivilegeEscalation: Invalid value: true: Allowing privilege escalation for containers is not allowed]
```

[更多细节参考](https://www.cnblogs.com/sunsky303/p/11090540.html)
