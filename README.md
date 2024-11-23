
# Assignment 3: Nginx & UFW 

## Introduction

This assignment will teach you how to set up a Bash script to generate a static index.html file containing system information. The script will run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script will be hosted by an nginx web server on your Arch Linux droplet, which will aslo be using a ufw firewall setup to help secure your server.

## Table of Contents

something


# Setup Instructions for New Server

## Task 1 - Create the System User and Directory Structure

Before we start, we need to create a system user with a home directory and a login shell for a non-login user. The reason we would want to create a system user is because it enhances security by limiting permissions. It prevents direct logins to the system user, which reduces the risk of unauthorized access and accidental changes.[^1]

**1. Creating the system user**

Type the following command to create a system user with a specified custom home directory[^1]:

```
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

**-r:** Creates a system account.[^1][^2]

**-d:** Specifies the home directory.

**-s /usr/sbin/nologin webgen:** Specifies a non-login shell to prevent interactive logins.[^1]

>[!NOTE]
>The reason the `-d` option was used is to specify a custom home directory for the system user. Although system users don't require a home directory, in this case, it helps in organizing and managing files related to the user's tasks. 

**2. Copy and Paste the following command to create the home directory, as it doesn't exist yet:**

```
sudo mkdir -p /var/lib/webgen
```

**3. Create the Necessary Directory Structure and Files**

Type the following to create the sub-directories `bin` and `HTML` for the `webgen` directory:
```
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
```

**4. Create generate_index File**

Type the following to create and enter the generate_index file:

```
sudo nvim /var/lib/webgen/bin/generate_index
```

Copy and paste the following script. When executed, this will generate a HTML structure into your `index.html` file.
```
#!/bin/bash

set -euo pipefail

# this is the generate_index script
# you shouldn't have to edit this script

# Variables
KERNEL_RELEASE=$(uname -r)
OS_NAME=$(grep '^PRETTY_NAME' /etc/os-release | cut -d '=' -f2 | tr -d '"')
DATE=$(date +"%d/%m/%Y")
PACKAGE_COUNT=$(pacman -Q | wc -l)
OUTPUT_DIR="/var/lib/webgen/HTML"
OUTPUT_FILE="$OUTPUT_DIR/index.html"

# Ensure the target directory exists
if [[ ! -d "$OUTPUT_DIR" ]]; then
    echo "Error: Failed to create or access directory $OUTPUT_DIR." >&2
    exit 1
fi

# Create the index.html file
cat <<EOF > "$OUTPUT_FILE"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System Information</title>
</head>
<body>
    <h1>System Information</h1>
    <p><strong>Kernel Release:</strong> $KERNEL_RELEASE</p>
    <p><strong>Operating System:</strong> $OS_NAME</p>
    <p><strong>Date:</strong> $DATE</p>
    <p><strong>Number of Installed Packages:</strong> $PACKAGE_COUNT</p>
</body>
</html>
EOF

# Check if the file was created successfully
if [ $? -eq 0 ]; then
    echo "Success: File created at $OUTPUT_FILE."
else
    echo "Error: Failed to create the file at $OUTPUT_FILE." >&2
    exit 1
fi
```

**5. Create `index.html` File in the /webgen/HTML directory**
```
sudo nvim /var/lib/webgen/HTML/index.html
```

**6. Set Ownership**

Type the following commmand to set ownership of the home directory and its contents to webgen:

```
sudo chown -R webgen:webgen /var/lib/webgen
```

>[!NOTE] 
>Running this command changes the ownership to the webgen user, ensuring that only the webgen user has the necessary permissions to access and manage its home directory and all files within it.


## Task 2 - Creating the generate-index.service and generate-index.timer scripts


**1. Create the generate-index.service File**

Type the following to create and enter a service file named `generate-index.service` in your system directory:

```
sudo nvim /etc/systemd/system/generate-index.service
```

Type the following into the `generate-index.service` file:

```ini
[Unit]
Description=Generate Index Service


[Service]
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

**2. Create the generate-index.timer File**

Type the following to create and enter a service file named `generate-index.timer` in your system directory:

```
sudo nvim /etc/systemd/system/generate-index.timer
```

Type the following into the `generate-index.service` file:

```ini
[Unit]
Description=Timer for Generate Index Service

[Timer]
OnCalendar=*-*-* 05:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```

**3. Enable and Start the Timer**

Type the following command to set the timer to start automatically at boot:

```
sudo systemctl enable generate-index.timer
```

Type the following command to iniate the timer immediately:

```
sudo systemctl start generate-index.timer
```

>[!IMPORTANT]
>Check timer status is **active** by typing `sudo systemctl status generate-index.timer`

**4. Manual Trigger Service for Testing**
>[!NOTE]
>Do not enable service. We just want to start it to check if the `generate-index.service` will work when active.

Type the following command to start the service:
```
sudo systemctl start generate-index.service
```


>[!NOTE]
>When you start `generate-index.service` , index.html's structure will be successfully written if the service output message shows that it has started, executed, and deactivated successfully.

5. Check if Status is Active and if the Service Executed

Type the following command to check the status of the service:

```
sudo systemctl status generate-index.service
```

Type the folowing to view the logs of the service:

```
sudo journalctl -u generate-index.service
```

>[!NOTE]
>When you check the service status, Active will say **inactive (dead)** because the `generate-index.service` successfully completed its task and then stopped, as it only executes the `generate_index` script once. However, when you check the logs, you should see detailed entries confirming the script's successful execution and any relevant output generated during its run.


## Task 3 - Modifying nginx.conf and Creating Server Blocks

**1. Install nginx**
```
sudo pacman -S nginx
```

**2. Modify the Main `nginx.conf` File**

Type the following command to access and edit the `nginx.conf` file[^3]:
```
sudo nvim /etc/nginx/nginx.conf
```

**3. Locate `user` in nginx.conf and change it to `webgen webgen`**
```
user webgen webgen;
```
>[!NOTE]
>The first user indicates the username and the second user indicates the usergroup. This allows nginx.conf the correct permissions for managing files and directories associated with the webgen user and group.


**4. Create the `sites-available` and `sites-enabled` directory**

Creates the `sites-available` directory[^3]:
```
sudo mkdir -p /etc/nginx/sites-available
```

Creates the `sites-enabled` directory[^3]:
```
sudo mkdir -p /etc/nginx/sites-enabled
```


**5. Create a Separate Server Block File into `sites-available`**

Type the following command to create a new server block file that'll configure Nginx to host index.html[^3]:

```
sudo nvim /etc/nginx/sites-available/webgen
```

>[!NOTE]
>We created a separate server block file instead of modifying the main nginx.conf file because it helps manage configurations more easily[^3]. Through this approach, it allows you to split a large configuration into smaller, manageable files.Therefore, you can enable or disable specific parts quickly without affecting the entire setup, keeping your configuration organized and easier to maintain.

Copy and paste the following server block into your webgen file:

```
server {
   listen 80;
   listen [::]:80;

   server_name localhost.webgen;

   root /var/lib/webgen/HTML;
   index index.html;

        location / {
        try_files $uri $uri/ =404;
    }
}
```

>[!NOTE]
>This server block is set to listen for HTTP requests on port 80, which is the default port for http traffic[^4]. For the server name, we named it localhost.webgen. Make sure the `server_name` is always unique. The root directory `/var/lib/webgen/HTML` specifies where Nginx will look for files to host, and the default file `index.html` ensures that users can access the website's main content correctly.


**6.Enable the Server Block**

Type the following command to symlink and enable the server block[^3]:

```
sudo ln -s /etc/nginx/sites-available/webgen /etc/nginx/sites-enabled/
```

**7. Add the `sites-enabled` Directory in nginx.conf file**

```
http {
    ...
    include /etc/nginx/sites-enabled/*;
}
```

>[!NOTE]
>Type `sudo nginx -t` to verify if `nginx.conf` has any errors

**8. Restart Nginx**

Copy and paste the following command to restart Nginx:

```
sudo systemctl restart nginx
```

**9. Start Nginx**

Copy and paste the following command to start Nginx:

```
sudo systemctl start nginx
```

>[!IMPORTANT]
> Type `sudo systemctl status nginx` to check if Nginx is **active** and **running**


## Task 4 - Configuring UFW for SSH and HTTP




























# References
[^1]: "Users and groups - ArchWiki." Arch Linux, 23 Nov. 2024. [Online]. https://wiki.archlinux.org/title/Users_and_groups#Example_adding_a_system_user. [Accessed: 23-Nov-2024].


[^2]: `man useradd` - Use `-r` options for creating a system user.

[^3]: "nginx - ArchWiki." Arch Linux, 7 Nov. 2024. [Online] https://wiki.archlinux.org/title/Nginx. [Accessed: 22-Nov-2024].

[^4]: [1] "Week Twelve Notes," CIT2420 Notes, 2024. https://gitlab.com/cit2420/2420-notes-f24/-/blob/main/2420-notes/week-twelve.md. [Accessed: Nov. 23, 2024].









