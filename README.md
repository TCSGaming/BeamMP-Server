# BeamMP-Server

[![CMake Windows Build](https://github.com/BeamMP/BeamMP-Server/workflows/CMake%20Windows%20Build/badge.svg?branch=master)](https://github.com/BeamMP/BeamMP-Server/actions?query=workflow%3A%22CMake+Windows+Build%22)
[![CMake Linux Build](https://github.com/BeamMP/BeamMP-Server/workflows/CMake%20Linux%20Build/badge.svg?branch=master)](https://github.com/BeamMP/BeamMP-Server/actions?query=workflow%3A%22CMake+Linux+Build%22)

This is the server for the multiplayer mod **[BeamMP](https://beammp.com/)** for the game [BeamNG.drive](https://www.beamng.com/).
The server is the point through which all clients communicate. You can write Lua mods for the server, there are detailed instructions on the [BeamMP Wiki](https://wiki.beammp.com).

**For Linux, you __need__ the runtime dependencies, which are listed below under [Runtime Dependencies](#runtime-dependencies)**

## Fork Changes: Client-Supplied Guest Names

This fork adds one opt-in feature on top of upstream `BeamMP-Server`: a way for guests to get a real display name locally, instead of a generic `GuestXXXX`, without depending on `auth.beammp.com` being reachable.

**Why:** normally, every connecting client — guest or forum account — gets named by the server calling out to BeamMP's central `auth.beammp.com/pkToUser` endpoint. If that backend is slow, down, or a guest simply never got a key from it, the server falls back to an anonymous guest ID. This fork adds a second, fully local path that doesn't depend on that backend at all.

**How it works:** a matching patched [BeamMP-Launcher](https://github.com/TCSGaming/BeamMP-Launcher) can send a specially-prefixed key (`GN:<name>`) instead of a real auth key. If this server has the feature enabled, it recognizes that prefix, skips the `auth.beammp.com` call entirely for that connection, sanitizes the supplied name (strips control characters and BeamMP's `^`-color formatting codes, trims whitespace, clamps to 24 characters), and uses it directly as the guest's display name.

**It's off by default and fully backward compatible.** A stock launcher never sends a `GN:`-prefixed key, so nothing changes for normal players unless you turn this on *and* they're using the matching patched launcher.

### Enabling it

In `ServerConfig.toml`, under `[General]`:
```toml
AllowGuests = true
AllowClientSuppliedGuestNames = true
```
Both must be `true` — the guest-name feature is gated behind guests being allowed at all.

### Security notes

- The supplied name is **not verified** against anything — there's no proof of Steam (or any) account ownership, only that the connecting launcher chose to send it. Fine for a small private/friends server; a technical player could send any string they want, same as if they'd typed any other guest name.
- The name is sanitized (control characters stripped, `^`-color codes stripped, 24-character cap) before being used anywhere, so it can't be used to impersonate staff formatting or inject garbage into chat/UI.
- Everything downstream (duplicate-name checks, roles, bans) works off the same `Client->GetName()` / `Client->IsGuest()` calls regardless of which path set them, so existing server-side Lua plugins don't need any changes.

### Companion repo

This only does anything paired with the matching [BeamMP-Launcher fork](https://github.com/TCSGaming/BeamMP-Launcher), which is what actually sends the `GN:` key — and only to servers explicitly allow-listed in that launcher's `Launcher.cfg`.

---

## Support + Contact

Feel free to ask any questions via the following channels:

- **Discord**: [click for invite](https://discord.gg/beammp)
- **BeamMP Forum**: [BeamMP Forum Support](https://forum.beammp.com/c/support/33)

## Minimum Requirements

These values are guesstimated and are subject to change with each release.

* RAM: 30-100 MiB usable (not counting OS overhead)
* CPU: >1GHz, preferably multicore
* OS: Windows, Linux (theoretically any POSIX)
* GPU: None
* HDD: 10 MiB + Mods/Plugins
* Bandwidth: 5-10 Mb/s upload

## Contributing

TLDR; [Issues](https://github.com/BeamMP/BeamMP-Server/issues) with the "help wanted" or "good first issue" label or with nobody assigned.

To contribute, look at the active [issues](https://github.com/BeamMP/BeamMP-Server/issues). Any issues that have the "help wanted" label or don't have anyone assigned are good tasks to take on. You can either contribute by programming or by testing and adding more info and ideas.

Fork this repository, make a new branch for your feature, implement your feature or fix, and then create a pull-request here. Even incomplete features and fixes can be pull-requested.

If you need support with understanding the codebase, please write us in the Discord. You'll need to be proficient in modern C++.

## About Building from Source

We only allow building unmodified (original) source code for public use. `master` is considered **unstable** and we will not provide technical support if such a build doesn't work, so always build from a tag. You can checkout a tag with `git checkout tags/TAGNAME`, where `TAGNAME` is the tag, for example `v3.4.1`. See [the tags](https://github.com/BeamMP/BeamMP-Server/tags) for possible versions/tags, as well as [the releases](https://github.com/BeamMP/BeamMP-Server/releases) to check which version is marked as a release/prerelease. We recommend using the [latest release](https://github.com/BeamMP/BeamMP-Server/releases/latest).

## Supported Operating Systems

The code itself supports (latest stable) Linux, Windows and FreeBSD. In terms of actual build support, for now we usually only distribute Windows binaries and Linux. For any other distro or OS, you just have to find the same libraries listed in [Runtime Dependencies](#runtime-dependencies) further down the page, and it should build fine.

Recommended compilers: MSVC, GCC, CLANG. 

You can find precompiled binaries under [Releases](https://github.com/BeamMP/BeamMP-Server/releases/).

## Build Instructions

On Linux, you need some dependencies to **build** the server (on Windows, you don't):

```
liblua5.3-dev curl zip unzip tar cmake make git g++
```

You can install these with your distribution's package manager. You will need sudo or need root for ONLY this step.

The names of each package may change depending on your platform.

If you are building for ARM (like aarch64), you need to run `export VCPKG_FORCE_SYSTEM_BINARIES=1` before the following commands.

You can build on **Windows, Linux** or other platforms by following these steps:

1. Check out the repository with git: `git clone --recursive https://github.com/BeamMP/BeamMP-Server`.
2. Go into the directory `cd BeamMP-Server`.
3. Specify the server version to build by checking out a tag: `git checkout tags/v3.4.1` - [Possible versions/tags](https://github.com/BeamMP/BeamMP-Server/tags)
4. Run CMake `cmake -S . -B bin -DCMAKE_BUILD_TYPE=Release` - this can take a few minutes and may take a lot of disk space and bandwidth.
5. Build via `cmake --build bin --parallel --config Release -t BeamMP-Server`.
6. Your executable can be found in `bin/`.

When you make changes to the code, you only have to run step 4 again.
### Building for FreeBSD
Building is only supported for major release branches of FreeBSD that are currently not EOL. The build process is the same as on Linux, although build dependencies can be universally installed from ports via pkg:
```
pkg install git cmake-core zip bash devel/ninja devel/pkgconf lua53
```
After installing the necessary build dependencies, follow the Linux build instructions beginning from step 3. Beware that running the initial cmake command will compile vcpkg from source, as vcpkg has no native FreeBSD port - this may take some time.

On systems with a single logical CPU core, `make` may fail to build the server when using the `--parallel` option when calling CMake. If you see error messages related to make, simply omit the `--parallel` from the command: `cmake --build bin --config Release -t BeamMP-Server`.

### Runtime Dependencies

These are needed to *run* the server.

Debian, Ubuntu and friends: `liblua5.3-0`

Other Linux distros: `liblua` of *some kind*.

Windows: No libraries.

## Support
The BeamMP project is supported by community donations via our [Patreon](https://www.patreon.com/BeamMP). This brings perks such as Patreon-only channels on our Discord, early access to new updates, and more server keys.
