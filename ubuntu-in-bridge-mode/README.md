## Configure Vagrantfile for Bridged Networking

Edit the Vagrantfile. You'll need to uncomment (or add) the configuration for a public network and specify it to use a bridged adapter.

Here's an example configuration snippet to add to your Vagrantfile:
[Vagrantfile](/ubuntu-in-bridge-mode/Vagrantfile)

Replace "en0: Wi-Fi (AirPort)" with the name of the network interface you want to bridge on your laptop. If you're not sure, you can remove the bridge: option entirely, and Vagrant will prompt you to choose an interface when you start the VM.
