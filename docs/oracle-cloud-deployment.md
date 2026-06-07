# Deploying to Oracle Cloud "Always Free" Tier

This guide documents how to deploy ChurchCRM in a production-ready, secure, and completely free configuration on the **Oracle Cloud Infrastructure (OCI) Always Free Tier** using Docker and FrankenPHP (Caddy).

---

## Why Oracle Cloud Free Tier?

Oracle Cloud offers one of the most generous free tiers in the cloud hosting industry. Specifically:
* **Compute VM**: Up to 4 ARM Ampere A1 Compute CPU cores and 24 GB of RAM (can be allocated to a single instance).
* **Storage**: Up to 200 GB of persistent block volume storage.
* **Uptime**: Always-on 24/7/365. Services do not sleep or turn off.
* **Networking**: Mapped public IPv4 address with 10 TB of free outbound bandwidth per month.

---

## Step 1: Create Your Compute Instance in OCI

1. Sign up/log in to the [Oracle Cloud Console](https://cloud.oracle.com).
2. Navigate to **Compute** -> **Instances** -> **Create Instance**.
3. **Configure Image and Shape**:
   * *Image*: Choose **Ubuntu Server** (latest LTS version).
   * *Shape*: Click **Change Shape**, select **Ampere (ARM)**, and allocate **1-2 OCPUs** and **6-12 GB RAM** (this is more than enough for a large ChurchCRM instance).
4. **Configure Networking**:
   * Assign a public IPv4 address.
5. **Save SSH Keys**:
   * Download the private key (`.key`) file. You will need it to connect.
6. Click **Create** and wait for the status to show **Running**.

---

## Step 2: Open Ingress Ports (80/443) in Oracle Security Lists

Oracle Cloud blocks all ingress traffic by default except SSH. You must open HTTP and HTTPS ports:

1. In the instance details panel, click on the **Virtual Cloud Network (VCN)** link.
2. Select your subnet, then click on the **Default Security List**.
3. Click **Add Ingress Rules** and add the following two rules:
   * **Rule 1 (HTTP)**:
     * *Source CIDR*: `0.0.0.0/0`
     * *IP Protocol*: `TCP`
     * *Destination Port Range*: `80`
   * **Rule 2 (HTTPS)**:
     * *Source CIDR*: `0.0.0.0/0`
     * *IP Protocol*: `TCP`
     * *Destination Port Range*: `443`

---

## Step 3: Install Docker and Git on the Server

Connect to your instance via SSH:
```bash
ssh -i /path/to/your-key.key ubuntu@YOUR_INSTANCE_PUBLIC_IP
```

Inside the VM, update and install Docker:
```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io docker-compose-v2 git

# Enable and start Docker service
sudo systemctl enable --now docker

# Add your user to the docker group to avoid using sudo
sudo usermod -aG docker ubuntu
newgrp docker
```

---

## Step 4: Clone the Project and Build the Stack

1. Clone the ChurchCRM repository:
   ```bash
   git clone https://github.com/ChurchCRM/CRM.git
   cd CRM
   ```
2. Build the production build locally (or inside the container):
   ```bash
   # Install dependencies
   npm ci --ignore-scripts
   ```
3. Prepare the Docker files:
   Make sure you have your production configuration templates set up. For production, copy `docker/Config.php` to your configuration file:
   ```bash
   cp docker/Config.php src/Include/Config.php
   ```

---

## Step 5: Configure FrankenPHP (Caddy) for Production & SSL

FrankenPHP uses Caddy, which **automatically provisions and renews SSL certificates** from Let's Encrypt once your domain name is pointed to it.

1. Point your domain's **A Record** (e.g. `crm.mychurch.org`) to your Oracle Instance's **Public IP address**.
2. Open [docker/frankenphp/Caddyfile](file:///q:/blipping-projects/icgchft.com/crm/docker/frankenphp/Caddyfile) in the repo:
   * Replace `:80` at line 34 with your domain name (e.g. `crm.mychurch.org`).
3. Launch the production stack:
   ```bash
   docker compose -f docker/docker-compose.frankenphp.yaml up -d --build
   ```

*Caddy will instantly request a TLS certificate from Let's Encrypt and serve your ChurchCRM application over a secure HTTPS connection.*

---

## Step 6: Verify Persistence and Backups

The `docker-compose.frankenphp.yaml` uses named volumes for database storage (`churchcrm-db`) and application data (`churchcrm-www`). Because this is saved on Oracle's persistent block volumes, your database records, member profiles, and uploaded pictures are safe and will survive container updates and system reboots.

To backup your data, simply backup the volume contents located under `/var/lib/docker/volumes/` on your host VM.
