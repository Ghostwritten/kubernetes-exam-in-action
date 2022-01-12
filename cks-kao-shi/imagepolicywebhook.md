# ImagePolicyWebhook

#### 文章目录

*
  * [1. 介绍](broken-reference)
  * [2. Image 摘要](broken-reference)
  * [3. OPA注册白名单](broken-reference)
  * [4. ImagePolicyWebhook](broken-reference)

### 1. 介绍 <a href="#1__3" id="1__3"></a>

\
\
\


![](https://img-blog.csdnimg.cn/20210521162315240.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210521162413722.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210521162625720.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210521163827152.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

### 2. Image 摘要 <a href="#2_image__8" id="2_image__8"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052116390744.png)

```
root@master:~# k get pods -A -oyaml |grep image: |grep -v f:
      image: busybox
      image: ubuntu
      image: busybox:latest
      image: ubuntu:latest
    - image: nginx
      image: nginx:latest
    - image: nginx
      image: nginx:latest
    - image: nginx
      image: nginx:latest
      image: openpolicyagent/gatekeeper:469f747
      image: openpolicyagent/gatekeeper:469f747
      image: openpolicyagent/gatekeeper:469f747
      image: openpolicyagent/gatekeeper:469f747
      image: calico/kube-controllers:v3.16.6
      image: calico/kube-controllers:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
    - image: calico/pod2daemon-flexvol:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/pod2daemon-flexvol:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
    - image: calico/pod2daemon-flexvol:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/pod2daemon-flexvol:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
    - image: calico/pod2daemon-flexvol:v3.16.6
      image: calico/node:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/cni:v3.16.6
      image: calico/pod2daemon-flexvol:v3.16.6
      image: k8s.gcr.io/coredns:1.7.0
      image: k8s.gcr.io/coredns:1.7.0
      image: k8s.gcr.io/coredns:1.7.0
      image: k8s.gcr.io/coredns:1.7.0
      image: k8s.gcr.io/etcd:3.4.13-0
      image: k8s.gcr.io/etcd:3.4.13-0
      image: k8s.gcr.io/kube-apiserver:v1.20.7
      image: k8s.gcr.io/kube-apiserver:v1.20.7
      image: k8s.gcr.io/kube-controller-manager:v1.20.7
      image: k8s.gcr.io/kube-controller-manager:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-proxy:v1.20.7
      image: k8s.gcr.io/kube-scheduler:v1.20.7
      image: k8s.gcr.io/kube-scheduler:v1.20.7


root@master:~# k get pods -A -oyaml |grep image: |grep -v f: |grep api
      image: k8s.gcr.io/kube-apiserver:v1.20.7
      image: k8s.gcr.io/kube-apiserver:v1.20.7

#拷贝sha256
root@master:~# k get pod -n kube-system kube-apiserver-master -oyaml |grep image
            f:image: {}
            f:imagePullPolicy: {}
    image: k8s.gcr.io/kube-apiserver:v1.20.7
    imagePullPolicy: IfNotPresent
    image: k8s.gcr.io/kube-apiserver:v1.20.7
    imageID: docker://sha256:034671b24f0f15cf0023e70edad18026c6885c18a78d1eca9489fabc4b2a28f1


root@master:~/imagev1.20.7# vim /etc/kubernetes/manifests/kube-apiserver.yaml
.....
    image: k8s.gcr.io/kube-apiserver@sha256:034671b24f0f15cf0023e70edad18026c6885c18a78d1eca9489fabc4b2a28f1  #修改版本号改为sha256
.....................


root@master:~/imagev1.20.7# k get pod -n kube-system |grep api
kube-apiserver-master                      1/1     Running   0          2m42s
```

### 3. OPA注册白名单 <a href="#3_opa_99" id="3_opa_99"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521165928411.png?xshadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

```
# cat k8strustedimages_template.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8strustedimages
spec:
  crd:
    spec:
      names:
        kind: K8sTrustedImages
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8strustedimages

        violation[{"msg": msg}] {
          image := input.review.object.spec.containers[_].image
          not startswith(image, "docker.io/")
          not startswith(image, "k8s.gcr.io/")
          msg := "not trusted image!"
        }


# cat all_pod_must_have_trusted_images.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sTrustedImages
metadata:
  name: pod-trusted-images
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]


root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k -f k8strustedimages_template.yaml create
constrainttemplate.templates.gatekeeper.sh/k8strustedimages created
root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k -f all_pod_must_have_trusted_images.yaml create
k8strustedimages.constraints.gatekeeper.sh/pod-trusted-images created


root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k get k8strustedimages
NAME                 AGE
pod-trusted-images   2m56s



root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k describe k8strustedimages pod-trusted-images
Name:         pod-trusted-images
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sTrustedImages
Metadata:
  Creation Timestamp:  2021-05-21T09:07:04Z
  Generation:          1
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
    Manager:      kubectl-create
    Operation:    Update
    Time:         2021-05-21T09:07:04Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:auditTimestamp:
        f:byPod:
        f:totalViolations:
        f:violations:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2021-05-21T09:08:06Z
  Resource Version:  150646
  UID:               fc9f394e-a088-42c5-bcd5-9f506879b8fa
Spec:
  Match:
    Kinds:
      API Groups:
        
      Kinds:
        Pod
Status:
  Audit Timestamp:  2021-05-21T09:21:41Z
  By Pod:
    Constraint UID:       fc9f394e-a088-42c5-bcd5-9f506879b8fa
    Enforced:             true
    Id:                   gatekeeper-audit-65f658df68-jql6v
    Observed Generation:  1
    Operations:
      audit
      status
    Constraint UID:       fc9f394e-a088-42c5-bcd5-9f506879b8fa
    Enforced:             true
    Id:                   gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Observed Generation:  1
    Operations:
      webhook
  Total Violations:  11
  Violations:
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                app
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                app
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                pod
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                test-5f6778868d-r7pnj
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                test-5f6778868d-vl7jd
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-audit-65f658df68-jql6v
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                calico-kube-controllers-57fc9c76cc-t45js
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                calico-node-jkfm9
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                calico-node-lrkbp
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                calico-node-pdwqm
    Namespace:           kube-system
Events:                  <none>



root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k run nginx --image=nginx
Error from server ([denied by pod-trusted-images] not trusted image!): admission webhook "validation.gatekeeper.sh" denied the request: [denied by pod-trusted-images] not trusted image!

root@master:~/cks/supply-chain-security/secure-the-supply-chain/whitelist-registries/opa# k run nginx --image=docker.io/nginx
pod/nginx created
```

\


![](https://img-blog.csdnimg.cn/20210521172745471.png?x-oss-process=image/watermark,type\_ZmFuZ3poZW5naGVpdGk,shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210521172804290.png?x-oss-process=image/watermark,type\_ZmFuZ3poZW5naGVpdGk,shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521172908435.png)

```
root@master:~# ls cks/cks-course-environment-master/course-content/supply-chain-security/secure-the-supply-chain/whitelist-registries/ImagePolicyWebhook/
admission_config.yaml  apiserver-client-cert.pem  apiserver-client-key.pem  external-cert.pem  external-key.pem  kubeconf

root@master:~# cp /root/cks/cks-course-environment-master/course-content/supply-chain-security/secure-the-supply-chain/whitelist-registries/ImagePolicyWebhook/* root@master:~#/etc/kubernetes/admission/
root@master:~# mkdir /etc/kubernetes/admission


root@master:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.211.40:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --admission-control-config-file=/etc/kubernetes/admission/admission_config.yaml #添加此行
    - --advertise-address=192.168.211.40
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook  # #修改此行
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.20.7
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.211.40
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/admission  #添加此行
      name: k8s-admission    #添加此行
      readOnly: true    #添加此行
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:    #添加此行
      path: /etc/kubernetes/admission   #添加此行
      type: DirectoryOrCreate    #添加此行
    name: k8s-admission    #添加此行
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}


root@master:~# k get nodes
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   9d    v1.20.1
node1    Ready    <none>                 9d    v1.20.1
node2    Ready    <none>                 9d    v1.20.1

#创建pod失败
root@master:~# k run test --image=nginx
Error from server (Forbidden): pods "test" is forbidden: Post "https://external-service:1234/check-image?timeout=30s": dial tcp: lookup external-service on 8.8.8.8:53: no such host

#修改admission_config.yaml 配置
root@master:~# vim /etc/kubernetes/admission/admission_config.yaml 
root@master:~# cat /etc/kubernetes/admission/admission_config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: ImagePolicyWebhook
    configuration:
      imagePolicy:
        kubeConfigFile: /etc/kubernetes/admission/kubeconf
        allowTTL: 50
        denyTTL: 50
        retryBackoff: 500
        defaultAllow: true    #修改此行为true


root@master:/etc/kubernetes/manifests# mv kube-apiserver.yaml ../
root@master:/etc/kubernetes/manifests# ps -ef |grep api
root      78871  39023  0 20:17 pts/3    00:00:00 grep --color=auto api
root@master:/etc/kubernetes/manifests# ps -ef |grep api^C
root@master:/etc/kubernetes/manifests# mv ../kube-apiserver.yaml .

root@master:~/imagev1.20.7# k -n kube-system get pod
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57fc9c76cc-t45js   1/1     Running   0          9d
calico-node-jkfm9                          1/1     Running   0          9d
calico-node-lrkbp                          1/1     Running   0          9d
calico-node-pdwqm                          1/1     Running   0          9d
coredns-74ff55c5b-gmfsj                    1/1     Running   0          9d
coredns-74ff55c5b-s6lt8                    1/1     Running   0          9d
etcd-master                                1/1     Running   0          9d
kube-apiserver-master                      1/1     Running   0          39s
kube-controller-manager-master             1/1     Running   2          9d
kube-proxy-6m8q6                           1/1     Running   0          9d
kube-proxy-8vprs                           1/1     Running   0          9d
kube-proxy-b6r2s                           1/1     Running   0          9d
kube-scheduler-master                      1/1     Running   2          9d

#创建pod成功
root@master:~/imagev1.20.7# k run test --image=nginx
pod/test created
```
