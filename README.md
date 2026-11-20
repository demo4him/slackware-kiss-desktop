
# Slackware KISS Desktop

A KISS post-install configuration layer for Slackware64 15.0.

The project provides a collection of independent tools for configuring a lightweight Slackware desktop. It does not replace the official Slackware installer or modify the original installation media.

## Project structure

```text
slackware-kiss-desktop/
├── display-manager/
│   └── display-manager.sh
├── pkglists/
│   ├── packages-add
│   └── packages-remove
├── sbo/
│   └── sbo-manager.sh
├── scripts/
│   └── kiss-trim.sh
├── tagfiles/
│   ├── a/
│   ├── ap/
│   ├── d/
│   ├── k/
│   ├── l/
│   ├── n/
│   ├── t/
│   ├── x/
│   └── xap/
└── xfce/
    ├── DEPENDENCY-AUDIT.md
    ├── install-xfce.sh
    ├── packages-official
    └── profiles/
        ├── minimal
        └── standard
```

## XFCE installer

The official XFCE installer uses Slackware64 15.0 packages.

Run as root:

```sh
./xfce/install-xfce.sh
```

Available profiles:

- **Minimal** — 12 packages.
- **Standard** — 20 packages.
- **Custom** — all packages initially disabled.

A dialog checklist allows the user to review and change the selection before installation.

Dry-run examples:

```sh
./xfce/install-xfce.sh --profile minimal --dry-run
./xfce/install-xfce.sh --profile standard --dry-run
./xfce/install-xfce.sh --profile custom --dry-run
```

The dry-run validates the selection without synchronizing repositories or installing packages.

After installation, use `xwmconfig` to select XFCE for `startx`.

The installer does not automatically overwrite the user's X session configuration.

## SlackBuilds.org manager

The SBo manager provides a simple dialog checklist for optional SlackBuilds.org packages.

Run as root:

```sh
./sbo/sbo-manager.sh
```

Requires `sbopkg` configured for Slackware 15.0.

Optional packages:

- xarchiver
- thunar-archive-plugin
- xfce4-genmon-plugin
- xfce4-xkb-plugin

All options are disabled initially. Only selected packages are passed to sbopkg.

## LXDM display manager

The display manager module configures LXDM exclusively.

Run as root:

```sh
./display-manager/display-manager.sh
```

The module detects the active local desktop user and configures:

- LXDM installation, if not already installed.
- Automatic login for the detected user.
- elogind PAM integration.
- LXDM startup through `/etc/rc.d/rc.4`.
- Runlevel 4 as the default runlevel.

Configuration files are backed up before modification.

Automatic login can be disabled by commenting out the active `autologin=` line in `/etc/lxdm/lxdm.conf`.

To return to the console-oriented default, change `id:4:initdefault:` to `id:3:initdefault:` in `/etc/inittab`.

## Package reconciliation

`scripts/kiss-trim.sh` provides a conservative package reconciliation helper.

It reads:

- `pkglists/packages-add`
- `pkglists/packages-remove`

Both lists are intentionally empty by default.

Run a report without modifying the system:

```sh
./scripts/kiss-trim.sh --dry-run
```

The helper supports an explicit apply operation, with confirmation, after reviewing the proposed changes.

It does not resolve package dependencies.

## Tagfiles

The `tagfiles/` directory contains the nine Slackware package series supplied by the project:

`a`, `ap`, `d`, `k`, `l`, `n`, `t`, `x` and `xap`.

These files are intended to support a minimal and controlled Slackware installation.

## Design principles

- Keep it simple.
- Use native Slackware tools wherever possible.
- Keep modules independent.
- Make optional software opt-in.
- Prefer explicit confirmation before system changes.
- Back up configuration files before modifying them.
- Avoid unnecessary services, dependencies and automation.
- Keep the original Slackware installation philosophy intact.

## Target platform

- Slackware64 15.0
- XFCE
- LXDM
- dialog
- sbopkg (for optional SBo packages)

The project is developed and tested on Slackware64 15.0.
