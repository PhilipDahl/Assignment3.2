# Server Setup and Configuration Guide

For This assignment we were required to create a step-by-step instructions page for setting up and configuring two Arch Linux servers to generate and serve a static HTML file with system information, along with setting up a load balancer and a file server. 

## Part 1: Single Server Setup

### Task 1: Create a System User
1. First, we will be creating a system user `webgen` with a home directory at `/var/lib/webgen`:
    ```bash
    sudo useradd -r -d /var/lib/webgen -s /usr/bin/nologin webgen
    sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
    sudo chown -R webgen:webgen /var/lib/webgen
    ```
?1
### Task 2: Create and Configure Systemd Service and Timer
1. Create the `generate-index.service` file at `/etc/systemd/system/generate-index.service`:
    ```ini
    [Unit]
    Description=Generate index.html
    After=network-online.target
    Wants=network-online.target

    [Service]
    User=webgen
    Group=webgen
    ExecStart=/var/lib/webgen/bin/generate_index

    [Install]
    WantedBy=multi-user.target
    ```

?2

2. Create the `generate-index.timer` file at `/etc/systemd/system/generate-index.timer`:
    ```ini
    [Unit]
    Description=Run generate-index service daily

    [Timer]
    OnCalendar=*-*-* 05:00:00
    Persistent=true

    [Install]
    WantedBy=timers.target
    ```

?3

3. Enable and start the timer:
    ```bash
    sudo systemctl enable --now generate-index.timer
    ```

**Commands needed for setup and testing**:
- Enable and start the timer: `sudo systemctl enable --now generate-index.timer`
- Check if the timer is active: `sudo systemctl list-timers --all`
- Check logs for service execution: `sudo journalctl -u generate-index.service`

### Task 3: Configure Nginx
1. Modify the main `nginx.conf` to run as the `webgen` user:
    ```nginx
    user webgen;
    ```

2. Create a separate server block file at `/etc/nginx/sites-available/webgen`:
    ```nginx
    server {
        listen 80;
        server_name your_domain_or_ip;

        root /var/lib/webgen/HTML;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

?4

3. Enable the configuration by creating a symbolic link:
    ```bash
    sudo ln -s /etc/nginx/sites-available/webgen /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl reload nginx
    ```

**Commands needed for setup and testing**:
- Test Nginx configuration: `sudo nginx -t`
- Reload Nginx: `sudo systemctl reload nginx`
- Check Nginx status: `sudo systemctl status nginx`

?5

### Task 4: Configure UFW Firewall
1. Install and configure UFW:
    ```bash
    sudo pacman -S ufw
    sudo ufw allow ssh
    sudo ufw allow http
    sudo ufw limit ssh
    sudo ufw enable
    ```

**Commands needed for setup and testing**:
- Enable UFW: `sudo ufw enable`
- Check firewall status: `sudo ufw status`

### Task 5: Final Step
Visit your server's IP address in a browser to view the system information page.

---

## Part 2: Setting Up Two Servers with a Load Balancer

### Task 1: Create Two New Digital Ocean Droplets
1. Create two droplets running Arch Linux with the tag "web".

### Task 2: Set Up Load Balancer
1. Create a load balancer with the following settings:
    - **Region**: SFO3
    - **VPC**: Default
    - **Type**: External (public)
    - **Tag**: "web"

?6

### Login to each individual droplet

ssh -i .ssh\directory\of\private-key user@<ip_address>

### Setup for success
There are some important packages that are essential to the success of this assignment below shows how to download them 

1) sudo pacman -Syu
2) sudo pacman -S nginx
3) sudo pacman -S git
4) sudo pacman -S ufw


### Task 3: Clone Updated Starter Code
1. Clone the updated starter code repository containing the updated `generate_index` script.

### Task 4: Update Server Configuration
1. Update the server configuration to include a file server:
    - Create the `documents` directory:
        ```bash
        sudo mkdir -p /var/lib/webgen/documents
        sudo chown -R webgen:webgen /var/lib/webgen/documents
        ```
    - Add sample files:
        ```bash
        cd /var/lib/webgen/documents/file-one
        cd /var/lib/webgen/documents/file-two
        ```
insert the text "this is file one"
then do the same for file two except name it "file two"

2. Update Nginx configuration to serve files from the `documents` directory:
    ```nginx
    server {
        listen 80;
        server_name your_domain_or_ip;

        root /var/lib/webgen/HTML;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /documents/ {
            alias /var/lib/webgen/documents/;
            autoindex on;
        }
    }
    ```

3. Ensure the configuration is enabled and Nginx is reloaded:
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```


---



