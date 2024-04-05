# vagrant-kali

Vagrantfile template for Kali Linux, ready for use with Hyper-V and VirtualBox.

- The default `/vagrant` folder is not automatically mounted
- 6 vCPUs
- 8192 RAM
- The shell provisioner upgrades and autoremoves system packages
- Hyper-V: Ehanced session mode enabled
- Hyper-V: VM integration services all disabled
- VirtualBox: Clipboard + drag-and-drop are set to `hosttoguest`

Hyper-V is the default provider. Just comment the Hyper-V section and uncomment the VirtualBox section to change this.

Share a `provisioning/` folder containing your playbooks or roles from the current directory to run ansible from the Vagrant host by uncommenting the following lines:

```
  #config.vm.synced_folder "./provisioning", "/vagrant/provisioning", :mount_options => ["ro"]
...
  #config.vm.provision "ansible" do |ansible|
  #  ansible.playbook = "provisioning/playbook.yml"
  #end
```