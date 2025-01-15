Hello there!

Welcome everyone! Let's get started.

Now first things first,  my name is Patrick Pichler and  I am currently employed
at  Cast.AI as  a senior  software  engineer, developing  a Kubernetes  security
product.

Originally I  started my  career as  a Java  Software Developer,  but everything
changed when I stumbled upon Linux and the cloud. This definitely transformed me
into a  full-on Linux  nerd.

At my day job, I am part of the Kubernetes Runtime Security team. We develop an
agent, that can detect anomalous  behavior in your workloads, called kvisor.

Let me take you back to August 2024.  I was in call with my colleague Markus and
we were discussing ways of improving our detection capabilities.

During the  discussion, he  brought up  a blog  post by  a few  former Dynatrace
colleagues and him,  in which they outlined a way  of injecting Honeytokens into
containers and monitoring them. The main  premisse was that if they could detect
somebody accessing  one of the  Honeytokens, they  would know that  something is
off. To achieve this, they are using Kyverno for File injection and Tetragon for
monitoring the access.

I immediately fell  in love with the  idea. It sounded simple on  paper and from
what I can tell,  it would be highly effective. My only  issue with the solution
outlined in  the blog is,  that it requires  the installation of  two additional
components into  the cluster.  Kyverno and  Tetragon. I  thought to  myself that
there has to be an easier way. After a bit of searching the internet, I couldn't
find anyone doing this  in a single component. This was the  moment the idea for
Beesting was born.

Before I  will tell  you about  my journey of  implementing beesting,  we should
answer the question of what even is a Honeytoken.

Put simply,  you can think of  a Honeytoken as  digital bait. They are  put into
places where one would usually expect tokens of high value, meaning an potential
attacker  would access  those files.  A pretty  good example  for this  would be
Kubernetes service account tokens.

To better understand the usefulness of all of this, we will put on our Ski Masks
and Hoodie,  to see the  power of Honeytokens from  an evil hackers  eyes. After
searching the  internet for  potential targets, we  stumble across  this company
website running a simple  CMS. Some quick probing later, we  figured out that it
is in  fact SimpleCMS. A quick  trip to exploit-db  later and we found  a pretty
severe RCE  vulnerability we  can exploit. We  now have a  shell to  the machine
running  the  CMS.  Since  it  is common  to  inject  secrets  via  envisionment
variables, we check  the output of the  `env` commands for any  tokens of value.
Hmm no  secrets, but  there is  a large  number of  env variables  starting with
`KUBERNETES_`. This highly suggests the CMS  is running inside a Kubernetes Pod.
Next step, would be to see if  we can find any secrets under `/var/run/secrets`.
This is  a common location  to mount service account  tokens and any  other file
based  secrets.  And  we  got  lucky  it  seems,  we  find  a  directory  called
`/var/run/secrets/eks.amazonaws.com`. Let us check it's content. Oho the `access_key_token`
file sounds promising. We are going to use `cat` now to extract the secret.

What we  didn't know, the  file accessed  was a Honeytoken  put in place  by the
security team  of the  company hosting  the CMS.  By accessing  it, the  pod was
marked as  compromised and  the incident  response team  was informed  about our
presence. This is the power of Honeytokens.

Here we can see how I would envision  this on a high level. We see `Beesting` as
a  single component  injecting  Honeytokens  into pods  and  monitoring the  for
access.

To get  an understanding how  we could implement  something like this,  we first
need to  understand the architecture  of Kubernetes  a bit better.  A Kubernetes
cluster can be  separated into two parts  the control plane, this  is where ETCD
and the API  server runs, and the data  plane. Any app we submit  to the cluster
will  run on  nodes, which  are part  of the  data plane.  Kubernetes nodes  are
usually Linux machines, where the Kubelet runs.

It is kubelets job  to communicate with the API server to  figure out what needs
to  be running  on  it node.  Pods  are  then started  by  communicating with  a
Container Runtime, which in most cases will be containerD. The Container runtime
then download  whatever image was specified  in the container spec  and spawn it
via a low level runtime such as runC. Linux doesn't really have the concept of a
container. They are pretty much made  out of control groups, also called Cgroups
and  namespaces. Cgroups  allow to  limit the  resource usage,  such as  CPU and
memory, whereas with namespaces we can separate areas such as networking or file
mounts.

Awesome,  with this  out of  the way,  how  do we  get the  Honeytoken into  the
containers?

Let us have a  look at how the Dynatrace Gang achieved this  in their blog post.
They  used  Kyverno  as  a  mutating admission  webhook,  that  did  inject  the
Honeytoken file via a projected volume mount. Ok sounds easy, but those are some
fancy words. To refresh  our knowledge, we will go over those  words one by one.
What a  Honeytoken is, we already  learned earlier. Kyverno is  a policy engine,
that  acts as  a  webhook in  Kubernetes.  Webhooks are  an  extension point  in
Kubernetes  to allow  us  to  modify any  resources  submitted  before they  are
persisted into  ETCD. A projected  volume mount allows us  to mount a  secret or
config map at a specified location in  the filesystem of our pod. So to rephrase
that complicated  sounding sentence from before,  the blog post explains  how to
use Kyverno to inject a file into our pod before it is persisted.

Here an example of how the pod will look like after the webhook is done with it.
You  can see  that there  is a  `volume` named  `honeytoken`, that  references a
secret. This is our projected volume.  Also, there is a `volumeMount`, binding a
value  of the  referenced secret  at  a specific  path.  If we  would enter  the
container and have a look inside the file specified by `mountPath`, we would see
whatever value is set for the referenced key in the secret.

For Beesting,  I could also  have followed the  same approach. Instead  of using
Kyverno as the  mutating webhook, Beesting could have just  exposed the required
API.  There are  several reasons  I decided  against that  approach though.  The
biggest was that injecting the Honeytoken is  only half the bargain. So either I
expose  the mutating  webhook API  from all  agents, or  I would  need to  split
Beesting into two parts. This means my  initial goal of a single component would
be out  of the  window. Additionally, it  just didn't sounded  that much  fun. I
wanted  to start  Beesting  to research  into  something new  and  just using  a
mutating webhook  didn't get me  that exited. Maybe in  the future, I  give this
approach another shot  though, but since I  work on Beesting in my  free time, I
will spend my time on whatever looks fun first probably.

Ok, so  the Admission Controller is  off the table.  How else can we  inject our
sweet Honeytokens into containers? Looking  back at the Kubernetes architecture,
containers are  created by the Container  Runtime. As mentioned, in  most cases,
this runtime will be containerD. I do  not have exact numbers, but it feels like
99% of  the Kubernetes installations  out there run containerD.  Does containerD
offer  some kind  of plugin  mechanism  we could  leverage here?  It turns  out,
containerD  does have  plugins. Some  of plugin  types include  runtime plugins,
which allow  you to  add support for  your own low-level  runtime such  as runC,
differ plugins, in which you can calculate the differences are between two mount
points in  containers and how to  apply them, as  well as many more.

The  differ  plugin  did sound  as  it  would  allow  us to  inject  files  into
containers, the way we want. Though if we have a closer look at the interface we
would need  to implement,  it would  feel like quite  the hack.  We would  get a
`DiffRequest`  from  containerD,  that  contains  two  mount  points  we  should
calculate the  diff for.  I guess we  could in theory  inject our  token somehow
there, but it feels wrong.

During research  of containerD plugins,  I also stumbled  on a thing  called the
node resource  interface or NRI for  short. The main goal  of the NRI is  to add
domain specific custom logic to  OCI compatible runtimes. This sounds promising!
Honeytoken injection is domain specific custom logic  in a way. The docs for the
NRI  also mention  that it  keeps  the plugins  runtime agnostic  and that  both
containerD and CRI-O  are currently supporting it. Starting  with containerD v2,
the NRI plugin is even enabled by  default. This would be amazing, as this would
allow us to use Beesting with the  two most widely spread container runtimes and
maybe even more in  the future! NRI works by exposing a GRPC  API through a unix
socket, that can be used to register  plugins with it. The NRI API exposes quite
a  few different  callbacks  such as:  `CreateContainer`, `UpdateContainer`  and
various  state change  events  like  `PostCreateContainer` or  `RunPodStandbox`.
Callbacks, such  as `CreateContainer` grant us  the power to modify  anything we
want of the container  before it will be handed over  to runC, including mounts,
as well as so  called OCI hooks. OCI hooks are a  standardized way of specifying
commands that runC  will execute at certain stages when  running a container. We
will hear more about them later.

Awesome, NRI  sounds exactly  what we  need. We  only need  to figure  out, what
exactly do we even inject into the container?

Honeytokens, in our case, are just files.  All files in Linux must exist on file
systems. There is a variety of different file systems supported by Linux. All of
them  implement certain  functions  specified  by the  Virtual  File System.  It
defines methods on how  to interact with the file system in  a generic way.

What if we  somehow create our own  custom file system, which we  can mount into
the container at the location we want  the Honeytoken to be in? This should also
easily let us detect  if somebody tries to read our Honeytoken,  as they need to
go through  the methods we  define to  be part of  the VFS API.  Sounds amazing!
There is one catch though. In order to  make a VFS available to a Linux machine,
they need to be installed via a Kernel Module. Not only does this require higher
privileges, it  also is  quite dangerous,  as a  simple bug  in our  file system
implementation can  bring down the  whole machine.  Not something that  we want.
Besides Kernel Modules, there is also  the possibility to use FUSE, which stands
for 'Filesystem  in Userspace'.  Like Kernel Modules  though it  requires higher
privileges to install them,  plus it would add a hard  dependency on having FUSE
installed on all the nodes. The custom file system idea is not really a good fit
and out of the window.

Hmm we were hearing about those OCI  hooks before. Maybe we can leverage them to
create  the Honeytoken  in  the container?  OCI  hooks allow  us  to run  custom
commands at  various stages of  the container  lifecycle. Those hooks  should be
supported by any  OCI compatible low level runtime, which  runC is. They consist
of an absolute  path to the binary that  needs to be run, a set  of arguments to
pass to  the hook, as  well as some  env variables and  timeout for the  hook to
finish before being killed. The context of  the path argument can either be from
the point of view of the runtime,  or from within the container. It is depending
on  the hook  you  want to  use.  The full  list of  possible  hook points  are:
`prestart`,  `createRuntime`, `createContainer`,  `startContainer`, `poststart`,
`poststop`. The `createContainer` hook will  resolve the specified binary in the
same mount  namespace as the runtime,  but it will  run in the namespace  of the
resulting  container. It  allows us  to  customize the  containers file  system,
before it  will be  executed. We  should be able  to leverage  it to  create our
Honeytoken file.

By the way, for now we will solely focus on injecting the token file into the
container. The monitoring part, we will solve later. One step at a time.

The first  version of the PoC  looks as follows:  On start, Beesting will  put a
hook  binary  at a  predefined  location,  that  should  be reachable  from  the
cotnainer  runtime.  Next,  it  registers  itself as  a  NRI  plugin,  with  the
`CreateContainer` hook.  Once a container  is created,  the hook will  adapt the
container spec to feature a `createContainer` OCI hook, pointing at the location
we  extracted  the hook  to.  It  also will  pass  the  target location  of  the
honeytoken in the  container as the first  argument to the hook, as  well as the
files content as the second. The rest in then handled by the low level container
runtime, which will  execute the hook, causing  it to put the  honeytoken at the
wanted location inside the container.

In theory  it should be enough  to just mount `/tmp/beesting`  as a `hostPath`
volume and move a  binary there somehow. I decided to simply  embed the OCI hook
binary into Beesting itself,  as this should be easier to  handle. On startup
it will always override any existing hook binaries with a compatible version.

Let's  take our  solution  for a  ride  then!  I was  using  `skaffold` to  ease
development, so we start Beesting by  running `skaffold run`. This will deploy a
daemonset  with  the Beesting  agent  to  all nodes.  Next,  we  run some  dummy
deployment, that in  the end is just  sleeping forever. Looking at  the pods, we
see that the dummy  pod is having some issues. We can  use `kubectl describe` to
figure out what went wrong.

Oh no! `Permission denied`, what the hell?  How can we get a `Permission denied`
if runC itself is  ran as root? This doesn't make a lot  of sense! Has anyone an
idea why  we are  getting this error?  As a  small hint, here  is the  output of
running `mount` in the runtime namespace. Did  you spot it? `/tmp` is mounted as
`NOEXEC`, meaning  even though  our binary  has the  executable flag  set, Linux
refuses to run anything from there. This is a common thing you find when running
kubernetes clusters,  or even servers  in general.  It helps making  a potential
attackers life more difficult,  as they now need to figure  out a differnt place
to extract a malicious payload. The  reason for `/tmp` being very prominent with
hackers, is that anybody can write to it.  So we need to think about a different
location then. After some discussions about  Beesting with a colleague at Cast.AI,
he pointed out, that Istio is doing something similar. Shoutout to Samuel!

When installing istio,  it will extract its binary to  `/opt/cni/bin`. We should
be  fine changing  the  target  directory of  Beesting  from `/tmp/beesting`  to
`/opt/beesting/bin`. Let's try!  Look at that! It worked! If  we now `exec` into
the container, we can see that a  Honeytoken file is created under the specified
location!

Awesome, we  have a  first solution  to inject the  token! I  am not  100% happy
though. Embedding the OCI hook binary and  extracting it at startup , as well as
keeping track in which context the binaries are executed adds to much complexity
if you ask me.  Maybe there is a simpler way? Instead of  injecting an OCI hook,
coulnd't we  inject a  simple `bind`  mount of  our Honeytoken  file?

As a quick refresher, a `bind` mount allow  us to mount a file or directory from
one location to another, while it being accessible at both places. This means we
can mount a file from the runtimes namespace into the container.

The architecture  for PoCv2 then looks  as follows: Once again  is registering a
`CreateContainer`  NRI callback.  Since  the  Honeytoken is  just  a plain  file
without the  need to  be executable,  we can  use a  `hostPath` volume  mount to
`/tmp/beesting`. When  we get  notified by  the NRI, that  a new  container is
created, we simply create a file with the container ID under `/tmp/beesting` and
update  the containers  definition to  feature a  bind mount  to our  predefined
Honeytoken place. And that is pretty much it!

Let's see it in  action. We simply reuse the example  from before. The container
is created without any issues. Nice! To  see if the Honeytoken file was created,
we once again `exec`  into the container and `ls` the  location the file should
be created. Awesome there is a file  there! So everything seems to be working as
expected.

Great success!

I like this way of how we inject the token. We can now move on to the next part.
Monitoring its access.

This is where the Bee in Beesting comes in. We are going to use eBPF.

For those of you who do not know eBPF, it is to the kernel what javascript is to
the browser. Sounds strange, but is  incredibly useful. Before eBPF, it was very
hard to extend  the linux kernel in a  safe and secure manner. The  only way you
had were pretty much kernel modules. As  we have learned before, a single bug in
a kernel module can bring down your whole machine though. Not very secure.

eBPF  changed this,  by introducing  a safe  and secure  way of  attaching small
programs to predefined hooks in the kernel, as well as just arbitrary functions.

Usually, eBPF is written  in a subset of C, that is then  compiled to byte code.
The byte code is then loaded into the  linux kernel and runs on a VM. In reality
though, the byte  code is Just-In-Time compiled to native  code, for performance
reasons. To ensure safety,  the byte code must pass the  verifier, before it can
be loaded into the kernel. It is the verifiers job, to go through all code paths
of the eBPF program and ensure that everything is safe and sound.

For example, unbounded  loops are forbidden. If encountered by  the verifier, it
will  reject your  program. The  reason  for this  is,  that it  is pretty  much
impossible to  know if an unbounded  loop will ever terminate,  unless of course
you solved the halting problem, which you probably don't. This means that when
the kernel passes on control to the eBPF program, it could stall the kernel
forever. Not good.

Once the verifier gives its OK though, the program is safe to run.

In order  to store state  and share data, eBPF  introduced the concept  of maps.
Maps are data structures,  which can be access from both  kernel and user space.
They also  allow for data being  shared across different eBPF  programs. Typical
use  cases  of maps  include  sharing  events  such  as certain  syscalls  being
triggered with user space, writing configuration  from the user space program to
eBPF or storing context to further enhance collected data.

There is a variety of map types available. All of them are solving certain use
cases such  as first-in-first-out  queues,  LRU  data storage  or  simple
arrays. There is even a map type representing a hash map.

You cannot just  call any kernel function  from an eBPF program, as  it would be
impossible to  guarantee stability and  saftey of  your program. To  work around
this eBPF  has the  notion of so  called helper functions.  In a  nutshell, eBPF
helper functions  allow you to retrieve  data and interact with  the kernel. For
example there are helper functions to interact  with eBPF maps or read data from
memory.

Now that we have a basic understanding what eBPF is and allows us to do, how are
we going to detect if a file has been access?

For this, we  will have a quick look  at what even is a file  in linux? Spoiler,
this will be once again a great simplification, that should still give us enough
inside to  understand how  things work. As  we have heard  before, all  files on
linux live on a  file system. Each file is represented by a  so called inode, or
index node. Inodes consist  of a unique number, or at least  unique for the file
system, which in the  end is just the index of the inode  in the inode table. It
also contains some metadata of the file,  such as the file type, the permission,
as  well as  the  owner and  the  size. Inodes  also contain  a  pointer to  the
underlying block where the  content of the file lives. This  is where the binary
for that one funny cat video is stored.

The  monitoring part  is trickier  than it  sounds. There  are multiple  ways of
interacting with files. For example, we  could hook the `open` syscall, but then
we would miss out  on any access via the `openat` syscall. We  also need to take
into consideration  an attacker might simply  symlink our file in  order to mask
accessing it. There  are even a few  ways of getting access to  the file without
calling `open` at all.

So there are quite a few different options to watch out for.

Luckily for us,  there is this thing  called Linux Security Modules,  or LSM for
short. In a nutshell,  LSM can be used to evaluate permissions  of a process for
certain actions, such as creating files. We  are not going to dive any deeper on
LSM, as Beesting  is not really using  it, but instead hooks  the same functions
used to evaluate permissions.

It will look as follows. Imagine we ahvea  service called `my-fancy-service` that
is trying to open a file, through the `open` syscall. The kernel will now call the
`security_file_open` hook, which is part of LSM, to evaluate if if the process
has permissions to open that certain file. If it does, the kernel will go ahead
and call the corresponding open function defined via the virtual file system.
What beesting is doing, it is hooking the `security_file_open` function to get
notified, if a file is being opened.

To combine  everything: PoCv3  is based  on PoCv2 for  the file  injection part.
During file  injection, it  now also records  the inode, as  well as  the device
number of  the Honeytoken  to be injected.  This works, as  the inode  of `bind`
mounted files  stay the same. This  inode and device  number is then added  to a
eBPF hash map, which is used by our  eBPF program to check if the file a process
tries to  access is  one of  our Honeytokens.  If this  is the  case, we  send a
message to  userspace, via  a ringbuffer.  You can  think of  a ringbuffer  as a
simple  message  queue  that eBPF  in  kernel  space  can  put messages  on  and
userspace  can read  from  it. It  is  worth noting  though,  that just  hooking
`security_file_open`  would  probably  not  be enough  for  a  production  ready
application. For the sake of the PoC it was enough though.

And here is a quick demo of it in  action! We run the same sample as before. The
pod is  created without any  issues, nice. The honey  token is also  injected as
before. Now we  are going to `cat`  the content of the  file, simulating access.
Beesting should  now have  logged a warning  to the STDOUT,  so let's  check the
logs.  And there  it  is.  Beesting successfully  detected  file  access of  our
honeytoken.

Very nice!

This is  pretty much  it!

I have  had a few other  ideas that I would  like to explore in  the future, but
didn't had  time .

One would  be to take PoCv1  as the base for  file detection instead of  v2.
The OCI hook could  communicate with a Beesting socket that is
accessible through a `hostPath` volume. The benefit I see in doing this is, that
there is  no need to care  about thing such as  node restarts, as the  file will
live within the containers filesystem.  With the currently implementation a node
restart would cause Beesting  to loose pretty much all state,  which was a known
trade off I've chosen during the PoC to keep things simple.

Yet another approach I was thinking about, was to scan all read operations for a
certain unique  pattern, which  would represent our  Honeytoken. This  would, at
least in theory, be more flexible at detecting that a Honeytoken was read, as it
doesn't solely rely on inodes.

Anyway I hope to  explore this space further in the future. All  in all it was a
pretty nice learning and researching opportunity.  In case you are interested in
the code, you can find it under https://github.com/patrickpichler/beesting/.

I would also encourage you to  check out my personal blog at patrickpichler.dev.
Currently it  looks a  bit empty, but  I hopefully get  around to  transform all
research I  have done from past  talks into articles  and put them up  there. So
stay tuned, hopefully.

If you like to discuss more about eBPF or Beesting, feel free to approach me.

Thank you!
