# /proc and Env

### 1. /proc and env variables <a href="#4_proc_and_env_variables_514" id="4_proc_and_env_variables_514"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524140010274.png)

```
root@master:~/cks/runtime-security# k run apache --image=httpd -oyaml --dry-run=client > pod.yaml
root@master:~/cks/runtime-security# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: apache
  name: apache
spec:
  containers:
  - image: httpd
    name: apache
    resources: {}
    env: 
    - name: SECRET
      value: "5555666677778888"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/apache created


root@master:~/cks/runtime-security# k get pods -owide |grep apache
apache   1/1     Running   0          89s    10.244.104.3   node2   <none>           <none>

root@node2:~# ps aux |grep httpd
root     123888  0.0  0.2   5940  4420 ?        Ss   23:04   0:00 httpd -DFOREGROUND
daemon   123921  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
daemon   123922  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
daemon   123923  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
root     126181  0.0  0.0  14408  1004 pts/0    S+   23:05   0:00 grep --color=auto httpd
root@node2:~# docker ps |grep httpd
ced29b338f66        httpd                  "httpd-foreground"       About a minute ago   Up About a minute                       k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
root@node2:~# docker ps |grep apache
ced29b338f66        httpd                  "httpd-foreground"       About a minute ago   Up About a minute                       k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
55f829e84a8d        k8s.gcr.io/pause:3.2   "/pause"                 2 minutes ago        Up 2 minutes                            k8s_POD_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
root@node2:~# 



root@node2:~# pstree -p

...................
     │                 ├─containerd-shim(123862)─┬─httpd(123888)─┬─httpd(123921)─┬─{httpd}(123953)
           │                 │                         │               │               ├─{httpd}(123954)
           │                 │                         │               │               ├─{httpd}(123955)
           │                 │                         │               │               ├─{httpd}(123957)
           │                 │                         │               │               ├─{httpd}(123961)
           │                 │                         │               │               ├─{httpd}(123963)
           │                 │                         │               │               ├─{httpd}(123965)
           │                 │                         │               │               ├─{httpd}(123967)
           │                 │                         │               │               ├─{httpd}(123969)
           │                 │                         │               │               ├─{httpd}(123971)
           │                 │                         │               │               ├─{httpd}(123974)
           │                 │                         │               │               ├─{httpd}(123976)
           │                 │                         │               │               ├─{httpd}(123978)
           │                 │                         │               │               ├─{httpd}(123980)
           │                 │                         │               │               ├─{httpd}(123982)
           │                 │                         │               │               ├─{httpd}(123984)
           │                 │                         │               │               ├─{httpd}(123986)
           │                 │                         │               │               ├─{httpd}(123988)
           │                 │                         │               │               ├─{httpd}(123990)
           │                 │                         │               │               ├─{httpd}(123992)
           │                 │                         │               │               ├─{httpd}(123994)
           │                 │                         │               │               ├─{httpd}(123996)
           │                 │                         │               │               ├─{httpd}(123998)
           │                 │                         │               │               ├─{httpd}(124000)
           │                 │                         │               │               ├─{httpd}(124002)
           │                 │                         │               │               └─{httpd}(124004)
           │                 │                         │               ├─httpd(123922)─┬─{httpd}(123956)
           │                 │                         │               │               ├─{httpd}(123958)
           │                 │                         │               │               ├─{httpd}(123959)
           │                 │                         │               │               ├─{httpd}(123960)
           │                 │                         │               │               ├─{httpd}(123962)
           │                 │                         │               │               ├─{httpd}(123964)
           │                 │                         │               │               ├─{httpd}(123966)
           │                 │                         │               │               ├─{httpd}(123968)
           │                 │                         │               │               ├─{httpd}(123970)
           │                 │                         │               │               ├─{httpd}(123972)
           │                 │                         │               │               ├─{httpd}(123975)
           │                 │                         │               │               ├─{httpd}(123977)
           │                 │                         │               │               ├─{httpd}(123979)
           │                 │                         │               │               ├─{httpd}(123981)
           │                 │                         │               │               ├─{httpd}(123983)
           │                 │                         │               │               ├─{httpd}(123985)
           │                 │                         │               │               ├─{httpd}(123987)
           │                 │                         │               │               ├─{httpd}(123989)
           │                 │                         │               │               ├─{httpd}(123991)
           │                 │                         │               │               ├─{httpd}(123993)
           │                 │                         │               │               ├─{httpd}(123995)
           │                 │                         │               │               ├─{httpd}(123997)
           │                 │                         │               │               ├─{httpd}(123999)
           │                 │                         │               │               ├─{httpd}(124001)
           │                 │                         │               │               ├─{httpd}(124003)
           │                 │                         │               │               └─{httpd}(124005)
           │                 │                         │               └─httpd(123923)─┬─{httpd}(123926)
           │                 │                         │                               ├─{httpd}(123927)
           │                 │                         │                               ├─{httpd}(123928)
           │                 │                         │                               ├─{httpd}(123929)
           │                 │                         │                               ├─{httpd}(123930)
           │                 │                         │                               ├─{httpd}(123931)
           │                 │                         │                               ├─{httpd}(123932)
           │                 │                         │                               ├─{httpd}(123933)
           │                 │                         │                               ├─{httpd}(123934)
           │                 │                         │                               ├─{httpd}(123935)
           │                 │                         │                               ├─{httpd}(123936)
           │                 │                         │                               ├─{httpd}(123937)
           │                 │                         │                               ├─{httpd}(123938)
           │                 │                         │                               ├─{httpd}(123939)
           │                 │                         │                               ├─{httpd}(123940)
           │                 │                         │                               ├─{httpd}(123941)
           │                 │                         │                               ├─{httpd}(123942)
           │                 │                         │                               ├─{httpd}(123943)
           │                 │                         │                               ├─{httpd}(123944)
           │                 │                         │                               ├─{httpd}(123945)
           │                 │                         │                               ├─{httpd}(123946)
           │                 │                         │                               ├─{httpd}(123947)
           │                 │                         │                               ├─{httpd}(123948)
           │                 │                         │                               ├─{httpd}(123949)
           │                 │                         │                               ├─{httpd}(123950)
           │                 │                         │                               └─{




root@node2:~# cd /proc/123888
root@node2:/proc/123888# ls
attr       cgroup      comm             cwd      fd       io        map_files  mountinfo   net        oom_adj        pagemap      root       sessionid  stack  status   timers
autogroup  clear_refs  coredump_filter  environ  fdinfo   limits    maps       mounts      ns         oom_score      personality  sched      setgroups  stat   syscall  uid_map
auxv       cmdline     cpuset           exe      gid_map  loginuid  mem        mountstats  numa_maps  oom_score_adj  projid_map   schedstat  smaps      statm  task     wchan
root@node2:/proc/123888# ls -lh exe 
lrwxrwxrwx 1 root root 0 May 23 23:04 exe -> /usr/local/apache2/bin/httpd
root@node2:/proc/123888# cd fd
root@node2:/proc/123888/fd# ls -lh 
total 0
lrwx------ 1 root root 64 May 23 23:04 0 -> /dev/null
l-wx------ 1 root root 64 May 23 23:04 1 -> pipe:[15919327]
l-wx------ 1 root root 64 May 23 23:04 2 -> pipe:[15919328]
lrwx------ 1 root root 64 May 23 23:04 3 -> socket:[15919481]
lr-x------ 1 root root 64 May 23 23:04 4 -> pipe:[15919497]
l-wx------ 1 root root 64 May 23 23:04 5 -> pipe:[15919497]
l-wx------ 1 root root 64 May 23 23:04 6 -> pipe:[15919327]
root@node2:/proc/123888/fd# cd ..
root@node2:/proc/123888# cat environ 
KUBERNETES_SERVICE_PORT=443KUBERNETES_PORT=tcp://10.96.0.1:443HTTPD_VERSION=2.4.46HOSTNAME=apacheHOME=/rootHTTPD_PATCHES=KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1PATH=/usr/local/apache2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binKUBERNETES_PORT_443_TCP_PORT=443KUBERNETES_PORT_443_TCP_PROTO=tcpHTTPD_SHA256=740eddf6e1c641992b22359cabc66e6325868c3c5e2e3f98faf349b61ecf41eaSECRET=5555666677778888HTTPD_PREFIX=/usr/local/apache2KUBERNETES_SERVICE_PORT_HTTPS=443KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443KUBERNETES_SERVICE_HOST=10.96.0.1PWD=/usr/local/apache2root@node2:/proc/123888# 
root@node2:/proc/123888# 
```

### &#x20;<a href="#5_falco_and_installation_669" id="5_falco_and_installation_669"></a>
