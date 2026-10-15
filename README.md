# NAT gateway lab: masquerade and port forwarding with nftables in an Ubuntu VM

[![check](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml)

A ready-made **NAT lab** for classrooms, from the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning), where schools and colleges give every student their own machines and a terminal in the browser. Each student gets an Ubuntu VM with a small network built inside it: a private LAN host, a **gateway** and an outside server. They configure **masquerade (source NAT)** and a **port forward (destination NAT)** with **nftables** on the gateway, save the rules so they survive a restart, and see with their own eyes what the other end sees. Nothing is installed on the student's computer.

A VM is one machine, so each of the three is a **network namespace** and each link a pair of virtual cables. The real commands work, which is the point: `nft`, `conntrack -L`, `tcpdump`, `curl`, `ip route`.

## The lab

| Lab | File | What the student does | Help page |
| --- | --- | --- | --- |
| NAT gateway | [`nat-gateway.lab.yaml`](nat-gateway.lab.yaml) | Find why a private host gets no answer from the outside, translate its addresses on the gateway, forward one port from the outside to a web page on the LAN, allow that flow and nothing wider, and save the ruleset | [NAT gateway lab](https://mitsurutech.co.uk/help/lab-nat-gateway) |

The network: a LAN host on `10.0.0.0/24`, the gateway between it and an outside network, `203.0.113.0/24` (a range set aside for documentation), and an outside server that answers every request with the address it saw it come from. The outside has no route back to the LAN, as on the internet. The lab leaves a README in the student's home folder that says what to do and gives none of the answers.

It has a **check**, a script that marks a handed-in VM without the teacher reading it. It runs on a copy of the saved machine after a fresh start, so it sees only what the student saved in `/etc/nftables.conf`, and **it marks what the network does and not how the rules are spelt**: a request from the LAN host must reach the outside server, which must have seen the gateway's outside address; a request from the outside to the forwarded port must reach the LAN page, which must have seen the client's own address (so a proxy on the gateway does not pass); the other LAN page and every other port must stay shut; and the gateway's forward chain must still drop what it was not told to allow.

## What is in the file

The `.lab.yaml` is a whole **VM lab definition**, exactly what the platform runs:

| Key | What it is |
| --- | --- |
| `cpus`, `memory`, `disk` | the size of each student's VM: 1 CPU, 1024 MB and 10 GB |
| `open` | the ports that open in the browser: none, the lab is the terminal |
| `setup` | a script run once, as root, the first time each student's VM starts: it installs `nftables`, `conntrack`, `curl` and a few tools from Ubuntu's archive in about ten seconds, writes the `netlab` command, the topology, the starting `/etc/nftables.conf` and the lab's README |
| `check` | the marking script, run as root on a copy of the saved disk after a fresh boot; its exit status is the result and `0` passes |

The `netlab` command it installs: `netlab status` lists the machines, `netlab shell gw` opens a shell inside one, and `netlab reset` puts the starting rules back and starts the lab again.

## Using it on the platform

**To set it as work for a class:**

1. In your module, add a task and choose the **Ubuntu VM lab** card.
2. Press the NAT gateway sample, or paste the contents of the file here. Using a sample replaces the text, and Undo brings yours back.
3. Publish and try your own copy first. Every student gets a VM of their own, kept between sessions.
4. Assign it like any task. When it is handed in, run the check on a student's row in the class list: it starts a copy of their saved machine that is not saved back, and shows whether it passed and what it printed.

The help manual's [guide to Ubuntu VM labs](https://mitsurutech.co.uk/help/labs) and its [sample labs page](https://mitsurutech.co.uk/help/sample-labs) cover the rest.

No college account yet? [Your first year is free](https://mitsurutech.co.uk/pricing) for up to 50 students, and [signing up](https://mitsurutech.co.uk/signup) takes an email address.

## Checking the file

On every push, `.github/workflows/check.yml` checks that the file is valid YAML with the five keys above and that `setup` and `check` are each inside the platform's 16 KB limit. It does not boot a VM: that is done on the platform's own machines.

## What does not work in a lab

The VM reaches Ubuntu's package archive, PyPI, npm and GitHub, and nothing else, so a lab cannot be pointed at a host on the internet. It is one VM, so the "outside" is a namespace inside it and not the real internet.

## About

Made by [Mitsuru Technologies](https://mitsurutech.co.uk) for the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning): Python, Java, Go, Node, SQL and Linux in the browser for schools and colleges, with container labs, VM labs, web apps deployed from GitHub, and quizzes that mark themselves. Try code with no account in the [playground](https://mitsurutech.co.uk/playground).

More labs from the same place: [lab-netlab](https://github.com/mitsurutech/lab-netlab) (static routing, OSPF and BGP), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) and [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licence

The lab definition, the `netlab` tool and this documentation are MIT licensed, see [LICENSE](LICENSE): copy them, change them, teach with them. nftables and conntrack-tools are GPL-2.0 licensed and are installed from Ubuntu's archive when a VM starts; nothing of them is distributed here.
