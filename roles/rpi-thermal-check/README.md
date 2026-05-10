# RPi Thermal Check

This role monitors and reports CPU temperature on Raspberry Pi nodes.

## Purpose

Provides quick thermal diagnostics for Raspberry Pi clusters by:
- Reading CPU temperature via `vcgencmd measure_temp`
- Displaying temperature alongside hostname for easy identification
- Enabling temperature monitoring across multiple nodes simultaneously

## Behavior

- Queries the CPU temperature on each target host
- Outputs the result in debug format with the hostname for easy monitoring

## Requirements

- Raspberry Pi running Raspbian/Raspberry Pi OS
- `vcgencmd` utility available (included by default)
