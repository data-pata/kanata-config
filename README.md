```
 _  __                 _
| |/ /__ _ _ __   __ _| |_ __ _
| ' // _` | '_ \ / _` | __/ _` |
| . \ (_| | | | | (_| | || (_| |
|_|\_\__,_|_| |_|\__,_|\__\__,_|

⌨ Your keyboard isn't broken, but lets fix it anyway ⌨
```

This repo holds **opinionated** configuration examples, setup instructions and development utilities for your keyboard hacks with [Kanata](https://github.com/jtroo/kanata). Tested on Ubuntu 24.04 - your mileage may vary.

---

## Installation

### 1. Install Kanata

Install via Homebrew:

```bash
brew install kanata
```

The systemd service file uses the default Linuxbrew prefix
(`/home/linuxbrew/.linuxbrew/bin/kanata`). If your brew prefix differs, update `ExecStart` in
`kanata.service` accordingly — you can find the correct path with `$(brew --prefix)/bin/kanata`.

### 2. Set Up Permissions

Since Kanata intercepts hardware keys, it needs permission to access `/dev/uinput` and `/dev/input`.

**Create the groups and add yourself:**
```bash
sudo groupadd --force --system uinput
sudo usermod -aG input,uinput $USER
```

**Add a udev rule:**
```bash
echo 'KERNEL=="uinput", MODE="0660", GROUP="uinput", OPTIONS+="static_node=uinput"' | sudo tee /etc/udev/rules.d/99-input.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
sudo modprobe uinput
```

> **Note:** Log out and log back in for group changes to take effect.

### 3. Create Your Configuration

Create a `.kbd` file (e.g., `~/.config/kanata/config.kbd`). If no config file is supplied, kanata tries to open `kanata.kbd` in the current directory.

### 4. Set Up Systemd Autostart

Copy the systemd service file to your user systemd directory:
```bash
mkdir -p ~/.config/systemd/user
cp kanata.service ~/.config/systemd/user/
```

Enable and start the service:
```bash
systemctl --user daemon-reload
systemctl --user enable kanata.service
systemctl --user start kanata.service
systemctl --user status kanata.service
```

---

## Development

### Running Kanata Manually

**Basic run:**
```bash
kanata --cfg ~/.config/kanata/config.kbd
```

**With debug output:**
```bash
kanata --cfg ~/.config/kanata/config.kbd --debug
```

> **Emergency exit:** Press `Ctrl+Space+Esc` to kill Kanata.

### Auto-Reload with entr

Automatically restart kanata when your config file changes:

```bash
sudo apt install entr
ls config.kbd | entr -r kanata --cfg config.kbd
```

Now kanata will restart automatically whenever you save `config.kbd`.

### Testing Configs with Systemd Running

When you have systemd autostart enabled but want to test a new config:

```bash
# Stop the systemd service
systemctl --user stop kanata.service

# Test your new config (with auto-reload)
ls test-config.kbd | entr -r kanata --cfg test-config.kbd

# When done testing, restart the systemd service
systemctl --user start kanata.service
```

---

## Tips and Tricks

### Useful Tasks

> **Prerequisite:** Install [Task](https://taskfile.dev/installation/) - a modern task runner alternative to Make.

```bash
# Validate config syntax
task check

# Run kanata with debug output and auto-reload (stops systemd first)
task debug

# install kanata as a systemd service
task install

# Check if kanata service is running
task status

# View kanata service logs (follow)
task logs
```

### VS Code Extension

Kanata uses `.kbd` files. For syntax highlighting and error checking, install the **"Kanata Configuration Language"** extension by `rszyma`.
