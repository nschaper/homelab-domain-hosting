# homelab-domain-hosting - Home Domain Routing Configuration Guide

This document provides a concise overview of configuring and managing a domain setup for a home network on a consumer ISP (e.g., Xfinity). The setup leverages Cloudflare Tunnels along with a Synology Router and NAS to achieve subdomain routing (e.g., `nas.mydomain.net` → `http://192.168.1.200:5000`).

> **Note:** Replace `mydomain.net` with your actual domain name.

## Overview

- **Domain:** `mydomain.net`
- **Local Devices:**
  - Synology NAS (e.g., `192.168.1.200`)
  - Synology Router (e.g., `192.168.1.1`)
- **Routing Method:** Cloudflare Tunnels managed via Zero Trust
- **Docker Service:** `cloud flared` running on Synology Docker

## Prerequisites

- A Cloudflare account with Zero Trust access.
- Cloudflare Tunnels set up and a tunnel (e.g., `MyHome Tunnel`) configured.
- Synology NAS and Router configured on your local network.
- Docker installed on Synology with the `cloud flared` container running.
- Basic networking knowledge and admin access to your devices.

## Configuring Cloudflare Tunnels via Zero Trust

### Launch Zero Trust
- Open [Cloudflare Zero Trust](https://one.dash.cloudflare.com/) in your browser.

### Manage Tunnels
- Navigate to **Networks → Tunnels**.
- Find or create your tunnel (e.g., `MyHome Tunnel`) that routes traffic from the public internet to your local network.

> **Important:** Do not manually modify DNS records. All DNS changes are automated based on the public hostname settings defined in Zero Trust.

### Manage Public Hostnames
- In the Zero Trust dashboard, select the **Public Hostnames** tab.
- Create, edit, or remove entries as needed. For example:
  - **`nas.mydomain.net`** → `http://192.168.1.200:5000`

## Current Routing Configuration

The following routes are pre-configured in this setup:

- **`mydomain.net`** → `http://192.168.1.200`
- **`nas.mydomain.net`** → `http://192.168.1.200:5000`
- **`router.mydomain.net`** → `http://192.168.1.1:8000`
- **`channels.mydomain.net`** → `http://192.168.1.200:8089`

## Installing and Configuring cloud flared

This section details how to install and configure the `cloud flared` Docker service on your Synology NAS.

### Install Docker on Synology
- Install Docker from the Synology Package Center if not already installed.

### Pull the Official Image
- Open a terminal on your Synology NAS (or use the Docker UI) and pull the image:
  ```bash
  docker pull cloudflare/cloudflared

### Prepare the Configuration Directory
Create a directory on your NAS (e.g., /docker/cloudflared/) to store configuration files and tunnel credentials.

### Configure Your Tunnel
Obtain Credentials:
- When you create a tunnel via Cloudflare Zero Trust, download the credentials file (e.g., TUNNEL-UUID.json) and place it in your configuration directory.
- Create a Configuration File:
  - In the same directory, create a config.yml file with minimal settings. For example:
    ```tunnel: <TUNNEL-UUID>
    credentials-file: /etc/cloudflared/TUNNEL-UUID.json
    # Ingress rules are managed via the Zero Trust dashboard
    ingress:
      - service: http_status:404
    Replace <TUNNEL-UUID> with your actual tunnel ID.
    ```

### Run the cloud flared Container
Open Synology Docker and create a new container with the following settings:
- Image: cloudflare/cloudflared
- Container Name: cloudflared
- Volume Mount: Map your NAS configuration directory (e.g., /docker/cloudflared/) to /etc/cloudflared inside the container.
- Command: Set the container to run the tunnel:
tunnel run <TUNNEL-NAME>
Replace <TUNNEL-NAME> with your tunnel's name (as configured in the Zero Trust dashboard).

### Verify the Setup
Check the container logs via Synology Docker to ensure the tunnel is active and connected.
Ensure that any changes to public hostnames via Zero Trust are reflected without manually editing DNS records.

## Additional Considerations
- Zero Trust Administration:
Always manage your public hostnames through the Cloudflare Zero Trust dashboard rather than editing DNS settings manually.
- Troubleshooting:
Verify that the cloudflared container is running properly.
Confirm local IP addresses for your Synology devices.
Ensure the Cloudflare Tunnel is active and correctly configured via Zero Trust.
- Security Best Practices:
Regularly update your Synology firmware, Docker images, and Cloudflare settings to maintain a secure, zero trust environment.

## Conclusion
This guide is intended to help knowledgeable users manage domain routing on a home network with a consumer ISP, utilizing Cloudflare Tunnels, Synology hardware, and the cloud flared Docker service. For advanced configurations or troubleshooting, refer to the official documentation provided by Cloudflare and Synology.
