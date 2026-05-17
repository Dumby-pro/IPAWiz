# IPAWiz

A small Linux CLI for building iPhone `.ipa` files. Wraps [Theos] so you don't
have to remember the Makefile dance.

## Install

```bash
chmod +x ipawiz
mv ipawiz ~/.local/bin/
```

Make sure `~/.local/bin` is on your `$PATH`. If it isn't, add this to your
`~/.bashrc` or `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Then run the one-time setup:

```bash
ipawiz --setup         # installs Theos, iOS SDK, and build deps
ipawiz --setup --sign  # ...and also builds zsign for desktop-side signing
```

`--setup` will:

1. Install build deps via apt/dnf/pacman (build tools, `ldid`, `cmake`, …)
2. Clone Theos to `~/theos`
3. Pull iOS SDKs from `theos/sdks` into `~/theos/sdks` (~600 MB)
4. Add `export THEOS=~/theos` to your `~/.bashrc` / `~/.zshrc`
5. (with `--sign`) build `zsign` from source and drop it in `~/.local/bin`

## Usage

```bash
ipawiz MyApp.app                     # package an existing .app -> MyApp.ipa
ipawiz ./build                       # find a .app inside ./build, package it
ipawiz --web ./mysite                # wrap a web folder into mysite.ipa
ipawiz --web ./mysite -n "Cool App"  # custom display name
ipawiz --web ./site -b com.you.site  # custom bundle id
ipawiz --web ./site -o out/foo.ipa   # custom output path
ipawiz -h                            # full help
```

### Folder mode

`ipawiz FOLDER` expects either:

- a directory ending in `.app` (a real iOS app bundle, with `Info.plist` and
  the executable inside), or
- a directory containing exactly that.

It does **not** invent an `Info.plist` for raw files — for that, use `--web`.

### Web wrapper mode

`--web FOLDER` generates a tiny Obj-C app whose root view is a `WKWebView`
that loads `FOLDER/index.html` from the app bundle. Relative links to CSS, JS,
and other HTML files in the same folder Just Work. Network requests work too.

If `index.html` is missing you'll see a placeholder page when the IPA runs.

### Signing

The IPA is unsigned by default. Two common ways to install it:

- **Sideload it.** SideStore, AltStore, Feather, and TrollStore all sign IPAs
  on-device, so unsigned is fine.
- **Sign it yourself** with a `.p12` cert + `.mobileprovision`:

  ```bash
  ipawiz MyApp.app \
      --sign-p12 cert.p12 \
      --sign-pp  dev.mobileprovision \
      --sign-password "..."          # optional
  ```

  This runs `zsign` against the `.app` before packaging. Run
  `ipawiz --setup --sign` once to install zsign.

## Troubleshooting

- **`Theos not found`** — open a new terminal after `--setup` so `$THEOS` is
  exported, or run `source ~/.bashrc`.
- **`ldid` couldn't be installed via apt** — IPAWiz falls back to building it
  from source (ProcursusTeam/ldid). Needs `cmake` + `libssl-dev`, both of
  which `--setup` installs.
- **Build fails with missing SDK** — re-run `ipawiz --setup`; the SDK clone
  may have been interrupted. Check `ls ~/theos/sdks/`.

## Credits

- **Pierce** — concept, design, and direction. All the "this should exist on
  Linux" energy that produced the tool.
- **[Theos]** — the actual cross-platform iOS build system doing the heavy
  lifting. None of this works without it. Maintained by the Theos team and
  a long list of contributors.
- **[ProcursusTeam/ldid]** — the `ldid` fork used for pseudo-signing during
  the build.
- **[zsign]** by zhlynn — used for optional desktop-side signing with a `.p12`
  + `.mobileprovision`.
- **Claude (Anthropic)** — wrote the Python glue around the above based on
  Pierce's spec.

## License

Do whatever you want with it. The tools it wraps (Theos, ldid, zsign) keep
their own licenses.

[Theos]: https://theos.dev
[ProcursusTeam/ldid]: https://github.com/ProcursusTeam/ldid
[zsign]: https://github.com/zhlynn/zsign
