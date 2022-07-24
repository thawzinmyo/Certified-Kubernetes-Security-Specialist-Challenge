# Challenge2

When you press each icon in the architecture, will see the questions.

![Challenge2_Architecture](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/image/C2/Challenge2_Head.PNG)

- It's Free and here is the link for "[CKS Challenges](https://kodekloud.com/courses/cks-challenges/)"
- Participate challenges for CKS Exam preparation or Hands-on. Be enjoy learning and practing, Good Luck!

---
---


Let's inspect that how many namespace is there firstly.
```bash
root@controlplane ~ ➜  k get ns
NAME              STATUS   AGE
default           Active   39m
dev               Active   17m
kube-node-lease   Active   39m
kube-public       Active   39m
kube-system       Active   39m
prod              Active   17m
staging           Active   17m
```
---
---

Quesiton in `Dockerfile(webapp)` icon.
- Run as non root(instead, use correct application user)
- Avoid exposing unnecessary ports
- Avoid copying the 'Dockerfile' and other unnecessary files and directories in to the image. Move the required files and directories (app.py, requirements.txt and the templates directory) to a subdirectory called 'app' under 'webapp' and update the COPY instruction in the 'Dockerfile' accordingly.
- Once the security issues are fixed, rebuild this image locally with the tag 'kodekloud/webapp-color:stable'

Solution:

Let's check to Dockerfile.

```bash
$  cat webapp/Dockerfile
```
<details>
<summary>Dockerfile Output</summary>
<p>

```bash
## Install Flask
RUN pip install flask

## Copy All files to /opt
COPY . /opt/

## Flask app to be exposed on port 8080
EXPOSE 8080

## Flask app to be run as 'worker'
RUN adduser -D worker

## Expose port 22
EXPOSE 22

WORKDIR /opt

USER root

ENTRYPOINT ["python", "app.py"]
```
</p>
</details>

When I quick verify on all deployments,pods and services. There is no need access for SSH. And move `app.py`, `requirements.txt`, `template` directory to subdirectory `app` under `web` according to the question. Also use worker as user ( correct application user )
```bash
root@controlplane ~ ✖ ls
dev-webapp.yaml  staging-webapp.yaml  webapp

root@controlplane ~ ➜ cd webapp/

root@controlplane ~/webapp ➜ mkdir app

root@controlplane ~/webapp ➜ mv app.py requirements.txt template/ app/

root@controlplane ~/webapp ➜  ls
app  Dockerfile
```

<details>
<summary>Modified Dockerfile</summary>
<p>

```bash
## Install Flask
RUN pip install flask

## Copy All files to /opt
COPY ./app/ /opt/

## Flask app to be exposed on port 8080
EXPOSE 8080

## Flask app to be run as 'worker'
RUN adduser -D worker

WORKDIR /opt

USER worker

ENTRYPOINT ["python", "app.py"]
```
</p>
</details>
And then rebuild the image using the update Dockerfile using given tag.

```bash
root@controlplane ~/webapp ➜  docker build -t kodekloud/webapp-color:stable .
```
<details>
<summary>Docker image rebuild progress</summary>
<p>

```bash
root@controlplane ~/webapp ➜  docker build -t kodekloud/webapp-color:stable .
Sending build context to Docker daemon  8.192kB
Step 1/8 : FROM python:3.6-alpine
3.6-alpine: Pulling from library/python
59bf1c3509f3: Pull complete 
8786870f2876: Pull complete 
acb0e804800e: Pull complete 
52bedcb3e853: Pull complete 
b064415ed3d7: Pull complete 
Digest: sha256:579978dec4602646fe1262f02b96371779bfb0294e92c91392707fa999c0c989
Status: Downloaded newer image for python:3.6-alpine
 ---> 3a9e80fa4606
Step 2/8 : RUN pip install flask
 ---> Running in 0a4e883d4591
Collecting flask
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Collecting click>=7.1.2
  Downloading click-8.0.4-py3-none-any.whl (97 kB)
Collecting importlib-metadata
  Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp36-cp36m-musllinux_1_1_x86_64.whl (29 kB)
Collecting dataclasses
  Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Collecting zipp>=0.5
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, dataclasses, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 importlib-metadata-4.8.3 itsdangerous-2.0.1 typing-extensions-4.1.1 zipp-3.6.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 0a4e883d4591
 ---> c4bcf197b3d4
Step 3/8 : COPY ./app/ /opt/
 ---> e29c562d9925
Step 4/8 : EXPOSE 8080
 ---> Running in 1c954bdbce94
Removing intermediate container 1c954bdbce94
 ---> 215ccf687c0d
Step 5/8 : RUN adduser -D worker
 ---> Running in 78eed39c720b
Removing intermediate container 78eed39c720b
 ---> 7e842acbc871
Step 6/8 : WORKDIR /opt
 ---> Running in a3f965b9111f
Removing intermediate container a3f965b9111f
 ---> 8ff2aab405c9
Step 7/8 : USER 10001
 ---> Running in 0baa3c813b34
Removing intermediate container 0baa3c813b34
 ---> 426d299a2822
Step 8/8 : ENTRYPOINT ["python", "app.py"]
 ---> Running in 84bcef2aa9ea
Removing intermediate container 84bcef2aa9ea
 ---> a11596049504
Successfully built a11596049504
Successfully tagged kodekloud/webapp-color:stable
```
</p>
</details>

After complete rebuilding progress. Let's verify images.

```bash
root@controlplane ~/webapp ➜  docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
kodekloud/webapp-color               stable              a11596049504        7 minutes ago       51.8MB
nginx                                latest              7425d3a7c478        2 months ago        142MB
busybox                              latest              1a80408de790        3 months ago        1.24MB
nginx                                alpine              51696c87e77e        3 months ago        23.4MB
k8s.gcr.io/kube-apiserver            v1.23.0             e6bf5ddd4098        7 months ago        135MB
k8s.gcr.io/kube-controller-manager   v1.23.0             37c6aeb3663b        7 months ago        125MB
k8s.gcr.io/kube-proxy                v1.23.0             e03484a90585        7 months ago        112MB
k8s.gcr.io/kube-scheduler            v1.23.0             56c5af1d00b5        7 months ago        53.5MB
python                               3.6-alpine          3a9e80fa4606        7 months ago        40.7MB
k8s.gcr.io/etcd                      3.5.1-0             25f8c7f3da61        8 months ago        293MB
k8s.gcr.io/coredns/coredns           v1.8.6              a4ca41631cc7        9 months ago        46.8MB
k8s.gcr.io/pause                     3.6                 6270bb605e12        10 months ago       683kB
weaveworks/weave-npc                 2.8.1               7f92d556d4ff        18 months ago       39.3MB
weaveworks/weave-kube                2.8.1               df29c0a4002c        18 months ago       89MB
quay.io/coreos/flannel               v0.13.1-rc1         f03a23d55e57        20 months ago       64.6MB
quay.io/coreos/flannel               v0.12.0-amd64       4e9f801d2217        2 years ago         52.8MB
kodekloud/fluent-ui-running          latest              bd30270a8b9a        3 years ago         969MB
kodekloud/webapp-color               latest              32a1ce4c22f2        3 years ago         84.8MB
```


Just leave for a moment.

---
---


Question in `kubesec` icon.
- Before you proceed, make sure to fix all issues specified in the 'Dockerfile(webapp)' section of the architecture diagram. Using the 'kubesec' tool, which is already installed on the node, identify and fix the following issues:

- Fix issues with the '/root/dev-webapp.yaml' file which was used to deploy the 'dev-webapp' pod in the 'dev' namespace.
- Redeploy the 'dev-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'
- Fix issues with the '/root/staging-webapp.yaml' file which was used to deploy the 'staging-webapp' pod in the 'staging' namespace.
- Redeploy the 'staging-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'

Solution:

First quick check overall on given `dev-webapp.yaml` YAML file.
<details>
<summary>Output</summary>
<p>

```bash
root@controlplane ~ ➜  cat dev-webapp.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: dev-webapp
  name: dev-webapp
  namespace: dev
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      allowPrivilegeEscalation: true
      capabilities:
        add:
        - NET_ADMIN
        - SYS_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-z4lvb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-z4lvb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```
</p>
</details>

Let's start to scan using kubesec which tool is already installed.
```bash
root@controlplane ~ ➜  kubesec scan /root/dev-webapp.yaml 
```
<details>
<summary>Kubesec Scan Output with Critical issues</summary>
<p>

```bash
[
  {
    "object": "Pod/dev-webapp.dev",
    "valid": true,
    "fileName": "/root/dev-webapp.yaml",
    "message": "Failed with a score of -34 points",
    "score": -34,
    "scoring": {
      "critical": [
        {
          "id": "CapSysAdmin",
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided",
          "points": -30
        },
        {
          "id": "AllowPrivilegeEscalation",
          "selector": "containers[] .securityContext .allowPrivilegeEscalation == true",
          "reason": "",
          "points": -7
        }
      ],
      "passed": [
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]
```
</p>
</details>

I removed the Critical issues in /root/dev-webapp.yaml file by removing some YAML line that faced the security issues. Also remove the Critical issues in /root/staging-webapp.yaml file with the same process.

And then I try scan back for that critical issues which are exiting or not. Verify the result on these two Pod YAML Files.

<details>
<summary>Kubesec Scan Output After Fixed Issues</summary>
<p>

```bash
[
  {
    "object": "Pod/dev-webapp.dev",
    "valid": true,
    "fileName": "/root/dev-webapp.yaml",
    "message": "Passed with a score of 3 points",
    "score": 3,
    "scoring": {
      "passed": [
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]
```
</p>
</details>

So, redeploying these two pod (given question) will see after editing some configuration in yaml file later. See you soon on editing.


---
---



Question in `prod-web` icon.

- The deployment has a secret hardcoded. Instead, create a secret called 'prod-db' for all the hardcoded values and consume the secret values as environment variables within the deployment.

Let's inspect for that deployment under `prod` namespace.

```bash
root@controlplane ~ ➜  k -n prod get all
```

<details>
<summary>Status of Deployment,Pod,Service</summary>
<p>

```bash
root@controlplane ~ ➜  k -n prod get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/prod-db                    1/1     Running   0          16m
pod/prod-web-8747d8c84-l6vr5   1/1     Running   0          16m

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/prod-db    ClusterIP   10.100.248.237   <none>        3306/TCP         16m
service/prod-web   NodePort    10.96.157.153    <none>        8080:30081/TCP   16m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prod-web   1/1     1            1           16m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/prod-web-8747d8c84   1         1         1       16m
```
</p>
</details>

Let's inspect `prod-web` deployment only.
<details>
<summary>Manifest file of prod-web Deployment</summary>
<p>

```bash
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
    creationTimestamp: "2022-07-21T10:28:17Z"
    generation: 1
    labels:
      name: prod-web
    name: prod-web
    namespace: prod
    resourceVersion: "1793"
    uid: 0c0d9747-2943-4720-9817-04ba6f65dfce
  spec:
    progressDeadlineSeconds: 600
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        name: prod-web
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          name: prod-web
        name: dev-web
      spec:
        containers:
        - env:
          - name: DB_Host
            value: prod-db
          - name: DB_User
            value: root
          - name: DB_Password
            value: paswrd
          image: mmumshad/simple-webapp-mysql
          imagePullPolicy: Always
          name: webapp-mysql
          ports:
          - containerPort: 8080
            protocol: TCP
          resources: {}
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
              - all
            readOnlyRootFilesystem: true
            runAsUser: 10001
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 1
    conditions:
    - lastTransitionTime: "2022-07-21T10:28:33Z"
      lastUpdateTime: "2022-07-21T10:28:33Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2022-07-21T10:28:17Z"
      lastUpdateTime: "2022-07-21T10:28:33Z"
      message: ReplicaSet "prod-web-8747d8c84" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    observedGeneration: 1
    readyReplicas: 1
    replicas: 1
    updatedReplicas: 1
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
Error from server (NotFound): deployments.apps "deploy" not found
```
</p>
</details>


Deployment is using the database secret as environment variable as plain text. It's not best practice as a security perspective. So, let's create secret which encrpyt secret data with base64 and then use again environment variable from secret config file.

Now, let's check that how many secrets is exit now in `prod` namespace.

```bash
root@controlplane ~ ➜  k -n prod get secret
NAME                  TYPE                                  DATA   AGE
default-token-bb4vt   kubernetes.io/service-account-token   3      21m
```
There is default secret config. Now, let's create new secret with the given name `prod-db` under `prod` namespace.

```bash
root@controlplane ~ ➜  k -n prod create secret generic prod-db --from-literal DB_Host=prod-db --from-literal DB_User=root --from-literal DB_Password=paswrd
secret/prod-db created
```

Let's verfiy the output and inspect it.

```bash
root@controlplane ~ ➜  k -n prod get secret
NAME                  TYPE                                  DATA   AGE
default-token-bb4vt   kubernetes.io/service-account-token   3      21m
prod-db               Opaque                                3      56s
```
<details>
<summary>Inspect the prod-db secret</summary>
<p>

```bash
root@controlplane ~ ➜  k -n prod describe secret prod-db 
Name:         prod-db
Namespace:    prod
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_User:      4 bytes
DB_Host:      7 bytes
DB_Password:  6 bytes
```
</p>
</details>

Now, let's edit the `prod-web` deployment and replace environment type.

<details>
<summary>Manifest file using secret environment variable type</summary>
<p>

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2022-07-20T15:40:15Z"
  generation: 1
  labels:
    name: prod-web
  name: prod-web
  namespace: prod
  resourceVersion: "2364"
  uid: 8b294267-a5fe-4de7-b903-15d2be0f1e72
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: prod-web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: prod-web
      name: dev-web
    spec:
      containers:
      - envFrom:
          secretRef:
            name: prod-db        
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - all
          readOnlyRootFilesystem: true
          runAsUser: 10001
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2022-07-20T15:40:48Z"
    lastUpdateTime: "2022-07-20T15:40:48Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2022-07-20T15:40:15Z"
    lastUpdateTime: "2022-07-20T15:40:48Z"
    message: ReplicaSet "prod-web-8747d8c84" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```
</p>
</details>

Let's verify again deployment and pod is up and running.

```bash
root@controlplane ~/webapp ➜  k -n prod get deploy,pods
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prod-web   1/1     1            1           13m

NAME                            READY   STATUS        RESTARTS   AGE
pod/prod-db                     1/1     Running       0          13m
pod/prod-web-55b5898485-lvwws   1/1     Running       0          9s
pod/prod-web-8747d8c84-kqmmn    1/1     Terminating   0          13m
```
---
---

Another icon on `prod-netpol` icon.

Question:

- Use a network policy called 'prod-netpol' that will only allow traffic only within the 'prod' namespace. All the traffic from other namespaces should be denied.

Let's inspect networkpolicies under `prod` namespace.

```bash
root@controlplane ~ ➜  kubectl -n prod get netpol
No resources found in prod namespace.
```
Before create Network Policy, let's inspect `prod` namespace with its labels.

<details>
<summary>Prod Namespace and its labels</summary>
<p>

```bash
root@controlplane ~ ➜  k get ns --show-labels
NAME              STATUS   AGE   LABELS
default           Active   57m   kubernetes.io/metadata.name=default
dev               Active   36m   kubernetes.io/metadata.name=dev
kube-node-lease   Active   57m   kubernetes.io/metadata.name=kube-node-lease
kube-public       Active   57m   kubernetes.io/metadata.name=kube-public
kube-system       Active   57m   kubernetes.io/metadata.name=kube-system
prod              Active   36m   kubernetes.io/metadata.name=prod
staging           Active   36m   kubernetes.io/metadata.name=staging
```
</p>
</details>

Will use prod namespace label in Network Policy.


Let's create `Network Policy` from the given question under `prod` namespace.

<details>
<summary>Network Policy Output</summary>
<p>

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: prod-netpol
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
      - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: prod
```
</p>
</details>

```bash
root@controlplane ~ ➜  k apply -f netpol.yaml 
networkpolicy.networking.k8s.io/prod-netpol created
```

Let's inspect network policy named `prod-netpol`.
<details>
<summary>Inspect prod-netpol Network Policy</summary>
<p>

```bash
root@controlplane ~ ✖ k -n prod describe netpol prod-netpol 
Name:         prod-netpol
Namespace:    prod
Created on:   2022-07-20 16:17:29 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      NamespaceSelector: kubernetes.io/metadata.name=prod
  Not affecting egress traffic
  Policy Types: Ingress
```
</p>
</details>

---
---

Now, let's click on Question `staging-pod` icon.

- Ensure that the pod 'staging-webapp' is immutable:
- This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells (sh and ash) before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
- Image used: 'kodekloud/webapp-color:stable'
- Redeploy the pod as per the above recommendations and make sure that the application is up.

Let's inspect Pod and Service under `staging` namespace.

```bash
root@controlplane ~ ➜  k -n staging get po,svc
NAME                 READY   STATUS    RESTARTS   AGE
pod/staging-webapp   1/1     Running   0          38m

NAME                     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/staging-webapp   NodePort   10.99.41.93   <none>        8080:30082/TCP   38m
```
According to question, `exec` command can access the pod.
Herein

```bash
root@controlplane ~ ➜  k -n staging exec staging-webapp -- whoami
root
```
 They don't want that. So, need to edit the neccessary configuration in staging-pod YAML file.

`staging-webapp.yaml` YAML file is /root/staging-webapp.yaml. Let's inspect the default one.

<details>
<summary>Given staging-webapp YAML file</summary>
<p>

```bash
root@controlplane ~ ➜ cat /root/staging-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: staging-webapp
  name: staging-webapp
  namespace: staging
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
        - SYS_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-v78f2
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-v78f2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```
</p>
</details>

Let's edit the configuration using VI. And check the output result.

<details>
<summary>Modified staging-webapp YAML Output</summary>
<p>

```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-07-20T11:57:59Z"
  labels:
    name: staging-webapp
  name: staging-webapp
  namespace: staging
  resourceVersion: "1800"
  uid: 67942841-2d8c-4c08-b362-2eb848bb6ac3
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    startupProbe:
      exec:
        command: ["/bin/rm", "/bin/sh", "/bin/ash"]
      initialDelaySeconds: 5
      periodSeconds: 5
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - NET_ADMIN
      runAsUser: 0
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-h588r
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node01
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-h588r
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T11:57:59Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T11:58:07Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T11:58:07Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T11:57:59Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://1a0ab8848e38c24c6396d1f213410ba3c407aa0fd9e8972b02b8bcaef67d42a7
    image: kodekloud/webapp-color:latest
    imageID: docker-pullable://kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423
    lastState: {}
    name: webapp-color
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-07-20T11:58:06Z"
  hostIP: 10.30.189.9
  phase: Running
  podIP: 10.50.192.4
  podIPs:
  - ip: 10.50.192.4
  qosClass: BestEffort
  startTime: "2022-07-20T11:57:59Z"
```
</p>
</details>

Configure `startupProbe` and use `kodekloud/webapp-color:stable` stable image.

Delete the pod under `staging` namespace and recreate the pod. And then verfiy the pod that it is running or not.

```bash
root@controlplane ~ ➜  k -n staging delete po staging-webapp --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "staging-webapp" force deleted

root@controlplane ~ ➜  k apply -f staging-webapp.yaml 
pod/staging-webapp created
```

```bash
root@controlplane ~ ➜  k -n staging get pod,svc
NAME             READY   STATUS    RESTARTS   AGE
staging-webapp   1/1     Running   0          12s

NAME                     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/staging-webapp   NodePort   10.102.194.113   <none>        8080:30082/TCP   31m
```

---
---

Now, Question on `dev-webapp` icon.

- Ensure that the pod 'dev-webapp' is immutable:
- This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
- Image used: 'kodekloud/webapp-color:stable'
- Redeploy the pod as per the above recommendations and make sure that the application is up.

Question is the same with `staging-webapp` under `staging` namespace.

Verify Pod and Service under `dev` namespace.

```bash
root@controlplane ~ ✖ k -n dev get po,svc
NAME             READY   STATUS    RESTARTS   AGE
pod/dev-webapp   1/1     Running   0          50m

NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/dev-webapp   NodePort   10.97.201.53   <none>        8080:30083/TCP   50m
```
Access that pod using `exec` command according to the question.

```bash
root@controlplane ~ ➜  k -n dev exec dev-webapp -- whoami
root
```

They don't want that access to pod. So, configue the pod YAML File.

<details>
<summary>Modified dev-webapp YAML Output</summary>
<p>

```bash
root@controlplane ~ ➜  cat dev-webapp.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: dev-webapp
  name: dev-webapp
  namespace: dev
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    startupProbe:
      exec:
        command: ["/bin/rm", "/bin/sh", "/bin/ash"]
      initialDelaySeconds: 5
      periodSeconds: 5
    securityContext:
      runAsUser: 0
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - NET_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-z4lvb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-z4lvb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```
</p>
</details>

Delete the `dev-webapp` pod under `dev` namespace. And recreate that pod using /root/dev-webapp.yaml.

```bash
root@controlplane ~ ➜  k -n dev delete pod dev-webapp --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "dev-webapp" force deleted

root@controlplane ~ ➜  k apply -f dev-webapp.yaml 
pod/dev-webapp created
```

Verify Pod and Service under `dev` namespace again.
```bash
root@controlplane ~ ➜  k -n dev get po,svc
NAME             READY   STATUS    RESTARTS   AGE
pod/dev-webapp   1/1     Running   0          52s

NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/dev-webapp   NodePort   10.109.85.57   <none>        8080:30083/TCP   29m
```

Now, press the `Check` button to verify all. 

BOM!!!

All are correct!

![Challenge2_Complete_Architecture](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/image/C2/Challenge2_Complete.PNG)











