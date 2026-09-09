# Pi Clone Setup Automation

These scripts automate the repetitive parts of setting up a freshly cloned
Raspberry Pi SD card (new hostname, new user, copied project files, fixed
virtual environment paths). A few steps still require manual/interactive
input and are called out below.

## Files

- `clone_setup.sh` — Run first (as root), right after first boot on the OLD user.
- `cleanup_old_user.sh` — Run after confirming the new user works, to delete the old account.
- `verify_setup.sh` — Run as the new user to confirm environment, camera, and versions are correct.

## Usage

### 1. Copy scripts to the Pi

Copy these three files to the Pi (e.g. via `scp`), or keep them in your class
repo and `git pull` them onto each Pi.

### 2. Run the main setup script (as OLD user, with sudo)

```bash
sudo bash clone_setup.sh <OLD_USERNAME> <NEW_USERNAME> <NEW_HOSTNAME>
```

Example:
```bash
sudo bash clone_setup.sh lhsengr11 lhsengr05a lhsengr05a
```

This will:
1. Reset machine-id and SSH host keys (fixes RPi Connect identity collisions)
2. Set the new hostname
3. Create the new user (you will be prompted to set a password)
4. Copy group memberships from the old user
5. Copy `~/Documents` (including `scripts/pose_basic.py` and `cv_env2`) to the new user's home
6. Rewrite hardcoded paths inside the `cv_env2` virtual environment
7. Add auto-activation of `cv_env2` to the new user's `.bashrc`

### 3. Reboot and log in as the new user

```bash
sudo reboot
```

```bash
ssh <NEW_USERNAME>@<NEW_HOSTNAME>.local
```

### 4. Verify everything works

```bash
bash verify_setup.sh
```

### 5. Manual steps (cannot be automated)

**a. Enable Wayland compositor + desktop autologin** (required for RPi Connect
screen sharing):

```bash
sudo raspi-config
```
- Advanced Options → Wayland → Wayfire (or Labwc)
- System Options → Boot / Auto Login → Desktop Autologin
- Reboot when prompted

**b. Sign in to RPi Connect** (requires browser-based auth):

```bash
rpi-connect signout
rpi-connect on
rpi-connect signin
```

### 6. Remove the old user once confirmed working

```bash
sudo bash cleanup_old_user.sh <OLD_USERNAME>
```

This checks for and kills any lingering sessions/processes before deleting
the account, avoiding the "user is currently used by process" error.

## Notes

- These scripts assume the project lives at `~/Documents/scripts/cv_env2`
  (which includes `pose_basic.py` and later lesson scripts). Adjust paths
  inside the scripts if your layout differs.
- Always burn from the official Raspberry Pi OS Bookworm image and clone
  from a fully-tested "golden" Pi — do not skip the machine-id/SSH key
  reset step, or RPi Connect will show the wrong hostname for the device.
