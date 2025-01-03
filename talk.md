Hello there!

Welcome everyone! Let's get started.

Now first things first,  my name is Patrick Pichler and  I am currently employed
at  Cast.AI as  a senior  software  engineer, developing  a Kubernetes  security
product.

Originally I  started my  career as  a Java  Software Developer,  but everything
changed when I stumbled upon Linux and the cloud. This definitely transformed me
into a  full-on Linux  nerd. Do  not question why  I am  currently on  a Macbook
though.

At my day job,  I am part of a team that develops  a Kubernetes agent to perform
runtime detection of anomalous behavior.

Let me take you  back to August 2024, where I had a  discussion with a colleague
of mine, Markus. In  my day job, I am part of a  team that implements Kubernetes
runtime detections  via an agent running  on nodes and we  were discussing about
some high fidelity events. By high fidelity I mean events that when detected, we
can  be pretty  sure that  something fishy  is  going on  in a  pod. During  the
discussion,  he brought  up  a blog  post  of  his and  a  few former  Dynatrace
colleagues wrote, in  which they outlined how to inject  and monitor Honeytokens
into Pods via  Kyverno's mutating webhook features and using  Tetragon to detect
file access. The main premise of the  idea was that, if we could detect somebody
accessing that file, we would know  with a pretty high certainty, that something
is off in that pod.

I immediately fell in love with the idea. It sounded simple on paper and from
what I can tell, it would be highly effective. My only issue with the solution
outlined in the blog, it requires to install two additional components in the
form of Kyverno and Tetragon. After a bit of searching the internet, I couldn't
find anyone doing this in a single component. This was the moment the idea for
Beesting was born.

Before we look into how the thing  should work in more details, we should answer
the question of what even are Honeytokens?

Put simply, you can  think of a Honeytoken as digital bait. They are put into
places where one would usually expect tokens of high value, meaning an potential
attacker would probe those locations and access the Honeytoken. A pretty good
example for this would be Kubernetes service account tokens.

The main idea of Beesting would be, to inject some bait files into pods.
Injecting the bait is only half the story though, as we need to closely monitor
usage of the Honeytoken.

To better understand  the usefulness of all  of this, we will  now put ourselves
into the  shoes of  an attacker.  Imagine we are some evil Hackers that are
currently probing some public facing CMS for any vulnerabilities. We find a RCE
and now have a shell in the CMS machine. Let us see if we can find any secrets
by looking at the `env` variables. Hmm no secrets, but a lot of them start with
`KUBERNETES_`. This highly suggests the CMS is running inside a Kubernetes Pod.
Next step I would say we see if we can find any secrets under `/var/run/secrets`.
This is a common location to mount service account tokens and any other file
based secrets. And we got lucky it seems, we find a file called
`/var/run/secrets/eks.amazonaws.com/access_key_token`. Sounds promising! We
simply use `cat` to get the token then.

What our attacker does not know, the file accessed during recon was a Honeytoken
and marked the pod our CMS is running in as compromised and has informed our
security team. That is the power of Honeytokens.

So the goal of Beesting was to somehow build a service, that can inject such
Honeytokens and monitor for its access or usage.

For us to tackle this problem, we first need to understand the architecture of
Kubernetes a bit better. A Kubernetes cluster can be separated into two parts
the control plane, this is where ETCD and the API server runs, and the data
plane. Any app we submit to the cluster will run on nodes, which are part of
the data plane. Kubernetes nodes are usually Linux machines, where the Kubelet
runs. Kubelet will fetch what pods to run on the current machine from the API
server. It will then communicate with a container runtime, most often
containerD, and instructs it what containers to run. The Kubelet communicates
with the container runtime via the Container Runtime Interface or CRI in short.
Container runtimes, such as containerD or CRI-O, then download whatever image
we instruct it to run and spawn the "container" via a low level runtime such
as runC. Linux doesn't really have the concept of a container. They are pretty
much made out of control groups, also called Cgroups and namespaces. Cgroups
allow to limit the resource usage, such as CPU and memory, whereas with
namespaces we can separate areas such as networking or file mounts.

When somebody now  runs `kubectl apply -f pod.yaml`, `kubectl`  will talk to the
API server, to create a new resource of type `Pod` in `ETCD`. Next the scheduler
picks up that there is an unscheduled pod  and assigns a node to it. The Kubelet
running on our target node constantly checks  the API server for new pods to run
on its  node. If  a new  pod is  detected, Kubelet  will instruct  the container
runtime, to create the pod and run it. The interface used to communicate between
Kubelet and the container runtime is  called the CRI, which stands for Container
Runtime Interface.  Pretty descriptive  name. The  container runtime,  we assume
containerD  in this  example,  will pull  whatever image  was  specified in  the
containers making  up the pod, prepares  the mount roots for  the containers and
instructs runC to run them. runC will  create all the necessary Cgroups, as well
as  namespaces and  then runs  whatever binary  we specified  in the  containers
entrypoint. This is of course a bit of an over simplification, but enough for us
to understand the basic concept.

Our  goal with  Beesting  is to  inject  a Honeytoken  file  in the  resulting
containers. There are a few different ways on how we could achieve this.

Let us have a  look at how the Dynatrace Gang achieved this  in their blog post.
They  used  Kyverno  as  a  mutating admission  webhook,  that  did  inject  the
Honeytoken file via a projected volume mount. Ok sounds easy, but those are some
fancy words. To refresh  our knowledge, we will go over those  words one by one.
What a Honeytoken  we already learned earlier. Kyverno is  a policy engine, that
acts  as a  webhook in  Kubernetes, allowing  us enforce  certain standards,  or
modify pods before  they are persisted. This is exactly  what a mutating webhook
does. It is an extension point in Kubernetes to allow us to modify any resources
submitted before they  are persisted into ETCD. A projected  volume mount allows
us to mount a secret or config map  at a specified location in the filesystem of
our pod. So to rephrase that complicated sounding sentence from before, the blog
post explains  how to use  Kyverno to inject  a file into  our pod before  it is
persisted.

For Beesting,  I could also  have followed the  same approach. Instead  of using
Kyverno as the  mutating webhook, Beesting could have just  exposed the required
API.  There are  several reasons  I decided  against that  approach though.  The
biggest was that injecting the Honeytoken is  only half the bargain. So either I
the mutating webhook API from all agents, or I would need to split Beesting into
two parts. This means  my initial goal of easy to install and  operate is out of
the window. The  next problem I have  with this approach, it  is very Kubernetes
specific. Not the biggest deals, but still,  if I can make it more general, that
is a  nice win. Last,  but not least,  it just didn't  sounded that much  fun. I
wanted  to start  Beesting  to research  into  something new  and  just using  a
mutating webhook  didn't get me  that exited. Maybe in  the future, I  give this
approach another shot  though, but since I  work on Beesting in my  free time, I
will spend my time on whatever looks fun first probably.

Ok, so  the Admission Controller is  off the table.  How else can we  inject our
sweet Honeytokens into containers? Looking  back at the Kubernetes architecture,
containers are  created by the  Container Runtime.  In most cases,  this runtime
will be containerD.  I do not have exact  numbers, but it feels like  99% of the
Kubernetes installations  out there run  containerD. Does containerD  offer some
kind of plugin  mechanism we could leverage here? It  turns out, containerD does
have plugins. Some  of plugin types include runtime plugins,  which allow you to
add support  for your  own low-level  runtime such as  runC, differ  plugins, in
which you can calculate what mount points are missing from containers and how to
apply them, as well as many more. The  differ plugin did sound as it would allow
us to inject files into containers, the way  we want. Though if we have a closer
look at the interface  we would need to implement, it would  feel like quite the
hack, as we would get a `DiffRequest` from containerD, that contains the data we
should diff. In our case we would  need to inject something into the diff, which
feels plain wrong.

During research  of containerD plugins,  I also stumbled  on a thing  called the
node resource  interface of NRI for  short. The main goal  of the NRI is  to add
domain specific custom logic to  OCI compatible runtimes. This sounds promising!
Honeytoken injection is domain specific custom logic  in a way. The docs for the
NRI  also mention  that it  keeps  the plugins  runtime agnostic  and that  both
containerD and CRI-O  are currently supported. Starting with  containerD v2, the
NRI plugin  is even  enabled by default.  This would be  amazing, as  this would
allow us to use Beesting with the  two most widely spread container runtimes and
maybe even more in  the future! NRI works by exposing a GRPC  API through a unix
socket, that can be used to register  plugins with it. The NRI API exposes quite
a few different callbacks such  as: `CreateContainer`, `UpdateContainer` and various
state change events like  `PostCreateContainer` or `RunPodStandbox`. Callbacks, such
as `CreateContainer` or `UpdateContainer`, grant us  the power to modify anything we
want of the container  before it will be handed over  to runC, including mounts,
as well as so  called OCI hooks. OCI hooks are a  standardized way of specifying
commands that runC  will execute at certain stages when  running a container. We
will hear more about them later.

Awesome, NRI  sounds exactly  what we  need. We  only need  to figure  out, what
exactly do we even inject into the container?

Honeytokens in  our case are  just files. There is  a variety of  different file
systems supported by Linux. All of them implement certain functions specified by
the Virtual File System, or VFS in  short. It defines methods on how to interact
with the file system in a generic way.  What if we somehow create our own custom
file system, which we  can mount into the container at the  location we want the
Honeytoken to be in? This should also  easily let us detect if somebody tries to
read our Honeytoken, as they need to go through the methods we define to be part
of the VFS  API. Sounds amazing! There is  one catch though. In order  to make a
VFS available to a Linux machine, they need to be installed via a Kernel Module.
Not only does this  require higher privileges, it also is  quite dangerous, as a
simple bug in  our file system implementation can bring  down the whole machine.
Besides Kernel Modules, there is also  the possibility to use FUSE, which stands
for 'Filesystem  in Userspace'  to implement  a custom  FS. Like  Kernel Modules
though it requires higher  privileges to install them, plus it  would add a hard
dependency on  having FUSE installed  on all the  nodes. The custom  file system
idea is not really a good fit and out of the window.

Hmm we were hearing about those OCI  hooks before. Maybe we can leverage them to
create the Honeytoken in the container? As  heard earlier, with OCI hooks we can
run custom  commands at various stages  of the container lifecycle.  Those hooks
should be supported by any OCI compatible low level runtime, which runC is. They
consist  of an  absolute path  to the  binary that  needs to  be run,  a set  of
arguments to pass to the hook, as well as some env variables and timeout for the
hook to finish before being killed. The  context of the path argument can either
be from the  point of view of the  host, or from within the  container, which is
depending on  the hook you want  to use. The  full list of possible  hook points
are: prestart, createRuntime,  `createContainer`, `startContainer`, `poststart`,
`poststop`. The `createContainer` hook will  resolve the specified binary in the
same mount namespace as the runtime, but  it will run in the  namespace of the
resulting container. It allows us to  customize the container, before it will be
executed.  This means  we can  leverage  it to  create our  Honeytoken file.

The idea  the first version of  the PoC is then  as follows: We implement  a NRI
plugin.  On  startup,  Beesting  will  put a  binary  at  a  specified  location
reachable  from the  runtime mount  namespace. The  NRI plugin  will register  a
`CreateContainer` hook,  which will update any  given container spec and  add an
OCI `createContainer` hook, pointing to our extracted binary and gets the target
path passed as its argument. runC will then  handle the rest, as it will run the
hook once it tries to create the container. Sound like a plan!

By the way, this  focuses solely on the injecting of  the token file. Monitoring
it for access I postponed to a later stage. One step at a time.

The  first challenge  during implementation  of PoCv1  didn't take  too long  to
appear. What  is the best  way of  exposing a binary  from the container  to the
host?  In theory  it  should be  enough  to just  mount  `/tmp/beestinger` as  a
`hostPath` volume and move a binary there somehow. I decided to simply embed the
OCI hook binary into  Beesting. It should be easier to handle,  as on startup it
will always override any existing hook  binaries with a compatible version. So I
decided to have a two stage build. Stage 1 will build our OCI hook binary, Stage
2 Beesting. This makes it easy to embed the hook binary into Beesting.

The rest is rather straight forward. To ensure not getting some Heisenbugs with
the OCI hook, I implemented the OCI hook extraction as an atomic operation. Kudos
to Tailscale for having this very nice snippet in one of their license compatible
code bases.

Let's take our  solution for a ride  then! Oh no! `Permission  denied`, what the
hell? How can we  get a `Permission denied` if runC itself is  ran as root? This
doesn't make a lot  of sense! Has anyone an idea why we  are getting this error?
Here is the output of running `mount`  in the runtime namespace as a small hint.
Did you spot it?  `/tmp` is mounted as `NOEXEC`, meaning  even though our binary
has the  executable flag set,  Linux refuses to run  anything from there.  So we
need  to think  about  a different  folder then.  After  some discussions  about
Beesting  with a  colleague at  Cast.AI,  he pointed  out, that  Istio is  doing
something similar.  After checking  their source  code, we  found that  Istio is
extracting its binary under `/opt/cni/bin`. I guess it should be fine to extract
the Beesting hook then under `/opt/beestinger/bin`.  Let's try! Look at that! It
worked! If we now  `exec` into the container, we can see  that a Honeytoken file
is created under the specified location!

Awesome, we  have a  first solution  to inject the  token! I  am not  100% happy
though. Embedding the OCI hook binary and extracting it at startup adds too much
complexity if  you ask  me. Maybe  there is a  simpler way?  What if  instead of
injecting the OCI hook into the container  on startup, we simply inject a `bind`
mount of a Honeytoken file under the  specified location? Just as a refresher, a
`bind` mount allow  us to mount a  file or folder from one  location to another,
while it being  accessible at both places.  This means we can mount  a file from
the runtimes namespace into the container. So the plan would be, to generate our
Honeytoken file at a place accessible from  the runtime namespace and use NRI to
modify  the created  container to  feature  a `bind`  mount used  to inject  our
Honeytoken file  into the container. Since  the Honeytoken is just  a plain file
without the  need to  be executable,  we can  use a  `hostPath` volume  mount to
`/tmp/beestinger`. When  we get  notified by  the NRI, that  a new  container is
created, we simply create a file with the container ID under `/tmp/beesting` and
update  the containers  definition to  feature a  bind mount  to our  predefined
Honeytoken place. And that is pretty much  it! Let's see it in action. We simply
reuse the example  from before. The container is created  without any issues and
we also  do not  see any  problems on  the Beesting  logs. Nice!  To see  if the
Honeytoken file was  created, we once again `exec` into  the container and `cat`
the location  the file  should be  created. Awesome  there is  a file  there! So
everything seems to be working as expected.

With the injection of our Honeytoken out of the way, how are going to monitor it
for access?  This is where  the Bee in  Beesting comes in.  We are going  to use
eBPF.

For those of you who do not know  eBPF, it can be compared to what JavaScript is
for the  Browser, but for  the Linux kernel.  Sounds strange, but  is incredibly
useful. Before  eBPF it  was incredibly  hard to extend  the Linux  kernel. eBPF
changed this, by offering a secure way of doing this. One of the most common use
cases of eBPF is in monitoring. eBPF  allows us to attach small programs to hook
points,  which can  either be  predefined, arbitrary  kernel functions,  or even
arbitrary locations in the kernel. eBPF code  is compiled to a bytecode, that is
then run inside  the kernel space in a  VM. To ensure that eBPF code  is safe to
run and  doesn't crash the kernel,  there are various safety  measures in place.
For  example, it  is illegal  to have  unbounded loops  in eBPF,  as this  would
effectively stall  the kernel once it  transfers control to an  eBPF program. To
ensure our  program is safe  to run,  it must pass  the so called  verifier.
It is the verifiers job to go through all possible execution paths in our eBPF
program and ensure that we are not doing something stupid that could crash the
kernel.

In order  to store state  and share data, eBPF  introduced the concept  of maps.
Maps are  data structures, which  can be access from  both eBPF and  user space.
They also  allow for data being  shared across different eBPF  programs. Typical
use  cases  of maps  include  sharing  events  such  as certain  syscalls  being
triggered with user space, writing configuration  from the user space program to
eBPF or  storing context  from different programs  to further  enhance collected
data.

Maps come  in different  types and  shapes. There are  map types  for particular
operations,  such  as first-in-first-out  queues,  LRU  data storage  or  simple
arrays. There is even a map type representing a hash map.

You cannot just call any kernel function from an eBPF program, as this would not
only couple your  program to an exact  kernel version, but would  also be making
hard to  guarantee compatibility and stability  of your program. To  work around
this eBPF  has the  notion of so  called helper functions.  In a  nutshell, eBPF
helper functions  allow you to retrieve  data and interact with  the kernel. For
example there are helper functions to interact  with eBPF maps or read data from
memory.

Now that we have a basic understanding what eBPF is and allows us to do, how are
we going to detect if a file has been access? To keep things simple, all we need
to know is, that  in Linux, each file has an associated  inode. Inode stands for
Index  Node and  contains metadata  of the  file, such  as file  permissions and
owners. Each file in a filesystem has a unique inode number.

The  monitoring part  is trickier  than it  sounds. There  are multiple  ways of
interacting with files. For example, we  could hook the `read` syscall, but then
we would miss out on any access via the `splice` syscall. We also need to take
into consideration an attacker might simply symlink our file in order to mask
accessing it.

Another thing to watch  out for, even though we can hook  pretty much any kernel
function we want, this only works if the function is not inlined in the compiled
kernel. Meaning,  we need  to put some  thought into what  functions we  want to
hook.

Luckily for  us, Linux offers a  way of evaluating permissions  of processes via
LSM hooks. LSM stands for Linux Security  Modules and provide a set of functions
for various use cases, that will be evaluated before certain operations are run,
such a process  trying to create a  symlink. We could even  create eBPF programs
that function as LSM programs, but for our use case, we are just going to attach
ourself to those security functions and  report back to userspace, that our file
has been accessed.

PoCv3 is based on PoCv2, and makes use of attaching eBPF programs to certain LSM
hooks. Additionally to inject files  into containers via `bind` mounts, Beesting
now also records the inode, as well as the device number of the Honeytoken. This
works, as the inode of `bind` mounted files stay the same. This inode and device
number is then  added to a eBPF hash  map, which is used by our  eBPF program to
check if the file  a process tries to access is one of  our Honeytokens. If this
is the case, we send a message to  userspace, via a ringbuffer. You can think of
a  ringbuffer as  a simple  message  queue that  eBPF  in kernel  space can  put
messages on and userspace can read from  it. For simplicities sake, I decided to
only hook  the `security_file_open`  function. From the  testing and  tracing of
kernel code I did, it should be good enough to get informed when a file is read.

And here is a quick demo of it in action! We run the same sample as before.
Let's also open the Beesting logs in another terminal. We now `exec` into the
container and try accessing the file with `cat`. As you can see, the Beesting
logs now inform us that a process was accessing our Honeytoken.

This is  pretty much  it! I  have had  a few other  ideas that  I would  like to
explore in the future,  but didn't had time . One would be  to take PoCv1 as the
base  for file  detection  instead of  v2.  The  challenge here  is  in, how  to
communicate the inode  and device we need  to monitor back to  Beesting. The OCI
hook  could communicate  with a  Beesting socket  that is  accessible through  a
`hostPath` volume. The benefit I see in doing  this is, that there is no need to
care  about thing  such as  node  restarts, as  the  file will  live within  the
containers filesystem.  With the currently  implementation a node  restart would
cause Beesting to loose pretty much all  state, which was a known trade off I've
chosen during the PoC to keep things simple.

Yet another approach I was thinking about, was to scan all read operations for
a certain unique pattern. This would, at least in theory, be more flexible at
detecting that a Honeytoken was read, as it doesn't solely rely on inodes. On
the other hand, this approach for sure would be rather expensive, as all the
read data would need to be checked.

Anyway I  hope to explore this  space further in  the future. In case  anyone is
interested in the  code, you can check it  out here. All in all it  was a pretty
nice learning and researching opportunity. In case you are interested in the code,
you can find it under https://github.com/patrickpichler/beesting/.

I would also encourage you to  check out my personal blog at patrickpichler.dev.
Currently it  looks a  bit empty, but  I hopefully get  around to  transform all
research I  have done from past  talks into articles  and put them up  there. So
stay tuned, hopefully.

If you like to discuss more about eBPF, feel free to approach me and talk to me.
I am more than happy to talk about it!

Thanks!
