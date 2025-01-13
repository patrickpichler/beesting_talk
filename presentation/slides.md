---
colorSchema: light
theme: apple-basic
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: none
title: Of bees and Kubernetes Runtime Security
mdc: true
defaults:
    layout: center
layout: image
class: text-center
image: /bee.jpg
---

<div class="w-full h-full flex flex-justify-end flex-items-end">
    <span class="m-b-10" style="color: #181825; text-shadow: 2px 1px #bac2de;">
        <span class="fancy-headline">
            Beesting
        </span>
        <br/>
        <span class="fancy-headline-small">
            Can't touch this
        </span>
    </span>
</div>

<div class="attribution">
    <a href="https://pixabay.com/photos/sunflower-flower-bee-insect-6545123/">Image by mariya_m</a> on pixabay
</div>


---
layout: full
---

<div class="grid grid-cols-[1fr_35%] gap-6">

<div>
<h1 class="bold">Who am I?</h1>

<br/>

<h2>Software engineer turned Cloud Enthusiast <noto-cloud /></h2>
<br/>
<h2>Kubernetes wizard <noto-magic-wand /></h2>
<br/>
<h2>Linux Nerd <devicon-linux /></h2>
</div>

<div>
<img src="/profile_pic_compressed.jpg" style="border-radius: 50%;"/>
</div>

</div>

<!--
Originally I started my career as a Java Software Developer, but everything changed when I stumbled
upon Linux and the cloud. This definitely transformed me into a full-on Linux nerd. Do not question the MacBook though.
-->

---
layout: image
image: /time.jpg
backgroundSize: 110%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/close-up-woman-with-watch_21768055.htm#fromView=search&page=1&position=28&uuid=a87f4551-9928-476a-9f10-f078c92b901e">Image by Freepik</a> on Freepik
</div>

---
layout: image
image: /dyantrace_blog.png
backgroundSize: 80%
---

<div class="attribution">
    <a href="https://www.dynatrace.com/news/blog/context-aware-security-incident-response/">Link</a> to blog
</div>

---
layout: image
image: /dynatrace_diagram.excalidraw.svg
backgroundSize: 80%
---

---
layout: image
image: /honeytoken.jpg
backgroundSize: 100%
---

<div class="h-full" style="display: flex; justify-content: end; flex-direction: column">
    <span id="headline" style="color: #eff1f5">
        <span class="fancy-headline-small">What even is a</span><br/>
        <span class="fancy-headline">Honeytoken?</span>
    </span>
</div>

<style>
#headline {
    text-shadow: 3px 2px #11111b;
}
</style>

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/close-up-honey-dripping-off-wooden-dipper-with-copy-space_6073844.htm#fromView=search&page=1&position=4&uuid=a963c1e3-765e-4800-98fc-b014e200ee61">Image by Freepik</a> on Freepik
</div>

---
layout: fact
class: bg-[#8bd5ca] color-[#e64553]
---

# Digital bait

---
layout: image
image: /hacker.jpg
---

<span class="text-size-4xl absolute top-30 right-55 text-align-center color-red">This is us</span>
<Arrow x1="600" y1="150" x2="480" y2="200" color="red" />

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/side-view-male-hacker-with-laptop_8725474.htm#fromView=search&page=1&position=18&uuid=e4c7a153-84c8-4550-87fc-1e259fa8afb5">Image by freepik</a> on Freepik
</div>

---
layout: image
image: /cms.png
backgroundSize: 80%
---
---
layout: image
image: /cms_cve.png
backgroundSize: 80%
---

---
layout: full
class: code-fill
---

```shellsession
$ root@simpecms-...:/root#
```

---
layout: full
class: code-fill
---

```shellsession {|2,3,10,}
$ root@simpecms-...:/root# env
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=simpecms-6c4bfc97cd-wdnhv
PHP_VERSION=7.4.9
APACHE_CONFDIR=/etc/apache2
PHP_LDFLAGS=-Wl,-O1 -pie
PWD=/var/www/html
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
TERM=xterm
PHP_URL=https://www.php.net/distributions/php-7.4.9.tar.xz
...
```

---
layout: full
class: code-fill
---

```shellsession
$ root@simpecms-...:/root# ls -al /var/run/secrets/
```

---
layout: full
class: code-fill
---

```shellsession {|5}
$ root@simpecms-...:/root# ls -al /var/run/secrets/
total 20
drwxr-xr-x 4 root root 4096 Jan  5 09:57 .
drwxr-xr-x 1 root root 4096 Jan  5 09:57 ..
drwxr-xr-x 2 root root 4096 Jan  5 09:57 eks.amazonaws.com
drwxr-xr-x 3 root root 4096 Jan  5 09:57 kubernetes.io
```

---
layout: full
class: code-fill
---

```shellsession {|12}
$ root@simpecms-...:/root# ls -al /var/run/secrets/
total 20
drwxr-xr-x 4 root root 4096 Jan  5 09:57 .
drwxr-xr-x 1 root root 4096 Jan  5 09:57 ..
drwxr-xr-x 2 root root 4096 Jan  5 09:57 eks.amazonaws.com
drwxr-xr-x 3 root root 4096 Jan  5 09:57 kubernetes.io

$ root@simpecms-...:/root# ls -al /var/run/secrets/eks.amazonaws.com/
total 12
drwxr-xr-x 2 root root 4096 Jan  5 09:57 .
drwxr-xr-x 4 root root 4096 Jan  5 09:57 ..
-rw-r--r-- 1 root root   16 Jan  5 09:57 access_key_token
```
---
layout: full
class: code-fill
---

```shellsession {14}
$ root@simpecms-...:/root# ls -al /var/run/secrets/
total 20
drwxr-xr-x 4 root root 4096 Jan  5 09:57 .
drwxr-xr-x 1 root root 4096 Jan  5 09:57 ..
drwxr-xr-x 2 root root 4096 Jan  5 09:57 eks.amazonaws.com
drwxr-xr-x 3 root root 4096 Jan  5 09:57 kubernetes.io

$ root@simpecms-...:/root# ls -al /var/run/secrets/eks.amazonaws.com/
total 12
drwxr-xr-x 2 root root 4096 Jan  5 09:57 .
drwxr-xr-x 4 root root 4096 Jan  5 09:57 ..
-rw-r--r-- 1 root root   16 Jan  5 09:57 access_key_token

$ root@simpecms-...:/root# cat /v/r/s/eks.amazonaws.com/access_key_token
```

---
layout: image
image: /trap_card_meme.png
backgroundSize: 60%
---

---
layout: image
image: /beesting_overview.excalidraw.svg
backgroundSize: 80%
---

---
layout: image
image: /ship.jpg
backgroundSize: 100%
---

<div class="w-full h-full flex flex-justify-start flex-items-end">
    <span class="m-b-10 color-[#e64553]" style="text-shadow: 2px 1px #bac2de;">
        <span class="fancy-headline-small">
            How does <br/>
        </span>
        <span class="fancy-headline">
            Kubernetes <br/>
        </span>
        <span class="fancy-headline-small">
            work?
        </span>
    </span>
</div>

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/aerial-view-container-cargo-ship-sea_13180315.htm#fromView=search&page=1&position=6&uuid=e5f2b7b3-d2d2-49b2-926e-eba339ed6a5b">Image by tawatchai07</a> on Freepik
</div>

---
layout: image
image: /kubernetes_overview.excalidraw.svg
backgroundSize: 100%
---

<v-switch>

<template #1>
    <Arrow x1="300" y1="35" x2="200" y2="35" color="red" />
</template>

<template #2>
    <Arrow x1="730" y1="35" x2="630" y2="35" color="red" />
</template>

<template #3>
    <Arrow x1="810" y1="245" x2="710" y2="245" color="red" />
    <Arrow x1="810" y1="470" x2="710" y2="470" color="red" />
</template>

</v-switch>

---
layout: image
image: /kubernetes_overview_node.excalidraw.svg
backgroundSize: 100%
---

<v-switch>

<template #1>
    <Arrow x1="655" y1="455" x2="555" y2="455" color="red" />
</template>

<template #2>
    <Arrow x1="765" y1="170" x2="665" y2="170" color="red" />
</template>

<template #3>
    <Arrow x1="890" y1="455" x2="790" y2="455" color="red" />
</template>

</v-switch>

---
layout: section
---

## Cgroups

<span class="text-size-5xl">Limit resource usage (CPU, Memory)</span>

---
layout: section
---

## Namespaces

<span class="text-size-5xl">Separate areas (Networking, Mounts)</span>

---
layout: image
image: /woman_thinking.jpg
backgroundSize: 110%
---

<div class="w-full h-full flex flex-justify-end flex-items-start">
    <span class="" style="color: #fcfc64; text-shadow: 2px 1px #bac2de;">
        <span class="fancy-headline-smaller">
            How do we get the<br/>
        </span>
        <span class="fancy-headline-small">
            Honeytoken<br/>
        </span>
        <span class="fancy-headline-smaller">
            into the container?
        </span>
    </span>
</div>

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/contemplative-female-looks-seriously-pensively-aside-purses-lips-concentrated-dressed-green-loose-jumper-makes-choice-mind_12930208.htm#fromView=search&page=7&position=29&uuid=f0298062-b6e1-429b-95cb-12126fe0161b">Image by wayhomestudio</a> on Freepik
</div>

---
layout: image
image: /dynatrace_diagram_detailed.excalidraw.svg
backgroundSize: 80%
---

<v-switch>

<template #1>
    <Arrow x1="390" y1="150" x2="290" y2="150" color="red" />
</template>

<template #2>
</template>

</v-switch>


---
layout: full
class: code-fill code-small-font
---

```yaml {|15-16|17-18|15-18|10-14|13-14}
apiVersion: v1
kind: Pod
metadata:
  name: "myapp"
  namespace: default
spec:
  containers:
  - name: myapp
    image: "myapp:latest"
    volumeMounts:
    - name: honey-volume
      readOnly: true
      subPath: token
      mountPath: /run/secrets/eks.amazonaws.com/s3_token
  volumes:
    - name: honeytoken
      secret:
        secretName: honeytoken

```
---
layout: image
image: /beesting_overview_webhook.excalidraw.svg
backgroundSize: 80%
---
---
layout: section
---

# Problems

<div class="text-size-4xl">

<v-clicks>

* Lot of moving parts
* Kubernetes specific

</v-clicks>

</div>

---
layout: fact
class: bg-[#7287fd] color-[#e5c890]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #737994;">
 Boring
</span>

---
layout: image
image: /kubernetes_overview_node.excalidraw.svg
backgroundSize: 100%
---

<v-clicks>
<Arrow x1="800" y1="20" x2="660" y2="130" color="red" />
</v-clicks>

---
layout: image
image: /containerd_plugins.png
backgroundSize: 90%
---
---
layout: section
---

<div class="text-size-5xl">

* Runtime

<v-clicks>

* Differ
* and a few more

</v-clicks>

</div>

---
layout: fact
class: bg-[#ef9f76] color-[#1BA0E4]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #e6e9ef;">
 Differ Plugins
</span>

---

```proto {|10}
// Diff service creates and applies diffs
service Diff {
	// Apply applies the content associated with the provided digests onto
	// the provided mounts. Archive content will be extracted and
	// decompressed if necessary.
	rpc Apply(ApplyRequest) returns (ApplyResponse);

	// Diff creates a diff between the given mounts and uploads the result
	// to the content store.
	rpc Diff(DiffRequest) returns (DiffResponse);
}
```

---

```proto
message DiffRequest {
	repeated containerd.types.Mount left = 1;
	repeated containerd.types.Mount right = 2;
	string media_type = 3;
	string ref = 4;
	map<string, string> labels = 5;
	google.protobuf.Timestamp source_date_epoch = 6;
}
```

---
layout: fact
class: bg-[#f9e2af] color-[#7287fd]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #626880;">
    Node Resource Interface
</span>

---
layout: image
image: /nri_overview.excalidraw.svg
backgroundSize: 90%
---

---
class: code-small-font
---

```go {|8}
type handlers struct {
	Configure           func(...) (api.EventMask, error)
	Synchronize         func(...) ([]*api.ContainerUpdate, error)
	Shutdown            func(...)
	RunPodSandbox       func(...) error
	StopPodSandbox      func(...) error
	RemovePodSandbox    func(...) error
	CreateContainer     func(...) (*api.ContainerAdjustment, []*api.ContainerUpdate, error)
	StartContainer      func(...) error
	UpdateContainer     func(...) ([]*api.ContainerUpdate, error)
	StopContainer       func(...) ([]*api.ContainerUpdate, error)
	RemoveContainer     func(...) error
	PostCreateContainer func(...) error
	PostStartContainer  func(...) error
	PostUpdateContainer func(...) error
}
```

---
layout: section
---

<div class="p-10">

```go {|3|5}
type ContainerAdjustment struct {
	Annotations map[string]string
	Mounts      []*Mount
	Env         []*KeyValue
	Hooks       *Hooks
	Linux       *LinuxContainerAdjustment
	Rlimits     []*POSIXRlimit
	CDIDevices  []*CDIDevice
}
```

</div>

---
layout: image
image: /inject_container.jpg
---

<div class="w-full h-full flex flex-justify-end flex-items-start text-align-center">
    <span class="m-10 color-[#dce0e8]" style="text-shadow: 2px 1px #4c4f69;">
        <span class="fancy-headline-small">
            What to inject <br/> into the <br/> container?
        </span>
    </span>
</div>

---
layout: fact
class: bg-[#df8e1d] color-[#232634]
---

<span class="text-size-7xl" style="text-shadow: 2px 1px #e6e9ef;">
    Honeytokens = Files
</span>

---
layout: section
---

<div class="text-size-5xl">

* ext2, ext3, ext3
* BTRFS
* XFS
* ZFS
* and many more

</div>

---
layout: image
image: /linux_vfs.excalidraw.svg
backgroundSize: 60%
---

---
layout: image
image: /what_if.jpg
---

<div class="w-full h-full flex flex-justify-end flex-items-start">
    <span class="m-t-10 color-[#eff1f5]" style="text-shadow: 2px 1px #737994;">
        <span class="fancy-headline-small">
           What if ...<br/> we create our own <br/>
        </span>
        <span class="fancy-headline">
            File System?
        </span>
    </span>
</div>

---
layout: image
image: /linux_vfs.excalidraw.svg
backgroundSize: 60%
---

<v-clicks>

<Arrow color="red" x1="750" y1="105" x2="700" y2="305" />

</v-clicks>

---
layout: image
image: /eww_brother.jpg
backgroundSize: 100%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/displeased-man-refusing-stretching-hand-grey-wall_8357236.htm#fromView=search&page=3&position=44&uuid=0d989872-e200-4027-a513-1dd4035a9a91">Image by cookie_studio</a> on pixabay
</div>

---
layout: image
image: /linux_vfs_fuse.excalidraw.svg
backgroundSize: 60%
---

<v-clicks>

<Arrow color="red" x1="850" y1="105" x2="750" y2="150" />

</v-clicks>

---

<span class="text-size-7xl">
    What about OCI Hooks?
</span>

---
layout: section
---

```go {|2|3|4|5|}
type Hook struct {
	Path    string
	Args    []string
	Env     []string
	Timeout *OptionalInt
}
```

---
layout: section
---

```go {|4}
type Hooks struct {
	Prestart        []*Hook
	CreateRuntime   []*Hook
	CreateContainer []*Hook
	StartContainer  []*Hook
	Poststart       []*Hook
	Poststop        []*Hook
}
```

---
layout: image
image: /create_container_hook.png
backgroundSize: 90%
---

<div class="attribution">
    <a href="https://github.com/opencontainers/runtime-spec/blob/v1.2.0/config.md#createContainer-hooks">Link</a> to docs
</div>

---

<span class="text-size-7xl">
    For now just file injection
</span>

---
layout: image
image: /beesting_pocv1.excalidraw.svg
backgroundSize: 90%
---

<v-switch>

<template #1>
    <Arrow x1="740" y1="210" x2="590" y2="270" color="red" />
</template>

<template #2>
    <Arrow x1="740" y1="310" x2="590" y2="210" color="red" />
</template>

<template #3>
    <Arrow x1="400" y1="310" x2="400" y2="410" color="red" />
</template>

<template #4>
    <Arrow x2="400" y2="310" x1="400" y1="410" color="red" />
</template>

<template #5>
    <span class="absolute top-40 right-25 text-align-center color-red">mount /tmp/beesting<br/> as hostPath volume</span>
    <Arrow x1="740" y1="210" x2="580" y2="275" color="red" />
</template>

</v-switch>

---
layout: fact
---

<span class="text-size-5xl line-height-13">beesting-hook is embedded into Beesting</span>

---
layout: full
class: code-fill code-small-font
---

```shellsession
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.024708ms
You can also run [skaffold run --tail] to get the logs
```

---
layout: full
class: code-fill code-small-font
---

```shellsession
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.024708ms
You can also run [skaffold run --tail] to get the logs

$ k apply -f HACK/dummy.yaml
deployment.apps/dummy created
```

---
layout: full
class: code-fill code-small-font
---

```shellsession
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.024708ms
You can also run [skaffold run --tail] to get the logs

$ k apply -f HACK/dummy.yaml
deployment.apps/dummy created

$ k get pods
NAME                        READY   STATUS              RESTARTS     AGE
beesting-agent-q9s2d        1/1     Running             0            54s
dummy-8984df79-zpvnm        0/1     RunContainerError   2 (4s ago)   20s
```

---
layout: full
class: code-fill code-small-font
---

```shellsession {|19}
$ k describe pod dummy-8984df79-zpvnm
Name:             dummy-8984df79-zpvnm
Namespace:        default
Priority:         0
Service Account:  default
Node:             beestinger-control-plane/172.18.0.3
...
Events:
Type    Reason  Age              From     Message
----    ------  ----             ----     -------
...
Warning Failed  8s (x4 over 49s) kubelet  Error:   failed  to   create  containerd
                                          task:  failed to  create shim  task: OCI
                                          runtime   create  failed:   runc  create
                                          failed:   unable   to  start   container
                                          process:  error  during container  init:
                                          error   running   hook   #1:   fork/exec
                                          /tmp/beesting/beesting-hook:
                                          permission denied: unknown
```

---
layout: section
class: code-small-font
---

```shellsession
Error:  failed  to   create  containerd  task:  failed  to   create  shim  task:
OCI  runtime  create failed:  runc  create  failed:  unable to  start  container
process:  error  during  container  init:   error  running  hook  #1:  fork/exec
/tmp/beesting/beesting-hook: permission denied: unknown
```

---
layout: section
class: code-small-font
---

```shellsession {|9}
$ root@beestinger-control-plane:/# mount
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
...
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k,inode64)
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noexec,relatime,inode64)
/dev/vda1 on /var type ext4 (rw,relatime,discard,errors=remount-ro)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k,inode64)
...
```
---
layout: image
image: /istio.png
backgroundSize: 90%
---

<div class="w-full h-full flex flex-justify-end flex-items-start color-red">
    <span class="p-r-30 p-t-20">/opt/cni/bin</span>
</div>

<Arrow x1="750" y1="150" x2="540" y2="245" color="red" />

<div class="attribution">
    <a href="https://github.com/istio/istio/blob/1.24.2/cni/pkg/install/install.go#L57-L64">cni/pkg/install/install.go</a>
</div>

---
layout: image
image: /beesting_pocv1.excalidraw.svg
backgroundSize: 90%
---

<span v-click>
    <span class="absolute top-40 right-30 text-align-center color-red">extracted to <br/> /opt/beesting/bin</span>
    <Arrow x1="720" y1="210" x2="580" y2="275" color="red" />
</span>

<span v-click.hide="1">
    <span class="absolute top-40 right-30 text-align-center color-red">extracted to <br/> /tmp/beesting</span>
    <Arrow x1="740" y1="210" x2="580" y2="275" color="red" />
</span>

---
layout: full
class: code-fill code-small-font
---

```shellsession {|16}
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.024708ms
You can also run [skaffold run --tail] to get the logs

$ k delete -f HACK/dummy.yaml
deployment.apps/dummy deleted

$ k apply -f HACK/dummy.yaml
deployment.apps/dummy created

$ k get pods
NAME                        READY   STATUS              RESTARTS     AGE
beesting-agent-q9s2d        1/1     Running             0            30s
dummy-8984df79-zpvnm        1/1     Running             0            52s
```

---
layout: full
class: code-fill
---

```shellsession
$ k exec deploy/dummy -it -- ls -alh /var/run/secrets/eks.amazonaws.com/
total 12K
drwxr-xr-x    2 root     root        4.0K Jan  6 15:11 .
drwxr-xr-x    4 root     root        4.0K Jan  6 15:11 ..
-r--r--r--    1 root     root          16 Jan  6 15:11 access_key_token
```
---
layout: fact
class: bg-[#7287fd] color-[#e5c890]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #737994;">
    Too much complexity
</span>

---
layout: section
class: bg-[#e64553] color-[#eff1f5]
---

<span class="text-size-2xl" style="text-shadow: 2px 1px #737994;">
Next try <br/>
</span>

<span class="text-size-5xl" style="text-shadow: 2px 1px #737994;">
    Replace hook with bind mount
</span>

---
layout: image
image: /bind_mount.excalidraw.svg
backgroundSize: 90%
---

---
layout: image
image: /beesting_pocv2.excalidraw.svg
backgroundSize: 90%
---

<v-switch>

<template #1>
    <Arrow x1="400" y1="310" x2="400" y2="410" color="red" />
</template>

<template #2>
    <Arrow x1="750" y1="260" x2="610" y2="310" color="red" />
</template>

<template #3>
</template>

</v-switch>

---
layout: full
class: code-fill code-small-font
---

```shellsession {|16}
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.024708ms
You can also run [skaffold run --tail] to get the logs

$ k delete -f HACK/dummy.yaml
deployment.apps/dummy deleted

$ k apply -f HACK/dummy.yaml
deployment.apps/dummy created

$ k get pods
NAME                        READY   STATUS              RESTARTS     AGE
beesting-agent-x2g8w        1/1     Running             0            30s
dummy-8984df79-isz1s        1/1     Running             0            52s
```

---
layout: full
class: code-fill
---

```shellsession {|5}
$ k exec deploy/dummy -it -- ls -alh /var/run/secrets/eks.amazonaws.com/
total 12K
drwxr-xr-x    2 root     root        4.0K Jan  6 16:51 .
drwxr-xr-x    4 root     root        4.0K Jan  6 16:51 ..
-r--r--r--    1 root     root          16 Jan  6 16:51 access_key_token
```

---
layout: image
image: /great_success.png
backgroundSize: 80%
---

---
layout: image
image: /monitoring.jpg
---

<div class="w-full h-full flex flex-justify-end flex-items-start text-align-center">
    <span class="m-t-30 color-[#e64553]" style="text-shadow: 2px 1px #bac2de;">
        <span class="fancy-headline-small">
            How do we detect <br/>
        </span>
        <span class="fancy-headline">
            File access?
        </span>
    </span>
</div>

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/man-got-surprised-while-looking-through-magnifying-glass-saying-wow-awesome-product-standing_39673215.htm#fromView=search&page=2&position=31&uuid=b032d6d2-c816-43b2-93e1-81d01a4d49a8">Image by benzoix</a> on Freepik
</div>


---
layout: image
image: /ebpf_so_hot_meme.png
backgroundSize: 50%
---
---
layout: image
image: /EBPF_logo.png
backgroundSize: 80%
---
---
layout: image
image: /ebpf_comic.png
backgroundSize: 90%
---

<div class="attribution">
    eBPF Comic by Philipp Meier and Thomas Graf
</div>

---
layout: image
image: /ebpf_source_to_vm.excalidraw.svg
backgroundSize: 80%
---
---

<span class="text-size-7xl">
    Unbounded Loops
</span>

<v-click>
    <div class="absolute top-0 bottom-0 right-0 left-0 text-align-center p-20">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%" preserveAspectRatio="meet">
          <line x1="20" y1="20" x2="80" y2="80" stroke="red" stroke-width="10"/>
          <line x1="80" y1="20" x2="20" y2="80" stroke="red" stroke-width="10"/>
        </svg>
    </div>
</v-click>

---
layout: image
image: /verifier_approved.jpg
backgroundSize: 80%
---
<div class="attribution">
    <a href="https://www.freepik.com/free-photo/close-up-woman-with-watch_21768055.htm#fromView=search&page=1&position=28&uuid=a87f4551-9928-476a-9f10-f078c92b901e">Image by Freepik</a> on Freepik
</div>

---
layout: image
image: /map_architecture.png
backgroundSize: 70%
---

---
layout: section
---

# Map Types

<div class="text-size-4xl">

* HashTable, Arrays
* LRU (Least Recently Used)
* Perf and Ring Buffer
* ...

</div>

---
layout: image
image: /helper.png
backgroundSize: 90%
---

---
layout: section
---

# Helpers

<div class="text-size-4xl">

* bpf_get_current_pid_tgid
* bpf_map_lookup_elem
* bpf_map_delete_elem
* ...

</div>

---
layout: fact
---

<span>
    What is a file?
</span>

---
layout: image
image: /fs_basic.excalidraw.svg
backgroundSize: 80%
---

---
layout: section
---

<div class="text-size-5xl">

* open
* openat
* symlinks
* ...

</div>

---
layout: image
image: /choice_overload.jpg
backgroundSize: 80%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/frustrated-unhappy-man-cries-with-despair-expresses-negative-emotions-wears-sticky-notes-around-whole-body-head-poses-indoor-against-pink-wall-has-dejected-miserable-expression_13577814.htm#fromView=search&page=1&position=8&uuid=6e0e6875-60f9-49d9-930f-e418fd181715">Image by wayhomestudio</a> on Freepik
</div>

---
layout: fact
---

<span class="text-size-8xl">
    Linux Security Modules
</span>

---
layout: fact
---

<span>
    Beesting doesn't use LSM directly
</span>

---
layout: image
image: /lsm_illustration.excalidraw.svg
backgroundSize: 80%
---

<v-switch>

<template #1>
    <Arrow x1="300" y1="115" x2="400" y2="115" color="red" />
</template>

<template #2>
    <Arrow x1="300" y1="305" x2="400" y2="305" color="red" />
</template>

<template #3>
    <Arrow x1="800" y1="490" x2="700" y2="490" color="red" />
</template>

<template #4>
    <Arrow x1="190" y1="490" x2="290" y2="490" color="red" />
</template>

<template #5>
    <Arrow x1="800" y1="450" x2="800" y2="350" color="red" />
</template>

</v-switch>

---
layout: fact
---

<span>
    PoCv3 is based on PoCv2
</span>

---
layout: image
image: /beesting_pocv3.excalidraw.svg
backgroundSize: 95%
---

<v-switch>

<template #1>
    <Arrow x1="665" y1="310" x2="565" y2="310" color="red" />
</template>

</v-switch>

---
layout: image
image: /beesting_pocv3_monitoring.excalidraw.svg
backgroundSize: 70%
---

<v-switch>

<template #1>
    <Arrow x1="230" y1="205" x2="230" y2="305" color="red" />
</template>

<template #2>
    <Arrow x1="230" y1="505" x2="450" y2="405" color="red" />
</template>

<template #3>
    <Arrow x1="750" y1="55" x2="660" y2="160" color="red" />
</template>

<template #4>
    <Arrow x1="760" y1="510" x2="640" y2="510" color="red" />
</template>

</v-switch>

---
layout: full
class: code-fill code-small-font
---

```shellsession {|16}
$ skaffold run
...
Waiting for deployments to stabilize...
Deployments stabilized in 5.103810ms
You can also run [skaffold run --tail] to get the logs

$ k delete -f HACK/dummy.yaml
deployment.apps/dummy deleted

$ k apply -f HACK/dummy.yaml
deployment.apps/dummy created

$ k get pods
NAME                   READY   STATUS    RESTARTS   AGE
beesting-agent-svpzl   1/1     Running   0          20s
dummy-8984df79-kddjh   1/1     Running   0          13s
```

---
layout: full
class: code-fill code-small-font
---

```shellsession {|5}
$ k exec deploy/dummy -- ls -alh /var/run/secrets/eks.amazonaws.com/
total 12K
drwxr-xr-x    2 root     root        4.0K Jan 11 09:14 .
drwxr-xr-x    4 root     root        4.0K Jan 11 09:14 ..
-rw-r--r--    1 root     root          16 Jan 11 09:14 access_key_token
```

---
layout: full
class: code-fill code-small-font
---

```shellsession
$ k exec deploy/dummy -- ls -alh /var/run/secrets/eks.amazonaws.com/
total 12K
drwxr-xr-x    2 root     root        4.0K Jan 11 09:14 .
drwxr-xr-x    4 root     root        4.0K Jan 11 09:14 ..
-rw-r--r--    1 root     root          16 Jan 11 09:14 access_key_token

$ k exec deploy/dummy -- cat /var/run/secrets/eks.amazonaws.com/access_key_token
2yWeFNuHzb7wUw==
```

---
layout: full
class: code-fill code-small-font
---

```shellsession
$ k exec deploy/dummy -- ls -alh /var/run/secrets/eks.amazonaws.com/
total 12K
drwxr-xr-x    2 root     root        4.0K Jan 11 09:14 .
drwxr-xr-x    4 root     root        4.0K Jan 11 09:14 ..
-rw-r--r--    1 root     root          16 Jan 11 09:14 access_key_token

$ k exec deploy/dummy -- cat /var/run/secrets/eks.amazonaws.com/access_key_token
2yWeFNuHzb7wUw==

$ k logs ds/beesting-agent --tail=2
time=2025-01-11T09:14:37.455Z level=DEBUG msg="watch inode" token.Inode=92 token.Dev=78
2025-01-11T09:23:17Z 💥 Honey Token Access Detected!
    Pod: default/dummy-8984df79-kddjh
    Container: dummy-pod,
    PID: 379512,
    StartTime: 584352740276296
    Comm: cat
```

---
layout: image
image: /great_success.png
backgroundSize: 80%
---

---
layout: section
---

## Further research

<span class="text-size-5xl">File injection based on PoCv1</span>

---
layout: image
image: /beesting_pocv1_monitoring.excalidraw.svg
backgroundSize: 90%
---

<v-click>
    <span class="absolute top-90 left-10 text-align-center text-size-3xl">
        <span class="color-green">
            Would survive node restart<br/>
        </span>
        <span class="color-red">
            How to secure it?
        </span>
    </span>
</v-click>

---
layout: section
---

## Further research

<span class="text-size-5xl">
    Hook read operations and scan for pattern
</span>

---
layout: fact
---

<span class="text-size-5xl">Very flexible</span>

---
layout: fact
---

https://github.com/patrickpichler/beesting/

---
layout: fact
---

https://patrickpichler.dev

---
layout: image
image: /blog.png
backgroundSize: 80%
---
---
