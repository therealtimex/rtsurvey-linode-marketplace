# rtSurvey — Linode Marketplace Ansible Playbook

Ansible playbook used by the [rtSurvey Linode Marketplace 1-Click App](https://www.linode.com/marketplace/).

rtSurvey is a self-hosted smart survey and data collection platform designed for field research, NGOs, government agencies, and academic institutions that require full data sovereignty. Built by [RTA](https://rta.vn), it provides a complete CAPI (Computer-Assisted Personal Interviewing) environment with multi-user management, real-time Shiny analytics, and optional Keycloak SSO integration.

---

## What this repo contains

This repo is the Ansible playbook that gets cloned and executed by the StackScript bootstrap during Linode deployment. It is **not** meant to be run manually on your local machine — it runs on the Linode itself via `connection = local`.

```
.
├── ansible.cfg           # Local connection, no SSH required
├── collections.yml       # community.general + ansible.posix
├── requirements.txt      # Pins ansible + jinja2 versions
├── provision.yml         # Generates server-side credentials
├── site.yml              # Main playbook entry point
├── group_vars/
│   └── rtsurvey/vars     # Default variable values
└── roles/
    ├── common/           # sudo user, SSH hardening, UFW firewall
    ├── rtsurvey/         # Docker, Nginx, .env, SSL trigger
    └── post/             # Credentials file, MOTD, cleanup
```

---

## Deployment

### Requirements

| Field | Value |
|-------|-------|
| OS | Ubuntu 22.04 LTS |
| Recommended Plan | Dedicated 4 GB (4 CPU, 4 GB RAM) |
| Minimum Plan | Shared 4 GB |
| Disk | 80 GB SSD |

### StackScript parameters

At deploy time, the StackScript prompts for:

| Parameter | Description |
|-----------|-------------|
| `sudo_username` | Non-root sudo user to create |
| `sudo_password` | Password for the sudo user |
| `ssh_public_key` | (Optional) SSH public key for the sudo user |
| `tz` | Server timezone (default: `Asia/Ho_Chi_Minh`) |

All application passwords (database, admin, Keycloak) are generated automatically on the server — you do not set them at deploy time.

---

## After Deployment

### 1. Get your credentials

SSH into your Linode as the sudo user you created:

```bash
ssh <sudo_username>@<your-linode-ip>
cat ~/.credentials
```

### 2. Log in to rtSurvey

Open `http://<your-linode-ip>` in a browser.

- **Username**: `admin`
- **Password**: see `~/.credentials`

### 3. Configure domain and SSL

1. In the app go to **Configuration → System Properties → Domain & SSL**
2. Enter your domain name (DNS A record must already point to your Linode IP)
3. Select SSL type:
   - **Let's Encrypt** — recommended; issues a free TLS certificate automatically
   - **rtsurvey.com** — for `*.rtsurvey.com` subdomains (Cloudflare-proxied)
4. Click Save — the server obtains the certificate and switches to HTTPS automatically

### 4. Keycloak SSO (pre-installed)

Embedded Keycloak is available at `https://<domain>/auth/admin` after SSL is active.

- **Login**: `admin` / see `~/.credentials` (Keycloak Admin Password)

---

## Logs

| Log | Location |
|-----|----------|
| Deployment log | `/var/log/stackscript.log` |
| SSL trigger log | `/var/log/rtsurvey-ssl.log` |
| App files | `/opt/rtsurvey/` |

---

## Support

- Website: https://rta.vn
- Documentation: https://rta.vn/rtsurvey
- Support: info@rta.vn
