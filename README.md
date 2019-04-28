This Ansible playbook turns your fleet of Raspberry Pi into Kubernetes cattle with k3s, the streamlined version of Kubernetes by @rancher that runs silky smooth on the ARM processor powering your Pi.

# Manually prepare your Raspberries
- Burn Raspian Stretch Lite on an SD card with EtcherBalena, or something alike
- Create an empty `ssh` file in the root of the SD card (volume is called `boot`)

# Secure your Raspberries
Optional but advisable: use SSH key auth and disable password login.
The default ssh credentials for a Raspberry Pi are username `pi` and password `raspberry`.
- Copy your pubkey to the Pi with `ssh-copy-id pi@kube1.example.com`

# Configure ansible setup
- Copy `ansible/hosts.template` to `ansible/hosts` for your configuration
- Choose one of the Raspberries to lead / orchestrate your cluster; we'll call this the _server_
- In `ansible/hosts`, fill out the ip or hostname for the leading Raspberry under `[k3s-server]`
- Fill out the ip's or hostnames for the rest of your cattle under `[k3s-agents]`
- change the password in the group_vars
- pick a version of k3s to install

# Provision the nodes
When you're all set up and configured, run the Ansible playbook:

[Ansible in a conatiner](https://ruleoftech.com/2017/dockerizing-all-the-things-running-ansible-inside-docker-container)

### Build a player

```
docker build -t lcwyo/ansible-player .
```

### Run the player

```
docker run --rm -it \
    -v ~/.ssh/id_rsa:/root/.ssh/id_rsa \
    -v ~/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub \
    -v $(pwd):/ansible/ \
    lcwyo/ansible-player /ansible/site.yml -i /ansible/hosts
```


This will:
- Configure and prepare the Raspberries for k3s
- Install the k3s binary on the 'server' Pi (the leading node)
- Install a `k3s-server` service on the server and start it
- Fetch the node token from the server
- Install and start a `k3s-agent` service on the agents, joining the cluster
- Enable autostart on boot for k3s on all nodes

# See if it worked
Log into your server node and run:
```bash
$ sudo k3s kubectl get nodes -o wide
```
You should see all of your nodes broadcasting a _Ready_ status.

Happy cattle herding!

#todo

need to find some storage class...


#### Install Helm

Download helm

`curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh`

Make instalation script executable

`chmod u+x install-helm.sh`

Install helm
`./install-helm.sh
`

Prepare & Install tiller

```
kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller

```

tiller needs an arm image, so we use the arm immage from jessestuart

`helm init --tiller-image=jessestuart/tiller:v2.9.0 --service-account tiller --upgrade `

use arm helm charts

`helm repo add arm-stable https://peterhuene.github.io/arm-charts/stable`
