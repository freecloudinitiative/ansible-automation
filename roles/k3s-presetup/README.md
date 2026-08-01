# k3s-presetup

This role handles the initial system preparation required before installing K3s.

## What it does

- It immediately turns off the system swap memory.
- It disables swap memory permanently by removing it from the `/etc/fstab` file.
- It disables and stops the `dphys-swapfile` service if it is present.
- It updates the system's APT package cache.
- It installs all the required packages necessary for the K3s installation.
