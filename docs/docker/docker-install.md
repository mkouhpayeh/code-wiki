# Docker Installation

## Docker Desktop
Docker Desktop bundles the Docker Engine, CLI, and a friendly GUI for Windows and macOS.

1. Download and install:
   - https://www.docker.com/products/docker-desktop
2. After installation, verify Docker runs:
```powershell
docker run hello-world
```
3. Notes:
   - Docker Desktop requires virtualization enabled in BIOS/UEFI.
   - On Windows, use WSL2 backend for best compatibility (enable WSL2 and install a Linux distro).
   - Use the Docker Desktop UI to configure resources (CPU, memory, disk).

---

## Docker Machine 
Docker Machine historically created a small VM (often using VirtualBox) to run the Docker Engine on platforms that didn't support Docker natively. This is mostly superseded by Docker Desktop and WSL2 on modern systems.

---

## Install on Linux
Example (Debian/Ubuntu-based systems):

```bash
# Install prerequisites
sudo apt update
sudo apt install -y curl

# Download and run the official installer script
curl -fsSL https://get.docker.com -o /tmp/get-docker.sh
sudo sh /tmp/get-docker.sh

# Test Docker
sudo docker run hello-world
```

## Post-install
To run Docker without `sudo`, add your user to the `docker` group:

```bash
sudo usermod -aG docker $USER
# Apply the new group membership immediately (without logout) in the current session:
newgrp docker
# or log out and log back in for the change to take effect
```

Check group membership:

```bash
groups
# or
groups $USER
```

If `docker` is listed, you can run Docker commands without `sudo`:

```bash
docker run hello-world
```

---

## Common tips

- If `docker run hello-world` fails:
  - Ensure the Docker service is running: `sudo systemctl status docker`
  - Review daemon logs: `sudo journalctl -u docker --since "1 hour ago"`
- On Debian/Ubuntu, the installer script will configure the Docker repository and install Docker Engine and CLI.
- To manage Docker as a systemd service:
```bash
sudo systemctl enable docker
sudo systemctl start docker
```
- If you encounter permission issues after adding your user to the `docker` group, log out and log back in (or reboot) to refresh group membership.
- Use `docker version` to inspect client and server versions, and `docker info` for environment / runtime details.

---

## Useful links
- Official Docker docs: https://docs.docker.com
- Docker Desktop: https://www.docker.com/products/docker-desktop
- Get Docker installer script: https://get.docker.com
