# ProxyMan

ProxyMan is a shell script to manage system-wide proxy settings on Linux systems. It supports both Debian/Ubuntu-based (using `apt`) and RHEL/CentOS/Fedora-based systems (using `dnf` or `yum`). It can also configure proxies for `wget`, Docker (both system daemon and per-user Docker settings), and system-wide environment variables.

## Features

- Sets and unsets proxies for:
  - `/etc/environment`
  - Package managers: `/etc/apt/apt.conf` (apt), `/etc/dnf/dnf.conf` (dnf), or `/etc/yum.conf` (yum)
  - `/etc/wgetrc`
  - Docker system-wide: `/etc/docker/daemon.json` and `/etc/systemd/system/docker.service.d/http-proxy.conf`
  - Per-user Docker config: `~/.docker/config.json`
- Creates backups of these configuration files when you run `set`, and restores them when you run `unset`.
- Provides `export` and `unexport` commands to easily apply or remove proxy settings from your current shell session without logging out.

## Requirements

- Run as root (via `sudo`), because it modifies system files.
- `/etc/proxy.conf` must exist with these variables set:
  
  ```bash
  HTTP_PROXY=http://proxy.example.com:8080
  HTTPS_PROXY=http://proxy.example.com:8080
  NO_PROXY=localhost,127.0.0.1,::1
  ```
  
Adjust these values to match your actual proxy setup.

## Installation
1. Copy the proxyman.sh script to a location in your $PATH, for example:
   
```bash
sudo cp proxyman.sh /usr/local/bin/proxyman
sudo chmod +x /usr/local/bin/proxyman
 ```

2. Ensure /etc/proxy.conf is created and contains `HTTP_PROXY` , `HTTPS_PROXY`, and `NO_PROXY`.

## Usage

Run all commands as root (via sudo):
- **Set proxy:**
  
```bash
sudo proxyman set
```

This will configure all system-wide files, Docker, and the per-user Docker config. It will print instructions on how to apply the settings to your current shell.

- **UNset proxy:**
  
```bash
sudo proxyman unset
```

This restores the original configurations from backups and removes the proxy settings. It will print instructions on how to remove the settings from your current shell.

- **List current settings:**
  
```bash
sudo proxyman list
```

Shows the current state of all relevant files.

- **Export Commands**

Use the following command to print export statements for `http_proxy`, `https_proxy`, `no_proxy`, and their uppercase equivalents:

```bash
sudo proxyman export
```

To apply these proxy settings to your current shell without reopening it, use:

```bash
eval "$(sudo proxyman export)"
```

- **Unexport Commands**

To print unset statements for removing proxy variables from your current shell:

```bash
sudo proxyman unexport
```

To apply these unset statements, use:

```bash
eval "$(sudo proxyman unexport)"
```

- **Help**

For help and usage information, use:

```bash
sudo proxyman -h
```

## Examples

### Set and Apply Proxy in Current Shell

```bash
sudo proxyman set
eval "$(sudo proxyman export)"
```

- ### Unset and Remove Proxy from Current Shell

```bash
sudo proxyman unset
eval "$(sudo proxyman unexport)"
```

Alternatively, you can reopen your terminal session after setting or unsetting proxies to apply or remove the proxy environment.

## Notes

- The script attempts to detect your system's package manager:
  - If `apt` is found, it configures `/etc/apt/apt.conf`.
  - If `dnf` or `yum` is found, it configures those accordingly.
  - If none of these package managers are found, it skips package manager proxy configuration.
  
- **Docker Integration:**
  - Requires Docker 20.10+ for the `proxies` key in `daemon.json`.
  - Creates a systemd drop-in file at `/etc/systemd/system/docker.service.d/http-proxy.conf` to allow the Docker daemon to pull images via the proxy.
  - Configures per-user Docker settings in `~/.docker/config.json` for the user who invoked `sudo`. If run as root, it uses the root user's home directory.

- **Backup Behavior:**
  - Backups are created only once. Subsequent runs will not overwrite the backups.
