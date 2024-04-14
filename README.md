# Intro

This guide will walk you through the process of setting up simple notifications for whenever a user logs into your Linux server via SSH.


# Installation

## AIO setup script

Coming soon

## Manual setup

### Using Gotify

#### Requirements

- Gotify: https://gotify.net/
- Curl: https://curl.se/docs/manpage.html

#### Steps

1. Create a new file in `/etc/ssh` called `login-alert.sh` (i.e. with nano: `sudo nano /etc/ssh/login-alert.sh`)
2. Paste the following into the newly created file and save it:
```bash
#!/bin/sh

GOTIFY_URL="https://your-gotify-instance-url.com/"
GOTIFY_TOKEN="YOUR_APP_TOKEN"

MSG_TITLE="SSH Login Alert"
MSG_BODY="`env`"

if [ "$PAM_TYPE" != "close_session" ]; then
        message="`env`"
        curl "${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}" -F "title=${MSG_TITLE}" -F "message=${MSG_BODY}"
fi
```
3. Set required permissions: `sudo chmod +x login-alert.sh`
4. Change ownership to root, protecting it from unauthorized changes: `sudo chown root:root login-alert.sh`
5. Tell SSH to execute the script whenever someone logs in by adding the following to `/etc/pam.d/sshd`:
```bash
session optional pam_exec.so seteuid /etc/ssh/login-alert.sh
``` 

It is recommended to keep in the "optional" keyword for testing purposes. After verifying the alert works as intended, you can change it to "required". However, be aware that if the alert script exits with an error (i.e. if your Gotify instance is unreachable) it will prevent SSH login.
6. Make sure `UsePAM` is set to `yes` in `/etc/ssh/sshd_config`.

#### Example message

The default script will send the following message on login:
```
XDG_SESSION_TYPE=tty
PAM_USER=<USER_WHO_LOGGED_IN>
MOTD_SHOWN=pam
PAM_RHOST=<IP_ADDRESS_OF_USER>
PAM_TYPE=open_session
SSH_AUTH_INFO_0=<FOR_EXAMPLE_THE_PUBLIC_KEY>
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
XDG_SESSION_CLASS=user
XDG_SESSION_ID=<SESSION_ID>
PAM_SERVICE=sshd
PAM_TTY=ssh
XDG_RUNTIME_DIR=/run/user/1000
LANG=en_US.UTF-8
PWD=/
SSH_CONNECTION=<USER_IP_ADDRESS> <ID> <SSH_HOST_IP> <SSH_HOST_PORT>
```

# Used resources

- Fritz, 2014-04-16: [https://askubuntu.com/a/448602](https://askubuntu.com/a/448602)
- Nicolas BADIA, 2025-03-15: [https://askubuntu.com/questions/179889/how-do-i-set-up-an-email-alert-when-a-ssh-login-is-successful#comment829103_448602](https://askubuntu.com/questions/179889/how-do-i-set-up-an-email-alert-when-a-ssh-login-is-successful#comment829103_448602)