# TinkerGame

> ### About this fork
>
> This is the [Plasma-Deckery](https://github.com/Plasma-Deckery) fork of
> [360900/tinkergame](https://github.com/360900/tinkergame). Everything below is
> the upstream README; this section records what was contributed from here.
>
> **All of it is merged upstream.** The fork holds nothing but this note, so
> there is no reason to prefer it over upstream — use whichever you like.

### What was contributed

The theme: Non-Steam games get their artwork without you having to think about
it, and without anything being matched to the wrong game behind your back.

**Artwork arrives by itself.** A systemd user unit watches `shortcuts.vdf` for
every Steam account on the machine and reacts when it changes.
`tinkergame artwork watch install` sets it up, `systemctl --user` switches it on
and off.

**Nothing is matched to the wrong game silently.** Artwork used to be looked up
by shortcut name alone, taking SteamGridDB's top hit unconditionally. That is a
guess, and a wrong guess is worse than a blank entry because it looks correct: a
shortcut called `Caustic` got the artwork of *Caustic Bloom*, and one called
`Eden` got a game literally named *eden\**. The automatic pass no longer makes
that call — it fetches entries you have decided on and leaves the rest alone.

**You decide once per entry, with the arrow keys.**
`tinkergame artwork resolve` lists the candidates SteamGridDB already returned —
that list was fetched and thrown away before — and you pick one with up/down and
Enter. Skip, *never look this up*, and *search under a different name* are rows
in the same list, so no stray keypress can mark an entry wrongly. The choice is
stored as a SteamGridDB game ID in `sgdbnames.conf` and you are never asked
again. Entries that already have all five artwork types are not offered at all,
and each prompt names the types that are missing.

**Far fewer requests.** Artwork already on disk is no longer requested again. A
refresh over a complete library costs a handful of file checks instead of
roughly six API calls per entry.

**Non-Steam entries get their icon in Steam.** The icon field in `shortcuts.vdf`
was never written, so Steam showed a generic icon whatever was downloaded.

**A missing API key says so.** `artwork resolve` used to exit silently with no
output when no SteamGridDB key was configured. It now names what is missing,
links the page the key comes from, takes it there, and checks it before storing.

**TinkerGame starts again after it has seen a machine without yad.** An empty
`YAD` in the global config was unrecoverable: the X11 wrapper was built around
it anyway, which hid the problem from every later check, and even
`tinkergame set YAD global ...` aborted before it could write.

**Editing a shortcut no longer destroys the Non-Steam library.**
`editSteamShortcutEntry` stripped `0a` from the file's hex text with no regard
for byte boundaries, so any byte pair `X0 AY` lost a byte and everything after
it shifted. Steam then logged `CSteamDoc::LoadShortcuts: failed to load shortcut
file` and discarded every Non-Steam game. Found the hard way, on a
thirteen-entry library.

### Merge status

| Change | Upstream | Merged |
|---|---|---|
| Set the icon field for Non-Steam artwork | [#9](https://github.com/360900/tinkergame/pull/9) | 2026-09-22 |
| Keep a per-call SteamGridDB override out of the global setting | [#11](https://github.com/360900/tinkergame/pull/11) | 2026-09-22 |
| Skip the request when the artwork is already there | [#14](https://github.com/360900/tinkergame/pull/14) | 2026-09-22 |
| Refresh Non-Steam artwork automatically via systemd units | [#15](https://github.com/360900/tinkergame/pull/15) | 2026-09-22 |
| Pick the SteamGridDB game interactively, never guess automatically | [#16](https://github.com/360900/tinkergame/pull/16) | 2026-09-22 |
| Never wrap a yad that cannot run | [#18](https://github.com/360900/tinkergame/pull/18) | 2026-09-22 |
| Stop corrupting `shortcuts.vdf`, keep the watcher out of boot | [#20](https://github.com/360900/tinkergame/pull/20) | 2026-09-28 |

### Known rough edge

The watcher can still fail in the first minute or two after logging in.
`graphical-session.target` goes active before a new client can open the display,
so TinkerGame's yad version probe reads back empty — and `checkIntDeps` treats
"could not determine" as "too old" and aborts the run, even though fetching
artwork needs no GUI at all. Later runs are unaffected. Not yet fixed upstream.

---

[![Wiki](https://img.shields.io/badge/docs-wiki-66c0f4?logo=github)](https://github.com/360900/tinkergame/wiki)

![TinkerGame main menu](docs/img/tinkergame-main-gui.png)

> **Warning!** This project is still in alpha! Expect bugs and major changes.

TinkerGame is a Linux game-launch wrapper for Steam. It gives each game a
small graphical control panel for Proton, Wine, Gamescope, MangoHud, mod
managers, launch commands, environment variables, and troubleshooting tools.

It supports Proton games, native Linux games, non-Steam games launched through
Steam, X11, Wayland, and Steam Deck game mode.

> TinkerGame is independent software. It is not affiliated with Valve or
> Steam. Use third-party tools and game modifications at your own risk.

## Documentation

The [TinkerGame wiki](https://github.com/360900/tinkergame/wiki) covers every
feature category with detailed pages, including the [Install
guide](https://github.com/360900/tinkergame/wiki/Installation), use as a
[Steam Compatibility Tool](https://github.com/360900/tinkergame/wiki/Steam-Compatibility-Tool),
and per-feature options. Every TinkerGame window can also open the matching
wiki page with the F1 key.

## What It Provides

- Per-game environment variables, launch commands, executables, and scripts.
- Proton and Wine selection, downloads, and prefix tools.
- Gamescope, MangoHud, GameMode, DXVK, VKD3D, FSR, and related options.
- Mod Organizer 2, Vortex, ReShade, Special K, Hedge Mod Manager, and other
  integrations.
- Native Linux game support through a Steam launch option.
- A YAD-based interface that works from the desktop and Steam Deck game mode.
- Command-line tools for installation, diagnostics, compatibility tools, and
  configuration.

## Install

Use your distribution package manager when a TinkerGame package is available.
ProtonUp-Qt and ProtonPlus support depends on their current release and package
metadata.

The quickest way from a source checkout is the one-command installer:

```sh
git clone https://github.com/360900/tinkergame.git
cd tinkergame
./install.sh
```

`./install.sh` asks whether to install for the current user only (recommended,
no root rights) or system-wide. It can also run non-interactively:

```sh
./install.sh --user     # install to ~/.local, no sudo
./install.sh --system   # install to /usr, uses sudo
```

The same with plain make:

```sh
sudo make install       # system-wide (/usr)
make install-user       # current user (~/.local)
```

The install requires Bash, Make, YAD, Git, Wget, Tar, Unzip, `jq`
(custom Proton and shader repository data), `xxd`, and the X11 tools
`xprop`, `xrandr`, and `xwininfo`. Optional integrations add their own
dependencies.

Debian/Ubuntu:

```sh
sudo apt-get install bash yad git wget tar unzip jq xdotool xxd x11-xserver-utils x11-utils
```

Arch:

```sh
sudo pacman -S bash git jq tar unzip wget xdotool xxd xorg-xprop xorg-xrandr xorg-xwininfo yad
```

(`xdotool` is only needed for a few optional legacy features.)

When installed for the current user, make sure `~/.local/bin` is in `PATH`.

For distribution packaging, `DESTDIR` stages files without changing the
runtime prefix embedded in the installed scripts:

```sh
make PREFIX=/usr DESTDIR="$PWD/pkg" install
```

An Arch Linux package recipe is provided at
[`packaging/arch/PKGBUILD`](packaging/arch/PKGBUILD).

To uninstall, run the uninstaller from this checkout - it scans the usual
install locations (user and system), removes every TinkerGame installation it
finds, and also removes the Steam compatibility-tool registration:

```sh
./uninstall.sh
```

The default keeps your settings, cache, downloaded tools, and game data. Add
`--purge` to remove those as well, and `--yes` to skip the confirmation
prompt. If system files are not writable by your user, rerun it with `sudo`.

Older releases also installed a `tinkergame-uninstall` command; current
releases no longer ship it. Use `./uninstall.sh` instead - it removes a
leftover helper from old installations as well.

## Use With Steam

### Proton games

Installing TinkerGame registers it as a Steam compatibility tool
automatically. If the registration was removed, or you installed without
running the registration step, re-register it:

```sh
tinkergame compat add
```

Select TinkerGame in the game's compatibility settings, or set it as the
default compatibility tool in Steam's Steam Play settings.

### Native Linux games

Set this as the game's launch option:

```text
tinkergame %command%
```

Use only one integration method per game. Do not select TinkerGame as a
compatibility tool and add it as a launch option at the same time.

### Command line

Run `tinkergame help` for the complete command list. Useful commands include:

```sh
tinkergame settings
tinkergame config dir
tinkergame version
tinkergame help
```

## Configuration

User configuration is stored under:

```text
${XDG_CONFIG_HOME:-$HOME/.config}/tinkergame
```

Logs are stored in the configuration directory and temporary startup logs are
stored under `/dev/shm/tinkergame`.

This is a full breaking rename. Existing users should read
[MIGRATION.md](MIGRATION.md) before installing.

## Troubleshooting

Start with the latest log from the configured `logs` directory and the startup
log under `/dev/shm/tinkergame`. Verify a source checkout with:

```sh
make build   # syntax-check the entry point and all lib/ modules
make check   # smoke checks
```

When reporting a problem, include the TinkerGame version, distribution,
desktop or game mode, display server, YAD version, game AppID, and relevant log
sections. Remove personal paths and tokens first.

## Credits

TinkerGame is a rename and continuation of
[SteamTinkerLaunch](https://github.com/sonic2kk/steamtinkerlaunch), the project
this repo was forked from, and builds on the work of its many contributors.
TinkerGame's parent project was created by
[`frostworx`](https://github.com/frostworx) and long maintained by
[`sonic2kk`](https://github.com/sonic2kk), with help from the broader
[SteamTinkerLaunch](https://github.com/sonic2kk/steamtinkerlaunch) community
over the years.

The complete history of the code remains available through this repository's
git log. This project is licensed under the GPLv3, as was its upstream
[SteamTinkerLaunch](https://github.com/sonic2kk/steamtinkerlaunch); see
[LICENSE](LICENSE).

## Development

TinkerGame is primarily Bash. The main script remains intentionally portable,
but new code should use arrays for external command arguments, quote paths, and
handle optional dependencies explicitly.

Run the local checks with:

```sh
make build          # syntax-check the entry point and all modules
make check          # smoke checks
./tests/run.sh unit # full unit test suite
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for development notes.

## License

TinkerGame is licensed under the GNU General Public License v3.0. See
[LICENSE](LICENSE).
