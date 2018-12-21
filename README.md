# LinuxScripts
Scripts used in my Linux work - KVM, Shred, etc
Mostly to host KVM build scripts that worked and useful commands from previous and current jobs.  Possibly FOG installation and troubleshooting.

Good step by step guides for running windows 10 on linux kvm with vga passthrough:
https://heiko-sieger.info/running-windows-10-on-linux-using-kvm-with-vga-passthrough/
https://dennisnotes.com/note/20180614-ubuntu-18.04-qemu-setup/

# Correct wrong parameters/information
# Insert relevant info in {} braces

sudo virt-install \
--virt-type=kvm \
--name {} \
--ram {} \
--vcpus={} \
--os-variant={} \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/{}.iso \
--network=bridge=vibr0,model=virtio \
--graphics vnc \
--disk path=/var/lib/libvirt/images/{}.qcow2,size={},bus=virtio,format=qcow2

# Windows 10(?):
sudo virt-install \
--virt-type=kvm \
--name windows10 \
--ram 4096 \
--vcpus=2 \
--os-variant=windows8 \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/windows.iso \
--network=bridge=vibr0,model=virtio \
--graphics vnc \
--disk path=/var/lib/libvirt/images/windows10.qcow2,size=80,bus=virtio,format=qcow2

# UbuntuMATE:
sudo virt-install \
--virt-type=kvm \
--name ubuntumate \
--ram 2048 \
--vcpus=2 \
--os-variant=ubuntu16 \
--virt-type=kvm \
--hvm \
--cdrom=/var/lib/libvirt/boot/ubuntu-mate-16.04.5-desktop-amd64.iso \
--network=bridge=vibr0,model=virtio \
--graphics vnc \
--disk path=/var/lib/libvirt/images/ubuntumate.qcow2,size=40,bus=virtio,format=qcow2

Commands (may need to run as sudo/root):
List all VMs - virsh list --all

Start a server - virsh start serverName

Stop a server - virsh shutdown serverName

Steps for using Linux KVM: (taken from: https://www.cyberciti.biz/faq/installing-kvm-on-ubuntu-16-04-lts-server/)

On server side - 
1. Install Ubuntu Server (16.04 in my case)
   Install SSH, nothing else
2. After it is all set up, updated, and upgraded:
       sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager
   or
       sudo apt-get install qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker
   -Both of these had elements that failed, but combining them worked out
3. You may need to set up a bridged network, but one set itself up for me
   Make a copy of the network interfaces before changing them:
       sudo cp /etc/network/interfaces /etc/network/interfaces.backup-1-Nov-2018
4. Grab an ISO to start making a VM (CentOS 7 in my case):
       cd /var/lib/libvirt/boot/
       sudo wget https://mirrors.kernel.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1708.iso
5. Make the VM:
       sudo virt-install --virt-type=kvm --name centos7 --ram 2048 --vcpus=2 --os-variant=centos7.0 --virt-type=kvm --hvm --cdrom=/var/lib/libvirt/boot/CentOS-7-x86_64-DVD-1708.iso --network=bridge=virbr0,model=virtio --graphics vnc --disk path=/var/lib/libvirt/images/centos7.qcow2,size=40,bus=virtio,format=qcow2
   Obviously change this command to make it work for the ISO you downloaded
6. Finally, to get the info for connecting from the client:
       sudo virsh dumpxml centos7 | grep vnc
   You just need to use the output to determine the port it's on

On the client side - 
1. Install a VNC client if you don't have one yet
2. Make sure the client is hardlined on the same network as the server
3. In the VNC viewer, add a new connection using the IP Address and Port of the server
   -In my case, there was an issue with SSH-ASKPASS or something like that.  I launched my VNC client as follows:
    sudo virt-manager --no-fork
   You will be prompted to enter your client password first.  When you try and connect to the server, go back to the terminal you launched the VNC from and enter your SERVER password, then do that again when launching the VM
4. You should now be able to finish the VM setup as you see fit

**RAN INTO AN ISSUE**
After using a VNC client that initally worked, it completely stopped connecting to the server.  So I went back and actually
followed the guide to get it going again.  Here's what I used to connect on Ubuntu 18.04 (verbatim for my machine, YMMV):
1. In bash: ssh tyler@10.1.10.108 -L 5900:127.0.0.1:5900
   -Parsing out this command:
	-Typical ssh with the user (tyler) @ the server (10.1.10.108)
	-The -L command for ssh allows for port binding
	-5900 is the port on the server that one of the VMs is using
	-127.0.0.1:5900 is to bind the server port 5900 to the localhost port 5900
   -This will open an SSH instance in the terminal, so have another tab or window open if you need to do more commands on the client
2. In Remmina, change from RDP to VNC for the connection type, then type in the localhost bind (127.0.0.1:5900)
3. A new window should open up with the VM.  You can do this for multiple VMs as long as you use a different terminal window
for each VM - new tab or completely serparate window is fine.

