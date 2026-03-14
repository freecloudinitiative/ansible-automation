# k3s-master-install

This role handles the initial installation and configuration of the primary K3s master node.

## What it does

- It downloads the official K3s installation script.
- It checks if K3s is already installed on the system.
- It installs the K3s server with cluster initialization and specific configurations (disabling default CNI, Traefik, etc.).
- It waits for the K3s service to become active and ready on port 6443.
- It verifies the installation and displays the installed K3s version.
- It securely backs up the original bootstrap token.
- It removes the original bootstrap token file for security purposes.
