# Secure Docker & Docker-Compose Wrappers

These wrappers provide a defensive layer around docker and docker-compose by blocking all host bind-mounts to protected system paths (e.g., /etc, /root, /usr, /var/lib, /proc, /sys, etc.).
This prevents users in the docker group—or users allowed to run your wrappers via sudo—from performing root-equivalent actions such as:

Mounting /etc and modifying system configuration
Mounting /root and replacing SSH keys
Mounting /var/run/docker.sock and taking over the host
Exploiting symlinks to gain access to protected directories
These wrappers allow safe mounts from user-owned directories such as /home, /srv, /opt, /mnt, /media, and relative paths.
Features

Blocks -v /host:/container mounts into protected locations
Blocks long-syntax mounts (--mount type=bind,source=/etc)
Resolves symlinks to prevent bypasses
Works for both docker and docker-compose
Supports relative paths and named volumes
Drop-in replacement (no patching of Docker engine required)

## Installation

Place the wrappers in /usr/local/bin/:

sudo cp docker /usr/local/bin/docker
sudo cp docker-compose /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker /usr/local/bin/docker-compose


Ensure real binaries exist at:

/usr/bin/docker
/usr/bin/docker-compose


If not, edit the REAL_DOCKER= and REAL_DC= variables inside the wrappers.

Ensure /usr/local/bin comes before /usr/bin in PATH.

Check with:
echo $PATH
Testing
Run safe and unsafe test cases from the list provided (mounting /etc should be blocked, mounting ~/foo should be allowed).

Security Notes

These wrappers greatly reduce accidental or naive privilege escalation but do not make Docker safe for untrusted users.
Users allowed to run Docker can still:

Execute containers with excessive Linux capabilities

Use --privileged mode

Exploit kernel flaws

Use device passthrough

For full isolation:
Use rootless Docker, Podman, or VMs.

Allowing Users to Run These Wrappers Without a Password

If you want certain users to run docker and docker-compose without a password but only through your wrapper, do not allow direct access to /usr/bin/docker.

Instead, allow only the wrapper paths.

## Example: visudo (NOPASSWD)

Run:

`sudo visudo`


Add:
```
# Allow user 'developer' to run only the wrapper docker binaries without password
developer ALL=(root) NOPASSWD: /usr/local/bin/docker, /usr/local/bin/docker-compose
```
Important: DO NOT whitelist /usr/bin/docker !

That would bypass your protections.

Optional: Block direct access to the real Docker binary

You may set permissions so only root can execute /usr/bin/docker:
```
sudo chmod 750 /usr/bin/docker
sudo chown root:root /usr/bin/docker
```

This forces all non-root usage to go through the wrapper.

Usage

Now the user can run:

sudo docker run ...
sudo docker-compose up


With no password prompt, but still subject to your restrictions.
