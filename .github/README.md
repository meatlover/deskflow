> [!IMPORTANT]
> **This is an unofficial fork of [deskflow/deskflow](https://github.com/deskflow/deskflow).** It is not affiliated with the Deskflow project.
> It is upstream `master` ([`80221f0f7`](https://github.com/deskflow/deskflow/commit/80221f0f7b7ebd807fb1ffdf00906bbcfe561011)) plus a single fix:
> [**fix(mac): use a listen-only event tap while the cursor is on the server**](https://github.com/meatlover/deskflow/commit/ca982b4c114f497c0f172b8051c4089e93559a1c).
>
> **Problem.** On a macOS server, the cursor trails the hand by 1–2 s during fast motion, and often moves in jumps. Deskflow
> watches input through an *active* Quartz event tap (`kCGEventTapOptionDefault` at `kCGHIDEventTap`), and WindowServer waits for
> the tap to hand every event back. When WindowServer is busy, those waits add up faster than events arrive, so a backlog builds.
> Under that load WindowServer also merges queued moves, which is why the cursor jumps.
>
> **Evidence** (macOS 27.0.1, M3 Ultra, 125 Hz mouse):
> - A standalone active tap with the same settings, whose callback returns each event immediately, reproduces the lag.
>   WindowServer even disables it by timeout.
> - The same tap as listen-only does not lag.
> - A spindump of `deskflow-core` during the lag shows its tap thread blocked on WindowServer while handing events back
>   (`_CGSEventTapSendDataProcessedAck`), with no samples in Deskflow's own callback.
>
> **Fix.** While the cursor is on the server, Deskflow only reads input and never changes or drops it, so the active tap is not
> needed there. Input is read through a listen-only tap. The active tap, which swallows local input, is enabled only while the
> cursor is on a client. It is switched in `enter()`/`leave()` on the tap thread.
>
> **Notes**
> - Grant Deskflow **both** *Accessibility* and *Input Monitoring*. Without Input Monitoring, the server falls back to the old
>   single active tap and logs `no listen-only event tap, the cursor may lag`.
> - Only the macOS **server** role changes. The client role and the other platforms are identical to upstream.
> - There are no binaries; build from source. With Homebrew Qt 6.9 on the macOS 27 SDK, add
>   `-DWrapOpenGL_AGL="$(xcrun --show-sdk-path)/System/Library/Frameworks/OpenGL.framework"` to the CMake configure step.
>   A locally built app needs its own bundle ID if the official Deskflow is also installed, because macOS keys privacy grants
>   by bundle ID.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/deskflow/deskflow-artwork/blob/main/logo/deskflow-logo-dark-200px.png?raw=true">
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/deskflow/deskflow-artwork/blob/main/logo/deskflow-logo-light-200px.png?raw=true">
  <img alt="Deskflow" src="https://github.com/user-attachments/assets/f005b958-24df-4f4a-9bfd-4f834dae59d6">
</picture>

**Deskflow** is a free and open source keyboard and mouse sharing app.
Use the keyboard, mouse, or trackpad of one computer to control nearby computers,
and work seamlessly between them.
It's like a software KVM (but without the video).
TLS encryption is enabled by default. Wayland is supported. Clipboard sharing is supported.

> [!TIP]
>
> **Chat with us**
>
> - Main discussion on Matrix: [`#deskflow:matrix.org`](https://matrix.to/#/#deskflow:matrix.org) ([Matrix clients](https://matrix.org/ecosystem/clients/))
> - Discussion also happens on IRC: `#deskflow` or `#deskflow-dev` on [Libera Chat](https://libera.chat/)
> - Start a [new discussion](https://github.com/deskflow/deskflow/discussions) on our GitHub project.

## Download

[![Downloads: Stable Release](https://img.shields.io/github/downloads/deskflow/deskflow/latest/total?style=for-the-badge&logo=github&label=Download%20Stable)](https://github.com/deskflow/deskflow/releases/latest)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![Downloads: Continuous Build](https://img.shields.io/github/downloads/deskflow/deskflow/continuous/total?style=for-the-badge&logo=github&label=Download%20Continuous)](https://github.com/deskflow/deskflow/releases/continuous)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![Download From Flathub](https://img.shields.io/flathub/downloads/org.deskflow.deskflow?style=for-the-badge&logo=flathub&label=Download%20from%20flathub)](https://flathub.org/apps/org.deskflow.deskflow)

> [!NOTE]
> On Windows, you will need to install the
> [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#latest-microsoft-visual-c-redistributable-version).  
> Download latest: [`vc_redist.x64.exe`](https://aka.ms/vc14/vc_redist.x64.exe) [`vc_redist.arm64.exe`](https://aka.ms/vc14/vc_redist.arm64.exe)

> [!TIP]
> For macOS users, the easiest way to install and stay up to date is to use [Homebrew](https://brew.sh) with our [homebrew-tap](https://github.com/deskflow/homebrew-tap).
> macOS reports unsigned apps as damaged. This occurs because we do not use an Apple certificate for notarization. Clear the quarantine attribute to run the app: `xattr -c Deskflow.app`

To use Deskflow, download one of our [packages](https://github.com/deskflow/deskflow/releases), install `deskflow` (from your package repository), or [build it](https://github.com/deskflow/deskflow/wiki/Building) from source.

## Stats

[![GitHub commit activity](https://img.shields.io/github/commit-activity/m/deskflow/deskflow?logo=github)](https://github.com/deskflow/deskflow/commits/master/)
[![GitHub top language](https://img.shields.io/github/languages/top/deskflow/deskflow?logo=github)](https://github.com/deskflow/deskflow/commits/master/)
[![GitHub License](https://img.shields.io/github/license/deskflow/deskflow?logo=github)](LICENSE)
[![REUSE status](https://api.reuse.software/badge/github.com/deskflow/deskflow)](https://api.reuse.software/info/github.com/deskflow/deskflow)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=deskflow_deskflow&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=deskflow_deskflow)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=deskflow_deskflow&metric=coverage)](https://sonarcloud.io/summary/new_code?id=deskflow_deskflow)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=deskflow_deskflow&metric=code_smells)](https://sonarcloud.io/summary/new_code?id=deskflow_deskflow)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=deskflow_deskflow&metric=vulnerabilities)](https://sonarcloud.io/summary/new_code?id=deskflow_deskflow)

[![CI](https://github.com/deskflow/deskflow/actions/workflows/continuous-integration.yml/badge.svg)](https://github.com/deskflow/deskflow/actions/workflows/continuous-integration.yml)
[![CodeQL Analysis](https://github.com/deskflow/deskflow/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/deskflow/deskflow/actions/workflows/codeql-analysis.yml)
[![SonarCloud Analysis](https://github.com/deskflow/deskflow/actions/workflows/sonarcloud-analysis.yml/badge.svg)](https://github.com/deskflow/deskflow/actions/workflows/sonarcloud-analysis.yml)

## Contribute

[![Good first issues](https://img.shields.io/github/issues/deskflow/deskflow/good%20first%20issue?label=good%20first%20issues&color=%2344cc11)](https://github.com/deskflow/deskflow/labels/good%20first%20issue)

There are many ways to contribute to the Deskflow project.

We're a friendly, active, and welcoming community focused on building a great app.

Read our [Contributing](https://github.com/deskflow/deskflow/wiki/Contributing) page to get started.

For instructions on building Deskflow, use the wiki page: [Building](https://github.com/deskflow/deskflow/wiki/Building)

## Operating Systems

We support all major operating systems, including Windows, macOS, Linux, and Unix-like BSD-derived.

Windows 10 v1809 or higher is required.

macOS 13 or higher is required to use our CI builds for Apple Silicon machines. macOS 12 or higher is required for Intel macs or local builds.

Linux requires libei 1.3+ and libportal 0.8+ for the server/client. Additionally, Qt 6.7+ is required for the GUI.
Linux users with systems not meeting these requirements should use flatpak in place of a native package.

We officially support FreeBSD, and would also like to support: OpenBSD, NetBSD, DragonFly, Solaris.

## Repology

Repology monitors a huge number of package repositories and other sources comparing package
versions across them and gathering other information.

[![Repology](https://repology.org/badge/vertical-allrepos/deskflow.svg?columns=2&exclude_unsupported)](https://repology.org/project/deskflow/versions)

## Installing on macOS

When you install Deskflow on macOS, you need to allow accessibility access (Privacy & Security) to both the `Deskflow` app and the `deskflow` process.

If using Sequoia, you may also need to allow `Deskflow` under Local Network‍ settings (Privacy & Security).
When prompted by the OS, go to the settings and enable the access.

If you are upgrading and you already have `Deskflow` or `deskflow`
on the allowed list you will need to manually remove them before accessibility access can be granted to the new version.

macOS users who download directly from releases may need to run `xattr -c /Applications/Deskflow.app` after copying the app to the `Applications` dir.

It is recommended to install Deskflow using [Homebrew](https://brew.sh) from our [homebrew-tap](https://github.com/deskflow/homebrew-tap)

To add our tap, run:

```
brew tap deskflow/tap
```

Then install either:

- Stable: `brew install deskflow`
- Continuous: `brew install deskflow-dev`

## Similar Projects

In the open source developer community, similar projects collaborate for the improvement of all
mouse and keyboard sharing tools. We aim for idea sharing and interoperability.

- [**Lan Mouse**](https://github.com/feschber/lan-mouse) -
  Rust implementation with the goal of having native front-ends and interoperability with
  Deskflow/Synergy.
- [**Synergy**](https://symless.com/synergy) -
  Downstream commercial fork. Synergy sponsors Deskflow with financial support and contributes code ([learn more](https://github.com/deskflow/deskflow/wiki/Relationship-with-Synergy)).
- [**Input Leap**](https://github.com/input-leap/input-leap) -
  Inactive Deskflow/Synergy-derivative with the goal continuing Barrier development (now a dead fork).

## FAQ

### Is Deskflow compatible with Synergy, Input Leap, or Barrier?

Yes, Deskflow has network compatibility with all forks:

- Requires Deskflow >= v1.17.0.96
- Deskflow will _just work_ with Input Leap and Barrier (server or client).
- Connecting a Deskflow client to a Synergy 1 server will also _just work_.
- To connect a Synergy 1 client, you need to select the Synergy protocol in the Deskflow server settings.

_Note:_ Only Synergy 1 is compatible with Deskflow (Synergy 3 is not yet compatible).

### Is Deskflow compatible with Lan Mouse?

We would love to see compatibility with Lan Mouse. This may be quite an effort as currently the way they handle the generated input is very different.

### If I want to solve issues in Deskflow do I need to contribute to a fork?

We welcome PRs (pull requests) from the community. If you'd like to make a change, please feel
free to [start a discussion](https://github.com/deskflow/deskflow/discussions) or
[open a PR](https://github.com/deskflow/deskflow/wiki/Contributing).

### Is clipboard sharing supported?

Absolutely. The clipboard-sharing feature is a cornerstone feature of the product and we are
committed to maintaining and improving that feature.

### Is Wayland for Linux supported?

Yes! Wayland (the Linux display server protocol aimed to become the successor of the X Window
System) is an important platform for us.
The [`libei`](https://gitlab.freedesktop.org/libinput/libei) and
[`libportal`](https://github.com/flatpak/libportal) libraries enable
Wayland support for Deskflow. We would like to give special thanks to Peter Hutterer,
who is the author of `libei`, a major contributor to `libportal`, and the author of the Wayland
implementation in Deskflow. Others such as Olivier Fourdan and Povilas Kanapickas helped with the
Wayland implementation.

Some features _may_ be unavailable or broken on Wayland. Please see the [known Wayland issues](https://github.com/deskflow/deskflow/discussions/7499).

### Where did it all start?

Deskflow was first created as Synergy in 2001 by Chris Schoeneman.
Read about the [history of the project](https://github.com/deskflow/deskflow/wiki/History) on our
wiki.

## Meow'Dib (our mascot)

![Meow'Dib](https://github.com/user-attachments/assets/726f695c-3dfb-4abd-875d-ed658f6c610f)

## Deskflow Contributors

[![Sponsored by Synergy](https://raw.githubusercontent.com/deskflow/deskflow-artwork/b2c72a3e60a42dee793bd47efc275b5ee0bdaa5f/misc/synergy-sponsor.svg)](https://symless.com/synergy)

[Synergy](https://symless.com/synergy) sponsors the Deskflow project by contributing code and providing financial support ([learn more](https://github.com/deskflow/deskflow/wiki/Relationship-with-Synergy)).

Deskflow is made by possible by these contributors.

 <a href = "https://github.com/deskflow/deskflow/graphs/contributors">
   <img src = "https://contrib.rocks/image?repo=deskflow/deskflow"/>
 </a>

## License

This project is licensed under [GPL-2.0](LICENSE) with an [OpenSSL exception](../LICENSES/LicenseRef-OpenSSL-Exception.txt).
