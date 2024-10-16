# Configure Anything LLM Backup to Google Drive

This tutorial will guide you through creating a script to compress a folder, upload it to Google Drive using `rclone`, and schedule this task to run automatically every 12 hours on an linux server.

![https://rclone.org/img/logo_on_dark__horizontal_color.svg](https://rclone.org/img/logo_on_dark__horizontal_color.svg)

## Prerequisites

- Access to an Ubuntu 22.04 server.
- Google Drive account.
- `rclone` installed and configured.
- Administrator permissions to install packages and configure `crontab`.

## Step 1: Install Dependencies

Before starting, install the `zip` utility if it's not already installed:

```bash
sudo apt update
sudo apt install zip -y
```

## Step 2: Configure Rclone

Set up `rclone` to access Google Drive:

1. Run the configuration command:

   ```bash
   rclone config
   ```

2. Follow the instructions to create a new remote configuration for Google Drive.
3. Make sure to authenticate `rclone` with your Google account.

## Step 3: Create the Backup Script

Create a script named `backup_volume.sh` with the following content:

```bash
#!/bin/bash

SOURCE_DIRECTORY="/home/anythingllm/volume"
BACKUP_DIRECTORY="/home/rclone/backuptmp"
REMOTE_NAME="DockerBackup"
REMOTE_PATH="server/anythingllm"
DATE=$(date +%Y-%m-%d)

mkdir -p "$BACKUP_DIRECTORY"

ZIP_FILE="${BACKUP_DIRECTORY}/volume_backup_${DATE}.zip"
zip -r "$ZIP_FILE" "$SOURCE_DIRECTORY"

if [ -f "$ZIP_FILE" ]; then
    echo "Zip file created successfully: $ZIP_FILE"
    rclone copy "$ZIP_FILE" "$REMOTE_NAME:$REMOTE_PATH"
    if [ $? -eq 0 ]; then
        echo "Zip file uploaded successfully to Google Drive: $REMOTE_NAME:$REMOTE_PATH"
        rm "$ZIP_FILE"
        echo "Local zip file removed: $ZIP_FILE"
    else
        echo "Error uploading zip file to Google Drive"
    fi
else
    echo "Error creating zip file"
fi
```

## Step 4: Make the Script Executable

Make the script executable with the following command:

```bash
chmod +x /home/rclone/script/backup_volume.sh
```

## Step 5: Schedule with Crontab

Schedule the script to run every 12 hours:

1. Open `crontab` for editing:

   ```bash
   crontab -e
   ```

2. Add the following line to the file:

   ```bash
   0 */12 * * * /bin/bash /home/rclone/script/backup_volume.sh
   ```

3. Save and close the editor.

## Conclusion

Your backup script is now set up to run automatically every 12 hours, compressing the specified folder and uploading the zip file to Google Drive. Be sure to periodically check the logs to ensure everything is functioning as expected.
