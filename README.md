# Slackware KISS Desktop

**Version:** 0.3.4  
**Target:** Slackware64 15.0

A KISS post-install configuration layer for Slackware64 15.0, inspired by the
manifest-based desktop trimming approach used by Microlinux MLED. It uses
native Slackware tools and explicit package lists. It does not replace the
official Slackware installer or modify the Slackware installation media.

## Quick start

Clone the repository as a normal desktop user:

```sh
git clone https://github.com/demo4him/slackware-kiss-desktop
cd slackware-kiss-desktop
```

Switch to root without changing the working directory:

```sh
su
```

First preview the XFCE selection and package plan:

```sh
./desktop-kiss.sh --dry-run
```

Review the selected XFCE packages, generated ADD/REMOVE manifests, and trim
report. If the plan is correct, start the real workflow:

```sh
./desktop-kiss.sh
```

The real workflow modifies the installed system. Read **MLED Trim and package
reconciliation** before applying it. The project is intended for Slackware64
15.0 and should first be tested in a disposable virtual machine or on a
recoverable installation.

## Main workflow

`desktop-kiss.sh` is the main entry point. Run it as root from an active local
desktop session.

The real workflow runs in this order:

1. Run the official XFCE installer and choose a Minimal, Standard, or Custom
   profile. The exact final checklist selection is saved in
   `pkglists/xfce-selected`.
2. Read the nine supplied tagfiles and generate `pkglists/packages-add`.
3. Compare installed packages with the keep set: tagfile ADD packages, the
   selected XFCE packages, and entries in `pkglists/packages-keep`. Generate
   `pkglists/packages-remove` from installed packages outside that set.
4. Install missing ADD packages with `slackpkg` and verify that every ADD
   package is present.
5. Only after successful ADD verification, remove packages listed in
   `packages-remove` with `removepkg`.
6. Offer the optional SlackBuilds.org package manager.
7. Configure LXDM.

If ADD installation or verification fails, the REMOVE phase does not start.
The trim stage requires an active Slackware mirror in
`/etc/slackpkg/mirrors`.

Choose an XFCE preset directly:

```sh
./desktop-kiss.sh --profile minimal
./desktop-kiss.sh --profile standard
./desktop-kiss.sh --profile custom
```

Preview a specific preset without installing or removing system packages:

```sh
./desktop-kiss.sh --profile standard --dry-run
```

Optional stages can be skipped:

```sh
./desktop-kiss.sh --skip-trim
./desktop-kiss.sh --skip-sbo
./desktop-kiss.sh --skip-lxdm
```

Options can be combined, for example:

```sh
./desktop-kiss.sh --skip-sbo --skip-lxdm
```

Use `./desktop-kiss.sh --help` to display all options.

The orchestrator checks prerequisites before making package changes. The SBo
and LXDM stages require `sbopkg` configured for Slackware 15.0. LXDM setup
also requires an active local desktop session so the module can identify the
logged-in user. Use `--skip-sbo` and `--skip-lxdm` if those optional stages are
not wanted.

## MLED-style package reconciliation

The package policy is explicit and manifest-based:

- Every `ADD` entry in the nine supplied tagfiles is included in
  `pkglists/packages-add`.
- Only the XFCE packages actually selected in the Minimal, Standard, or Custom
  checklist are protected. The selection is recorded in
  `pkglists/xfce-selected` and validated against `xfce/packages-official`.
- Packages listed in `pkglists/packages-keep` are also protected.
- Installed packages outside the combined keep set are written to
  `pkglists/packages-remove`.
- Tagfile entries marked `SKP` are not selected by the ADD set; if installed
  and not otherwise protected, they become removal candidates.

The XFCE profile is selected and installed first. Then the ADD packages are
installed and verified. The REMOVE list is processed only if every ADD
package is present. This preserves the MLED-inspired explicit-list approach
while making ADD-before-REMOVE ordering a safety condition.

The project does not perform automatic dependency resolution or dependency
auditing during trimming. It is not a general-purpose Slackware cleanup tool.

Generate manifests and a TSV audit report independently, after creating the
XFCE selection:

```sh
./scripts/mled-trim-audit.sh
```

Preview the generated package plan:

```sh
./scripts/kiss-trim.sh --dry-run
```

The orchestrator runs these stages automatically. Applying package changes
requires typing `APPLY` at the confirmation prompt.

### Important

The REMOVE list can contain many packages when run against a full Slackware
installation. Inspect the generated report and dry-run output carefully
before applying the trim. Keep any required local or third-party package by
adding its Slackware package base name, one per line, to
`pkglists/packages-keep`.

The project does not guarantee dependency completeness for arbitrary package
selections. Use the trim with the project's intended tagfile-based KISS
selection, review the proposed removals, and keep a recovery path available.

## Official XFCE installer

The XFCE installer uses the official Slackware64 15.0 XFCE package repository.
Run it independently as root if desired:

```sh
./xfce/install-xfce.sh
```

Available starting profiles:

- **Minimal** — core XFCE session components.
- **Standard** — Minimal plus common desktop essentials.
- **Custom** — all packages initially disabled.

Each profile is a starting preset. A `dialog` checklist lets the user review
and change the selection before installation. A final confirmation is
required before `upgradepkg --install-new` runs.

Select or preview a profile directly:

```sh
./xfce/install-xfce.sh --profile minimal --dry-run
./xfce/install-xfce.sh --profile standard --dry-run
./xfce/install-xfce.sh --profile custom --dry-run
```

The package inventory is stored in `xfce/packages-official`; profile
membership is stored in `xfce/profiles/minimal` and
`xfce/profiles/standard`. The installer synchronizes the official XFCE package
directory into this dedicated cache:

```text
/var/cache/slackware-kiss-desktop/xfce-15.0
```

`KISS_XFCE_REPO_URL` and `KISS_XFCE_CACHE` can override the repository source
and cache directory. The cache must be a dedicated directory whose final
component is `xfce-15.0`; symbolic links in the cache path are refused.

After installation, run `xwmconfig` as the desktop user and select XFCE for
`startx`. The installer does not automatically overwrite the user's X session
configuration.

## SlackBuilds.org manager

Run as root:

```sh
./sbo/sbo-manager.sh
```

This optional module requires `sbopkg` configured for Slackware 15.0. It
presents a `dialog` checklist with all options disabled initially. Only
selected packages are passed to `sbopkg`:

- `xarchiver`
- `thunar-archive-plugin`
- `xfce4-genmon-plugin`
- `xfce4-xkb-plugin`

## LXDM display manager

Run as root from an active local desktop session:

```sh
./display-manager/display-manager.sh
```

The module configures LXDM, detects the active local desktop user, installs
LXDM through `sbopkg` if not already installed, enables automatic login for
that user, integrates `pam_elogind.so`, updates `/etc/rc.d/rc.4`, and sets
Runlevel 4 as the default in `/etc/inittab`.

Configuration files are backed up before modification. To disable automatic
login, comment out the active `autologin=` line in `/etc/lxdm/lxdm.conf`. To
return to the console-oriented default, change `id:4:initdefault:` to
`id:3:initdefault:` in `/etc/inittab`.

## Project structure

```text
slackware-kiss-desktop/
├── desktop-kiss.sh
├── display-manager/
│   └── display-manager.sh
├── docs/
│   ├── KDE-CLEANUP-AUDIT-STATUS.md
│   ├── MLED-PORTING-POLICY-0.3.4.md
│   └── MLED-TRIM-IMPLEMENTATION-0.3.4.md
├── pkglists/
│   ├── packages-add
│   ├── packages-keep
│   ├── packages-remove
│   └── xfce-selected
├── sbo/
│   └── sbo-manager.sh
├── scripts/
│   ├── kiss-trim.sh
│   └── mled-trim-audit.sh
├── tagfiles/
│   ├── a/tagfile
│   ├── ap/tagfile
│   ├── d/tagfile
│   ├── k/tagfile
│   ├── l/tagfile
│   ├── n/tagfile
│   ├── t/tagfile
│   ├── x/tagfile
│   └── xap/tagfile
└── xfce/
    ├── DEPENDENCY-AUDIT.md
    ├── install-xfce.sh
    ├── packages-official
    └── profiles/
        ├── minimal
        └── standard
```

## Design principles

- Keep it simple (KISS).
- Use native Slackware tools wherever possible.
- Preserve the nine supplied tagfiles.
- Use explicit ADD, REMOVE, and KEEP manifests.
- Select and install the XFCE profile before package reconciliation.
- Install and verify all tagfile ADD packages before removing unselected
  packages.
- Protect only the selected XFCE packages, not the entire XFCE inventory.
- Keep optional software opt-in.
- Require explicit confirmation before package changes.
- Back up configuration files before modifying them.
- Do not silently modify the user's X session configuration.
- Do not perform automatic dependency resolution during package trimming.

## Requirements

- Slackware64 15.0
- `dialog`
- `rsync`
- `xz`
- `slackpkg` with a configured mirror (for package reconciliation)
- `sbopkg` configured for Slackware 15.0 (for SBo/LXDM stages)

The project targets Slackware64 15.0.
