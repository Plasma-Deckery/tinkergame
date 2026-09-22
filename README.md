# TinkerGame

> ### About this fork
>
> This is the [Plasma-Deckery](https://github.com/Plasma-Deckery) fork of
> [360900/tinkergame](https://github.com/360900/tinkergame). Everything below is
> the upstream README; this section lists what was contributed from here and
> where each change stands.
>
> The theme of the work so far: **Non-Steam games get their artwork without you
> having to think about it, and without anything being matched to the wrong
> game behind your back.**

### What is better than before

**Your Non-Steam games get their artwork by themselves.** Adding a shortcut in
Steam used to mean remembering to run an artwork update afterwards. A systemd
user unit now watches `shortcuts.vdf` for every Steam account on the machine and
reacts when it changes. `tinkergame artwork watch install` sets it up,
`systemctl --user` switches it on and off.

**Nothing is matched to the wrong game silently.** Artwork used to be looked up
by shortcut name alone, taking SteamGridDB's top hit unconditionally. That is a
guess, and a wrong guess is worse than a blank entry because it looks correct:
a shortcut called `Caustic` got the artwork of *Caustic Bloom*, and one called
`Eden` got a game literally named *eden\**. The automatic pass no longer makes
that call. It fetches entries you have decided on and leaves the rest alone.

**You decide once per entry, with the arrow keys.** `tinkergame artwork resolve`
lists the candidates SteamGridDB already returned — that list was fetched and
thrown away before — and you pick one with up/down and Enter. Skip, *never look
this up*, and *search under a different name* are rows in the same list, so no
letter keys and no stray keypress can mark an entry wrongly. The choice is
stored as a SteamGridDB game ID in `sgdbnames.conf`, and you are never asked
about that entry again. Entries that already have all five artwork types are
not offered at all.

**It only asks about what is actually missing, and says what that is.** Each
prompt shows which of the five types are on disk, so `artwork missing: icon` is
one keypress away from fixed instead of a guess.

**Much less waiting and far fewer requests.** Artwork already on disk is no
longer requested from SteamGridDB again. A refresh over a library that is
already complete now costs a handful of file checks instead of roughly six API
calls per entry.

**Non-Steam entries get their icon in Steam.** The icon field in `shortcuts.vdf`
was never written, so Steam showed a generic icon no matter what was
downloaded.

**A missing API key tells you so.** `artwork resolve` used to exit silently with
no output at all when no SteamGridDB key was configured. It now says what is
missing, links the page the key comes from, takes it right there, and checks it
before storing it.

**TinkerGame starts again after it has seen a machine without yad.** An empty
`YAD` in the global config used to be unrecoverable: the X11 wrapper was built
around it anyway, which hid the problem from every later check, and even
`tinkergame set YAD global ...` aborted before it could write.

**Your Non-Steam library stops disappearing.** Editing a shortcut entry could
corrupt `shortcuts.vdf` so badly that Steam discarded every Non-Steam game in
it. See the table below.

### Merge status

| Change | Upstream | Status |
|---|---|---|
| Set the icon field for Non-Steam artwork | [#9](https://github.com/360900/tinkergame/pull/9) | Merged |
| Keep a per-call SteamGridDB override out of the global setting | [#11](https://github.com/360900/tinkergame/pull/11) | Merged |
| Skip the request when the artwork is already there | [#14](https://github.com/360900/tinkergame/pull/14) | Merged |
| Refresh Non-Steam artwork automatically via systemd units | [#15](https://github.com/360900/tinkergame/pull/15) | Merged |
| Pick the SteamGridDB game interactively, never guess automatically | [#16](https://github.com/360900/tinkergame/pull/16) | Merged |
| Never wrap a yad that cannot run | [#18](https://github.com/360900/tinkergame/pull/18) | Merged |
| **Stop corrupting `shortcuts.vdf` when editing an entry** | [#20](https://github.com/360900/tinkergame/pull/20) | **Open — see below** |

### If you are running upstream right now, read this

[#20](https://github.com/360900/tinkergame/pull/20) is not merged yet, and
[#9](https://github.com/360900/tinkergame/pull/9) is. That combination matters.

`editSteamShortcutEntry` converts `shortcuts.vdf` to a hex string and used to
strip the characters `0a` from it before writing it back — as text, with no
regard for byte boundaries. Any byte pair `X0 AY` lost a byte and everything
after it shifted. Steam then logs

```
CSteamDoc::LoadShortcuts: failed to load shortcut file
```

and discards every Non-Steam game in the file, deleting it on the next start.
Because the file vanishes later, the loss looks like it came from Steam.

#9 is what makes the icon field actually get written, so on current upstream
`main` a routine artwork refresh reaches that code every time. This was found
the hard way: a thirteen-entry Non-Steam library, gone.

Until #20 is merged upstream, use this fork, or keep a copy of your
`shortcuts.vdf` somewhere Steam does not write.

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

To uninstall, run the uninstaller - it scans the usual install locations
(user and system), removes every TinkerGame installation it finds, and also
removes the Steam compatibility-tool registration:

```sh
tinkergame-uninstall
# or, for a system installation:
sudo tinkergame-uninstall
```

The default keeps your settings, cache, downloaded tools, and game data. Add
`--purge` to remove those as well, and `--yes` to skip the confirmation
prompt. If the uninstaller lives outside your `PATH`, run it from the
repository instead:

```sh
bash uninstall.sh --purge
```

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
