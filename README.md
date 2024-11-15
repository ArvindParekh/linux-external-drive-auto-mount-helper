# Automatically Mount External Drives on Ubuntu

## Introduction
Managing external hard drives on Linux can be a tedious task, requiring manual steps to mount the drives every time they are connected. This documentation provides a step-by-step guide to automate the mounting process on Ubuntu, ensuring your external drives are ready to use as soon as they are plugged in.

## Benefits
- **Automatic Mounting**: The external drives are automatically mounted when connected, eliminating the need for manual intervention.
- **Dynamic Mount Points**: The script creates unique mount points for each drive, preventing conflicts when multiple drives are connected.
- **Filesystem Check**: The script runs a filesystem check (`fsck`) before mounting the drive, ensuring data integrity.
- **Improved Convenience**: With this automation, you can plug in your external drives and immediately start using them, without having to remember and execute a series of commands.

## Prerequisites
- Ubuntu operating system
- Administrative (root) access to your system

## Installation and Configuration

### 1. Determine the Device Name
The first step is to identify the device name of your external drive. This can be done using the `lsblk` command:

```bash
DEVICE=$(lsblk -r -o NAME | grep -E 'sd[a-z][0-9]+$' | head -1)
```
### 2. Create a Mount Point
Next, we need to create a mount point directory for the external drive. To ensure uniqueness and avoid conflicts, we'll first attempt to use a default mount point. If that location is already in use, a new mount point will be created based on the device name and timestamp:

```bash
MOUNT_POINT="/media/<name-of-your-mount-directory>"
# Check if the mount point is already in use
if mountpoint -q $MOUNT_POINT; then
  MOUNT_POINT="/media/external-$DEVICE-$(date +%s)"
  mkdir -p $MOUNT_POINT
fi
```
If you're not sure what's happening here, don't worry! This is just the approach we'll use to dynamically create the mount point. There's a ready-to-use script at the end. **To get started**, you can start executing the commands from here on. ⬇️

### 3. Set Up the udev Rule
Ubuntu uses the udev system to manage device events, such as the addition or removal of external drives. We'll create a udev rule to automatically trigger the mount script when a new block device is added:

1. Open the udev rules file:
    ```bash
    sudo nano /etc/udev/rules.d/99-mount-external.rules
    ```
2. Add the following line to the file:
    ```bash
    SUBSYSTEM=="block", KERNEL=="sd[a-z][0-9]", ACTION=="add", RUN+="/usr/local/bin/mount_external.sh"
    ```
    This rule will run the `mount_external.sh` script whenever a new block device (like an external drive) is added.
3. Save and close the file.

### 4. Create the Mount Script

The final step is to create the mount script that will be triggered by the udev rule. This script will handle the mounting process, including checking for existing mount points and creating new ones if necessary:

1. Open the mount script file:
    ```bash
    sudo nano /usr/local/bin/mount_external.sh
    ```
2. Add the following content to the file:
    ```bash
    #!/bin/bash

    DEVICE=$(lsblk -r -o NAME | grep -E 'sd[a-z][0-9]+$' | head -1)
    #DEVICE="sda1"
    MOUNT_POINT="/media/<name-of-your-mount-directory>"

    # Check if the mount point is already in use
    if mountpoint -q $MOUNT_POINT; then
    MOUNT_POINT="/media/external-$DEVICE-$(date +%s)"
    mkdir -p $MOUNT_POINT
    fi

    sudo fsck /dev/$DEVICE #>> /tmp/mount_external.log - optional, if you'd like to log the output of the command
    sudo mount /dev/$DEVICE $MOUNT_POINT #>> /tmp/mount_external.log
    ```
    This script:
    - Determines the device name using lsblk and awk
    - Checks if the default mount point is already in use
    - If so, creates a new mount point with a timestamp suffix
    - Performs the fsck and mount operations
3. Make the script executable:
    ```bash
    sudo chmod +x /usr/local/bin/mount_external.sh
    ```

## Usage
Once the setup is complete, you can connect your external drive, and it should be automatically mounted to the appropriate directory (e.g., `/media/external-sdd1`, `/media/external-sde1-1612345678`). The mount point will be unique for each drive, preventing conflicts when multiple drives are connected.
If you encounter any issues or have additional questions, please don't hesitate to reach out (@ [𝕏](https://x.com/ArvindParekh_21)/[Bsky](https://bsky.app/profile/arvindparekh.bsky.social)) for further assistance.

## Conclusion
Automating the mounting of external drives on Ubuntu enhances the user experience and reduces the risk of forgetting crucial steps. By implementing this solution, you can enjoy the convenience of plug-and-play external storage, focusing on your tasks rather than managing the technical details. This documentation provides a comprehensive guide to set up the automation, ensuring your external drives are always ready when you need them. Have fun, happy coding!
