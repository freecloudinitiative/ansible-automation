# Raspberry Pi Boot Config

This role configures Raspberry Pi boot parameters via `/boot/firmware/config.txt`.

## Purpose

Optimizes Raspberry Pi performance for Kubernetes clusters by:
- Enabling 64-bit ARM architecture and CPU boost
- Configuring fan control with temperature thresholds
- Disabling unnecessary hardware (Bluetooth, audio, HDMI)
- Minimizing GPU memory allocation to maximize RAM for k3s
- Managing LED indicators

## Behavior

- Deploys templated `config.txt` with customizable variables
- Automatically reboots the system if configuration changes

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `rpi_arm_64bit` | 1 | Enable 64-bit ARM architecture |
| `rpi_arm_boost` | 1 | Enable CPU frequency boost |
| `rpi_fan_temp0` | 40000 | Fan activation temperature (mC) |
| `rpi_fan_temp1` | 45000 | Secondary fan activation temperature (mC) |
| `gpu_mem` | 16 | GPU memory allocation in MB |
