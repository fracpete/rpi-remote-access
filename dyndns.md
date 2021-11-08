# DynDNS

## noip.com

[noip.com](https://noip.com/) offers three DNS names with free accounts.

* Install and configure the client

  https://www.noip.com/support/knowledgebase/installing-the-linux-dynamic-update-client/
  
* Create systemd server `/etc/systemd/system/noip2.service` with the following content
  ```ini
  [Unit]
  Description=noip2 service
  
  [Service]
  Type=forking
  ExecStart=/usr/local/bin/noip2
  Restart=always
  
  [Install]
  WantedBy=default.target
  ```
  
* Enable the service
  ```bash
  sudo systemctl enable noip2.service
  ```
  
* Start the service
  ```bash
  sudo systemctl start noip2.service
  ```
