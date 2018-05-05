# login root user
# add sudo user
chmod 740 /etc/sudoers
sudo sh -c 'echo "debian  ALL=(ALL:ALL) ALL" >> /etc/sudoers'
chmod 440 /etc/sudoers

/etc/ssh/ssh_config
StrictHostKeyChecking no

lxc-attach -n test1 -- su -c 'echo -e "root:debian" | chpasswd'

# login non root user
sudo apt-get install -y git ansible avahi-daemon sshpass
git config --global user.name tnov
git config --global user.email sightlight999@gmail.com

mkdir ~/repos
cd ~/repos
git clone https://github.com/tnov/toolbox.git
cd /toolbox/playbook
ansible-playbook root.yml
# sudo apt-get install -y lxc python-lxc lxc-dev debootstrap libvirt0 libpam-cgroup libpam-cgfs bridge-utils
# sudo lxc-create -t debian -n test1 

# sudo brctl addbr br0
# sudo sh -c 'echo "auto br0\niface br0 inet dhcp\n      bridge_ports enp2s0f2\n      bridge_stp off\n      bridge_maxwait 0" >> /etc/network/interfaces'
# sudo /etc/init.d/networking restart
# sudo /etc/init.d/network-manager restart



# sudo sh -c 'echo "#lxc.network.type = empty\n# architecture configuration\nlxc.arch = amd64\n# Network configuration\nlxc.network.type = veth\nlxc.network.flags = up\nlxc.network.link = br0\nlxc.network.hwaddr = 00:FF:AA:xx:xx:xx" > /etc/lxc/default.conf'

# sudo sh -c 'echo USE_LXC_BRIDGE=\"true\" > /etc/default/lxc-net'


lxc-checkconfig
# unprivileged user can clone
# sudo sh -c 'echo "kernel.unprivileged_userns_clone=1" > /etc/sysctl.d/80-lxc-userns.conf'
# unprivileged user can clone external
# sudo sh -c 'echo "kernel.unprivileged_userns_clone=1" > /etc/sysctl.d/80-lxc-userns.conf'
# sudo sysctl --system

# login non root user
# check subuids and subgids
cat /etc/s*id|grep $USER
# not found 'cat /etc/s*id|grep $USER'
sudo usermod --add-subuids 1258512-1324047 $USER
sudo usermod --add-subgids 1258512-1324047 $USER
# Permission denied - Could not access /home/$USER/.local. Please grant it x access, or add an ACL for the container root.
sudo setfacl -m u:1258512:x . .local .local/share

mkdir -p ~/.config/lxc
echo -e "lxc.include = /etc/lxc/default.conf\nlxc.id_map = u 0 100000 65536\nlxc.id_map = g 0 100000 65536\nlxc.mount.auto = proc:mixed sys:ro cgroup:mixed" > ~/.config/lxc/default.conf

# USERNAME TYPE BRIDGE COUNT
echo "$USER veth br0 10"| sudo tee -i /etc/lxc/lxc-usernet

lxc-create --name test1 -t download

debian
stretch
amd64

lxc-start --name test1 --logfile $HOME/lxc_ubuntu.log --logpriority DEBUG

sudo lxc-ls --fancy

lxc-attach -n test1 -- sh -c "echo hello world"

export LANGUAGE=en_US.UTF-8 && export LANG=en_US.UTF-8 && export LC_ALL=en_US.UTF-8 && locale-gen en_US.UTF-8 && dpkg-reconfigure locales
  - name: "add sudo user"
    shell: lxc-attach -n {{ item.vm }} -- su -c "chmod 740 /etc/sudoers && echo debian  ALL=(ALL:ALL) ALL >> /etc/sudoers && chmod 440 /etc/sudoers
    with_items: "{{ vms }}"

    shell: lxc-attach -n {{ item.vm }} -- su -c 'echo "StrictHostKeyChecking no\nPermitRootLogin yes\nPasswordAuthentication yes\nPermitEmptyPasswords no" >> /etc/ssh/sshd_config'
    
