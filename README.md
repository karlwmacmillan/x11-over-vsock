X11 over Vsock
==============

![CI](https://github.com/nbdd0121/x11-over-vsock/workflows/CI/badge.svg?branch=master)

## Background

Windows will reset all external TCP connections when network changes or when PC resumes from disconencted sleep/hibernation, which include connections on the WSL bridge. If you are using X11, this can be annoying because all X11 connections over TCP will also drop.

## Solution

Unlike TCP connections, Vsock connections will not be dropped. Vsock is VM socket for communication between the guest VM and the host, mostly used to provide [integration service](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/make-integration-service). In WSL2, Vsock is used for many interops (e.g. file/network/executable). This program is just another integration service.

Two executables are to be ran, one inside WSL2 and another outside. The program inside WSL2 will listen on Unix socket /tmp/.X11-unix/X0 (DISPLAY=:0) and forward it the program outside WSL2 via Vsock. The program outside WSL2 will listen on the Vsock and forward it to TCP port 6000 to which your X server should listen.

## Build

This program is written in Rust. If you do not have Rust toolchain installed you can get it from https://rustup.rs/.

Install in both WSL and Windows using `cargo install --git https://github.com/nbdd0121/x11-over-vsock` (The binary will be installed to `~/.cargo/bin/x11-over-vsock` and `%USERPROFILE%\.cargo\bin\x11-over-vsock.exe`).

You can also download pre-built binaries from [GitHub Actions artifacts](https://github.com/nbdd0121/x11-over-vsock/actions).

## Usage

In WSL, run `x11-over-vsock` and set `DISPLAY=:0`.

In Windows, start a X server (e.g. VcXsrv) on TCP port 6000, and you can either:
* Execute `hcsdiag list` with administrator privilege to get the VMID of your WSL instance, then `x11-over-vsock.exe <VMID>` (no administrator privilege required). WSL must be running before execution, and you will need to kill and start the process again if shutdown you shutdown the WSL utility VM.
* Execute `x11-over-vsock.exe` with administrator privilege. It will automatically retrieve WSL VMID. WSL must be running before execution, and you will need to kill and start the process again if you shutdown the WSL utility VM.
* Execute `x11-over-vsock.exe --daemon` with administrator privilege. It will poll WSL status every 5 seconds, and start/shutdown server automatically. A service manager such as NSSM (http://nssm.cc/) can be used to run this automatically in the background.

## Example Configuration

On Windows:

(Chocolately instructions are included, but all of this can be done manually)

1. Install an X server (e.g., `choco install xcxsrv`)
1. Install NSSM: `choco install nssm`
1. Use nssm to install service: `nssm.exe install x11-over-vsock` - set Path to where you put the `x11-over-vsock.exe` and add the Arguments to `--daemon`.

![NSSM Configuration Image](images/nssm.png?raw=true)

1. Start the x11-over-vsock service in PowerShell: `Start-Service x1l-over-vsock`.
1. Start the X server

In WSL:
1. Place `x11-over-vsock` binary somewhere in your path (e.g., `~/bin`)
1. Start `x11-over-vsock` automatically in `.bashrc` or `.zshrc` and set environment variables:

    ```
    if ! pgrep -u $USER x11-over-vsock >/dev/null 2>&1; then
            x11-over-vsock &
    fi
    export DISPLAY=:0
    ```
1. Source the configuration (e.g., `source ~/.zshrc`)
1. Start an X app (e.g., `xterm`)

At this point, everything should be working as normal.