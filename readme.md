# Immich App

## Commands

```sh
sudo docker compose up
sudo docker compose stop
```

## UPDATE VERSION STEPS:

1- Change to version needed in the env file: IMMICH_VERSION= <HERE>
2-

```sh
sudo docker compose up
sudo docker compose stop
```

> **Note:** After modifying the `.env` file, you may need to recreate data:

```sh
sudo docker compose up -d --force-recreate
```

UBUNUTU BACKUP:

## Adding a New Drive

When adding a new drive, make sure it's set to auto-mount!

### How to Add Your SSD to fstab for Auto-Mount

1. **Get the UUID for your SSD:**

   ```sh
   sudo blkid
   ```

   Look for a line like:

   ```
   UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ```

2. **Add a line to `/etc/fstab`:**

   - Replace the UUID with your actual value.
   - Use `\040` for spaces in the mount point.

3. **Create the mount point if it doesn't exist:**

   ```sh
   sudo mkdir -p "/media/gonzalo/Data Main1"
   ```

4. **Test the fstab entry (without rebooting):**
   ```sh
   sudo mount -a
   ```

If there are no errors, your SSD will auto-mount at `/media/gonzalo/Data Main1` on every boot.
