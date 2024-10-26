# vagrant-templates

## How to Install Ubuntu Server from ISO image using Vagrant

There are 2 Vagrantfile available:

1. `Vagrantfile` is for installation of Ubuntu Server 24-04 from ISO image
2. `Vagrantfile-2` is used after Ubuntu Server installation to boot from the virtual hard disk where the OS installed and configure SSH connection.

### Important Notes

1. The basic structure of a Vagrantfile in Vagrant, which defines the configuration for setting up virtual machines.

```
Vagrant.configure("2") do |config|
  # configuration code goes here
end

```

2. General settings (`config.vm.box`, `config.vm.network`, `config.vm.ssh`) are defined at the top level in `config.vm` to keep them consistent across all providers.

3. Vagrant supports multiple providers (e.g., VirtualBox, VMware, Docker), therefore provider-specific settings go in `config.vm.provider`, as they may differ between VirtualBox, VMware, and other providers. The code block below applies only if you are using VirtualBox which specifies that the provider for the virtual machine is VirtualBox and customizes how the virtual machine will run on VirtualBox.

```
config.vm.provider "virtualbox" do |vb|
  # VirtualBox-specific configuration settings go here
end
```

4. Vagrant expects a base box to be defined, even when booting from an ISO file since Vagrant is primarily designed to manage pre-built boxes. Specify a very lightweight box to satisfy Vagrant's requirement, and it should not interfere with the ISO installation process.

```
config.vm.box = "bento/ubuntu-20.04"
```

5. If you were using another pre-configured Vagrant box before, remove that cached box because it might still be lingering in the Vagrant environment: `vagrant destroy` and `vagrant box remove ubuntu/focal64`

6. If your VirtualBox machine uses a different SATA controller name (other than "SATA Controller"), replace "SATA Controller" in the Vagrantfile with the exact name of your controller (check in VirtualBox settings if unsure).
7. SATA controller is a virtual device that emulates a physical SATA (Serial ATA) controller, commonly used in real hardware for connecting storage devices like hard drives and optical drives to the motherboard. In a virtual machine, the SATA controller helps manage how virtual storage devices, like virtual hard disks and ISO images, are connected to the VM.\
   The SATA controller inside the VM only exists virtually. It doesn’t connect to the physical SATA controller or motherboard of your host machine. Instead, VirtualBox handles everything by virtualizing hardware and using the host's resources (like CPU, RAM, and disk storage) to simulate real devices for the VM.\
   Under the hood, VirtualBox maps the virtual SATA controller to files on your host machine (like a .vdi for the virtual hard disk or an ISO file for installation media).\
   Mapping a virtual disk to a file on the host’s disk means that the virtual hard disk used by the virtual machine (VM) is stored as a file on the host machine's storage system. Instead of the VM interacting with a real physical hard disk, VirtualBox (or another hypervisor) creates and manages a virtual disk file on the host, which emulates a real hard disk for the guest OS. Virtual disk file acts as a container that stores all the data (like the guest OS, files, and applications) used by the VM.

8. There could be a mismatch in user credentials or SSH keys as a result of manually installing the OS and SSH on the Ubuntu Server. Therefore, Vagrant's SSH connection might fail to authenticate with the server. Vagrant uses SSH key-based authentication with a default insecure private key.

9. Vagrant's insecure key pair consists of 2 keys:\
   **Private Key** that is stored locally on your host machine at `~/.vagrant.d/insecure_private_key`. If Vagrant can't find or use the private key, it will default to password-based login, which is why it can be prompted for a password.\
   **Public Key** which gets copied to the Ubuntu Server VM's `/home/vagrant/.ssh/authorized_keys` file during setup to allow passwordless login.\
    **NOTE:** The insecure key pair is convenient for local development. Replace the key with a custom one and avoid exposing the VM to public networks.

10. The `.vagrant.d` folder is a configuration directory used by Vagrant to store various settings, plugins, boxes, and SSH keys. It is typically located in your home directory (`~/.vagrant.d` on Linux/macOS).

11. The VM’s SSH service listens on port 22. If port 22 is not properly forwarded to port 2222 (host machine), SSH won't work. Log into your Ubuntu Server and ensure the server is listening on port 22: `sudo ss -tlnp | grep 22`. Try manually connecting to the VM using port 2222, which is forwarded to port 22 on the VM: `ssh -i ~/.vagrant.d/insecure_private_key -p 2222 vagrant@127.0.0.1`. If this works, the issue lies with how Vagrant is forwarding or managing SSH connections.

12. When you use the `vagrant ssh` command, it connects to `127.0.0.1:2222` (host machine). The traffic is forwarded to `127.0.0.1:22` inside the VM. SSH service on port 22 inside the VM responds and grants access.
