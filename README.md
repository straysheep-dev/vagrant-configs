# Vagrant

*Like Ansible, but for provisioning Virtual Machines, such as VMware, VirtualBox, qemu, or Hyper-V.*


Useful links to get started:

- [Vagrant Docs: Quick Start Tutorials](https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-index)
- [Kali Docs: Kali Vagrant Guest](https://www.kali.org/docs/virtualization/install-vagrant-guest-vm/)
- [Vagrant Docs: Intro to Ansible](https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro)
- [Ansible Docs: Vagrant Guide](https://docs.ansible.com/ansible/latest/scenario_guides/guide_vagrant.html)


Install
=====

Validate your install with one of these public keys:

- [hashicorp public keys](https://www.hashicorp.com/trust/security) `C874 011F 0AB4 0511 0D02 1055 3436 5D94 72D7 468F`
- [hashicorp public keys on keybase.io](https://keybase.io/hashicorp) `C874 011F 0AB4 0511 0D02 1055 3436 5D94 72D7 468F`
- [hashicorp gpg apt key](https://apt.releases.hashicorp.com/gpg) `798A EC65 4E5C 1542 8C8E 42EE AA16 FCBC A621 E701`
- [How to Verify a Hashicorp Binary](https://developer.hashicorp.com/well-architected-framework/operational-excellence/verify-hashicorp-binary)

[Download](https://developer.hashicorp.com/vagrant/downloads) the binaries for Windows and macOS.

Ubuntu / Debian install:
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```


Usage
=====

Kali provides the most straight forward [examples](https://www.kali.org/docs/virtualization/install-vagrant-guest-vm/) for getting started and what Vagrant can achieve.

The commands below do the following:

1. Obtain the Vagrantfile from [Kali's listings on Vagrant Cloud](https://app.vagrantup.com/kalilinux/boxes/rolling) and initialize this directory for a new VM
2. Downloads the `.box` file from Vagrant Cloud if it's missing or a new version is available, and starts the VM using the default provider installed (usually VirtualBox, VMware or Hyper-V)
	- If you already have the box specified in the Vagrantfile as `config.vm.box` available locally, `vagrant up` will provision, register, and start the VM without downloading anything
	- Basically with the `.box` available, and a Vagrantfile, you can skip `vagrant init`
3. Gracefully powers off the VM

```bash
mkdir -p ~/vagrant/kali; cd ~/vagrant/kali
vagrant init kalilinux/rolling
vagrant up
vagrant halt
```

***The `.box` file is a template for creating separate VM's. Create a new directory and Vagrantfile for each Virtual Machine.***


## How it Works

When you run `vagrant up`:

- If a `.box` file wasn't previously imported manually, and no URL is provided, Vagrant will attempt to search the public Vagrant Cloud and download it
- Vagrant checks if a new version of the `.box` file is available in Vagrant Cloud and if so, downloads it (this is skipped if you imported a box locally or from a URL)
	- *NOTE: if Vagrant tries to download a `.box` file, you'll see this on the command line
- [Calculates and compares the box checksum to the metadata files](https://developer.hashicorp.com/vagrant/docs/cli/box#options-for-direct-box-files) unless you specified a checksum
- Forwards `localhost:2222/tcp` of the host to `localhost:22/tcp` of the guest VM for ssh access and provisioning access
- Automatically mounts the cwd `.` containing the Vagrantfile and any subdirectories, in the guest at `/vagrant` with read / write access (you should disable write access)
- Starts the provider's GUI window of the VM
- You can then run `vagrant ssh` to open an ssh session into the guest

How Vagrant itself works:

- The `.box` file is an archive that contains the `.vmdk` and metadata files used as a template for every VM you create using that box as a base.
- All invocation of the `vagrant` command should be done from the directory of the Vagrantfile. Otherwise you must specifiy a machine ID.

> [For boxes from HashiCorp's Vagrant Cloud, the checksums are embedded in the metadata of the box. The metadata itself is served over TLS and its format is validated.](https://developer.hashicorp.com/vagrant/docs/cli/box#options-for-direct-box-files)

Obtain the status (includes machine ID) of all known Vagrant machines on the system:
```bash
vagrant global-status
```


## Boxes

Boxes are the `.box` files used as templates to build VM's. [They're tarballs containing four components including the `.vmdk` virtual disk file itself](https://developer.hashicorp.com/vagrant/docs/boxes/format).

- [Vagrant Docs: Install a Box](https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-boxes)
- [Vagrant Docs: Start a Box](https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-up)

Boxes are stored per-user at the default location of `~/.vagrant.d/boxes/` on both Linux and Windows.

This is a condensed overview of the lifecycle of commands you might run affecting boxes and VM provisioning. Most importantly: 

- Boxes are *not* VM's, they're templates to build VM's from locally
- Deleting (destroying) a VM does not harm or remove the configuration files

```bash
vagrant box add hashicorp/bionic64       # Downloads the box from Vagrant Cloud if a local .box file is not specified
vagrant up                               # Start the VM
vagrant ssh                              # SSH into the VM
vagrant halt                             # Shutdown the VM
vagrant destroy                          # Delete the VM from your hypervisor (based on a Vagrantfile in cwd, or specify an ID)
vagrant box list                         # List all boxes (remember, these are templates, not the VM, "vagrant destroy" deletes the VM
vagrant box remove hashicorp/bionic64    # Remove a box (aka a VM template) from your local Vagrant environment
```


### Sourcing Boxes

When using Vagrant's Cloud, keep in mind anyone can upload a box. [The users `bento` and `ubuntu` are generally trusted](https://developer.hashicorp.com/vagrant/vagrant-cloud/boxes/catalog#choosing-the-right-box). It's important to ensure the username space on Vagrant's Cloud is owned by who you expect it to be. Similar to sourcing OS ISO files, check each distro's official releases for Vagrant boxes or documentation. Similarly if there's a trusted user (like `bento`) uploading boxes, double check any documentation to ensure the username matches.

- [Kali](https://www.kali.org/docs/virtualization/install-vagrant-guest-vm/)
- [Ubuntu](https://cloud-images.ubuntu.com/)
- [Debian](https://wiki.debian.org/Teams/Cloud/VagrantBaseBoxes)


### Manually Adding Boxes

The best way to demonstrate this is with [Ubuntu cloud's Vagrant images](https://cloud-images.ubuntu.com/). Download the following files just like you would an Ubuntu ISO.

- [SHA256SUMS](https://cloud-images.ubuntu.com/jammy/current/SHA256SUMS)
- [SHA256SUMS.gpg](https://cloud-images.ubuntu.com/jammy/current/SHA256SUMS.gpg)
- [jammy-server-cloudimg-amd64-vagrant.box ](https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64-vagrant.box)

See these pages for more detail on the following `box add` command:

- [`box add`](https://developer.hashicorp.com/vagrant/docs/cli/box#box-add)
- [`config-vm-box_url`](https://developer.hashicorp.com/vagrant/docs/vagrantfile/machine_settings#config-vm-box_url)

```bash
vagrant box add --name 'my-local-box' file://home/dev/vagrant/boxes/custom.box
```

## Snapshots

Snapshots can be managed in the following ways:

```bash
vagrant snapshot list
vagrant snapshot save [<vm-name>] <snapshot-name>
vagrant snapshot restore [<vm-name>] <snapshot-name> [--no-start]
vagrant snapshot delete [<vm-name>] <snapshot-name>
```

If you're in a directory with a Vagrantfile, you do not need to specify the VM name or ID.

There's also `snapshot push|pop` which are more for temporary usage, as you need to include `--no-delete` when `pop`ing (restoring) a previous snapshot. Otherwise it restores the previous state and deletes the latest snapshot.


## Provisioning

Automation is at the heart of Vagrant with the following provisioning options:

- Shell commands
- Ansible / Chef / Puppet / Salt / Docker integration

Provisioning normally only happens during the first boot of the VM. However, [as demonstrated at the bottom of the Kali docs on Vagrant usage](https://www.kali.org/docs/virtualization/install-vagrant-guest-vm/), provisioning can occur in the following ways manually.

```bash
vagrant provision           # (re)provision the powered on VM after making configuration changes
vagrant up --provision      # when VM is powered off, power it on then provision
vagrant reload --provision  # reboot the VM then provision
```

The configuration section example below shows how to include shell commands and ansible plays in a Vagrantfile.


Configuration
===========

- The Vagrantfile directory and all subdirectories are shared with the guest and writable by default, but can be disabled with `config.vm.synced_folder ".", "/vagrant", disabled: true`
- You can reload a modified configuration file into a running VM with:

```bash
vagrant reload
```

- [Vagrant Docs: Shell Provisioner](https://developer.hashicorp.com/vagrant/docs/provisioning/shell)
- [Vagrant Docs: Ansible Provisioner](https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro)
- [Vagrant Docs: Provider Configuration](https://developer.hashicorp.com/vagrant/docs/providers/configuration)

You could script updating a VM, taking a new snapshot, and deleting the oldest snapshot, to occur on a scheduled cycle via cron.

The Vagrantfile could include the following lines in the provisioning section:
```conf
  config.vm.provision "shell", inline: <<-SHELL
    export PATH=$PATH:/usr/bin
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get full-upgrade -yq
    apt-get autoremove --purge -yq
  SHELL
```

The script could look like:
```bash
vagrant up --provision
vagrant halt
vagrant snapshot save vm-name "snapshot-$(date +%F)"
vagrant snapshot delete vm-name "$(vagrant snapshot list vm-name | tail -n 2 | head -n 1)"
```

Here's an example snippet from a Vagrantfile for Kali Linux using VirtualBox as the provider:

```ruby
...

  # VirtualBox provider configuration:
  config.vm.provider "virtualbox" do |vb|
    vb.name = "kali-vagrant"
    vb.memory = "8192"
    vb.cpus = "6"
    vb.customize ["modifyvm", :id, "--clipboard-mode", "hosttoguest"]
    vb.customize ["modifyvm", :id, "--drag-and-drop", "hosttoguest"]
    #vb.customize ["modifyvm", :id, "--cable-connected1", "off"]
  end

...
```

The `vb.customize` lines allow you to make adjustements specific to the VirtualBox hyperviser. Each hyperviser (provider) has customization options like this.


Providers
========

*Providers basically means hypervisors.*


## VirtualBox

This is the most common provider, most boxes you find on Vagrant's public cloud will (only) support VirtualBox.


## Hyper-V

[Working with Hyper-V boxes has a few differences](https://developer.hashicorp.com/vagrant/docs/providers/hyperv/limitations):

- You need to run `vagrant.exe` from an Administrator shell
- You'll be prompted to select a network adapter on first boot (change this later in the Hyper-V manager)
- Vagrantfile network configurations are ignored
- Vagrant cannot enforce a static IP
	- You can however, do this with the provisioner section of the Vagrantfile
- Hyper-V's built in Default Switch (NAT network) changes its subnet range and gateway on each reboot
- Vagrant *will* report the VM's IP on each boot if it can reach it
- Vagrant *should* be able to ssh into the VM, even without an IPv4 address, using the IPv6 link-local address range of fe80::/10

*If you have multiple virtual network adapters in Hyper-V, choose the **Default Network switch**, or **any internal switch the host can reach**, during the initial boot and provisioning of the guest. Just don't use an external adapter (not safe considering `vagrant:vagrant` are the default ssh credentials) or private adapter (only VM's can communicate on private network adapters, the host can't).*

- This allows Vagrant to `vagrant ssh` into the guest (likely using IPv6 link-local scope fe80::/10 addressing if DHCP is not working on the Default Switch)
- You can connect a second adapter with additional network access to the guest in the Hyper-V manager once Vagrant finishes creating the VM


### Hyper-V Usage Quirks

In case you plan to move a Vagrant guest to a network where `vagrant ssh` cannot reach it:

- Provision the guest initally in a network the host can reach
- Move the provisioned guest to the desired network adapter after provisioning
- You can still start the guest, manage snapshots, and other things using Vagrant in these cases
- Shut the guest down manually instead of using `vagrant halt` if `vagrant ssh` cannot reach the guest


### Hyper-V Provider Configuraiton

Example Hyper-V Provider section in a Vagrantfile:

```ruby
  # Hyper-V provider configuration:
  config.vm.provider "hyperv" do |hv|
    hv.vmname = "kali-rolling-vagrant"
    hv.cpus = "4"
    hv.memory = "8192"
    #hv.ip_address_timeout = 240
    hv.enable_enhanced_session_mode = true
    hv.enable_virtualization_extensions = false
    hv.vm_integration_services = {
      guest_service_interface: false,
      heartbeat: false,
      key_value_pair_exchange: false,
      shutdown: false,
      time_synchronization: false,
      vss: false
    }
  end
```


### Hyper-V Mounting Shared Folders

[Hyper-V uses SMB as it's default shared folder mechanism](https://developer.hashicorp.com/vagrant/docs/synced-folders/smb).

- Creates an SMB share in the specified directory as admin
- Requires your host's local admin account credentials be passed to Vagrant
- You may need to specify an `smb_host` IP address

Example line using `smb_host`:

```ruby
config.vm.synced_folder "./provisioning", "/vagrant/provisioning", :mount_options => ["ro"], smb_host: "10.55.55.1"
```

To clean up any leftover Vagrant SMB shares:

```powershell
Get-SmbShare | where {$_.Name -imatch "vgt-*"} | Remove-SmbShare
```


### Hyper-V Ansible Provisioner

[Ansible support is limited to WSL on a Windows host](https://docs.ansible.com/ansible/latest/os_guide/windows_faq.html#windows-faq-ansible).

An easy work around is to use the shell provisioner to install ansible within the guest, clone or retrieve any playbooks, and run them from there.


### Hyper-V Networking

*This assumes your Windows host's firewall is set to `Set-NetFirewallProfile -All -AllowInboundRules Block`, which blocks ALL inbound traffic, even if a rule exists.*

- [Create a static NAT network](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network), this way you have a predictable subnet that does not change on reboot how Hyper-V's Default Switch does
- Set the IP, route, and interface information using the provisioner section of the Vagrantfile based on that custom NAT network's information
- This is an easy way to automate and avoid networking issues in Hyper-V

If you require special firewall rules, keep in mind this adapter can be mentioned in rules specifically, by providing the name of the adapter as the argument to `-InterfaceAlias` when executing the `Get-NetAdapater`, `New-NetFirewallRule` cmdlets.


#### Hyper-V Networking: Create a Static NAT Network

- [Set up a NAT network | Hyper-V User Guide](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network)

This helps resolve issues with the Default Switch changing it's subnet range and gateway on each reboot of the host.

[As long as `Get-NetNat` returns no existing NAT interfaces](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network#multiple-nat-networks-are-not-supported), you can create a custom NAT switch and network using the following:

- 10.55.55.1/24 is chosen since it's relatively obscure and unique, meaning it won't collide with common home wifi, office, or Hyper-V subnets
- SOHO routers will often default to the range of 192.168.0.0/16
- Hyper-V tends to use the subnet range of 172.16.0.0/12 
- VPNs and corporate networks will use the range of 10.0.0.0/8, be sure to review any existing network configurations or requirements before using 10.55.55.1/24

```powershell
New-VMSwitch -SwitchName "CustomNATSwitch" -SwitchType Internal
$ifindex = (Get-NetAdapter -IncludeHidden | where { $_.Name -eq "vEthernet (CustomNATSwitch)" }).ifIndex
New-NetIPAddress -IPAddress 10.55.55.1 -PrefixLength 24 -InterfaceIndex $ifindex
New-NetNat -Name CustomNATNetwork -InternalIPInterfaceAddressPrefix 10.55.55.0/24
```

To remove the NAT network and switch:

```powershell
Get-NetNat | where { $_.Name -eq "CustomNATNetwork" } | Remove-NetNat
Get-VMSwitch | where { $_.SwitchName -eq "CustomNATSwitch" | Remove-VMSwitch
```

If you want to be able to reach VM's on this switch from the host or WSL, you may need to enable forwarding. Just remember, the Windows host is likely filtering `ping` packets. Try `ssh`, [`Test-NetConnection`](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection?view=windowsserver2022-ps), [`nmap`](https://nmap.org/), or [`naabu`](https://github.com/projectdiscovery/naabu) to verify connectivity. 

Forwarding can be enabled or disabled per interface with the following:

```powershell
# Apply
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Enabled

# Remove
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Disabled
```

#### Hyper-V Network Troubleshooting

I've needed to do the following during the [`hashicorp/bionic64` tutorial](https://developer.hashicorp.com/vagrant/tutorials/getting-started):

- Set an internal switch (it can be any internal switch the host can reach) as the initial network adapter
- This is so `vagrant ssh` can reach the guest over link-local IPv6 (fe80::/10) if DHCP fails on the Default Switch
- Add a secondary network adapter that has desired network access (e.g., to the public internet or to another internal network)
- Run `sudo ip link set eth1 up; sudo dhclient -v` for the second adapter to pick up an IP address via DHCP

In some cases Vagrant could never find the link-local IPv6 address. In the event this happens, you can log in manually to the VM once and set a static configuration with either `netplan` or NetworkManager using `nmcli`.

#### Linux Guest: netplan Static Network Configuration

On hosts without `nmcli` you can use netplan if it's available. Edit an existing file under `/etc/netplan/`:

```bash
sudo nano /etc/netplan/01-some-file.yml
```

Replace contents with the following:

```conf
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      dhcp6: false
      addresses:
        - 10.55.55.28/24
      nameservers:
        addresses: [1.1.1.1, 9.9.9.9]
      routes:
        - to: default # Use `to: 0.0.0.0/0` on Ubuntu 18.04
          via: 10.55.55.1
```

See [this stack exchange post](https://unix.stackexchange.com/questions/688152/raspberry-pi-route-not-working-when-using-to-default-with-netplan) regarding use of `0.0.0.0/0` instead of `default`. This appears to be an issue with Ubuntu 18.04.


#### Linux Guest: nmcli Static Network Configuration (Recommended)

*With this method, the guest will still obtain DHCP leases on networks that provide it. Otherwise it will always have the static IP configuration available on it's `eth0` interface for use with the static NAT network.*

- [Red Hat: Configure IP Networking with nmcli](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli#sec-Adding_and_Configuring_a_Static_Ethernet_Connection_with_nmcli)

To enable and disable an interface:

```bash
nmcli device status         # Get the status of all network devices
nmcli con up id eth0        # Enable the profile named "eth0"
nmcli dev disconnect eth0   # Disable the profile named "eth0"
```

To configure an interface profile from scratch with `nmcli` (this follows Red Hat's example, swapping in our own information):

```bash
nmcli con add type ethernet con-name test-lab ifname eth0 ip4 10.55.55.11/24 gw4 10.55.55.1 #[ip6 abbe::cafe gw6 2001:db8::1]
nmcli con mod test-lab ipv4.dns "1.1.1.1 1.0.0.1"
nmcli con mod test-lab ipv6.dns "2606:4700:4700::1111 2606:4700:4700::1001"
nmcli con up test-lab ifname eth0
nmcli device status
nmcli -p con show test-lab
```

If you get the error that "device is strictly unmanaged" check `/etc/NetworkManager/` and `/etc/NetworkManager/conf.d`.

- [Red Hat: Permanently Setting a Device as Unmanaged](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli#permanently-configuring-a-device-as-unmanaged-in-networkmanager)

Look for any lines under `[keyfile]` or `[ifupdown]` sections in files that might be setting interfaces as unmanaged or strictly managed.

In Kali, you need to change `managed=false` to `managed=true` in `/etc/NetworkManager.conf` to use `nmcli`.

To delete a network configuration:

```bash
nmcli show
nmcli con delete id <profile-name>  # Alternatively, use `uuid <profile-uuid>` instead of `id <profile-name>`
```


Windows, AD, Home Labs
=====================

The [GOAD (Game of AD) project](https://github.com/Orange-Cyberdefense/GOAD) uses Windows boxes prepared by [StefanScherer/packer-windows](https://github.com/StefanScherer/packer-windows), which are referenced in the [Microsoft documentation for deploying Vagrant images](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/manage/hybrid/server/best-practices/local-vagrant-windows) locally. The [Vagrantfile](https://github.com/microsoft/azure_arc/blob/main/azure_arc_servers_jumpstart/local/vagrant/windows/Vagrantfile) is available on Microsoft's [azure_arc GitHub](https://github.com/microsoft/azure_arc).

Windows versions from 2016 / 10 to 2022 / 11 are available on [Stefan Scherer's Vagrant Cloud](https://app.vagrantup.com/StefanScherer).