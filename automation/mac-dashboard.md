# Mac Homelab Dashboard Launcher

**Skills demonstrated:** Bash | macOS shell | PATH | Symlinks | Workflow automation

## Project Overview

I created a small launcher on my Mac to open the main homelab dashboards from one command.

The script is located at:

```text
~/homelab-dashboard.sh
```

It opens the services I commonly use for administration and monitoring:

- Uptime Kuma
- Netdata
- Portainer
- Pi-hole
- Immich
- Cockpit

## Making the Script Executable

I made the launcher executable with:

```bash
chmod +x ~/homelab-dashboard.sh
```

## Creating a Simple Command

I created a shortcut command named:

```text
homelab-dashboard
```

using a symlink under:

```text
~/bin
```

and ensured that directory was available in my shell `PATH`.

Commands and concepts used included:

```bash
mkdir -p ~/bin
ln -sf
source ~/.zshrc
```

## Why I Built It

This is a small project, but it removes repetitive browser setup during routine administration.

Instead of manually opening several dashboards, I can use one command to start my normal monitoring workflow.

## What I Learned

This project reinforced:

- executable permissions
- shell scripts
- symbolic links
- the `PATH` environment variable
- shell configuration
- small-scale workflow automation

## Result

A single `homelab-dashboard` command opens the dashboards I use most often, making routine homelab checks faster and more consistent.
