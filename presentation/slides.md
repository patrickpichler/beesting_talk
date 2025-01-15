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
layout: fact
class: bg-[#dd7878] color-[#eff1f5]
---

<span style="text-shadow: 2px 1px #181926;">
<span class="text-size-6xl">
    Let me take you back to<br/>
</span>
<span class="text-size-7xl" >
    August 2024
</span>
</span>

<!--
* voice call with my colleague markus
* we were discussing how to improve detection capabilities of our agent
* he brought up a blog post of his and few former dynatrace coworkers
-->

---
layout: image
image: /dyantrace_blog.png
backgroundSize: 80%
---

<div class="attribution">
    <a href="https://www.dynatrace.com/news/blog/context-aware-security-incident-response/">Link</a> to blog
</div>

<!--
* outlined a way of detecting container breaches by injecting honeytokens into pods and monitoring them
-->

---
layout: image
image: /dynatrace_diagram.excalidraw.svg
backgroundSize: 80%
---

<!--
* they achieved this by using kyverno to inject the honeytoken
* tetragon for monitoring its access
* i fell in love with the idea
* sounds simple on paper
* highly effective
* only issue was two additional components are required
* i was thinking to myself there has to be an easier way
* after searching the internet for some time, i didn't find anything
* hence the idea of beesting was born
* before i will tell you about my journey, we should answer the question of what even is a honeytoken
-->

---
layout: image
image: /honeytoken.jpg
backgroundSize: 100%
class: color-[#eff1f5]
---

<div class="h-full" style="display: flex; justify-content: end; flex-direction: column">
    <span id="headline">
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

<!--
* put simply, honeytokens are digital bait
-->

---
layout: fact
class: bg-[#8bd5ca] color-[#e64553]
---

# Digital bait

<!--
* they are placed into locations one would expect high value token
* meaning a potential attacker will try to access them
* a good target for a honeytoken would be kubernetes SA tokens
* to better understand the usefulness of this all
-->

---
layout: image
image: /hacker.jpg
---

<span class="text-size-4xl absolute top-30 right-55 text-align-center color-red">This is us</span>
<Arrow x1="600" y1="150" x2="480" y2="200" color="red" />

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/side-view-male-hacker-with-laptop_8725474.htm#fromView=search&page=1&position=18&uuid=e4c7a153-84c8-4550-87fc-1e259fa8afb5">Image by freepik</a> on Freepik
</div>

<!--
* we will put on ski mask and hoddi and try viewing what we just learned from the lense of an evil hacker
* we are surfing the internet and stumble across this public facing CMS by some random company
-->

---
layout: image
image: /cms.png
backgroundSize: 80%
---

<!--
* after some fiddling around, we figured out that it is running on top of GetSimple CMS
* a quick trip to exploit-db later and we found a pretty severe RCE CVE we can exploit
-->

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

<!--
* we now have a shell to the machine running the CMS
* since it is common to inject secrets via env variables, we first check the output of the `env` command for tokens
-->

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

<!--
* no secrets, but large amount of env variables with `KUBERNETES_` prefix
* suggests we are running inside a kubernetes pod
-->

---
layout: full
class: code-fill
---

```shellsession
$ root@simpecms-...:/root# ls -al /var/run/secrets/
```

<!--
* next step is to check `/var/run/secrets` as it is common to mount secrets there in kubernetes environments
-->

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

<!--
* we got lucky, there are secrets
* interesting lookin directory called `eks.amazonaws.com`
* let us check its content
-->

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

<!--
* now the `acccess_key_token` file looks as it would contain a sensitive token
-->

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

<!--
* we are going to use `cat` now to extract the secret
-->

---
layout: image
image: /trap_card_meme.png
backgroundSize: 60%
---

<!--
* what we didn't know, file was injected by security team
* accessing it activated their trap card, marked the pod as compromised and triggered the incident response team
* this is the power of Honeytokens
-->

---
layout: image
image: /beesting_overview.excalidraw.svg
backgroundSize: 80%
---

<!--
* here is how i envision all of this on a high level
* beesting should be a single component injecting honeytokens into pods and finally watching them for access
* to get an better understanding how to implement something like this, we first need to understand the architecture of kubernetes a bit better
-->

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

<!--
* the following explanation is a simplification of course, but it should be good enough for us to gain a better understanding
-->

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

<!--
* Kubernetes cluster can be separated into two parts
* control plane, which is where ETCD and API server live and data plane
* if you submit an app to the cluster, it will run on nodes in the data plane
* nodes are usually linux machines
-->

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

<!--
* this is where the kubelet runs
* kubelets job is to figure out what needs to be run on the node by talking to the API server and make it happen by instruct the container runtime, which is most often containerD, to start whatever needs to be run.
* container runtime will download whatever image was specified in the container specs and instruct low level container runtime, such as runc, to spawn the containers
* linux doesn't really have the concept of containers though
* made up of two components: cgroups, namespaces
-->

---
layout: section
---

## Cgroups

<span class="text-size-5xl">Limit resource usage (CPU, Memory)</span>

<!--
* cgroups, or control groups, allow to limit the resource of processes.
* examples of such resources would be CPU or memory
-->

---
layout: section
---

## Namespaces

<span class="text-size-5xl">Separate areas (Networking, Mounts)</span>

<!--
* namespaces on the other hand allow to segregate areas such as networking or file system mounts. for example, each network in a network namespace can only see the network devices associated with the namespace.
-->

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

<!--
* let us have a look how the dynatrace gang achieved this in their blog post
* they are using kyverno as a mutating admission webhook, that injects the honeytoken file via a project volume mount
* sounds simple, but those are fancy words we will go over them 1 by 1
* kyverno is a policy engine, that allow us impose rules on any resource submitted to kubernetes, as well as mutating them
* the mutating part is achieved by a kubernetes extension point called admission webhook, defined webhooks are called before a resource, e.g. a pod, is persisted into ETCD
* a projected volume mount allows us to mount a secret/config map at a specified location in the pods file system
* to rephrase: the blog explains how to use kyverno to inject a file backed by a secret into our pod before it is persisted
-->

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

<!--
* here an example how the pod yaml will look like after the webhook is done with it
* there is a volume named honeytoken, that references a secret
* there is a volume mount binding a value from the secret into the pods filesyste
* if we would print the content of the file inside the pod, we will see it is same as whatever the value in the secret for `token` is
-->

---
layout: image
image: /beesting_overview_webhook.excalidraw.svg
backgroundSize: 80%
---

<!--
* for beesting i could have followed the same approach, but instead of kyverno expose the mutating webhook from the service itself
* injecting is only one part of the bargain though, we also need to monitor
* could have split beesting into two parts, one injecting the files into the pods, the other being deployes as a daemon set on all the nodes to handle monitoring
* didn't like this solution too much though
* first of all, not the super simple one component solution i envisioned. in theory all agents could expose the admission webhook api, but idk, felt wrong
-->

---
layout: fact
class: bg-[#e5c890] color-[#7287fd]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #737994;">
    Lots of moving parts
</span>

---
layout: fact
class: bg-[#7287fd] color-[#e5c890]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #737994;">
    Boring
</span>

<!--
* additionally, it sounded a bit... boring
* i started beesting as a project to research some new approaches of doing things. using an mutating webhook just didn't get me exited.
* i work on beesting in my free time, so i will focus on whatever sounds exciting to me first
* maybe i explore this path in the future
* for now admission controller is off the hook
* how else can we inject our sweet honeytokens then?
-->

---
layout: image
image: /kubernetes_overview_node.excalidraw.svg
backgroundSize: 100%
---

<v-clicks>
<Arrow x1="800" y1="20" x2="660" y2="130" color="red" />
</v-clicks>

<!--
* looking back at the architecture of a kubernetes node, we learned all pods and containers are created by kubelet calling the container runtime.
* as mentioned it is containerd in like 99% of the cases
* i do not have absolute numbers, but i never encountered a different runtime before
* does maybe containerd offer some way of plugin mechanism we could leverage?
-->

---
layout: image
image: /containerd_plugins.png
backgroundSize: 90%
---

<!--
* it turns out yes, containerd does have plugins
* some notable examples are runtime plugins
-->

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

<!--
* runtime plugins allow to add support for custom low level runtime such as runC
* differ plugins, used to calculate the difference between two mount points and figure out how to apply them
* and a few more
-->

---
layout: fact
class: bg-[#ef9f76] color-[#1BA0E4]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #e6e9ef;">
 Differ Plugins
</span>

<!--
* differ plugins did sound like something we can use to inject our files
-->

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

<!--
* having a closer look at the api we need to implement unveals it is maybe more of a hack
-->

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

<!--
* we get a diff request with two mount points that needs diffing
* in theory we can somehow inject our honeytoken there, but this doesn't feel right
* during research of containerd plugins, i stumbled on another better solution though
-->

---
layout: fact
class: bg-[#f9e2af] color-[#7287fd]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #626880;">
    Node Resource Interface
</span>

<!--
* the node resource interface or NRI for short
* NRI allows us to extend OCI compatible runtimes with domain specific custom logic
* thinking about what we want to do, injecting honeytokens is domain specific custom logic in a way
* NRI also keeps plugins runtime agnostic, meaning we can support more than just containerD
* currently only containerD and CRI-O support NRI though
* starting with containerD v2, NRI is even enabled by default
-->

---
layout: image
image: /nri_overview.excalidraw.svg
backgroundSize: 90%
---

<!--
* NRI works by exposing a GRPC API through a unix domain socket
* the api can be used to register plugins
-->

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

<!--
* plugins expose a variety of different callbacks to implement
* including `CreateContainer` `UpdateContainer` and a various state change events like `PostCreateContainer` or `RunPodSandbox`
* Callback such as `CreteContainer` grant us special powers to modify the container spec before containerD will hand it over to the low level runtime
-->

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

<!--
* this includes file mounts, environment variables and hooks
* the hooks in this case are OCI hooks and provide a standardized way of running commands during various stages of a container lifecycle. we will hear more about them later
* awesome NRI sounds exactly what we need, now we need to figure out, what to inject into the container
-->

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

<!--
* honeytokens in our case are just files
-->

---
layout: fact
class: bg-[#df8e1d] color-[#232634]
---

<span class="text-size-7xl" style="text-shadow: 2px 1px #e6e9ef;">
    Honeytokens = Files
</span>

<!--
* all files in linux must exist on a file system
* there is a variety of different filesystems supported by linux
-->

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

<!--
* all of them implement a common interface called, the virtual file system
-->

---
layout: image
image: /linux_vfs.excalidraw.svg
backgroundSize: 60%
---

<!--
* it provides a common set of functions used to communicate with the underlying datastores
* this sounds pretty interesting,
-->

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

<!--
* what if we try to create our own custom file system and mount it into the location we want the honeytoken to be in
-->

---
layout: image
image: /linux_vfs.excalidraw.svg
backgroundSize: 60%
---

<v-clicks>

<Arrow color="red" x1="750" y1="105" x2="700" y2="305" />

</v-clicks>

<!--
* this would also make monitoring easier, as we control all interactions with the filesystem
* there is one catch though, in order to make our custom file system available to linux, we must install it via a kernel module
* installing kernel modules takes higher privileges
* kernel modules are also quite dangerous, as a bug in our file system can in theory bring down the whole machine
-->

---
layout: image
image: /eww_brother.jpg
backgroundSize: 100%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/displeased-man-refusing-stretching-hand-grey-wall_8357236.htm#fromView=search&page=3&position=44&uuid=0d989872-e200-4027-a513-1dd4035a9a91">Image by cookie_studio</a> on pixabay
</div>

<!--
* this is definitely not something we want
-->

---
layout: image
image: /linux_vfs_fuse.excalidraw.svg
backgroundSize: 60%
---

<v-clicks>

<Arrow color="red" x1="850" y1="105" x2="750" y2="150" />

</v-clicks>

<!--
* another way of implementing custom file systems would be with the help of libfuse
* it provides a user space library used to implement the methods required by the VFS
* sadly, as with kernel modules, libfuse requires higher privilges
* additionally, it would add a hard dependency on libfuse being present on all nodes
* the custom file system idea is out of the window then
-->

---

<span class="text-size-7xl">
    What about OCI Hooks?
</span>

<!--
* but what about those OCI hooks we were hearing before
* as we learned, they allow us to run some sort of command at various stages of the container lifecycle
* the definition of a hook looks as follows
-->

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

<!--
* a path pointing to whatever binary we want to run
* some arguments passed to the executing binary
* some environment variables to be set
* a time out the hook needs to finish before it being killed
* the path argument is a bit special though, as the context in which it evaluates is depending on what hook point you are use
-->

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

<!--
* the available hook points are `prestart` `createRuntime` `createContainer` and so on
* the create container hook will execute when the container is being created by the runtime, but before it has been started.
-->

---
layout: image
image: /create_container_hook.png
backgroundSize: 90%
---

<div class="attribution">
    <a href="https://github.com/opencontainers/runtime-spec/blob/v1.2.0/config.md#createContainer-hooks">Link</a> to docs
</div>

<!--
* according to the docs, the hook will resolve the file path in the runtime namespace, but will run the binary in the context of the container
* we can use this to customize the containers file system and inject our token
-->

---

<span class="text-size-7xl">
    For now just file injection
</span>

<!--
* as a quick side note, we only focus on honeytoken injection for now.
* monitoring will be tackled later on
* one step at a time
-->

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
    <Arrow x1="400" y1="310" x2="400" y2="410" color="red" />
</template>

<template #3>
    <Arrow x2="400" y2="310" x1="400" y1="410" color="red" />
</template>

<template #4>
    <Arrow x1="740" y1="310" x2="590" y2="210" color="red" />
</template>


<template #5>
    <span class="absolute top-40 right-25 text-align-center color-red">mount /tmp/beesting<br/> as hostPath volume</span>
    <Arrow x1="740" y1="210" x2="580" y2="275" color="red" />
</template>

</v-switch>

<!--
* the first version of beesting look as follows
* we extract our hook binary at startup at a specified location, that has to be reachable from the container runtime
* next we register a create container hook with the NRI
* once a container gets created and our hook gets called, we inject an OCI `createContainer` hook into the container spec to run our extracted hook, pass the target honeytoken path as the first argument and the content of the honeytoken file as the second
* the rest is up to the container runtime, which will call our hook at the right time, which then creates the honeytoken at the right location in the container
* sounds like a plan
* in theory it should be enough to mount `/tmp/beesting` as a `hostPath` volume and move the binary there somehow.
-->

---
layout: fact
---

<span class="text-size-5xl line-height-13">beesting-hook is embedded into Beesting</span>

<!--
* I decided to embedd the hook inside the beesting binary, as it will make it quite easy to extract during startup.
* additionally it helps with keeping the hook version compatible with whatever beesting version is running
* let's take the solution for a ride then
-->

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

<!--
* to ease development i was using `skaffold`, so to start beesting we will call `skaffold run`
-->

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

<!--
* this will deploy beesting as a daemonset to all nodes in the cluster
* next we create some dummy deployment. all it does is sleeping forever, so not too important
* ok deployment has been created, let's see if everything is running as it should
-->

---
layout: full
class: code-fill code-small-font
---

```shellsession {|13}
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

<!--
* oh no wth, the pod failed
* what does `kubectl describe` tell us
-->

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

<!--
* what?? permission denied?
* here is the full error again
-->

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

<!--
* it says runc failed to run our hook with a permission denied error??
* that doesn't make too much sense, since runC is running as root and we the executable flag is set on the hook binary
* does anyone have an idea what could be the issue here?
* i give you a hint, here is the output of` the `mount` command in the runtime namespace
-->

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

<!--
* do you see it?
* `/tmp` is mounted with the `NOEXEC` flag
* even if a binary on there has the executable flag, linux will refuse to run it
* this is a common sight in kuberentes nodes and server in general
* /tmp is writeable by the whole system, meaning hackers using it to extract and run payloads
* by preventing executing of anything from it, we make the life of hackers a bit more difficult
* for us this is bad news though
* after some discussions with a colleague about beesting, shoutout to samuel, he mentioned istio is doing something similar than beesting is trying to do
-->

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

<!--
* during installation it extracts a binary to `/opt/cni/bin`
-->

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

<!--
* i assume we should then be fine to extract to `/opt/bessting/bin`
* let's give it a shot
-->

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

<!--
* here we use the same example as before
* the dummy pod is running nice
* we can now exec into the container and use `ls` to check if the honeytoken has been injected
-->

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

<!--
* awesome, everything seems to be working
* i am not 100% happy with this solution though
* embedding the OCI hook and extracting it at startup, as well as keeping track of the different execution contexts adds up quite a lot of complexity
-->

---
layout: fact
class: bg-[#7287fd] color-[#e5c890]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #737994;">
    Too much complexity
</span>

<!--
* there has to be an easier way
* what if
-->

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

<!--
* we replace our OCI hooks with some sort of bind `mount`
-->

---
layout: image
image: /bind_mount.excalidraw.svg
backgroundSize: 90%
---

<!--
* as a quick refresher `bind` mount allow us to mount a file or directory from one location to another, while being accessible from both
* this means we can mount a file from the runtime namespace into the container
-->

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

<!--
* the architecture for PoCv2 hence changes as follows
* we still register a NRI plugin with the `CreateContainer` hook
* instead of injecting an OCI hook, we create a file named the same as the container ID, at some predefined location reachable by the container runtime and inject it via a bind mount
* that is pretty much it!
* let's see it in action
-->

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

<!--
* same setup as before
* container is running nice
* the price question is if the injection worked
* we exec into the container again and run `ls`
-->

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

<!--
* the file is there, everything looks good
* great success
-->

---
layout: image
image: /great_success.png
backgroundSize: 80%
---

<!--
* i personally like this approach
* it has less movable parts and is easy to follow
* with the file injection out of the way
-->

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

<!--
* how do we detect file access?
* now this is where the bee in beesting comes from
* we are going to use eBPF
-->

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

<!--
* for those who have never heard of ebpf, you can think about it to the linux kernel, what javascript is to the browser
* sounds weird, but is incredibly useful
-->

---
layout: image
image: /ebpf_comic.png
backgroundSize: 90%
---

<div class="attribution">
    eBPF Comic by Philipp Meier and Thomas Graf
</div>

<!--
* before ebpf, it was very hard to extend the linux kernel in a safe and secure way
* the only way you pretty much had were kernel modules, which as we have heard before, can crash your whole machine with a single bug
* not very secure
* ebpf changed this by introducing a secure and safe way of attaching small programs to predefined hook points, as well as just arbitrary kernel functions
-->

---
layout: image
image: /ebpf_source_to_vm.excalidraw.svg
backgroundSize: 80%
---

<!--
* ebpf is usually written in a subset of c, that is then compiled to a byte code
* the byte code is then loaded into the linux kernel and executed on a vm
* under the hood, the byte code will be JIT compiled to native code though
* to ensure the byte code is safe to run, it must pass the so called verifier
* it is the verifiers job to go over all code paths in the program and ensure nothing crazy is safe and sound
-->

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

<!--
* for example, if you try to have an unbounded loop in your ebpf program
* the verifier will reject it
* the reason being, that it is impossible to know if a loop will terminate, unless you solve the halting problem of course
* this means that when the kernel passes control to the ebpf program, it could stall the kernel forever
* not good
-->

---
layout: image
image: /verifier_approved.jpg
backgroundSize: 80%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/close-up-woman-with-watch_21768055.htm#fromView=search&page=1&position=28&uuid=a87f4551-9928-476a-9f10-f078c92b901e">Image by Freepik</a> on Freepik
</div>

<!--
* if the verifier gives its ok, we know that the program is safe to run
-->

---
layout: image
image: /map_architecture.png
backgroundSize: 70%
---

<!--
* to store state and share data, ebpf has the concept of maps
* maps are data structures, which can be accessed by both the ebpf program in kernel space, as well as from user space
* typical use cases for maps are, sharing events with userspace when for example a syscall has happened, passing configuration from userspace to eBPF, or even storing context to further enhance the collected data.
-->

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

<!--
* there is a variety of map types available, all solving a dedicated use case
* such as FIFO queues, LRU data storage or arrays
* there even exist a hash map
* since you cannot just call any function from ebpf, as it would be impossible for the verifier to ensure stability and safety
-->

---
layout: image
image: /helper.png
backgroundSize: 90%
---

<!--
* the concept of helpers was introduced
* in a nutshell, those are special functions the verifier knows about  that allow interaction with the kernel
-->

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

<!--
* for example, there are helpers to get the current PID of the process running
* as well as helpers to interact with maps
* and many more
* now that we have a basic understanding of what ebpf is and what it allows us to do, how can we now detect file access
* for this, we first will have a quick look at what even is a file
-->

---
layout: fact
---

<span>
    What is a file?
</span>

<!--
* once again spoiler alert, this will be a simplification, that still should give us a good enough understanding how things work
* as we have heard before, in linux all files live on a file system
* each file is represented by a so called inode
-->

---
layout: image
image: /fs_basic.excalidraw.svg
backgroundSize: 80%
---

<!--
* an inode consists of a unique number, which btw is just the an index in the inode table, as well as metadata, such as file type, permissions, owner and so on
* the inode also contains a pointer to the underlying block the real data of our file is stored in
* this is where the binary data for that one funny cat video is locate
* the monitoring part of file access is trickier than it sounds
* there are a multitude of ways to interact with files
-->

---
layout: section
---

<div class="text-size-5xl">

* open
* openat
* symlinks
* ...

</div>

<!--
* for example, we could hook the open syscall with ebpf, only to figure out that we will be missing any calls through `openat`
* additionally, it is common for attackers to create symlinks to bypass access restrictions and detections
* there are even ways to read files that do not call `open` at all
-->

---
layout: image
image: /choice_overload.jpg
backgroundSize: 80%
---

<div class="attribution">
    <a href="https://www.freepik.com/free-photo/frustrated-unhappy-man-cries-with-despair-expresses-negative-emotions-wears-sticky-notes-around-whole-body-head-poses-indoor-against-pink-wall-has-dejected-miserable-expression_13577814.htm#fromView=search&page=1&position=8&uuid=6e0e6875-60f9-49d9-930f-e418fd181715">Image by wayhomestudio</a> on Freepik
</div>

<!--
* so there are quite a few different options to watch out for
* luckily for us, there is this thing called Linux Security Modules or LSM for short
-->

---
layout: fact
class: bg-[#1e66f5] color-[#eff1f5]
---

<span class="text-size-8xl" style="text-shadow: 2px 1px #232634;">
    Linux Security Modules
</span>

<!--
* LSM can be used to evaluate permission of a process for certain predefined actions, such as opening files
-->

---
layout: fact
---

<span>
    Beesting doesn't use LSM directly
</span>

<!--
* we are not going to dive any deeper into LSM though as beesting doesn't use LSM directly, just the function it exposes for evaluation
-->

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

<!--
* it will look like as follows
* imagine we have a service called my-fancy-service, that is opening some file
* the linux kernel will now call the `security_file_open` function, this is the LSM hook btw, to verify if the process is allowed to open the file
* if the process has permissions, the kernel will go ahead and call the corresponding open function from the VFS
* what beesting is doing, is attaching a hook at the `security_file_open` function, to get notified if a file is being opened
* now to combine everything
-->

---
layout: fact
---

<span>
    PoCv3 is based on PoCv2
</span>

<!--
* pocv3 is based on v2 for file injection
* so the injection part did not really change
-->

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

<!--
* when the token file is created though, beesting will go ahead and record the inode number, as well as the device number of the file being injected
* this works, since inode and device number will not change when using bind mounds
-->

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

<!--
* the inode number and device number is then added to a ebpf hashmap, which is used by beestings ebpf program hooking `security_file_open` to see if one of our honeytokens has been accessed
* if such a case is detected, the ebpf program will send a message to userspace through a so called ringbuffer
* you can think of a ring buffer as a simple message queue, that ebpf programs can write to and the userspace can read from
* the usespace will then print out a warning to stdout, to let the cluster admin know
* as a word of warning though, just hooking `security_file_open` is not enough in a production ready implementation
* for the PoC it was good enough though
* now a quick demo
-->

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

<!--
* we run the same sample as before
* we can see the dummy pod is started
-->

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

<!--
* execing into the pod and using ls shows us that the honeytoken was injected
* nothing new here
-->

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

<!--
* now the interesting part, if we use cat to access the honey token file, beesting should log a warning to stdout
* so let's check beestings log file
-->

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

<!--
* and there you have it
* beesting did detect the file access
-->

---
layout: image
image: /great_success.png
backgroundSize: 80%
---

<!--
* Great success
* this is pretty much it
* i have a few other ideas i would love to explore further int he future, but didn't have time yet
-->

---
layout: section
---

## Further research

<span class="text-size-5xl">File injection based on PoCv1</span>

<!--
* the first would be to use pocv1 as the base
-->

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
    </span>
</v-click>

<!--
* the challenge would be how to communicate the inode an device number back to the beesting agent
* in theory it could be done via some sort of unix socket
* the nice part of this would be, that this will survive node restarts, as it lives in the containers file system
* the current version of beesting is dropping state on node restarts
-->

---
layout: section
---

## Further research

<span class="text-size-5xl">
    Hook read operations and scan for pattern
</span>

<!--
* another idea would be to hook all read operations and scan for the honeytoken pattern to occur
-->

---
layout: fact
---

<span class="text-size-5xl">Very flexible</span>

<!--
* this should be very flexible, at least in theory, as we do no longer just rely on inodes
* i guess we could also detect tokens being extracted from env variables though
* the downside is, that there are not only a lot of read variations, but is also quite expensive to run
-->

---
layout: fact
---

https://github.com/patrickpichler/beesting/

<!--
* in case you are interested in the source code, you can find it under this github repository
* overall it was quite the nice research opportunity and i learned quite a lot during my journey
-->

---
layout: fact
---

https://patrickpichler.dev

<!--
* i would also suggest you to check out my personal blog at patrickpichler.dev
-->

---
layout: image
image: /blog.png
backgroundSize: 80%
---

<!--
* is still looks a bit empty, but i am planning on transforming my research into articles i will put there, so stay tuned
* in case you have any question or simply want to discuss beesting, ebpf or linux, feel free to approach me later. i am more than happy to talk
* thanks
-->

---
