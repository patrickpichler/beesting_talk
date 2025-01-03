* introduction

* setup stage by talking about blog post from Markus
    * i was reading this article and thought how to do it in single agent
    * maybe bit of background that i am working on kVisor, which is a eBPF agent

* explain end goal
    * showcase path traversal attack to read service account token
        * TODO: implement dummy service or find easy to exploit existing one

* what are Honeytokens
    * why are Honeytokens so effective when detecting malicious behavior

* quick overview over k8s architecture
    * what is actually running on a k8s node
    * what is Kubelet
    * what is CRI
    * how does runC fit into the picture

* lay out potential solutions on how to solve the problem of injecting (pros/cons)
    * use mutating admission controller to inject honey token
        * would work, but design would not be as sleek and low config as I want it to be
    * custom FS
        * to create a custom FS a custom kernel module would be required or FUSE
    * somehow bring the agent to inject the token
        * would be nice if it can simply live on the node via DaemonSet
        * but how to get it working?

* dive a bit into containerD plugins
    * few different set of plugins
    * most of the plugin types are not really useful
    * NRI looks interesting

* deep dive into NRI
    * not only supported by containerD, but also CRI-O
    * hooks to modify container definitions before them being passed to runC
    * hooks to run before containers are but FS is prepped

* back to implementing Beesting, PoCv1
    * for now focus only on getting file into container
    * uses OCI hooks
        * should work with any OCI compliant low level runtime
        * will place a hook binary at a specified location on the host FS
            * cannot be /tmp as it is often mounted `NOEXEC` (hook needs to be exec)
            * needs to be a static compiled binary, to ensure it working pretty much everywhere
    * highlight problems with approach
        * self unpacking logic is somewhat ugly
            * requires atomic unpack in case of update
        * more complicated build logic
        * need a persistent `hostPath` volume with exec flag (not nice)

* PoCv2
    * also focuses on getting file into container
    * instead of OCI hooks, which need to be executable, v2 bind mount files from
      `hostPath` volume into container
        * nice benefit of not caring about `NOEXEC` mounts
        * no complicated multi staged build logic
        * node restart would lead to a loss of tokens though, as `/tmp` is in memory only
    * overall solution seemed pretty decent and I decided to go forward with it

* short intro into eBPF
    * it is to the kernel what JavaScript is to the browser
    * byte code that runs on a VM in kernel space
    * will be JIT compiled
    * can be used to monitor file system calls
    * short bpftrace sample for hooking file writes might be nice

* short intro into how filesystems work
    * what are inodes
    * how can files be accessed

* there are a few challenges when trying to monitor file access with eBPF
    * symlinks
        * will create new inodes
        * will have different path than file we want to protect
    * io_uring
    * figuring out what kernel methods to hook to get coverage for all
      ways of reading a file

* PoCv3
    * keep things simple
    * files for injection will be created in `hostPath` volume
    * record inode of created files
        * will be used in monitoring
    *  inject files via bind mount into containers
        * can be done with one of the NRI hooks
        * bind mount will not change inode
    * eBPF program will also be running
        * has info of all the inodes to monitor in a map
        * will send message to ringbuf if file access has been detected
    * print out PID, container name and pod name where the file has been accessed
    * Beesting also uses special `Synchronize` hook that gets passed all running containers
      to figure out what inodes to watch on startup
        * it could also inject tokens into existing containers here, but it is not
          implemented yet
    * show e2e demo of it working

* other approaches I want to test, but didn't have a chance yet
    * based on PoCv1 report back inodes to Beesting and monitor that way
        * would require unix socket in the mounted `hostPath` volume
        * would also require some sort of auth
            * probably pre generated token is enough
    * PoCv3 generates a single file per container, would a single be enough?
        * only one inode to watch
        * potential attackers could notice that it is the same inode though
        * not too much benefits, besides no need for cleanup
    * in theory we could also check what has been read from anywhere and see if it
      contains a special pattern
        * expensive to run (needs benchmarking though)
        * FP rate for sure higher than inode based approaches
        * would be more flexible though (no need to check inodes)
