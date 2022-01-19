# Falco

### 1. Falco 安装 <a href="#5_falco_and_installation_669" id="5_falco_and_installation_669"></a>

[falco官网](https://falco.org)\
github: [https://github.com/falcosecurity/falco](https://github.com/falcosecurity/falco)\
k8s wtih falco: [https://v1-17.docs.kubernetes.io/docs/tasks/debug-application-cluster/falco/](https://v1-17.docs.kubernetes.io/docs/tasks/debug-application-cluster/falco/)\
\
\
官方下载安装：[https://falco.org/docs/getting-started/installation/](https://falco.org/docs/getting-started/installation/)

![](https://img-blog.csdnimg.cn/20210524141120519.png?x-oss-process=image/watermark,type\_ZmFuZ3poZW5naGVpdGk,shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

![](https://img-blog.csdnimg.cn/20210524141128843.png)

```
# install falco
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list 
apt-get update -y
apt-get -y install linux-headers-$(uname -r)
apt-get install -y falco=0.26.1
```

```
root@node2:~/falco# systemctl start falco
root@node2:~/falco# systemctl enable falco
Created symlink from /etc/systemd/system/multi-user.target.wants/falco.service to /usr/lib/systemd/system/falco.service.
root@node2:~/falco# systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2021-05-23 23:20:59 PDT; 12s ago
     Docs: https://falco.org/docs/
 Main PID: 28817 (falco)
   CGroup: /system.slice/falco.service
           └─28817 /usr/bin/falco --pidfile=/var/run/falco.pid

May 23 23:21:00 node2 falco[28817]: Falco initialized with configuration file /etc/falco/falco.yaml
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/falco_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/falco_rules.local.yaml:
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Starting internal webserver, listening on port 8765
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Starting internal webserver, listening on port 8765
May 23 23:21:05 node2 systemd[1]: [/usr/lib/systemd/system/falco.service:19] Unknown lvalue 'ProtectKernelTunables' in section 'Service'
May 23 23:21:05 node2 systemd[1]: [/usr/lib/systemd/system/falco.service:20] Unknown lvalue 'RestrictRealtime' in section 'Service'


root@node2:~/falco# ls /etc/falco/
falco_rules.local.yaml  falco_rules.yaml  falco.yaml  k8s_audit_rules.yaml  rules.available  rules.d
root@node2:~/falco# tail /var/log/syslog|grep falco
May 23 23:21:00 node2 kernel: [192079.038231] falco: initializing ring buffer for CPU 1
May 23 23:21:00 node2 kernel: [192079.088336] falco: CPU buffer initialized, size=8388608
May 23 23:21:00 node2 kernel: [192079.088339] falco: starting capture
May 23 23:21:00 node2 falco: Starting internal webserver, listening on port 8765
```

### 2. Falco 发现恶意进程 <a href="#6_use_falco_to_find_malicious_processes_723" id="6_use_falco_to_find_malicious_processes_723"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052414235822.png)

```
root@master:~/cks/runtime-security# k exec -ti apache -- bash
root@apache:/usr/local/apache2# echo user >> /etc/passwd
root@apache:/usr/local/apache2# apt-get update
Get:1 http://deb.debian.org/debian buster InRelease [121 kB]
Get:2 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]                  
Get:3 http://deb.debian.org/debian buster/main amd64 Packages [7907 kB]                 
Get:4 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:5 http://security.debian.org/debian-security buster/updates/main amd64 Packages [289 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [10.9 kB]
Fetched 8446 kB in 5s (1842 kB/s)                          
Reading package lists... Done


root@node2:~/falco# tail -f /var/log/syslog|grep falco
May 23 23:25:17 node2 falco[28817]: 23:25:16.992066800: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 (id=ced29b338f66) shell=bash parent=runc cmdline=bash terminal=34816 container_id=ced29b338f66 image=httpd)
May 23 23:25:17 node2 falco: 23:25:16.992066800: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 (id=ced29b338f66) shell=bash parent=runc cmdline=bash terminal=34816 container_id=ced29b338f66 image=httpd)
May 23 23:25:46 node2 falco[28817]: 23:25:46.131128350: Error File below /etc opened for writing (user=root user_loginuid=-1 command=bash parent=<NA> pcmdline=<NA> file=/etc/passwd program=bash gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=ced29b338f66 image=httpd)
May 23 23:25:46 node2 falco: 23:25:46.131128350: Error File below /etc opened for writing (user=root user_loginuid=-1 command=bash parent=<NA> pcmdline=<NA> file=/etc/passwd program=bash gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=ced29b338f66 image=httpd)
May 23 23:26:18 node2 falco[28817]: 23:26:18.336286131: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=ced29b338f66 container_name=k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 image=httpd:latest)
May 23 23:26:18 node2 falco: 23:26:18.336286131: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=ced29b338f66 container_name=k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 image=httpd:latest)
```

修改配置

```
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
    readinessProbe: 
      exec:
        command:
        - apt-get
        - update
      initialDelaySeconds: 5
      periodSeconds: 3
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/runtime-security# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "apache" force deleted
root@master:~/cks/runtime-security# k -f pod.yaml create
pod/apache created
root@master:~/cks/runtime-security# k get pod apache -o wide
NAME     READY   STATUS    RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
apache   0/1     Running   0          105s   10.244.104.4   node2   <none>           <none>


#发现报错进程
root@node2:~/falco# tail -f /var/log/syslog|grep falco
May 23 23:33:01 node2 falco[28817]: 23:33:01.783656151: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)
May 23 23:33:01 node2 falco: 23:33:01.783656151: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)
May 23 23:33:04 node2 falco[28817]: 23:33:04.833053968: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)
```

### 3. 分析 Falco 策略 <a href="#7_practice__investigate_falco_rules_801" id="7_practice__investigate_falco_rules_801"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524144913460.png)

官方：[https://falco.org/docs/rules/](https://falco.org/docs/rules/)

### 4. 改变 Falco 策略 <a href="#8_change_falco_rule_806" id="8_change_falco_rule_806"></a>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524145008291.png?shadow\_10,text\_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size\_16,color\_FFFFFF,t\_70)

```
root@master:~/cks/runtime-security# k get pods -owide
NAME     READY   STATUS    RESTARTS   AGE     IP             NODE    NOMINATED NODE   READINESS GATES
apache   1/1     Running   0          24s     10.244.104.5   node2   <none>           <none>
test     1/1     Running   0          3h32m   10.244.104.2   node2   <none>           <none>
root@master:~/cks/runtime-security# k exec -ti apache -- bash



root@node2:~# systemctl stop falco
root@node2:~# falco
Sun May 23 23:53:14 2021: Falco version 0.28.1 (driver version 5c0b863ddade7a45568c0ac97d037422c9efb750)
Sun May 23 23:53:14 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Sun May 23 23:53:15 2021: Starting internal webserver, listening on port 8765




23:53:30.491825091: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0 (id=84dd6fe8a9ad) shell=bash parent=runc cmdline=bash terminal=34816 container_id=84dd6fe8a9ad image=httpd)


root@node2:~# cd /etc/falco/
root@node2:/etc/falco# ls
falco_rules.local.yaml  falco_rules.yaml  falco.yaml  k8s_audit_rules.yaml  rules.available  rules.d


root@node2:/etc/falco# grep -r "A shell was spawned in a container with an attached terminal" *
falco_rules.yaml:    A shell was spawned in a container with an attached terminal (user=%user.name user_loginuid=%user.loginuid %container.info

#更新配置
root@node2:/etc/falco# cat falco_rules.local.yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec point into a container with an attached terminal.
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
    and not user_expected_terminal_shell_in_container_conditions
  output: >
    %evt.time,%user.name,%container.name,%container.id
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline terminal=%proc.tty container_id=%container.id image=%container.image.repository)
  priority: WARNING
  tags: [container, shell, mitre_execution]




root@master:~/cks/runtime-security# k exec -ti apache -- bash
root@apache:/usr/local/apache2#




root@node2:/etc/falco# falco
Mon May 24 00:07:13 2021: Falco version 0.28.1 (driver version 5c0b863ddade7a45568c0ac97d037422c9efb750)
Mon May 24 00:07:13 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:  #配置生效
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mon May 24 00:07:14 2021: Starting internal webserver, listening on port 8765
00:07:30.297671117: Warning Shell history had been deleted or renamed (user=root user_loginuid=-1 type=openat command=bash fd.name=/root/.bash_history name=/root/.bash_history path=<NA> oldpath=<NA> k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0 (id=84dd6fe8a9ad))

格式改变
00:07:33.763063865: Warning 00:07:33.763063865,root,k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0,84dd6fe8a9ad shell=bash parent=runc cmdline=bash terminal=34816 container_id=84dd6fe8a9ad image=httpd)
```
