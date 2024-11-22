
# Assignment 3: Nginx & UFW 

## Introduction

This assignment will teach you how to set up a Bash script to generate a static index.html file containing system information. The script will run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script will be hosted by an nginx web server on your Arch Linux droplet, which will aslo be using a ufw firewall setup to help secure your server.

## Table of Contents

something


# Setup Instructions for New Server

## Task 1

Before we start, we need to create a system user with a home directory and a login shell for a non-login user.

1. Type the following command to create a system user with a home directory

```
sudo useradd -r -m -d /var/lib/webgen -s /bin/bash webgen
```

-r: Creates a system account.

-m: Creates the home directory if it doesn't exist.

-d: Specifies the home directory.

-s: Specifies the login shell, which we'll use /bin/bash.


