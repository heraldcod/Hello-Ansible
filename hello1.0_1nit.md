# hello1.0_1nit.yml

*Hello, Ansible — Episode 1*

---

> *3 AM. E Corp tower. Forty-seven switches. One typo. Half the building goes dark. That's when he showed up. "Kiddo, there's a tool. You write once. They all listen." He called it Ansible.*
>
> *Welcome to fsociety, friend. Let's take back control. — Elliot*

---

Hello, friend.

That's lame, right? I know. But here's the thing — you're reading this, which means you're tired. Tired of clicking through GUIs. Tired of SSH-ing into boxes one by one. Tired of doing the same thing a hundred times and calling it "work."

You want control. Real control. The kind where you type once and machines listen.

Welcome to **Hello, Ansible** — a series about network automation. Not theory. Not slides. Actual control over actual devices.

---

## What Is This Series?

This is **Hello, Ansible**. A hands-on journey through network automation using Ansible.

No fluff. No "in this tutorial, we will learn..." garbage. You'll build, break, and fix things. By the end, you'll automate networks like you've been doing it for years.

Oh, and this series has a certain... *tone*. If you've watched Mr. Robot, you'll feel at home. If you haven't, just know we name things after characters, quote Elliot, and treat automation like a quiet revolution. Welcome to fsociety.

---

## What Is Network Automation?

You manage a network. Routers. Switches. Firewalls. Maybe ten devices. Maybe a thousand.

Every time something changes — a VLAN, an ACL, an IP address — you SSH in. You type commands. You copy-paste configs. You pray you didn't typo.

Now multiply that by every device. Every change. Every audit. Every compliance check.

That's not engineering. That's labor.

**Network automation** is writing code that does this for you. Consistently. Repeatedly. Without typos. Without forgetting a device. Without burning out.

You define *what* you want. The machine figures out *how* to make it happen.

---

## What Is Ansible?

Ansible is an automation engine. Open source. Written in Python. Built by Red Hat.

You write simple YAML files — called **playbooks** — that describe the state you want. Ansible connects to your devices and makes it so.

No agents to install on devices. No complex setup. Ansible connects using what the device already speaks:

| Connection | Used For |
|------------|----------|
| **SSH** | Linux servers, network CLI |
| **NETCONF** | Structured config over SSH (Juniper, some Cisco) |
| **REST API** | Modern devices with HTTP APIs |
| **WinRM** | Windows machines |

No special software on the targets. Just protocols they already understand.

It was built for servers first, but the network automation community adopted it. Now it speaks to Cisco, Juniper, Arista, Palo Alto, F5 — basically anything with a CLI or API.

---

## Why Ansible for Networks?

| Feature | Why It Matters |
|---------|----------------|
| **Agentless** | No software on network devices. Just SSH/NETCONF/API. |
| **YAML syntax** | Human-readable. Version-controllable. No programming degree required. |
| **Idempotent** | Run it once or a hundred times — same result. No duplicate configs. |
| **Multi-vendor** | One tool for Cisco, Juniper, Arista, and more. |
| **Collections** | Pre-built modules for network devices. Don't reinvent the wheel. |
| **Community** | Massive. Active. Someone already solved your problem. |

---

## The Control Machine

Ansible needs a home. A place to run from. That's the **control machine** (or control node).

This is your laptop. Your workstation. The machine where you write playbooks and execute them.

```
┌─────────────────┐
│ Control Machine │  ← Ansible lives here
│  (Your Laptop)  │  ← Playbooks live here
└────────┬────────┘
         │
         │ SSH / API
         ▼
┌─────────────────┐
│ Network Devices │  ← No Ansible here
│ (Routers, etc.) │  ← Just receive commands
└─────────────────┘
```

**Key point:** Ansible runs *on* the control machine, not on the network devices. The devices just receive commands and respond. They don't know Ansible exists.

Requirements for a control machine:
- Linux or macOS (Windows needs WSL)
- Python 3
- Network connectivity to target devices

That's it. Your laptop is the command center.

---

## How Ansible Talks to Network Devices

Traditional Ansible SSHs into a Linux box and runs commands *on that box*.

Network devices are different. You can't install Python on a switch. So Ansible runs everything locally on the control machine and sends commands *to* the device over:

| Method | How It Works |
|--------|--------------|
| **network_cli** | SSH to the device, send CLI commands. Like you typing, but automated. |
| **netconf** | XML-based API. Structured. Supported by Juniper, some Cisco. |
| **httpapi** | REST API. Modern devices with web APIs. |

For this series, we'll mostly use **network_cli** — SSH to Cisco IOS devices. The way most real networks work.

---

## The Approach

Theory is useless without practice. You can't learn to ride a bicycle by looking at it. You climb on. You fall. You get back up.

This series is hands-on from day one. You'll need:
- A laptop
- An internet connection
- A free lab environment (we'll set that up)

No expensive hardware. No VMs eating your RAM. Just you and the cloud.

Let's ride.

---

## The Problem

"I want to learn, but I don't have a lab."

The barrier they want you to believe exists. Expensive hardware. Enterprise licenses. A rack in your garage.

It's a lie.

You need:
- A laptop
- Internet
- A free Cisco DevNet account

That's the whole conspiracy. Let's exploit it.

---

## Your Machine

**Linux / macOS** — You're already on the right side. Skip to Ansible.

**Windows** — You're running their OS, but you can still break free. WSL2 gives you Linux inside the cage.

Open PowerShell as Administrator:

```powershell
wsl --install    # Enables WSL2 and installs Ubuntu by default
```

Restart. Ubuntu installs. Open it from the Start menu.

Welcome to Linux, friend. The revolution starts here.

---

## Ansible

Modern Ubuntu won't let you install Python packages system-wide. It protects itself. Smart, but annoying.

The workaround: a virtual environment. An isolated space where you can install whatever you want.

```bash
sudo apt update                       # Update package list
sudo apt install python3-pip python3-venv -y   # Install pip3 and venv
python3 -m venv ~/fsociety            # Create virtual environment
source ~/fsociety/bin/activate        # Activate it — prompt changes to (fsociety)
pip install ansible                   # Install Ansible inside the venv
```

Verify you're armed:

```bash
ansible --version                     # Shows version, config path, python location
```

Output. Version numbers. Paths. You're in.

**Important:** Every time you open a new terminal, activate the venv first:

```bash
source ~/fsociety/bin/activate        # Always run this before using Ansible
```

---

## ansible vs ansible-core

| Package | What |
|---------|------|
| `ansible-core` | The engine. Stripped. Minimal. |
| `ansible` | Engine + collections. Everything you need to talk to network devices. |

We installed `ansible`. Batteries included. No extra downloads. We move fast.

---

## The Lab: Cisco DevNet CML

Cisco runs a free cloud lab. Real routers. Real switches. Real IOS. They call it a "sandbox." I call it our playground.

1. Go to [developer.cisco.com](https://developer.cisco.com)
2. Create an account — they just want an email
3. Go to [devnetsandbox.cisco.com](https://devnetsandbox.cisco.com)
4. Search **CML**
5. Reserve it

An email arrives:
- VPN URL
- Credentials
- The IP address to the control center

They're handing you the keys. Take them.

---

## Connect

Download **Cisco AnyConnect**. It's their VPN client. Yeah, we're using their tools to access their systems. Poetic.

Connect using the VPN details.

Once you're through, open your browser. Enter the CML IP.

You're looking at the control center. A default lab is waiting. Start it.

The nodes light up. The connections form. The city wakes.

We're in.

---

## The Topology

Four devices. Pre-configured. Waiting.

We're going to name them. Because devices without names are just IP addresses, and IP addresses are forgettable.

| Device | Hostname | The Reference |
|--------|----------|---------------|
| Router 1 | **tyrell** | Ambitious. Always routing toward power. |
| Router 2 | **whiterose** | Controls the paths. Time is everything. |
| Switch 1 | **darlene** | The connector. Holds the group together. |
| Switch 2 | **angela** | Between both worlds. |

---

## The Catch: SSH

Here's what they don't tell you upfront.

These devices come with telnet enabled. Telnet — unencrypted, ancient, trusting. A protocol from a time before paranoia was survival.

Ansible *can* speak telnet. But we won't. We're better than that.

SSH. Encrypted. Authenticated. Secure. The way it should be.

We need to teach these devices the new language.

Right-click on a device. Open console. You're inside.

**tyrell (R1):**

```
configure terminal                      ! Enter global config mode
hostname tyrell                         ! Set device name
ip domain name ecorp.local              ! Domain needed for RSA key generation
crypto key generate rsa modulus 2048    ! Generate 2048-bit RSA keys (enables SSH)
ip ssh version 2                        ! Use SSH version 2 only (more secure)
username elliot secret fsociety         ! Create local user with encrypted password
line vty 0 4                            ! Configure virtual terminal lines (remote access)
 login local                            ! Use local username/password for auth
 transport input ssh                    ! Allow only SSH (no telnet)
end                                     ! Exit config mode completely
write memory                                  ! Save config to NVRAM
```

Repeat on **whiterose**, **darlene**, and **angela**. Same commands, just change the hostname.

Verify on each:

```
show ip ssh                             ! Confirm SSH is enabled and version 2.0
```

SSH version 2.0 enabled. The secure door is open. But only for those who know the knock.

---

## The Test

Back to your terminal. Time to make contact.

### What Are Modules?

Ansible doesn't do anything by itself. It uses **modules** — small units of code that perform specific tasks.

Think of modules as tools in a toolbox:
- `cisco.ios.ios_facts` — gather information from a Cisco IOS device
- `cisco.ios.ios_config` — push configuration to a Cisco IOS device
- `cisco.ios.ios_command` — run show commands on a Cisco IOS device
- `ansible.builtin.ping` — test connectivity (not ICMP, just Ansible reachability)

You tell Ansible *which module* to use, and the module does the actual work.

For network devices, modules come in **collections** — bundles organized by vendor:
- `cisco.ios` — Cisco IOS modules
- `cisco.nxos` — Cisco Nexus modules
- `arista.eos` — Arista modules
- `junipernetworks.junos` — Juniper modules

When we installed `ansible` (not `ansible-core`), these collections came bundled. Ready to use.

---

### The Ad-Hoc Command

An **ad-hoc command** is a one-liner. No playbook. No files. Just a quick command to do one thing.

Open your terminal (Ubuntu/WSL). Make sure you're in the fsociety venv:

```bash
source ~/fsociety/bin/activate
```

Get all four device IPs from the CML topology canvas.

```bash
ansible all -i <ip1>,<ip2>,<ip3>,<ip4>, -c ansible.netcommon.network_cli -u elliot -k -m cisco.ios.ios_facts -e ansible_network_os=cisco.ios.ios
```

Password when prompted: `fsociety`

**Breaking it down:**

| Flag | What It Does |
|------|--------------|
| `ansible` | The command itself |
| `all` | Target all hosts in inventory |
| `-i <ip1>,<ip2>,<ip3>,<ip4>,` | **Inventory** — all four devices, comma-separated. Trailing comma required. |
| `-c ansible.netcommon.network_cli` | **Connection plugin** — SSH to network CLI |
| `-u elliot` | **Username** — the user we created |
| `-k` | **Ask for password** — prompts you |
| `-m cisco.ios.ios_facts` | **Module** — gathers device facts (hostname, IOS version, interfaces, etc.) |
| `-e ansible_network_os=cisco.ios.ios` | **Variable** — tells Ansible it's Cisco IOS |

Four devices. One command. All respond.

Watch.

Facts flood back. Interfaces. IOS version. Hostname. Memory. Serial numbers. Everything the devices know about themselves, delivered to your screen.

One command. No clicking. No GUI. No manual labor.

You just interrogated four machines at once, friend. And they all answered.

---

## What Just Happened

```
Your laptop → VPN → CML → 4 Devices → SSH → Ansible → Facts returned
```

You reached through the internet, through their VPN, into their cloud, found four devices, asked them who they are, and they all answered. At once.

That's not a tutorial. That's power.

---

## What's Next

You ran an ad-hoc command. Quick. Direct. Proof of concept.

But ad-hoc commands are improvisation. Next, we write a proper playbook — structured, repeatable, version-controlled.

The difference between a hack and a system.

---

## Ending the Lab

When you're done, stop the lab in CML. Resources aren't infinite — even in the cloud.

Click **Stop** on your lab. Disconnect the VPN. Close the terminal.

The city sleeps. For now.

---

## Stuck?

Lab not working? SSH refused? Ansible throwing errors?

It happens. Reach out — leave a comment, open an issue, find me on socials. We'll debug it together.

No one learns alone.

---

## Quick Reference

| What | Value |
|------|-------|
| Sandbox | [devnetsandbox.cisco.com](https://devnetsandbox.cisco.com) |
| Ansible Network Docs | [docs.ansible.com/network](https://docs.ansible.com/projects/ansible/latest/network/index.html) |

---

*Next: hello1.1_1nventory.ini*

---
