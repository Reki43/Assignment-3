
# Assignment 3: Nginx & UFW 

## Introduction

This assignment will teach you how to set up a Bash script to generate a static index.html file containing system information. The script will run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script will be hosted by an nginx web server on your Arch Linux droplet, which will aslo be using a ufw firewall setup to help secure your server.

## Table of Contents

something


# Setup Instructions for New Server

## Task 1 - Create the System User and Directory Structure

Before we start, we need to create a system user with a home directory and a login shell for a non-login user. The reason we would want to create a system user is because it enhances security by limiting permissions. It prevents direct logins to the system user, which reduces the risk of unauthorized access and accidental changes.[^1]

**1. Creating the system user**

Type the following command to create a system user with a home directory:

```
sudo useradd -r -m -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

**-r:** Creates a system account.[^2]

**-m:** Creates the home directory if it doesn't exist.

**-d:** Specifies the home directory.

**-s /usr/sbin/nologin webgen:** Specifies a non-login shell to prevent interactive logins.[^1]

**2. Create the Necessary Directory Structure and Files**

Type the following to create the sub-directories `bin` and `HTML` for webgen directory:
```
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
```

**3. Create generate_index File**

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

**3. Create `index.html` File in the /webgen/HTML directory**
```
sudo nvim /var/lib/webgen/HTML/index.html
```

**4. Set Ownership**

Type the following commmand to set ownership of the home directory and its contents to webgen:

```
sudo chown -R webgen:webgen /var/lib/webgen
```

[!NOTE] Running this command changes the ownership to the webgen user, ensuring that only the webgen user has the necessary permissions to access and manage its home directory and all files within it.






























# References
[^1]: A. I. Nagori and H. Gerganov, "Creating a Non-login User on Linux," Baeldung. https://www.baeldung.com/linux/create-non-login-user. [Accessed: 19-Nov-2024].

[^2]: `man useradd` - Use `-r` options for creating a system user.







