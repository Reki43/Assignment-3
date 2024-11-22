
# Assignment 3: Nginx & UFW 

## Introduction

This assignment will teach you how to set up a Bash script to generate a static index.html file containing system information. The script will run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script will be hosted by an nginx web server on your Arch Linux droplet, which will aslo be using a ufw firewall setup to help secure your server.

## Table of Contents

something


# Setup Instructions for New Server

## Task 1 - Create the System User and Directory Structure

Before we start, we need to create a system user with a home directory and a login shell for a non-login user.

1. Creating the system user

Type the following command to create a system user with a home directory

```
sudo useradd -r -m -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

**-r:** Creates a system account.[^2]

**-m:** Creates the home directory if it doesn't exist.

**-d:** Specifies the home directory.

**-s /usr/sbin/nologin webgen:** Specifies a non-login shell to prevent interactive logins.[^1]





2. 


























# References

[^1]: A. I. Nagori and H. Gerganov, "Creating a Non-login User on Linux," Baeldung. https://www.baeldung.com/linux/create-non-login-user. [Accessed: 19-Nov-2024].


[^2]: `man useradd` - Use `-r` options for creating a system user.


