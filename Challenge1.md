# Challenge1

Let's start the first question. Here is the architecture!

![Architecture]()

- It's Free and here is the link for "[CKS Challenges](https://kodekloud.com/courses/cks-challenges/)"
- Participate challenges for CKS Exam preparation or Hands-on. Be enjoy learning and practing, Good Luck!

---
---

Question on `pv` icon.

- A persistentVolume called `alpha-pv` has already been created. Do not modify it and inspect the parameters used to create it.

Solution:

Quick verify persistentvolume named `alpha-pv`.
```bash
$ kubectl get persistentvolume --namespace alpha
  NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
  alpha-pv   1Gi        RWX            Delete           Available           local-storage            2m36s
```
---
---

Quesiton on `alpha-pvc` icon.

- `alpha-pvc` should be bound to 'alpha-pv'. Delete and Re-create it if necessary.

Solution:

Let's quick check on exiting `persistentvolumeclaim` under `alpha` namespace. 

```bash
root@controlplane ~ ➜  k -n alpha get pvc
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
alpha-pvc   Pending                                      local-storage   11m
```

PVC named `alpha-pvc` is pending. Someting is needed to modify. Let's check `alpha-pvc` yaml file.
<details>
<summary>PVC YAML Output</summary>
<p>

```bash
root@controlplane ~ ➜  cat alpha-pvc.yaml 
apiVersion: v1
items:
- apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"alpha-pvc","namespace":"alpha"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"1Gi"}},"storageClassName":"local-storage"}}
    creationTimestamp: "2022-07-24T04:07:37Z"
    finalizers:
    - kubernetes.io/pvc-protection
    name: alpha-pvc
    namespace: alpha
    resourceVersion: "27558"
    uid: c84da3e3-0a89-49e8-b24d-ffc1ff87b899
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
    storageClassName: local-storage
    volumeMode: Filesystem
  status:
    phase: Pending
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

</p>
</details>

When I check with yaml output, pv and pvc parameters were not match to bound.

Okay, after quick verify persistentvolume, memorize some pv's information to create persistentvolumeclaim named `alpha-pvc`. Let's create "YAML Manifest File for PVC". 

Delete pvc and create using new PVC YAML manifest file.

```bash
root@controlplane ~ ➜  k -n alpha delete pvc alpha-pvc --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
persistentvolumeclaim "alpha-pvc" force deleted

root@controlplane ~ ➜  k apply -f pvc.yaml 
persistentvolumeclaim/alpha-pvc created
```

<details>
<summary>New PVC YAML Manifest File</summary>
<p>

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpha-pvc
  namespace: alpha
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
</p>
</details>

Let's verify! Is it bound? If it's not bound or success, delete that exiting pvc and create from the PVC manifest. It will complete successfully. 
```bash
$ kubectl --namespace alpha get persistentvolumeclaim
  NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
  alpha-pvc   Bound    alpha-pv   1Gi        RWX            local-storage   30s
```

Yes, it's bound successfully.

---
---

Question on `apparmor` icon

- Move the AppArmor profile '/root/usr.sbin.nginx' to '/etc/apparmor.d/usr.sbin.nginx' on the controlplane node
- Load the 'AppArmor` profile called 'custom-nginx' and ensure it is enforced.

[AppArmor](https://kubernetes.io/docs/tutorials/security/apparmor/) is under covered Certified Kubernetes Security course. I am not that level. So, I don't even know what is this. After 1 hour searching and watching, I understand it - AppArmor is a Linux kernel security module that supplements the standard Linux user and group based permissions to confine programs to a limited set of resources. AppArmor can be configured for any application to reduce its potential attack surface and provide greater in-depth defense

Solution:

Okay, let's start with the given question. Let's move the AppArmor profile.
```bash
$ mv /root/usr.sbin.nginx /etc/apparmor.d/
```
Verify that task.
```bash
$ ls -al /etc/apparmor.d/ | grep -i nginx
  -rw-rw-r--   1 root root  1307 Mar 14 19:55 usr.sbin.nginx
```

Before load the "custom-nginx", let's check with "apparmor_status"

<details>
<summary>Apparmor_Status</summary>
<p>

```bash
$ apparmor_status 
  apparmor module is loaded.
  56 profiles are loaded.
  19 profiles are in enforce mode.
     /sbin/dhclient
     /usr/bin/lxc-start
     /usr/bin/man
     /usr/lib/NetworkManager/nm-dhcp-client.action
     /usr/lib/NetworkManager/nm-dhcp-helper
     /usr/lib/chromium-browser/chromium-browser//browser_java
     /usr/lib/chromium-browser/chromium-browser//browser_openjdk
     /usr/lib/chromium-browser/chromium-browser//sanitized_helper
     /usr/lib/connman/scripts/dhclient-script
     /usr/lib/snapd/snap-confine
     /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
     /usr/sbin/tcpdump
     docker-default
     lxc-container-default
     lxc-container-default-cgns
     lxc-container-default-with-mounting
     lxc-container-default-with-nesting
     man_filter
     man_groff
37 profiles are in complain mode.
   /usr/lib/chromium-browser/chromium-browser
   /usr/lib/chromium-browser/chromium-browser//chromium_browser_sandbox
   /usr/lib/chromium-browser/chromium-browser//lsb_release
   /usr/lib/chromium-browser/chromium-browser//xdgsettings
   /usr/lib/dovecot/anvil
   /usr/lib/dovecot/auth
   /usr/lib/dovecot/config
   /usr/lib/dovecot/deliver
   /usr/lib/dovecot/dict
   /usr/lib/dovecot/dovecot-auth
   /usr/lib/dovecot/dovecot-lda
   /usr/lib/dovecot/dovecot-lda///usr/sbin/sendmail
   /usr/lib/dovecot/imap
   /usr/lib/dovecot/imap-login
   /usr/lib/dovecot/lmtp
   /usr/lib/dovecot/log
   /usr/lib/dovecot/managesieve
   /usr/lib/dovecot/managesieve-login
   /usr/lib/dovecot/pop3
   /usr/lib/dovecot/pop3-login
   /usr/lib/dovecot/ssl-params
   /usr/sbin/avahi-daemon
   /usr/sbin/dnsmasq
   /usr/sbin/dnsmasq//libvirt_leaseshelper
   /usr/sbin/dovecot
   /usr/sbin/identd
   /usr/sbin/mdnsd
   /usr/sbin/nmbd
   /usr/sbin/nscd
   /usr/sbin/smbd
   /usr/sbin/smbldap-useradd
   /usr/sbin/smbldap-useradd///etc/init.d/nscd
   /usr/{sbin/traceroute,bin/traceroute.db}
   klogd
   ping
   syslog-ng
   syslogd
16 processes have profiles defined.
16 processes are in enforce mode.
   docker-default (2339) 
   docker-default (2352) 
   docker-default (2375) 
   docker-default (2380) 
   docker-default (2479) 
   docker-default (2512) 
   docker-default (2584) 
   docker-default (2607) 
   docker-default (4172) 
   docker-default (4229) 
   docker-default (4377) 
   docker-default (4438) 
   docker-default (7027) 
   docker-default (7047) 
   docker-default (7248) 
   docker-default (7303) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```
</p>
</details>

There is no "custom-nginx" yet.

Okay, time to load "custom-nginx".

```bash
$ apparmor_parser /etc/apparmor.d/usr.sbin.nginx
```
What's about now?

<details>
<summary>After Load Apparmor "custom-nginx"</summary>
<p>

```bash
$ apparmor_status 
  apparmor module is loaded.
  57 profiles are loaded.
  20 profiles are in enforce mode.
     /sbin/dhclient
     /usr/bin/lxc-start
     /usr/bin/man
     /usr/lib/NetworkManager/nm-dhcp-client.action
     /usr/lib/NetworkManager/nm-dhcp-helper
     /usr/lib/chromium-browser/chromium-browser//browser_java
     /usr/lib/chromium-browser/chromium-browser//browser_openjdk
     /usr/lib/chromium-browser/chromium-browser//sanitized_helper
     /usr/lib/connman/scripts/dhclient-script
     /usr/lib/snapd/snap-confine
     /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
     /usr/sbin/tcpdump
     custom-nginx
     docker-default
 lxc-container-default
     lxc-container-default-cgns
     lxc-container-default-with-mounting
     lxc-container-default-with-nesting
     man_filter
     man_groff
37 profiles are in complain mode.
   /usr/lib/chromium-browser/chromium-browser
   /usr/lib/chromium-browser/chromium-browser//chromium_browser_sandbox
   /usr/lib/chromium-browser/chromium-browser//lsb_release
   /usr/lib/chromium-browser/chromium-browser//xdgsettings
   /usr/lib/dovecot/anvil
   /usr/lib/dovecot/auth
   /usr/lib/dovecot/config
   /usr/lib/dovecot/deliver
   /usr/lib/dovecot/dict
   /usr/lib/dovecot/dovecot-auth
   /usr/lib/dovecot/dovecot-lda
   /usr/lib/dovecot/dovecot-lda///usr/sbin/sendmail
   /usr/lib/dovecot/imap
   /usr/lib/dovecot/imap-login
   /usr/lib/dovecot/lmtp
   /usr/lib/dovecot/log
   /usr/lib/dovecot/managesieve
   /usr/lib/dovecot/managesieve-login
   /usr/lib/dovecot/pop3
   /usr/lib/dovecot/pop3-login
   /usr/lib/dovecot/ssl-params
   /usr/sbin/avahi-daemon
   /usr/sbin/dnsmasq
   /usr/sbin/dnsmasq//libvirt_leaseshelper
   /usr/sbin/dovecot
   /usr/sbin/identd
   /usr/sbin/mdnsd
   /usr/sbin/nmbd
   /usr/sbin/nscd
   /usr/sbin/smbd
   /usr/sbin/smbldap-useradd
   /usr/sbin/smbldap-useradd///etc/init.d/nscd
   /usr/{sbin/traceroute,bin/traceroute.db}
   klogd
   ping
   syslog-ng
   syslogd
16 processes have profiles defined.
16 processes are in enforce mode.
   docker-default (2339) 
   docker-default (2352) 
   docker-default (2375) 
   docker-default (2380) 
   docker-default (2479) 
   docker-default (2512) 
   docker-default (2584) 
   docker-default (2607) 
   docker-default (4172) 
   docker-default (4229) 
   docker-default (4377) 
   docker-default (4438) 
   docker-default (7027) 
   docker-default (7047) 
   docker-default (7248) 
   docker-default (7303) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```
</p>
</details>

Here is the "custom-nginx". That apparmor profile will use in pod's manifest file later.


---
---

Question on `images` icon.

- Permitted images are: 'nginx:alpine', 'bitnami/nginx', 'nginx:1.13', 'nginx:1.17', 'nginx:1.16'and 'nginx:1.14'. Use 'trivy' to find the image with the least number of 'CRITICAL' vulnerabilities.

Solution:

So, how to find the image with the least number of vulnerabilities? Okay, there is a hint. What is [**trivy**](https://aquasecurity.github.io/trivy/v0.24.2/) in the question? After two hours searching and watching about trivy? I am understand a quite. **Trivy** is a simple and comprehensive vulnerability/misconfiguration scanner for containers and other artifacts.

 There are images in the given question. So, I need to try one by one with trivy that scan the images with least the vulnerability. I will not show the result. Its output is so long. I use this command
 ```bash
$ trivy image --severity CRITICAL <image>
 ```

 <details>
 <summary>Scanning for "nginx:1.17"</summary>
 <p>
 
 ```bash
 root@controlplane /etc ✖ trivy image --severity CRITICAL nginx:1.17
2022-07-24T04:33:58.492Z        INFO    Need to update DB
2022-07-24T04:33:58.492Z        INFO    Downloading DB...
28.79 MiB / 28.79 MiB [----------------------------------------------------------------------------------------------------------------------] 100.00% 16.12 MiB p/s 2s
2022-07-24T04:34:04.719Z        INFO    Detecting Debian vulnerabilities...
2022-07-24T04:34:04.748Z        INFO    Trivy skips scanning programming language libraries because no supported file was detected

nginx:1.17 (debian 10.4)
========================
Total: 42 (CRITICAL: 42)

+--------------+------------------+----------+---------------------------+-------------------+-----------------------------------------+
|   LIBRARY    | VULNERABILITY ID | SEVERITY |     INSTALLED VERSION     |   FIXED VERSION   |                  TITLE                  |
+--------------+------------------+----------+---------------------------+-------------------+-----------------------------------------+
| dpkg         | CVE-2022-1664    | CRITICAL | 1.19.7                    | 1.19.8            | Vhcalc 0.2.5 updates Dockerfile         |
|              |                  |          |                           |                   | to "python:3.9-slim-buster"             |
|              |                  |          |                           |                   | to include security fixes.              |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-1664    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libbsd0      | CVE-2019-20367   |          | 0.9.1-2                   | 0.9.1-2+deb10u1   | nlist.c in libbsd before                |
|              |                  |          |                           |                   | 0.10.0 has an out-of-bounds             |
|              |                  |          |                           |                   | read during a comparison...             |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2019-20367   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libc-bin     | CVE-2019-1010022 |          | 2.28-10                   |                   | glibc: stack guard protection bypass    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2019-1010022 |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2021-33574   |          |                           |                   | glibc: mq_notify does                   |
|              |                  |          |                           |                   | not handle separately                   |
|              |                  |          |                           |                   | allocated thread attributes             |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-33574   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2021-35942   |          |                           |                   | glibc: Arbitrary read in wordexp()      |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-35942   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-23218   |          |                           |                   | glibc: Stack-based buffer overflow      |
|              |                  |          |                           |                   | in svcunix_create via long pathnames    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23218   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-23219   |          |                           |                   | glibc: Stack-based buffer               |
|              |                  |          |                           |                   | overflow in sunrpc clnt_create          |
|              |                  |          |                           |                   | via a long pathname                     |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23219   |
+--------------+------------------+          +                           +-------------------+-----------------------------------------+
| libc6        | CVE-2019-1010022 |          |                           |                   | glibc: stack guard protection bypass    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2019-1010022 |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2021-33574   |          |                           |                   | glibc: mq_notify does                   |
|              |                  |          |                           |                   | not handle separately                   |
|              |                  |          |                           |                   | allocated thread attributes             |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-33574   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2021-35942   |          |                           |                   | glibc: Arbitrary read in wordexp()      |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-35942   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-23218   |          |                           |                   | glibc: Stack-based buffer overflow      |
|              |                  |          |                           |                   | in svcunix_create via long pathnames    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23218   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-23219   |          |                           |                   | glibc: Stack-based buffer               |
|              |                  |          |                           |                   | overflow in sunrpc clnt_create          |
|              |                  |          |                           |                   | via a long pathname                     |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23219   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libdb5.3     | CVE-2019-8457    |          | 5.3.28+dfsg1-0.5          |                   | sqlite: heap out-of-bound               |
|              |                  |          |                           |                   | read in function rtreenode()            |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2019-8457    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libexpat1    | CVE-2022-22822   |          | 2.2.6-2+deb10u1           | 2.2.6-2+deb10u2   | expat: Integer overflow in              |
|              |                  |          |                           |                   | addBinding in xmlparse.c                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-22822   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-22823   |          |                           |                   | expat: Integer overflow in              |
|              |                  |          |                           |                   | build_model in xmlparse.c               |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-22823   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-22824   |          |                           |                   | expat: Integer overflow in              |
|              |                  |          |                           |                   | defineAttribute in xmlparse.c           |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-22824   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-23852   |          |                           |                   | expat: Integer overflow                 |
|              |                  |          |                           |                   | in function XML_GetBuffer               |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23852   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-23990   |          |                           |                   | expat: integer overflow                 |
|              |                  |          |                           |                   | in the doProlog function                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-23990   |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-25235   |          |                           | 2.2.6-2+deb10u3   | expat: Malformed 2- and                 |
|              |                  |          |                           |                   | 3-byte UTF-8 sequences can              |
|              |                  |          |                           |                   | lead to arbitrary code...               |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-25235   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-25236   |          |                           |                   | expat: Namespace-separator characters   |
|              |                  |          |                           |                   | in "xmlns[:prefix]" attribute           |
|              |                  |          |                           |                   | values can lead to arbitrary code...    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-25236   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2022-25315   |          |                           |                   | expat: Integer overflow                 |
|              |                  |          |                           |                   | in storeRawNames()                      |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-25315   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libfreetype6 | CVE-2022-27404   |          | 2.9.1-3+deb10u1           |                   | FreeType: Buffer Overflow               |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-27404   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libgnutls30  | CVE-2021-20231   |          | 3.6.7-4+deb10u3           | 3.6.7-4+deb10u7   | gnutls: Use after free in               |
|              |                  |          |                           |                   | client key_share extension              |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-20231   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2021-20232   |          |                           |                   | gnutls: Use after free                  |
|              |                  |          |                           |                   | in client_send_params in                |
|              |                  |          |                           |                   | lib/ext/pre_shared_key.c                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-20232   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| liblz4-1     | CVE-2021-3520    |          | 1.8.3-1                   | 1.8.3-1+deb10u1   | lz4: memory corruption                  |
|              |                  |          |                           |                   | due to an integer overflow              |
|              |                  |          |                           |                   | bug caused by memmove...                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-3520    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libseccomp2  | CVE-2019-9893    |          | 2.3.3-4                   |                   | libseccomp: incorrect generation        |
|              |                  |          |                           |                   | of syscall filters in libseccomp        |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2019-9893    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libssl1.1    | CVE-2021-3711    |          | 1.1.1d-0+deb10u3          | 1.1.1d-0+deb10u7  | openssl: SM2 Decryption                 |
|              |                  |          |                           |                   | Buffer Overflow                         |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-3711    |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-1292    |          |                           | 1.1.1n-0+deb10u2  | openssl: c_rehash script                |
|              |                  |          |                           |                   | allows command injection                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-1292    |
+              +------------------+          +                           +-------------------+-----------------------------------------+
|              | CVE-2022-2068    |          |                           | 1.1.1n-0+deb10u3  | openssl: the c_rehash script            |
|              |                  |          |                           |                   | allows command injection                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2022-2068    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libtiff5     | CVE-2017-9117    |          | 4.1.0+git191117-2~deb10u1 |                   | libtiff: Heap-based buffer              |
|              |                  |          |                           |                   | over-read in bmp2tiff                   |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2017-9117    |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libwebp6     | CVE-2018-25009   |          | 0.6.1-2                   | 0.6.1-2+deb10u1   | libwebp: out-of-bounds read             |
|              |                  |          |                           |                   | in WebPMuxCreateInternal                |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25009   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2018-25010   |          |                           |                   | libwebp: out-of-bounds                  |
|              |                  |          |                           |                   | read in ApplyFilter()                   |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25010   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2018-25011   |          |                           |                   | libwebp: heap-based buffer              |
|              |                  |          |                           |                   | overflow in PutLE16()                   |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25011   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2018-25012   |          |                           |                   | libwebp: out-of-bounds read             |
|              |                  |          |                           |                   | in WebPMuxCreateInternal()              |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25012   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2018-25013   |          |                           |                   | libwebp: out-of-bounds                  |
|              |                  |          |                           |                   | read in ShiftBytes()                    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25013   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2018-25014   |          |                           |                   | libwebp: use of uninitialized           |
|              |                  |          |                           |                   | value in ReadSymbol()                   |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2018-25014   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2020-36328   |          |                           |                   | libwebp: heap-based buffer overflow     |
|              |                  |          |                           |                   | in WebPDecode*Into functions            |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2020-36328   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2020-36329   |          |                           |                   | libwebp: use-after-free in              |
|              |                  |          |                           |                   | EmitFancyRGB() in dec/io_dec.c          |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2020-36329   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2020-36330   |          |                           |                   | libwebp: out-of-bounds read             |
|              |                  |          |                           |                   | in ChunkVerifyAndAssign()               |
|              |                  |          |                           |                   | in mux/muxread.c                        |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2020-36330   |
+              +------------------+          +                           +                   +-----------------------------------------+
|              | CVE-2020-36331   |          |                           |                   | libwebp: out-of-bounds                  |
|              |                  |          |                           |                   | read in ChunkAssignData()               |
|              |                  |          |                           |                   | in mux/muxinternal.c                    |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2020-36331   |
+--------------+------------------+          +---------------------------+-------------------+-----------------------------------------+
| libx11-6     | CVE-2021-31535   |          | 2:1.6.7-1                 | 2:1.6.7-1+deb10u2 | libX11: missing request length checks   |
|              |                  |          |                           |                   | -->avd.aquasec.com/nvd/cve-2021-31535   |
+--------------+                  +          +                           +                   +                                         +
| libx11-data  |                  |          |                           |                   |                                         |
|              |                  |          |                           |                   |                                         |
+--------------+------------------+----------+---------------------------+-------------------+-----------------------------------------+
 ```
 </p>
 </details>

<details>
<summary>Scanning for "nginx:alpine"</summary>
<p>

```bash
root@controlplane /etc ➜  trivy image --severity CRITICAL nginx:alpine
2022-07-24T04:39:08.681Z        WARN    This OS version is not on the EOL list: alpine 3.16
2022-07-24T04:39:08.682Z        INFO    Detecting Alpine vulnerabilities...
2022-07-24T04:39:08.696Z        INFO    Trivy skips scanning programming language libraries because no supported file was detected
2022-07-24T04:39:08.696Z        WARN    This OS version is no longer supported by the distribution: alpine 3.16.1
2022-07-24T04:39:08.696Z        WARN    The vulnerability detection may be insufficient because security updates are not provided

nginx:alpine (alpine 3.16.1)
============================
Total: 0 (CRITICAL: 0)
```
</p>
</details>
I found "nginx:alpine" is less the vulnerability than other images. So, I will use this image under deployment as a image later.


---
---

Question on `middleware` icon.

- Pod called 'middleware' is already deployed in the 'alpha' namespace. Inspect it but do not alter it in anyway!


Solution:

Just verify the pod named "middleware" with labels under namespace "alpha".
```bash
$ kubectl --namespace alpha get pod --show-labels
  NAME         READY   STATUS    RESTARTS   AGE   LABELS
  external     1/1     Running   0          18m   app=external
  middleware   1/1     Running   0          18m   app=middleware
```
It's runnig.

---
---

Question on `external` icon.

- Pod called 'external' is already deployed in the 'alpha' namespace. Inspect it but do not alter it in anyway!
- 'external' pod should NOT be able to connect to 'alpha-svc' on port 80

Solution:

Let's verify the pod with labels.
```bash
$ kubectl --namespace alpha get pod --show-labels
  NAME         READY   STATUS    RESTARTS   AGE   LABELS
  external     1/1     Running   0          18m   app=external
  middleware   1/1     Running   0          18m   app=middleware
```
Yeah, just leave it for that question. It's enough.

---
---

Question on `alpha-xyz` icon.

- Create a deployment called 'alpha-xyz' that uses the image with the least 'CRITICAL' vulnerabilities? (Use the sample YAML file located at '/root/alpha-xyz.yaml' to create the deployment. Please make sure to use the same names and labels specified in this sample YAML file!)
- Deployment has exactly '1' ready replica
- 'data-volume' is mounted at '/usr/share/nginx/html' on the pod


Solution:

Let's deploy and add the necessary info to that "alpha-xyz" under the given path.
```bash
$ vi /root/alpha-xyz.yaml
```
<details>
<summary>Alpha-XYZ YAML Manifest File</summary>
<p>

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: alpha-xyz
  name: alpha-xyz
  namespace: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-xyz
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: alpha-xyz
      annotations:
        container.apparmor.security.beta.kubernetes.io/nginx: localhost/custom-nginx
    spec:
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: alpha-pvc
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - name: data-volume
          mountPath: /usr/share/nginx/html
```
</p>
</details>

Create the Deployment named "alpha-xyz".
```bash
$ kubectl create -f alpha-xyz.yaml
```

Let's verify. Is it running? 
```bash
$  kubectl --namespace alpha get pod
   NAME                         READY   STATUS    RESTARTS   AGE
   alpha-xyz-7c59b9c98f-p69jk   1/1     Running   0          15s
   external                     1/1     Running   0          21m
   middleware                   1/1     Running   0          21m
```

It's running successfully.

---
---

Question on "restrict-inbound" icon.

- Create a NetworkPolicy called 'restrict-inbound' in the 'alpha' namespace
Policy Type = 'Ingress'
Inbound access only allowed from the pod called 'middleware' with label 'app=middleware'
Inbound access only allowed to TCP port 80 on pods matching the policy

Solution:

Create "Network Policy" manifest file.
<details>
<summary>Restrict-Inbound YAML Manifest File</summary>
<p>

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-inbound
  namespace: alpha
spec:
  podSelector:
    matchLabels:
      app: alpha-xyz
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          namespace: alpha
      podSelector:
        matchLabels:
          app: middleware
    - podSelector:
        matchLabels:
          app: middleware
    ports:
    - protocol: TCP
      port: 80
```
</p>
</details>

Let's verify.
```bash
$ kubectl --namespace alpha get networkpolicy
  NAME               POD-SELECTOR    AGE
  restrict-inbound   app=alpha-xyz   19s
```

Let's describe that "restrict-inbound".
<details>
<summary>Describe "restrict-inbound"</summary>
<p>

```bash
$ kubectl --namespace alpha describe networkpolicy restrict-inbound
Name:         restrict-inbound
Namespace:    alpha
Created on:   2022-03-18 03:14:08 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=alpha-xyz
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      NamespaceSelector: namespace=alpha
      PodSelector: app=middleware
    From:
      PodSelector: app=middleware
  Not affecting egress traffic
  Policy Types: Ingress
```
</p>
</details>

I spent a lot of hour on this manifest file. There is a trick for me. I read [**Kubernetes Document**](https://kubernetes.io/docs/concepts/services-networking/network-policies/) again and again. Now, it's easy for you when you read this **Network Policy** YAML manifest file. For me, It's not easy. After that, I can solve it by elimating my mistake or misconfiguration.

---
---

Question on `alpha-service` icon.

- Expose the 'alpha-xyz' as a 'ClusterIP' type service called 'alpha-svc'
'alpha-svc' should be exposed on 'port: 80' and 'targetPort: 80'

Solution:

Let's expose the "alpha-xyz" deployment as a "ClusterIP" with named "alpha-svc".

```bash
$ kubectl --namespace alpha expose deployment alpha-xyz --name alpha-svc --port 80 --target-port 80
```

Let's verify the resource that created.
```bash
$ kubectl --namespace alpha get service
  NAME        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
  alpha-svc   ClusterIP   10.111.92.13   <none>        80/TCP    12s
```
<details>
<summary>Alpha-SVC Describe</summary>
<p>

```bash
$ kubectl --namespace alpha describe service alpha-svc
Name:              alpha-svc
Namespace:         alpha
Labels:            app=alpha-xyz
Annotations:       <none>
Selector:          app=alpha-xyz
Type:              ClusterIP
IP Families:       <none>
IP:                10.111.92.13
IPs:               10.111.92.13
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.50.0.6:80
Session Affinity:  None
Events:            <none>
```
</p>
</details>

---
---

Finally, time to press final "Check" button. Let's do it.
Bomb!!! Bomb!!! Bomb!!!
It's complete successfully.

![Challenge1_Complete_Architecture]()
