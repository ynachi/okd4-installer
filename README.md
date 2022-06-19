# okd4-hetzner
Deploy an okd4 cluster on hetzner dedicated. While this has been tested on hetzner, it can work on any dedicated host. Deploying a working okd4 cluster in non-cloud environment is hard. It requiet several steps that are sometimes easy to misconfigure. They are been completely automated using Ansible. You issue one command, wait about 30 minutes and you get a fully functional cluster with the following:

- Dynamic storage (NFS backed)
- Openshift registry already patched to use the storage backend and prune images
- A local user based on http basic auth
- Automatic DNS registration if you use Cloudflare as your registrar
- Automatic letsencrypt certificates request and configuration for your public endpoints (can also serve for your workloads)
- A cluster isolated in a private network. Only the required APIs are exposed to the public Internet (Kube API, Workload services and consoles)

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