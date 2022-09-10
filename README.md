# okd4-Installer

Deploying an okd4 cluster (upstream for OpenShift) is not an easy task and tend to fail from one deployment to another.
The purspose of this repo is to provide a tool to quickly spin up a working cluster in a Linux bare metal hypervisor without friction. While this has been tested on hetzner, it can work on any dedicated host. Again, deploying a working okd4 cluster in non-cloud environment is hard. It requiet several steps that are sometimes easy to misconfigure. They are been completely automated using Ansible. The deployment time using this tool was about 30 min (tested several times).

The following features are provided:

- Dynamic storage (NFS backed)
- Openshift registry already patched to use the storage backend and prune images
- A local user based on http basic auth
- Automatic DNS registration if you use Cloudflare as your registrar (If you don't have cloudflare dns, you can manually add A records pointing the public IP of your server to *apps.<YOUR_CLUSTER_NAME>.<YOUR_DOMAIN> and api.<YOUR_CLUSTER_NAME>.<YOUR_DOMAIN>.)
- Automatic letsencrypt certificates request and configuration for your public endpoints (can also serve for your workloads)
- A cluster isolated in a private network. Only the required APIs are exposed to the public Internet (Kube API, Workload services and consoles)
- Support both single and multi-master cluster

## Prerequisite

- A baremetal server with enough RAM/CPU/Disk to handle your Lab (mine was 12x3.6 GHZ Core, 128GB RAM and 1 TB SSD)
- Centos 8 stream as base OS
- A public IP if you want your Lab to be available other the Internet (Otherwise, you can use your home LAN)
- Passwordless SSH to the server for the user Ansible will run with
- Ansible < 2.12 on the machine from with you will run the deployment (client machine or Ansible controller)
- jmespath package must be installed

Exemple of client machine setup (in a venv, ideally)
```bash
pip3 install --upgrade pip; \
pip3 install pywinrm[kerberos]; \
pip3 install pywinrm; \
pip3 install jmespath; \
pip3 install requests; \
python3 -m pip install "ansible<2.12"
```

## Architecture

```flow
                                                   Baremetal Server (With one public IP)
                                      ┌───────────────────────────────────────────────────────────────┐
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │                                                               │
                                      │Firewalld                            DHCP/NAT/Internal         │
                          Public VIP  │                                     Internal Network          │
                                      ├─────────────────┐                  ┌──────────────┐           │
┌──────────┐                          │     LB          │   80/443/6443    │ Linux        │           │
│Internet  ├──────────────────────────►  (Container)    ├──────────────────► Bridge (kvm) │           │
└──────────┘     80/443/6443          │                 │   22623(Internal)│              │           │
                                      ├─────────────────┘                  └─'───.────..──┘           │
                                      │Private VIP                          ''   .     ..             │
                                      │                                    ''    .      ..            │
                                      │                                  ''      .        ..          │
                                      │                                 ''       .         ..         │
                                      │                                '         ..         .         │
                                      │                               '           .          ..       │
                                      │                          ┌───'──┐     ┌───.──┐     ┌──.───┐   │
                                      │                          │  C   │     │  M   │     │   B  │   │
                                      │                          │(VM)  │     │(VM)  │     │ (VM) │   │
                                      │                          └──────┘     └──────┘     └──────┘   │
                                      │                                                               │
                                      │                             Computes/Masters/bootstrap        │
                                      │                                                               │
                                      └───────────────────────────────────────────────────────────────┘
```

## Usage

1. Fill the inventory with the IP/hostname of your baremetal server
2. Configure passwordless ssh to that server
3. Fill the global variable file [all.yml](ansible/inventory/group_vars/all.yml). The parmameters are self explainatory.
4. Run the setup playbook (`ansible playbook -u USERNAME -i inventory setup.yml`). If you use vault as I do, supply the vault password or file accordingly.

## Okd4 official installation guide

https://docs.okd.io/latest/installing/installing_bare_metal/installing-bare-metal.html
