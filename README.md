# Jitsi Meet Docker Installation Guide

This guide explains how to quickly set up **Jitsi Meet** using Docker and Docker Compose.

## Prerequisites

- A machine with **Docker** and **Docker Compose** installed.
- Basic knowledge of the terminal (Linux or Windows).
- A valid domain name configured to point to your server.

---

## Installation Steps

### 1. Download the Latest Release

As the repository is private to the Abintrax organization, you cannot use `wget` or `curl` to fetch the release automatically.

Instead, follow these steps:

1. Go to the [Abintrax Docker Jitsi Meet GitHub Releases](https://github.com/abintrax/docker-jitsi-meet/releases) page.
2. Manually download the `.zip` archive of the latest release.
3. Transfer the `.zip` file to your server using a method such as `scp`, `rsync`, or via a file upload through your preferred terminal manager.

Example using `scp`:

```bash
scp docker-jitsi-meet-vX.X.X.zip user@your-server-ip:/path/to/your/target/directory
```

### 2. Extract the Archive

Unzip the downloaded package:

```bash
unzip <filename>
```

### 3. Create the `.env` Configuration File

Generate your local `.env` file by copying the example provided:

```bash
cp env.example .env
```

### 4. Edit Required Configuration Parameters

Open the `.env` file in a text editor and **configure all parameters marked with `# ---- REQUIRED ----`**. The most important required fields include:

- `PUBLIC_URL` - Public URL for the web service.
- `JVB_ADVERTISE_IPS` - Comma-separated IP addresses for the Jitsi Videobridge to advertise.
- `JWT_APP_ID` - Application identifier for JWT authentication.
- `JWT_APP_SECRET` - Secret used to sign JWT tokens.
- `JWT_ACCEPTED_ISSUERS` - Comma-separated list of accepted JWT token issuers.
- `JWT_ACCEPTED_AUDIENCES` - Comma-separated list of accepted JWT token audiences.

Make sure each required setting is properly filled before continuing. These values are critical for authentication and network configuration.

### 5. Set Secure Passwords

Run the included script to automatically populate the `.env` file with secure passwords:

```bash
./gen-passwords.sh
```

### 6. Configuration Directory Setup

The project now includes a pre-built `CONFIG` directory within the root of the repository. This directory already contains all necessary subdirectories for each Jitsi module:

- `web/`
- `transcripts/`
- `prosody/config/`
- `prosody/prosody-plugins-custom/`
- `jicofo/`
- `jvb/`
- `jigasi/`
- `jibri/`

You can customize configurations for each component by editing the files within these subdirectories.

> **SSL Certificates:** By default, the `CONFIG/web/keys` directory contains wildcard SSL certificates for the `*.abintrax.com` domain.  
> These should be kept as-is if you're using any `abintrax.com` subdomain (e.g., `meet.abintrax.com`).  

> **Note:** If you have set a custom `PUBLIC_URL` in your `.env` file, make sure to place your SSL certificates in `CONFIG/web/keys/` as follows:
>
> - The certificate file should be named: `your_domain.crt`
> - The private key file should be named: `your_domain.key`

### 7. Start the Services

Use Docker Compose to start the Jitsi Meet stack:

```bash
docker compose up -d
```

### 8. Access the Jitsi Meet Web UI

Once the services are running, visit the following URL in your browser:

```
https://meet.abintrax.com
```

> If you edited the `.env` file to use a custom domain, replace the above URL accordingly.

---

## Notes

- Ensure your firewall allows incoming traffic on the following ports:

  **External Ports:**

  - `80/tcp` – for Web UI HTTP (primarily used for redirecting to HTTPS; requires `ENABLE_HTTP_REDIRECT=1` in `.env`)
  - `443/tcp` – for Web UI HTTPS access
  - `10000/udp` – for RTP media over UDP
  - `20000-20050/udp` – for Jigasi SIP access (only if Jigasi is deployed)

- For production deployments, consider setting up SSL certificates with Let's Encrypt or your preferred certificate provider.

---

## System Requirements 🖥️

To ensure Jitsi Meet runs smoothly, your infrastructure should meet the following requirements, ordered by importance:

### 🌐 Network

- **Speed & Stability:** Use tools like `iperf3` to verify latency and any file download to confirm speed.
- **Estimated bandwidth per video stream resolution:**
  - 180p → 200 kbit/s
  - 360p → 500 kbit/s
  - 720p (HD) → 2.5 Mbit/s
  - 4K → 10 Mbit/s

> Example: 20 users streaming 4K cannot run on a server with only 100 Mbit/s upload.

- For serious deployments, a **1 Gbit/s** link is the baseline, and **10 Gbit/s** is recommended.  
- Large infrastructures often deploy multiple Videobridges each with 10 Gbit/s.

> These requirements mostly affect the **Jitsi Videobridge**. If using only external bridges, bandwidth is less critical on the main server.

### 🧠 RAM

- **8 GB** is generally recommended.
- **4 GB** may suffice for small meetings.
- **2 GB** could work for testing or very small groups.
- For large meetings, opt for **horizontal scaling** over excessive memory.

### 🧮 CPU

- Avoid low-performance CPUs or shared virtual cores.
- Ensure **dedicated CPU** if on VPS.
- Prosody (XMPP server) is **single-threaded**, so high core counts don’t always help.
- **4 dedicated cores** is typically sufficient.

### 💽 Disk

- Around **20 GB** is typically enough.
- SSDs are helpful but not essential.
- More space may be needed for **logs**, **recordings**, or **custom modules**.

---

## Architecture

A Jitsi Meet installation can be broken down into the following components:

- **A web interface**
- **An XMPP server**
- **A conference focus component**
- **A video router** (can be multiple for scaling)
- **A SIP gateway** (for audio call support)
- **A broadcasting infrastructure** (for recording or streaming)

This Docker setup splits each of these components into dedicated, interlinked containers, providing modular deployment and easy scalability.

![Jitsi Meet Docker Architecture](https://jitsi.github.io/handbook/assets/images/docker-jitsi-meet-afafdf87fea30a2fa6412baa4a3f8248.png)

---

## Backup Strategy 💾

Keeping your data safe is crucial. Here's how to handle backups for your Jitsi Meet deployment:

- **Config Directory**  
  Regularly back up the `CONFIG/` directory. It includes all component settings and any customizations.

- **Volumes & Recordings**  
  If using Jibri for recording, ensure recordings are written to a persistent volume. Back this up on a schedule.

- **SSL Certificates**  
  Backup custom SSL files in `CONFIG/web/keys/`, especially if you replace the default `*.abintrax.com` wildcard certificates.

> Consider using tools like `rsync`, `Restic`, or cloud sync to automate backups and store them offsite.

---

## License

This project uses the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
