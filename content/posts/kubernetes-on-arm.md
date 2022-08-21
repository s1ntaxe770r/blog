# Kubernetes on ARM

It’s a nice Tuesday morning, the sun is out and I have had my first cup of coffee, what a lovely time to attempt to install Kubernetes on my Raspberry Pi.

I’m using the Raspberry Pi 4 with a 128GB SD card running [Raspbian](https://www.raspbian.org/). Would this be enough? No idea but I’m about to find out.

![lets-go-the-rock.gif](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/lets-go-the-rock.gif)

## Current Setup

So far the only configuration I have made is assigning a static IP address and connected the PI to my router over Ethernet. 

![current setup ](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/20220503_105557.jpg)

current setup 

## Kubernetes Distributions

Much Like Ice-Cream Kubernetes comes in different flavors picking the right combination would be key to how well this works ( so I think). Here are a few I evaluated.

### [KinD](https://kind.sigs.k8s.io/) (Kubernetes in Docker)

KinD is a tool for running Kubernetes in Docker, it’s lightweight and works well.  I use it for testing stuff on my Mac. But where’s the fun in Installing docker and calling it a day?. I’m not looking to do [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way) but I want some resistance. For this reason, I would not be picking KinD. 

### [But what about K3s?](https://k3s.io/)

K3s has a reputation for being lightweight and super fast. Promising a 30sec startup time along with being optimized for ARM. It is a very logical choice for my use case. 

But here’s the thing. I don’t always like being “logical” and something about the next distribution screams INSTALL to me. 

### I give you K0s !!!

![Untitled](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/Untitled.png)

K0s advertises itself as a “**The Zero Friction Kubernetes”,** obviously this is not what I am here for, I could sit here and re-word every feature on the k0s homepage but I don’t have time for that today,  However, there are a few features I really like. 

1. In-Cluster SQLite: this is the default for single-node clusters, this is probably not as performant as ETCD but I’m not trying to [destroy my SD](https://github.com/rancher/docs/issues/3134) (prematurely).
2. Horizontal Pod Autoscaling: In case I need to scale my home network to handle traffic from a thousand users 🤣.  

The features I mentioned most likely exist on k3s but like I said something in me really wants to give k0s a shot.

After combing the docs for several ~~days~~ hours I finally realize why the controller has refused to start. I was doing two things wrong: 

- I was using the installation instructions for Debian machines and k0s had [instructions](https://docs.k0sproject.io/v1.24.3+k0s.0/raspberry-pi4/) for the raspberry pi all along 🤦🏽.
- Needed to enable `cgroups` on the pi and a few other [options](https://docs.k0sproject.io/v1.24.3+k0s.0/raspberry-pi4/#set-up-nodes) and reboot the PI.

Well that worked so I should have a working cluster right ? 

![sddefault.jpg](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/sddefault.jpg)

After running `k0s start` I decide to watch the journalctl logs , which doesn’t sound like a bad idea at first , however I was met with some horrible node errors and at some point [coreDNS](https://coredns.io) refused to start. A solid 10 minutes go by and the node errors seemed to have resolved itself and coreDNS seemed to have initialized , but I still see some “partial failures” in the logs and I decide to wait it out a little longer.

![journalctl logs ](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/Untitled%201.png)

journalctl logs 

Eventually I take the leap of faith and run `kubectl get nodes` …. 

![Untitled](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/Untitled%202.png)

```bash
NAME      STATUS   ROLES           AGE    VERSION
thev01d   Ready    control-plane   23m   v1.23.6+k0s
```

The node is finally ready , well Mission accomplished right ?  Not quite I still need a load balancer that I can use to expose my applications, I’ve heard good things about MetalLB so I would be trying that first. Thankfully k0s has a [dedicated pag](https://docs.k0sproject.io/v1.23.6+k0s.2/examples/metallb-loadbalancer/)e for MetalLB. 

## What about Nginx Ingress controller ?

Ever since I learned about ingress controllers I have always stuck with Nginx because it simple and gives me the least headaches(staring at you traefik)  plus A/B testing is super easy with Nginx. The problem here is I don’t really need the routing capabilities of Nginx ingress , here all I  need is a single static IP I can access my services on and MetalLB seems to do that. 

## Reality strikes

![6qj1by.jpg](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/6qj1by.jpg)

Sadly this turned out to be the case and for whatever reason my pods where not getting an external IP. Tried a googling but I am honestly not ready to battle with my crappy router and get this thing working. So I did the sane thing a defaulted to Nginx which worked flawlessly.

![Untitled](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/Untitled%203.png)

Ah yes sweet sweet 404 , but this wouldn’t be complete without a “simple https server” would it? 

![Untitled](Kubernetes%20on%20ARM%200e009dbeaab34051952bd5f37cc78c7f/Untitled%204.png)

## Conclusion

Overall this was a great learning experience ,  I got to learn about different Kubernetes components and how they interact with one another.  Also ran into an issue where my PI refused to connect to my router because the power supply was not enough, this got me thinking about what goes on in the average datacenter and how routers can fail ( mine did) and host of other problems that the modern day cloud abstracts from users.

If you made it this far thanks for reading , I hope to blog more about hardware and other PI projects is decide to make in the near future , until then goodbye and keep chewing!