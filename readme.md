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
git config --global user.name XXXX
git config --global user.email XXXXXXXXX@gmail.com

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

lxc-ls --fancy
lxc-start -n test1 -d
lxc-stop -n test1
lxc-attach -n test1 -- sh -c "echo hello world"

# setting init var
vi /lxc/lxc-init.yml
# install lxc and create container
ansible-playbook ./root.yml
ansible-playbook ./lxc/lxc-create.yml
# stop & destroy
ansible-playbook ./lxc/lxc-destroy.yml

-------------------------------------------
  - name: "add sudo user"
    shell: lxc-attach -n {{ item.vm }} -- su -c "chmod 740 /etc/sudoers && echo debian  ALL=(ALL:ALL) ALL >> /etc/sudoers && chmod 440 /etc/sudoers
    with_items: "{{ vms }}"

    shell: lxc-attach -n {{ item.vm }} -- su -c 'echo "StrictHostKeyChecking no\nPermitRootLogin yes\nPasswordAuthentication yes\nPermitEmptyPasswords no" >> /etc/ssh/sshd_config'

git add --all
git push    

apt-get install -y nginx
  tasks:
    - name: install nginx
      apt: name=nginx update_cache=yes cache_valid_time=3600
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted

    - name: copy TLS certificate
      copy: src=files/nginx.crt dest={{ cert_file }}
      notify: restart nginx
    - name: copy nginx config file
      template: src=templates/nginx.conf.j2 dest={{ conf_file }}
      notify: restart nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;
  listen 443 ssl;

  root /usr/share/nginx/html;
  index index.html index.htm;

  server_name {{ server_name }};
  ssl_certificate {{ cert_file }};
  ssl_certificate_key {{ key_file }};

  location / {
          try_files $uri $uri/ =404;
  }
}

/etc/nginx/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 1024;
        multi_accept on;
        # use epoll;
}

http {

        ##
        # Basic Settings
        ##
        
        charset UTF-8;
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        #keepalive_timeout 65;
        keepalive_timeout 10;
        types_hash_max_size 2048;
        # server_tokens off;
        server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
# 
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}

/etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  {{ domainname }};

    access_log  /var/log/nginx/log/{{ domainname }}.access.log  main;

    location / {
        root   /usr/share/nginx/html/{{ domainname }};
        index  index.html index.htm;
    } 
}

server {
    listen       80;
    server_name  $hostname;

    access_log  /var/log/nginx/log/{{ domainname }}.access.log  main;

    location {{ location }} {
        proxy_pass /{{ proxy_pass }};
    }
}

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

server {
    listen 80;
    server_name (バーチャルホスト名);
    rewrite ^(.*)$ https://$host$1 permanent;
}
/etc/nginx/sites-available/revproxy.conf
# # redirect http->https
# server {
#     listen 80 default;
#     server_name www.example.com;
#     return 301 https://$host$request_uri;
# }
# template reverse proxy
server {
        listen 80;
        listen [::]:80;
        server_name $hostname;
        proxy_set_header Host               $host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-Host   $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        
        for ( vhost in vhosts ) {
            location {{ vhost.location }} {
                proxy_pass {{ vhost.proxy_pass }};
            }
        }
}
server {
    listen 443;
    server_name (バーチャルホスト名);
    ssl on;
    ssl_certificate /etc/nginx/cert/ssl.crt;
    ssl_certificate_key /etc/nginx/cert/ssl.key;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-Proto $scheme;
    location / {
        proxy_pass http://(バックエンド Web サーバの IP アドレス);
        proxy_redirect http:// https://;
    }
}
# Example action to start service httpd, if not running
- service: name=httpd state=started

# Example action to stop service httpd, if running
- service: name=httpd state=stopped

# Example action to restart service httpd, in all cases
- service: name=httpd state=restarted

# Example action to reload service httpd, in all cases
- service: name=httpd state=reloaded

# Example action to enable service httpd, and not touch the running state
- service: name=httpd enabled=yes

# Example action to start service foo, based on running process /usr/bin/foo
- service: name=foo pattern=/usr/bin/foo state=started

# Example action to restart network service for interface eth0
- service: name=network state=restarted args=eth0

lxc-attach -n {{ item.vm }} -- su -c 'echo -e "# This file describes the network interfaces available on your system\n# and how to activate them. For more information, see interfaces(5).\n# The loopback network interface\nauto lo\niface lo inet loopback\n\nauto eth0\niface eth0 inet static\n    address 192.168.0.100\n    netmask 255.255.255.0\n    gateway 192.168.0.1" > /etc/network/interfaces'
lxc-attach -n {{ item.vm }} -- su -c 'systemctl restart networking.service'
lxc-attach -n {{ item.vm }} -- su -c 'ifdown eth0'
lxc-attach -n {{ item.vm }} -- su -c 'ifup eth0'

sudo apt-get install python-certbot-nginx -t stretch-backports

sudo certbot --agree-tos --staging --authenticator webroot --webroot /var/www/html -d sightlight666.dip.jp -m sightlight666@gmail.com --installer nginx

// If you usually use a command like systemctl or service to start and stop Nginx, you should use those commands instead in the hooks above. By using a command with hooks like this, if you automate renewal as described below, Certbot will automatically stop and start Nginx when you need to renew your certificates. If you configure Nginx to use the symlinks in the "live" directory as instructed by Certbot, Nginx will automatically begin using any renewed certificates. 
sudo certbot certonly --authenticator standalone --pre-hook "nginx -s stop" --post-hook "nginx"

// Automating renewal
sudo certbot renew --dry-run

Nginxで海外からのアクセスを拒否する | 稲葉サーバーデザイン
https://inaba-serverdesign.jp/blog/20140616/nginx_deny_ipaddress.html

GitHub - LWJGL/lwjgl3: LWJGL is a Java library that enables cross-platform access to popular native APIs useful in the development of graphics (OpenGL), audio (OpenAL) and parallel computing (OpenCL) applications.
https://github.com/LWJGL/lwjgl3

GitHub - ansible/ansible-examples: A few starter examples of ansible playbooks, to show features and how they work together. See http://galaxy.ansible.com for example roles from the Ansible community for deploying many popular applications.
https://github.com/ansible/ansible-examples

Ansibleでnginxを構築 - Qiita
https://qiita.com/takaheraw@github/items/e38010c176aa2937752b

おっさんエンジニアの実験室: 9月 2015
http://ossan-engineer.blogspot.jp/2015/09/

Ansible ～serviceモジュール～ - Qiita
https://qiita.com/moiwasaki/items/c320c7073a7ef086c5c9
