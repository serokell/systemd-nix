<!--
SPDX-FileCopyrightText: 2020 Serokell <https://serokell.io/>

SPDX-License-Identifier: MPL-2.0
-->

# `systemd-nix`

Generate systemd units from NixOS-style descriptions

## Descriptions of functions

Given a NixOS-style description of a systemd unit, like

```nix
{
  description = "A daemon to upload nix store paths to a remote store asynchronously";
  wantedBy = [ "default.target" ];
  path = with pkgs; [ nix upload-daemon ];
  script =
    ''upload-daemon \
    --target "ssh://some-remote-server" \
    --unix "/tmp/upload-daemon.sock" \
    -j $(nproc) \
    +RTS -N$(nproc)'';
  serviceConfig.Restart = "always";
}
```

### `mkService`

Will generate a systemd service description, like

```conf
[Unit]
Description=A daemon to upload nix store paths to a remote store asynchronously

[Service]
Environment="LOCALE_ARCHIVE=/nix/store/<...>/lib/locale/locale-archive"
Environment="PATH=<...>"
Environment="TZDIR=<...>/share/zoneinfo"

ExecStart=/nix/store/<...>-unit-script-upload-daemon-start/bin/upload-daemon-start
Restart=always
```

### `mkUserService`

Will generate an activation script that \[re\]installs and \[re\]starts
the service generate as described above.

## Usage example

```nix
{
  inputs = {
    systemd-nix = {
      url = github:serokell/systemd-nix;
      inputs.nixpkgs.follows =
        "nixpkgs"; # Make sure the nixpkgs version matches
    };
    deploy.url = github:serokell/deploy;
  };

  outputs = { self, nixpkgs, systemd-nix, deploy }:
    {
      # `nix run` will deploy
      inherit (deploy) defaultApp;
      deploy.nodes.example = {
        hostname = "localhost";
        profiles.hello = {
          path = systemd-nix.lib.x86_64-linux.mkUserService "hello" {
            description = "Produce a greeting and exit";
            path = [ nixpkgs.legacyPackages.x86_64-linux.hello ];
            serviceConfig = {
              Type = "oneshot";
              ExecStart = "hello";
            };
          };
          # Just to test that it's working
          activate = "$PROFILE/bin/activate";
        };
      };
    };
}
```

## License

systemd-nix is licensed under the Mozilla Public License Version 2.0.
You can read it in [./LICENSE](LICENSE).

## About Serokell

systemd-nix is maintained and funded with ❤️ by
[Serokell](https://serokell.io/). The names and logo for Serokell are
trademark of Serokell OÜ.

We love open source software! See [our other
projects](https://serokell.io/community?utm_source=github) or [hire
us](https://serokell.io/hire-us?utm_source=github) to design, develop
and grow your idea!
