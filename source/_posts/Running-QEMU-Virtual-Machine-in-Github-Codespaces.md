---
title: Running QEMU Virtual Machine in Github Codespaces
date: 2024-07-27 12:21:13
tags:
  - Tutorial
categories:
  - Linux
---
Create a new codespace:

```bash
gh codespace create
```

Generate SSH config:

```bash
gh codespace ssh --config > ~/.ssh/codespaces
printf 'Match all\nInclude ~/.ssh/codespaces\n' >> ~/.ssh/config
```

Connect to the codespace:

```bash
ssh cs.<codespace name>.<branch>
```

Run these commands in your codespace:

```bash
# Install QEMU
sudo apt install qemu-system-x86
# Set up kvm group
sudo groupadd -rg 109 kvm
sudo adduser codespace kvm
```

Relogin(or you will get permission denied on `/dev/kvm`) and run QEMU on the codespace:

```bash
qemu-system-x86_64 -enable-kvm -display vnc=:0 <other parameters>
```

Open SSH tunnel in a new terminal:

```bash
ssh cs.<codespace name>.<branch> -L <local port>:localhost:5900 -N
```

Finally, use a VNC client to connect to `127.0.0.1:<local port>`.
