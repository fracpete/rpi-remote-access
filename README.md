# rpi-remote-access
In order to get ssh access to a Raspberry Pi (e.g., through a 4G modem or if it is behind a firewall), the [frp](https://github.com/fatedier/frp/) reverse proxy can be used.

## Server (eg cloud VM)

Inbound ports that need to be open:

* 22 - for general ssh access
* 7000 - general inbound connections from clients
* 6000 - for accepting ssh connections and forwarding them to the client (unique to each client)

Server requires DNS name or fixed IP address. DynDNS, like [noip.com](https://noip.com/), 
works as well. See the [DynDNS](dyndns.md) article for instructions.

For this example, we are assuming `mydevice.ddns.net` as the server DNS name.

Install frp:
* Download appropriate [release binary](https://github.com/fatedier/frp/releases/tag/v0.37.1)
  ```bash
  sudo bash
  cd /opt
  wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz
  tar -xzf frp_0.37.1_linux_amd64.tar.gz
  ln -s frp_0.37.1_linux_amd64 frp
  ```

* Create `/etc/frps.ini` with the following content:
  ```ini
  [common]
  bind_port = 7000
  ```
  
* Create systemd service `/etc/systemd/system/frps.service` with the following content:

  ```ini
  [Unit]
  Description=frp reverse proxy server
  After=network.target
  
  [Service]
  User=ubuntu
  Group=ubuntu
  WorkingDirectory=/opt/frp
  ExecStart=/opt/frp/frps -c /etc/frps.ini
  
  [Install]
  WantedBy=multi-user.target
  ```
  
* Install systemd service
  ```bash
  sudo systemctl enable /etc/systemd/system/frps.service
  ```
  
* Start service
  ```bash
  sudo systemctl start frps.service
  ```

## Client (Raspberry Pi)

Inbound ports that need to be open:

* 22 - for ssh access

Install frp:
* Download appropriate [release binary](https://github.com/fatedier/frp/releases/tag/v0.37.1)
  ```bash
  sudo bash
  cd /opt
  wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz
  tar -xzf frp_0.37.1_linux_amd64.tar.gz
  ln -s frp_0.37.1_linux_amd64 frp
  ```

* Create `/etc/frpc.ini` with the following content:
  ```ini
  [common]
  server_addr = mydevice.ddns.net
  server_port = 7000
  
  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6000
  ```
  
* Create systemd service `/etc/systemd/system/frpc.service` with the following content:

  ```ini
  [Unit]
  Description=frp reverse proxy client
  After=network.target
  
  [Service]
  User=pi
  Group=pi
  Restart=on-failure
  RestartSec=15s
  WorkingDirectory=/opt/frp
  ExecStart=/opt/frp/frpc -c /etc/frpc.ini
  
  [Install]
  WantedBy=multi-user.target
  ```
  
* Install systemd service
  ```bash
  sudo systemctl enable /etc/systemd/system/frpc.service
  ```
  
* Start service
  ```bash
  sudo systemctl start frpc.service
  ```

## Raspberry Pi access
Changing remote access to the Raspberry Pi to using ssh-keys only (as user `pi`):
* On admin laptop create a ssh key in `$HOME/.ssh`:
  ```bash
  ssh-keygen -f mydevice
  ```
* Output the content of the public key (`mydevice.pub`) and paste it on the Raspberry Pi into `/home/pi/.ssh/authorized_keys`
* On admin laptop, create the following entry in `$HOME/.ssh/config`:
  ```
  Host mydevice
    User pi
    Hostname mydevice.ddns.net
    Port 6000
    IdentityFile ~/.ssh/mydevice
  ```
* On Raspberry Pi, edit the `/etc/ssh/sshd_config` file and disable password authentication:
  ```
  PasswordAuthentication no
  ```
* Restart the `ssh` service on the Raspberry Pi
  ```bash
  sudo systemctl restart ssh
  ```

