# Challenge4

Here is the architecture for Challenge 4.

![Architecture]()

- It's Free and here is the link for "[CKS Challenges](https://kodekloud.com/courses/cks-challenges/)"
- Participate challenges for CKS Exam preparation or Hands-on. Be enjoy learning and practing, Good Luck!

---
---

I spend a lot of time on this Challenge4 with single misconfiguration YAML file. I success that Challenge4 at 23_07_2022 12:30 PM. I spent at 22_07_2022 (After work hour - 18:00 PM - 22:00 PM), 21_07_2022 (After work hour - 17:00 PM - 23:00 PM). Total 11 hours on that single misconfiguration. Yeah, finally, I can solve it.

Before you go to solve, let quick check overall in this `Single Node K8s cluster`.

<details>
<summary>Quick Check on Single Node K8s Cluster</summary>
<p>

```bash
root@controlplane ~ ➜  k get no
NAME           STATUS   ROLES                  AGE    VERSION
controlplane   Ready    control-plane,master   128m   v1.23.0

root@controlplane ~ ➜  k get all -A
NAMESPACE     NAME                                       READY   STATUS              RESTARTS       AGE
citadel       pod/webapp-color                           0/1     ContainerCreating   0              2s
eden-prime    pod/eden-fe-77574c68cd-sf9mn               1/1     Running             0              83s
eden-prime    pod/eden-software1                         1/1     Running             0              83s
eden-prime    pod/eden-software2                         1/1     Running             0              83s
eden-prime    pod/eden-software3                         1/1     Running             0              83s
kube-system   pod/coredns-64897985d-98jvq                1/1     Running             0              128m
kube-system   pod/coredns-64897985d-x8fxn                1/1     Running             0              128m
kube-system   pod/etcd-controlplane                      1/1     Running             0              128m
kube-system   pod/kube-apiserver-controlplane            1/1     Running             0              128m
kube-system   pod/kube-controller-manager-controlplane   1/1     Running             0              128m
kube-system   pod/kube-proxy-psqzd                       1/1     Running             0              128m
kube-system   pod/kube-scheduler-controlplane            1/1     Running             0              128m
kube-system   pod/weave-net-mpfqn                        2/2     Running             1 (127m ago)   128m
omega         pod/omega-fe-678c4ccf75-zfbmw              1/1     Running             0              82s
omega         pod/omega-software4                        1/1     Running             0              82s
omega         pod/omega-software5                        1/1     Running             0              82s
omega         pod/omega-software6                        1/1     Running             0              82s

NAMESPACE     NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
citadel       service/webapp-color   NodePort    10.98.69.53   <none>        8080:32192/TCP           85s
default       service/kubernetes     ClusterIP   10.96.0.1     <none>        443/TCP                  128m
kube-system   service/kube-dns       ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   128m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   128m
kube-system   daemonset.apps/weave-net    1         1         1       1            1           <none>                   128m

NAMESPACE     NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
eden-prime    deployment.apps/eden-fe    1/1     1            1           83s
kube-system   deployment.apps/coredns    2/2     2            2           128m
omega         deployment.apps/omega-fe   1/1     1            1           82s

NAMESPACE     NAME                                  DESIRED   CURRENT   READY   AGE
eden-prime    replicaset.apps/eden-fe-77574c68cd    1         1         1       83s
kube-system   replicaset.apps/coredns-64897985d     2         2         2       128m
omega         replicaset.apps/omega-fe-678c4ccf75   1         1         1       82s
```
</p>
</details>


---
---

Question in `auditing` icon.
- The audit policy file should be stored at `/etc/kubernetes/audit-policy.yaml`.
- Use a volume called `audit` that will mount only the file `/etc/kubernetes/audit-policy.yaml` from the controlplane inside the api server pod in a read only mode.
- Create a single rule in the audit policy that will record events for the 'two' objects depicting abnormal behaviour in the 'citadel'   namespace. This rule should however be applied to all 'three' namespaces shown in the diagram at a 'metadata' level. Omit the 'RequestReceived' stage.

Solution:

On this challenge, 

Let's move to `/etc/kubernetes/` and create YAML file named `audit-policy.yaml` then create single rule policy to record events according to question.

```bash
root@controlplane /etc/kubernetes ➜  cat audit-policy.yaml 
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
- level: Metadata
  resources:
  - resources: ["pods", "configmaps"]
  namespaces: ["omega", "citadel", "eden-prime"]
```

Let's also modify `kube-apiserver` YAML file to use volume called `audit` and mount only file. I solved that challenge using Kubernetes Document [Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/). Necessary configuration are already ready in document. Need some modifying. 

<details>
<summary>Modified kube-apiserver</summary>
<p>

```bash
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.121.109:6443
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
    - --advertise-address=192.168.121.109
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
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
    image: k8s.gcr.io/kube-apiserver:v1.23.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.121.109
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
        host: 192.168.121.109
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
        host: 192.168.121.109
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
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/
      name: audit-log
      readOnly: false
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
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
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
  - name: audit
    hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/kubernetes/audit/
      type: DirectoryOrCreate
status: {}
```
</p>
</details>

---
---

Question in `audit.log` icon.
- audit-log-path set to '/var/log/kubernetes/audit/audit.log'.

Solution:
> This question will be solved after you solved above `auditing` icon. Solution is need to add `audit-log-path` in `kube-apiserver` YAML file. Let's go to next icons.

---
---

Let's move to Question `Falco` icon.
- Install the 'falco' utility on the controlplane node and start it as a systemd service

Solution:

<details>
<summary>Installing Falco</summary>
<p>

```bash
root@controlplane /etc/kubernetes ➜  curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
OK

root@controlplane /etc/kubernetes ➜  echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list
deb https://download.falco.org/packages/deb stable main

root@controlplane /etc/kubernetes ➜  apt-get update -y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9,383 B]                                                                                                     
Get:2 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                                                                                                             
Get:3 https://download.falco.org/packages/deb stable InRelease [4,168 B]                                                                                                              
Get:4 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [57.6 kB]                                                              
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                          
Hit:6 http://archive.ubuntu.com/ubuntu bionic InRelease                                                    
Get:7 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages [26.4 kB]                     
Get:8 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]                                            
Get:9 https://download.falco.org/packages/deb stable/main amd64 Packages [4,008 B]                                   
Get:10 http://security.ubuntu.com/ubuntu bionic-security/main i386 Packages [1,216 kB]                                     
Get:11 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]       
Get:12 http://archive.ubuntu.com/ubuntu bionic-updates/main i386 Packages [1,515 kB]           
Get:13 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [2,336 kB]         
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2,678 kB]          
Get:15 http://security.ubuntu.com/ubuntu bionic-security/main Translation-en [406 kB]            
Get:16 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [830 kB]          
Get:17 http://security.ubuntu.com/ubuntu bionic-security/restricted i386 Packages [28.1 kB]
Get:18 http://security.ubuntu.com/ubuntu bionic-security/restricted Translation-en [114 kB] 
Get:19 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1,218 kB]      
Get:20 http://security.ubuntu.com/ubuntu bionic-security/universe i386 Packages [1,030 kB]   
Get:21 http://archive.ubuntu.com/ubuntu bionic-updates/main Translation-en [496 kB]          
Get:22 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [861 kB]      
Get:23 http://security.ubuntu.com/ubuntu bionic-security/universe Translation-en [280 kB]
Get:24 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [19.0 kB] 
Get:25 http://security.ubuntu.com/ubuntu bionic-security/multiverse i386 Packages [6,020 B]
Get:26 http://security.ubuntu.com/ubuntu bionic-security/multiverse Translation-en [3,836 B]
Get:27 http://archive.ubuntu.com/ubuntu bionic-updates/restricted i386 Packages [34.7 kB]     
Get:28 http://archive.ubuntu.com/ubuntu bionic-updates/restricted Translation-en [119 kB]
Get:29 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1,831 kB]
Get:30 http://archive.ubuntu.com/ubuntu bionic-updates/universe i386 Packages [1,620 kB]
Get:31 http://archive.ubuntu.com/ubuntu bionic-updates/universe Translation-en [397 kB]
Get:32 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse i386 Packages [11.2 kB]
Get:33 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [24.9 kB]
Get:34 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [6,012 B]
Get:35 http://archive.ubuntu.com/ubuntu bionic-backports/main i386 Packages [10.8 kB]
Get:36 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [10.8 kB]
Get:37 http://archive.ubuntu.com/ubuntu bionic-backports/main Translation-en [5,016 B]
Get:38 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [11.6 kB]
Get:39 http://archive.ubuntu.com/ubuntu bionic-backports/universe i386 Packages [11.6 kB]
Get:40 http://archive.ubuntu.com/ubuntu bionic-backports/universe Translation-en [5,864 B]
Fetched 17.6 MB in 5s (3,523 kB/s)                                  
Reading package lists... Done
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'https://download.falco.org/packages/deb stable InRelease' doesn't support architecture 'i386'
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2

root@controlplane /etc/kubernetes ➜  

root@controlplane /etc/kubernetes ➜  apt-get -y install linux-headers-$(uname -r)
Reading package lists... Done
Building dependency tree       
Reading state information... Done
linux-headers-4.15.0-171-generic is already the newest version (4.15.0-171.180).
The following package was automatically installed and is no longer required:
  dkms
Use 'apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 107 not upgraded.

root@controlplane /etc/kubernetes ➜  apt-get install -y falco
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  falco
0 upgraded, 1 newly installed, 0 to remove and 107 not upgraded.
Need to get 19.3 MB of archives.
After this operation, 45.5 MB of additional disk space will be used.
Get:1 https://download.falco.org/packages/deb stable/main amd64 falco amd64 0.32.1 [19.3 MB]
Fetched 19.3 MB in 0s (43.4 MB/s)
debconf: unable to initialize frontend: Dialog
debconf: (Dialog frontend requires a screen at least 13 lines tall and 31 columns wide.)
debconf: falling back to frontend: Readline
Selecting previously unselected package falco.
(Reading database ... 76419 files and directories currently installed.)
Preparing to unpack .../falco_0.32.1_amd64.deb ...
Unpacking falco (0.32.1) ...
Setting up falco (0.32.1) ...
Installing new version of config file /etc/falco/falco.yaml ...
debconf: unable to initialize frontend: Dialog
debconf: (Dialog frontend requires a screen at least 13 lines tall and 31 columns wide.)
debconf: falling back to frontend: Readline
Loading new falco-2.0.0+driver DKMS files...
Building for 4.15.0-171-generic
Building initial module for 4.15.0-171-generic
Done.

falco:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-171-generic/updates/dkms/

depmod...

DKMS: install completed.
```
</p>
</details>

Let's check its service that up and running. 
```bash
root@controlplane /etc/kubernetes ➜  systemctl status falco.service 
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: https://falco.org/docs/
```

It's inactive(dead). Need to start `falco` service. But, before you start service, click on `file-output` icon. Change necessary configuration. So, next question will aslo be sovled by doing this. Make changes in `/etc/falco/falco.yaml`. Then start the service.

Before changes in Falco File,
```bash
# If keep_alive is set to true, the file will be opened once and
# continuously written to, with each output message on its own
# line. If keep_alive is set to false, the file will be re-opened
# for each output message.
#
# Also, the file will be closed and reopened if falco is signaled with
# SIGUSR1.

file_output:
  enabled: false
  keep_alive: false
  filename: ./events.txt

stdout_output:
  enabled: true
```

After changes in Falco File,

```bash
# If keep_alive is set to true, the file will be opened once and
# continuously written to, with each output message on its own
# line. If keep_alive is set to false, the file will be re-opened
# for each output message.
#
# Also, the file will be closed and reopened if falco is signaled with
# SIGUSR1.

file_output:
  enabled: true
  keep_alive: false
  filename: /opt/falco.log

stdout_output:
  enabled: true
```

<details>
<summary>Making changes in Falco YAML</summary>
<p>

```bash
root@controlplane /etc/kubernetes ➜  cat /etc/falco/falco.yaml 
#
# Copyright (C) 2022 The Falco Authors.
#
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# File(s) or Directories containing Falco rules, loaded at startup.
# The name "rules_file" is only for backwards compatibility.
# If the entry is a file, it will be read directly. If the entry is a directory,
# every file in that directory will be read, in alphabetical order.
#
# falco_rules.yaml ships with the falco package and is overridden with
# every new software version. falco_rules.local.yaml is only created
# if it doesn't exist. If you want to customize the set of rules, add
# your customizations to falco_rules.local.yaml.
#
# The files will be read in the order presented here, so make sure if
# you have overrides they appear in later files.
rules_file:
  - /etc/falco/falco_rules.yaml
  - /etc/falco/falco_rules.local.yaml
  - /etc/falco/rules.d

#
# Plugins that are available for use. These plugins are not loaded by
# default, as they require explicit configuration to point to
# cloudtrail log files.
#

# To learn more about the supported formats for
# init_config/open_params for the cloudtrail plugin, see the README at
# https://github.com/falcosecurity/plugins/blob/master/plugins/cloudtrail/README.md.
plugins:
  - name: k8saudit
    library_path: libk8saudit.so
    init_config:
    #   maxEventSize: 262144
    #   webhookMaxBatchSize: 12582912
    #   sslCertificate: /etc/falco/falco.pem
    open_params: "http://:9765/k8s-audit"
  - name: cloudtrail
    library_path: libcloudtrail.so
    # see docs for init_config and open_params:
    # https://github.com/falcosecurity/plugins/blob/master/plugins/cloudtrail/README.md
  - name: json
    library_path: libjson.so

# Setting this list to empty ensures that the above plugins are *not*
# loaded and enabled by default. If you want to use the above plugins,
# set a meaningful init_config/open_params for the cloudtrail plugin
# and then change this to:
# load_plugins: [cloudtrail, json]
load_plugins: []

# Watch config file and rules files for modification.
# When a file is modified, Falco will propagate new config,
# by reloading itself.
watch_config_files: true

# If true, the times displayed in log messages and output messages
# will be in ISO 8601. By default, times are displayed in the local
# time zone, as governed by /etc/localtime.
time_format_iso_8601: false

# Whether to output events in json or text
json_output: false

# When using json output, whether or not to include the "output" property
# itself (e.g. "File below a known binary directory opened for writing
# (user=root ....") in the json output.
json_include_output_property: true

# When using json output, whether or not to include the "tags" property
# itself in the json output. If set to true, outputs caused by rules
# with no tags will have a "tags" field set to an empty array. If set to
# false, the "tags" field will not be included in the json output at all.
json_include_tags_property: true

# Send information logs to stderr and/or syslog Note these are *not* security
# notification logs! These are just Falco lifecycle (and possibly error) logs.
log_stderr: true
log_syslog: true

# Minimum log level to include in logs. Note: these levels are
# separate from the priority field of rules. This refers only to the
# log level of falco's internal logging. Can be one of "emergency",
# "alert", "critical", "error", "warning", "notice", "info", "debug".
log_level: info

# Falco is capable of managing the logs coming from libs. If enabled,
# the libs logger send its log records the same outputs supported by
# Falco (stderr and syslog). Disabled by default.
libs_logger:
  enabled: false
  # Minimum log severity to include in the libs logs. Note: this value is
  # separate from the log level of the Falco logger and does not affect it.
  # Can be one of "fatal", "critical", "error", "warning", "notice",
  # "info", "debug", "trace".
  severity: debug

# Minimum rule priority level to load and run. All rules having a
# priority more severe than this level will be loaded/run.  Can be one
# of "emergency", "alert", "critical", "error", "warning", "notice",
# "informational", "debug".
priority: debug

# Whether or not output to any of the output channels below is
# buffered. Defaults to false
buffered_outputs: false

# Falco uses a shared buffer between the kernel and userspace to pass
# system call information. When Falco detects that this buffer is
# full and system calls have been dropped, it can take one or more of
# the following actions:
#   - ignore: do nothing (default when list of actions is empty)
#   - log: log a DEBUG message noting that the buffer was full
#   - alert: emit a Falco alert noting that the buffer was full
#   - exit: exit Falco with a non-zero rc
#
# Notice it is not possible to ignore and log/alert messages at the same time.
#
# The rate at which log/alert messages are emitted is governed by a
# token bucket. The rate corresponds to one message every 30 seconds
# with a burst of one message (by default).
#
# The messages are emitted when the percentage of dropped system calls
# with respect the number of events in the last second
# is greater than the given threshold (a double in the range [0, 1]).
#
# For debugging/testing it is possible to simulate the drops using
# the `simulate_drops: true`. In this case the threshold does not apply.

syscall_event_drops:
  threshold: .1
  actions:
    - log
    - alert
  rate: .03333
  max_burst: 1

# Falco uses a shared buffer between the kernel and userspace to receive
# the events (eg., system call information) in userspace.
#
# Anyways, the underlying libraries can also timeout for various reasons.
# For example, there could have been issues while reading an event.
# Or the particular event needs to be skipped.
# Normally, it's very unlikely that Falco does not receive events consecutively.
#
# Falco is able to detect such uncommon situation.
#
# Here you can configure the maximum number of consecutive timeouts without an event
# after which you want Falco to alert.
# By default this value is set to 1000 consecutive timeouts without an event at all.
# How this value maps to a time interval depends on the CPU frequency.

syscall_event_timeouts:
  max_consecutives: 1000

# Falco continuously monitors outputs performance. When an output channel does not allow
# to deliver an alert within a given deadline, an error is reported indicating
# which output is blocking notifications.
# The timeout error will be reported to the log according to the above log_* settings.
# Note that the notification will not be discarded from the output queue; thus,
# output channels may indefinitely remain blocked.
# An output timeout error indeed indicate a misconfiguration issue or I/O problems
# that cannot be recovered by Falco and should be fixed by the user.
#
# The "output_timeout" value specifies the duration in milliseconds to wait before
# considering the deadline exceed.
#
# With a 2000ms default, the notification consumer can block the Falco output
# for up to 2 seconds without reaching the timeout.

output_timeout: 2000

# A throttling mechanism implemented as a token bucket limits the
# rate of falco notifications. This throttling is controlled by the following configuration
# options:
#  - rate: the number of tokens (i.e. right to send a notification)
#    gained per second. Defaults to 1.
#  - max_burst: the maximum number of tokens outstanding. Defaults to 1000.
#
# With these defaults, falco could send up to 1000 notifications after
# an initial quiet period, and then up to 1 notification per second
# afterward. It would gain the full burst back after 1000 seconds of
# no activity.

outputs:
  rate: 1
  max_burst: 1000

# Where security notifications should go.
# Multiple outputs can be enabled.

syslog_output:
  enabled: true

# If keep_alive is set to true, the file will be opened once and
# continuously written to, with each output message on its own
# line. If keep_alive is set to false, the file will be re-opened
# for each output message.
#
# Also, the file will be closed and reopened if falco is signaled with
# SIGUSR1.

file_output:
  enabled: true
  keep_alive: false
  filename: /opt/falco.log

stdout_output:
  enabled: true

# Falco contains an embedded webserver that can be used to accept K8s
# Audit Events. These config options control the behavior of that
# webserver. (By default, the webserver is enabled).
#
# The ssl_certificate is a combination SSL Certificate and corresponding
# key contained in a single file. You can generate a key/cert as follows:
#
# $ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
# $ cat certificate.pem key.pem > falco.pem
# $ sudo cp falco.pem /etc/falco/falco.pem
#
# It also exposes a healthy endpoint that can be used to check if Falco is up and running
# By default the endpoint is /healthz
webserver:
  enabled: true
  listen_port: 8765
  k8s_healthz_endpoint: /healthz
  ssl_enabled: false
  ssl_certificate: /etc/falco/falco.pem

# Possible additional things you might want to do with program output:
#   - send to a slack webhook:
#         program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/XXX"
#   - logging (alternate method than syslog):
#         program: logger -t falco-test
#   - send over a network connection:
#         program: nc host.example.com 80

# If keep_alive is set to true, the program will be started once and
# continuously written to, with each output message on its own
# line. If keep_alive is set to false, the program will be re-spawned
# for each output message.
#
# Also, the program will be closed and reopened if falco is signaled with
# SIGUSR1.
program_output:
  enabled: false
  keep_alive: false
  program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/XXX"

http_output:
  enabled: false
  url: http://some.url
  user_agent: "falcosecurity/falco"

# Falco supports running a gRPC server with two main binding types
# 1. Over the network with mandatory mutual TLS authentication (mTLS)
# 2. Over a local unix socket with no authentication
# By default, the gRPC server is disabled, with no enabled services (see grpc_output)
# please comment/uncomment and change accordingly the options below to configure it.
# Important note: if Falco has any troubles creating the gRPC server
# this information will be logged, however the main Falco daemon will not be stopped.
# gRPC server over network with (mandatory) mutual TLS configuration.
# This gRPC server is secure by default so you need to generate certificates and update their paths here.
# By default the gRPC server is off.
# You can configure the address to bind and expose it.
# By modifying the threadiness configuration you can fine-tune the number of threads (and context) it will use.
# grpc:
#   enabled: true
#   bind_address: "0.0.0.0:5060"
#   # when threadiness is 0, Falco sets it by automatically figuring out the number of online cores
#   threadiness: 0
#   private_key: "/etc/falco/certs/server.key"
#   cert_chain: "/etc/falco/certs/server.crt"
#   root_certs: "/etc/falco/certs/ca.crt"

# gRPC server using an unix socket
grpc:
  enabled: false
  bind_address: "unix:///var/run/falco.sock"
  # when threadiness is 0, Falco automatically guesses it depending on the number of online cores
  threadiness: 0

# gRPC output service.
# By default it is off.
# By enabling this all the output events will be kept in memory until you read them with a gRPC client.
# Make sure to have a consumer for them or leave this disabled.
grpc_output:
  enabled: false

# Container orchestrator metadata fetching params
metadata_download:
  max_mb: 100
  chunk_wait_us: 1000
  watch_freq_sec: 1
```
</p>
</details>

After that, start the service.
```bash
root@controlplane /etc/kubernetes ➜  systemctl start falco.service 
```

Verify again the service.
```bash
root@controlplane /etc/kubernetes ➜  systemctl status falco.service 
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-07-23 06:06:19 UTC; 11s ago
     Docs: https://falco.org/docs/
  Process: 1698 ExecStartPre=/sbin/modprobe falco (code=exited, status=0/SUCCESS)
 Main PID: 1712 (falco)
    Tasks: 17 (limit: 2314)
   CGroup: /system.slice/falco.service
           └─1712 /usr/bin/falco --pidfile=/var/run/falco.pid
```

Okay, Output File is already configured and service is up and running.

---
---

Next Question `file-output` icon.

- Configure falco to save the event output to the file '/opt/falco.log'.

Solution:

> According to my previcious solution, that question is already solved by making changes in `/etc/falco/falco.yaml`.

---
---

Question on `Security Report` icon.
- Inspect the API server audit logs and identify the user responsible for the abnormal behaviour seen in the 'citadel' namespace. Save the name of the 'user', 'role' and 'rolebinding' responsible for the event to the file '/opt/blacklist_users' file (comma separated and in this specific order).
- Inspect the 'falco' logs and identify the pod that has events generated because of packages being updated on it. Save the namespace and the pod name in the file '/opt/compromised_pods' (comma separated - namespace followed by the pod name).

Solution:

According to `auditing` icon, I alrady configured necessary configuratin. So, let's check `audit-file-path` for the abnormal behaviour in `/var/log/kubernetes/audit/audit.log`.

<details>
<summary>Output</summary>
<p>

```bash
root@controlplane log/kubernetes/audit ➜  cat audit.log | grep -i citadel | egrep -v "\"get|\"watch|\"list" | jq
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "74497b2c-a34f-44c3-9f11-0dc235d315a7",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create",
  "verb": "create",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "impersonatedUser": {
    "username": "agent-smith",
    "groups": [
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.121.1"
  ],
  "userAgent": "kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47",
  "objectRef": {
    "resource": "pods",
    "namespace": "citadel",
    "name": "webapp-color",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 201
  },
  "requestReceivedTimestamp": "2022-07-23T06:01:21.938336Z",
  "stageTimestamp": "2022-07-23T06:01:21.942884Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"",
    "pod-security.kubernetes.io/enforce-policy": "privileged:latest"
  }
}
```
</p>
</details>

That log will generate when every changes in `pods` and `configmaps` under namespace `citadel` because of audit-policy.
So, after I get log file result, I copy and paste to `/opt/blacklist_users`.

```bash
root@controlplane /etc/kubernetes ➜  echo "agent-smith,important_role_do_not_delete,important_binding_do_not_delete/citadel" > /opt/blacklist_users
```

And then I inspect `falco.log` in file path `/opt/falco.log`.
```bash
root@controlplane /etc/kubernetes ➜  cat /opt/falco.log 
09:39:46.915763996: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=f671d8c2b199 container_name=k8s_eden-software2_eden-software2_eden-prime_13b9b0f5-3ba4-4b89-a2e6-615b3a16e88f_0 image=ubuntu:latest)
```

And then, also check container update using container_id.
```bash
root@controlplane log/kubernetes/audit ➜  crictl ps | grep -i "f671d8c2b199"
f671d8c2b199f       ubuntu@sha256:b6b83d3c331794420340093eb706a6f152d9c1fa51b262d9bf34594887c2c7ac                   26 minutes ago      Running             eden-software2            0                   f0210cb3ea8fc
```
Then check using pod_id
```bash
root@controlplane log/kubernetes/audit ✖ crictl pods | grep -i "f0210cb3ea8fc"
f0210cb3ea8fc       30 minutes ago       Ready               eden-software2                         eden-prime          0                   (default)
```
After cheking details, I found the namespace and pod and write to file `/opt/compromised_pods`.

```bash
root@controlplane /etc/kubernetes ➜  echo "eden-prime,eden-software2" > /opt/compromised_pods
```

---
---

Question on `eden-prime` icon.

- Delete pods belonging to the 'eden-prime' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

Solution:

After already solved `Security Report` question, It is ready to delete pods that were flagged in the `Security Report` file `/opt/compromised_pods`.

Let's check that `/opt/compromised_pods` file.

```bash
root@controlplane /etc/kubernetes ➜  cat /opt/compromised_pods 
eden-prime,eden-software2
```

Also check pods under namespace `eden-prime`
```bash
root@controlplane /etc/kubernetes ➜  k -n eden-prime get po
NAME                       READY   STATUS    RESTARTS   AGE
eden-fe-77574c68cd-zjvrw   1/1     Running   0          38m
eden-software1             1/1     Running   0          38m
eden-software2             1/1     Running   0          38m
eden-software3             1/1     Running   0          38m
```
So, `eden-softwar2` pod need to delete for the reason of `Security Report`.

Let's delete that pod.
```bash
root@controlplane /etc/kubernetes ➜  k -n eden-prime delete po eden-software2 --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "eden-software2" force deleted
```

---
---

Question on `omega` icon.

- Delete pods belonging to the 'omega' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

Solution:

First, check the running pods
```bash
root@controlplane log/kubernetes/audit ➜  k -n omega get po
NAME                        READY   STATUS    RESTARTS   AGE
omega-fe-678c4ccf75-zfbmw   1/1     Running   0          33m
omega-software4             1/1     Running   0          33m
omega-software5             1/1     Running   0          33m
omega-software6             1/1     Running   0          33m
```
Let's check the `Security Report` file `/opt/compromised_pods`.

```bash
root@controlplane log/kubernetes/audit ➜  cat /opt/compromised_pods 
eden-prime,eden-software2
```

There is no pods in `Security Report` file `/opt/compromised_pods`. So, no more changes in under `omega` namespace.

---
---

Question on `citadel` icon.

- Delete the role causing the constant deletion and creation of the configmaps and pods in this namespace. Do not delete any other role!

- Delete the rolebinding causing the constant deletion and creation of the configmaps and pods in this namespace. Do not delete any other rolebinding!

Solution:

For this challenge, according to `Security Report`, I query some info and write to `user,role and rolebinding` to `/opt/blacklist_users` that responsible for the abnormal behaviour seen in the `citadel` namespace. So, let's verify the username, role and rolebinding.
<details>
<summary>Inspect Role,Rolebinding</summary>
<p>

```bash
root@controlplane /etc/kubernetes ➜  k -n citadel get role,rolebinding
NAME                                                          CREATED AT
role.rbac.authorization.k8s.io/dev1                           2022-07-23T05:33:46Z
role.rbac.authorization.k8s.io/important_citadel_user_role    2022-07-23T05:33:47Z
role.rbac.authorization.k8s.io/important_role_do_not_delete   2022-07-23T05:33:45Z

NAME                                                                    ROLE                                AGE
rolebinding.rbac.authorization.k8s.io/dev1                              Role/dev1                           39m
rolebinding.rbac.authorization.k8s.io/important_binding_do_not_delete   Role/important_role_do_not_delete   39m
rolebinding.rbac.authorization.k8s.io/important_citadel_user_binding    Role/important_citadel_user_role    39m

root@controlplane log/kubernetes/audit ➜  k -n citadel describe role dev1 
Name:         dev1
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *          []                 []              [list create delete]

root@controlplane log/kubernetes/audit ➜  k -n citadel describe role important_citadel_user_role 
Name:         important_citadel_user_role
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *          []                 []              [list create delete]

root@controlplane log/kubernetes/audit ➜  k -n citadel describe role important_role_do_not_delete 
Name:         important_role_do_not_delete
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *          []                 []              [list create delete]

root@controlplane log/kubernetes/audit ➜  k -n citadel describe rolebinding dev1 
Name:         dev1
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  dev1
Subjects:
  Kind  Name      Namespace
  ----  ----      ---------
  User  dev-user  

root@controlplane log/kubernetes/audit ➜  k -n citadel describe rolebinding important_citadel_user_binding 
Name:         important_citadel_user_binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  important_citadel_user_role
Subjects:
  Kind  Name          Namespace
  ----  ----          ---------
  User  citadel-user  

root@controlplane log/kubernetes/audit ➜  k -n citadel describe rolebinding important_binding_do_not_delete 
Name:         important_binding_do_not_delete
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  important_role_do_not_delete
Subjects:
  Kind  Name         Namespace
  ----  ----         ---------
  User  agent-smith  
```
</p>
</details>

Let's delete role and rolebinding that responsible for the abnormal behaviour.

```bash
root@controlplane /etc/kubernetes ➜  k -n citadel delete role important_role_do_not_delete 
role.rbac.authorization.k8s.io "important_role_do_not_delete" deleted

root@controlplane /etc/kubernetes ➜  k -n citadel delete rolebinding important_binding_do_not_delete 
rolebinding.rbac.authorization.k8s.io "important_binding_do_not_delete" deleted
```
---
---

Finally, let's click on Question `kube-apiserver` icon.

- API server running?

After making changes on kube-apiserver YAML file and restart kubelet.service. kube-apiserver pod is up and running by checking pods and containers. So, it is success.

Let's verify `kube-apiserver` pod.
```bash
root@controlplane / ➜  k -n kube-system get pods
NAME                                   READY   STATUS    RESTARTS       AGE
coredns-64897985d-98jvq                1/1     Running   0              168m
coredns-64897985d-x8fxn                1/1     Running   0              168m
etcd-controlplane                      1/1     Running   0              168m
kube-apiserver-controlplane            1/1     Running   0              35m
kube-controller-manager-controlplane   1/1     Running   1 (35m ago)    168m
kube-proxy-psqzd                       1/1     Running   0              168m
kube-scheduler-controlplane            1/1     Running   1 (35m ago)    168m
weave-net-mpfqn                        2/2     Running   1 (168m ago)   168m
```

Let's verify container.
```bash
root@controlplane / ➜  crictl ps | grep -i kube-api
4f8ba05a67b74       e6bf5ddd40982                                                                                    35 minutes ago      Running             kube-apiserver            0                   4ac9d18a1eb6c
```
---
---

After all final verify, let's press `Check` button to verify success or not.

Cheer up! It's all success.

![Challenge4_Complete_Architecture]()



