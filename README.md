# ProxyMan

ProxyMan is a shell script to manage system-wide proxy settings on Linux systems. It supports both Debian/Ubuntu-based (using `apt`) and RHEL/CentOS/Fedora-based systems (using `dnf` or `yum`). In addition, it configures proxies for `wget`, Docker (both system daemon and per-user Docker settings), and system-wide environment variables like `http_proxy`, `https_proxy`, and `no_proxy`.


## Features

- **Package Manager Support:**  
  Detects your system’s package manager (`apt`, `dnf`, or `yum`) and configures the proxy settings accordingly.
  
- **System-Wide Environment Variables:**  
  Configures `/etc/environment` to set `http_proxy`, `https_proxy`, `no_proxy` and their uppercase variants globally.

- **Wget Proxy Configuration:**  
  Updates `/etc/wgetrc` so `wget` commands use the configured proxy.

- **Docker Proxy Configuration:**
  - Sets system-wide Docker proxy via `/etc/docker/daemon.json`.
  - Creates or updates a systemd drop-in file at `/etc/systemd/system/docker.service.d/http-proxy.conf` to allow the Docker daemon to pull images via the proxy.
  - Configures per-user Docker proxy settings in `~/.docker/config.json` for the user who ran `sudo`.

- **Backup and Restore Mechanism:**
  - When you run `set`, ProxyMan creates backups of all files it modifies, if not already existing.
  - When you run `unset`, it restores these backups, effectively removing the proxy settings.
  - When you run `purge`, it removes all proxy settings and all backup files permanently.

- **Interactive Mode:**
  - If `/etc/proxy.conf` does not exist, ProxyMan can prompt you for `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` interactively.
  - Default `NO_PROXY` values cover local IP ranges and docker0 networks, so you don’t have to guess.

- **Shell Integration:**
  - The `export` command prints `export` statements so you can `eval` them to apply proxy vars to your current shell session without reopening it.
  - The `unexport` command prints `unset` statements to remove the proxy vars from your current shell.

- **Safe and User-Friendly:**
  - Color-coded output helps distinguish success messages (green), prompts/warnings (yellow), and errors (red).
  - Before destructive operations like `purge`, it asks for confirmation.
  - Clear instructions are printed after `set` and `unset` so you know the next steps.
    
## Requirements

- Run as root (via `sudo`), since system-wide configuration files are modified.
- If you prefer a non-interactive setup, ensure `/etc/proxy.conf` exists with:
  
  ```bash
  HTTP_PROXY=http://proxy.example.com:8080
  HTTPS_PROXY=http://proxy.example.com:8080
  NO_PROXY=localhost,127.0.0.1,::1
  ```
  
Adjust these values to match your actual proxy setup.
If `/etc/proxy.conf` is missing, ProxyMan will prompt you interactively to enter these values when you run `set`.

## Installation
1. Copy the proxyman.sh script to a location in your $PATH, for example:
   
```bash
sudo cp proxyman.sh /usr/local/bin/proxyman
sudo chmod +x /usr/local/bin/proxyman
 ```

2. Ensure /etc/proxy.conf is created and contains `HTTP_PROXY` , `HTTPS_PROXY`, and `NO_PROXY`.

## Usage

Run all commands with `sudo`:
- **Set proxy:**
  
```bash
sudo proxyman set
```

This configures all system-wide files, Docker, and per-user Docker config. If `/etc/proxy.conf` is missing, it will prompt you interactively. After setting, ProxyMan will print instructions on how to apply these settings to your current shell (using `eval "$(sudo proxyman export)"`).

- **Unset proxy:**
  
```bash
sudo proxyman unset
```

Restores original configurations from backups and removes proxy settings. After unsetting, it will print instructions on how to remove the settings from your current shell (using `eval "$(sudo proxyman unexport)"`).

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
