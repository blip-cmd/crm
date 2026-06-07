# Buying and Pointing a Domain Name

This guide walks you through purchasing a custom domain name and pointing it to your ChurchCRM server (e.g., hosted on Oracle Cloud, a VPS, or a dedicated server) so that users can access the system securely over HTTPS via a clean URL.

---

## Step 1: Purchase a Domain Name

To get a domain name, you must buy one from an ICANN-accredited domain registrar.

### Recommended Registrars
* **Cloudflare Registrar**: Offers domains at wholesale cost (no markup fees or price hikes on renewals). Excellent built-in DNS management.
* **Porkbun**: Very affordable, simple interface, includes free WHOIS privacy.
* **Namecheap**: Highly popular, user-friendly, with good support and free domain privacy.

### Tips for Choosing a Domain
* **Congregation-focused**: Choose something recognizable to your members (e.g., `ourchurch.org` or `firstbaptistcity.org`).
* **Use a Subdomain**: If you already own a domain for your main church website (e.g., `mychurch.org`), you **do not** need to buy a new one. You can create a free subdomain like **`crm.mychurch.org`** or **`portal.mychurch.org`** using your existing domain provider's DNS panel.

---

## Step 2: Access Your DNS Management Panel

Once you buy a domain (or if you are using your existing church domain):
1. Log into the account dashboard of the registrar where the domain is registered.
2. Search for **DNS Settings**, **DNS Zone Editor**, or **Manage DNS**.
3. You will see a table of DNS records (records of type `A`, `CNAME`, `MX`, `TXT`, etc.).

---

## Step 3: Create an A Record to Point to Your Server

An **A Record** (Address Record) maps a human-readable domain or subdomain name to the physical IPv4 address of your server.

### Option A: Hosting on a Subdomain (e.g., `crm.mychurch.org`)
This is the recommended setup if you already have a main church website.
1. Click **Add New Record**.
2. Set **Type** to `A`.
3. Set **Name / Host** to `crm` (or whatever word you want before your domain name).
4. Set **Value / IP Address** to your server's Public IPv4 address (e.g., the public IP of your Oracle Cloud instance).
5. Set **TTL (Time to Live)** to `Auto` or `3600` (1 hour).
6. Save the record.

### Option B: Hosting on the Root Domain (e.g., `mychurchcrm.org`)
Use this if you bought a dedicated domain name just for the CRM.
1. Click **Add New Record**.
2. Set **Type** to `A`.
3. Set **Name / Host** to `@` (the `@` symbol represents the root domain).
4. Set **Value / IP Address** to your server's Public IPv4 address.
5. Set **TTL** to `Auto` or `3600`.
6. Save the record.

---

## Step 4: Verify DNS Propagation

DNS changes do not happen instantly. It takes time for DNS servers around the world to update their cache records (a process called *propagation*). This typically takes **5 to 30 minutes**, but can occasionally take up to 24–48 hours.

### How to Check if Your Domain is Pointing Correctly

#### 1. Use Web Tools (Easiest)
Go to [whatsmydns.net](https://www.whatsmydns.net/) and enter your domain name (e.g., `crm.mychurch.org`). Select `A` from the dropdown and click search. You will see a map showing which IP address your domain resolves to in different parts of the world.

#### 2. Use Terminal / Command Prompt
Run a ping or lookup command on your local machine:
```powershell
# In Windows PowerShell / Command Prompt
nslookup crm.mychurch.org

# In macOS / Linux Terminal
dig crm.mychurch.org
```
Ensure the output displays your server's public IP address.

---

## Step 5: Secure with SSL (HTTPS)

Once the domain resolves to your server's IP, you are ready to enable secure connections.

If you are using the **FrankenPHP (Caddy)** Docker setup described in the [Oracle Cloud Deployment Guide](oracle-cloud-deployment.md), you don't need to do anything else. Caddy will detect the domain name, communicate with Let's Encrypt to provision a free SSL certificate, and secure your site automatically.
