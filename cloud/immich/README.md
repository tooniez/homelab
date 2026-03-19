# Immich - Self-Hosted Photo and Video Backup

Immich is a self-hosted alternative to Google Photos with a focus on privacy and control. It keeps your photos and videos on your own hardware with support for RAW files, facial recognition, and automatic organization. The mobile app for both [Android](https://play.google.com/store/apps/details?id=app.immich) and [iOS](https://apps.apple.com/app/immich/id1613945652) includes built-in photo backup support.

> [!NOTE]
> Most of the steps below are from their [official docs](https://docs.immich.app/install/docker-compose). Please check those out as well.

## Prerequisites
1. A server running Docker. Checkout our [docker guide](https://techhut.tv/7-docker-basics-for-beginners/) to get this going.
2. Recommended 6GB of RAM and 4 core CPU. ([learn more](https://immich.app/docs/install/requirements))
3. Enough storage for your media. Your needed amount will depend on your photography habits and current media collection.

## Setup

### Prepare Your Environment
Create directories for Immich data. You may want to change the `library` location to a network drive or another location if your root file system has less storage than you may need. Checkout this example of [mounting drives](https://techhut.tv/auto-mount-drives-in-linux-fstab/) in Linux.
```bash
mkdir -p ~/docker/immich
```

Navigate to the directory and download the `compose.yaml` and `.env` files from this repo.
```bash
cd ~/docker/immich
wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/cloud/immich/compose.yaml && wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/cloud/immich/.env
```

### Configure the .env File
```bash
nano .env
```

**Mandatory Changes:**
- `DB_PASSWORD` — Change to a random strong password. Use only `A-Za-z0-9` to avoid issues with Docker parsing.
- `TZ` — Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List).
- `UPLOAD_LOCATION` — Verify or change the path where your photos will be stored.
- `DB_DATA_LOCATION` — Path for the PostgreSQL database. Network shares are **not** supported here.

### Deploy the Stack
```bash
docker compose up -d
```
Wait a few minutes for all containers to initialize. Check logs if any containers fail to start.
```bash
docker logs immich_server
```

### Access Immich
1. Open `http://your-server-ip:2283` in your browser
2. Create an admin account (first user becomes admin)
3. Configure settings:
   - **Storage Template** — Configure to make your library [human readable](https://immich.app/docs/install/post-install).
   - **Machine Learning** — Enable facial recognition if using the ML container.

## Mobile App Setup
1. Install Immich from the [App Store](https://apps.apple.com/app/immich/id1613945652) or [Play Store](https://play.google.com/store/apps/details?id=app.immich)
2. Connect to your server:
   - Server URL: `http://your-server-ip:2283`
   - Use your admin credentials
3. Enable auto-backup in the app settings

## Remote Access
- **[NetBird](https://netbird.io/) (Recommended)** — A zero-trust networking solution using WireGuard. Access Immich securely from anywhere without exposing ports. Follow our [NetBird setup guide](https://techhut.tv/self-host-netbird-pocketid) to get started.
- **Reverse Proxy** — Setup NGINX Proxy Manager for domain-based access ([video guide](https://www.youtube.com/watch?v=79e6KBYcVmQ)). Forward `photos.yourdomain.com` to `http://immich-server:2283`.

## Optional Configuration

### Hardware Transcoding
Uncomment the `devices` section in the `immich-server` service in `compose.yaml` to enable Intel QuickSync for video transcoding. [Learn more](https://immich.app/docs/features/hardware-transcoding/#single-compose-file).

### Hardware-Accelerated Machine Learning
Uncomment and adjust the `immich-machine-learning` section in `compose.yaml`. [Learn more](https://immich.app/docs/features/ml-hardware-acceleration).

### External Libraries
You can add existing photo directories as [external libraries](https://immich.app/docs/features/libraries/).

### Disable Machine Learning
If you don't need facial recognition or smart search, remove the entire `immich-machine-learning` service from `compose.yaml`.

## Storage Templates

Storage templates let you automatically sort photos into folders based on metadata like date, camera model, or custom tags. This makes your library human-readable and easier to back up.

Example template:
```
{{y}}/{{MMMM}}-{{DD}}/{{filename}}
```
Generates: `2024/July-15/IMG_1234.jpg`

Go to **Settings** > **Storage Template** in the Immich web UI to configure this. You can apply templates to new uploads and re-run them on existing libraries.

## Backup Strategy
- Regularly back up your database directory (`DB_DATA_LOCATION`)
- Use an external backup tool or service to keep a copy of your photo library offsite

## Troubleshooting

**Port Conflicts** — Ensure port 2283 is available on your host.

**Permission Errors** — Verify volume permissions with `ls -l` on your upload and database directories.

**ML Container Failing** — Try disabling GPU support or allocate more RAM.

**Reset Admin Password:**
```bash
docker exec immich_server npm run reset-admin-password
```

## Maintenance
- Re-deploy with updated image tags to update. Follow [Immich releases](https://github.com/immich-app/immich/releases).
- Monitor your `UPLOAD_LOCATION` directory for storage growth.
- Set retention policies in the Immich web UI as needed.
