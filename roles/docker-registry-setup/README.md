# docker-registry-setup

Deploys a lightweight [Docker Registry v2](https://hub.docker.com/_/registry) along with an interactive Web UI ([joxit/docker-registry-ui](https://github.com/Joxit/docker-registry-ui)) on K3s.

## Features

- **Docker Registry API**: Listens on port `5000` (`http://<PUBLIC_IP>:5000`).
- **Web UI**: Listens on port `5001` (`http://<PUBLIC_IP>:5001`).
- **CORS & Image Deletion**: CORS headers enabled for frontend access and image deletion capability configured.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `docker_registry_namespace` | `"docker-registry"` | Target Kubernetes namespace |
| `docker_registry_image` | `"registry:2"` | Official Docker Registry container image |
| `docker_registry_port` | `5000` | Registry API service port |
| `docker_registry_storage_size` | `"10Gi"` | PVC storage request size |
| `docker_registry_ui_enabled` | `true` | Enable Docker Registry Web UI deployment |
| `docker_registry_ui_image` | `"joxit/docker-registry-ui:latest"` | Container image for Web UI |
| `docker_registry_ui_port_forward_local_port` | `5001` | Public port-forward port for Web UI |

## Example Playbook

```yaml
- hosts: "{{ groups['masters'][0] }}"
  become: true
  roles:
    - docker-registry-setup
```
