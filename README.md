# krops (krebs ops)

krops is a lightweigt toolkit to deploy nixos systems, remotely or locally.

fancy features include:
- store your secrets in passwordstore
- build your system remotely
- minimal overhead
- run from custom nixpkgs branch/checkout/fork

minimal example:

create a krops.nix somewhere
```
let
  #krops = ./.;
  krops = (import <nixpkgs> {}).fetchgit {
    url = https://cgit.krebsco.de/krops/;
    rev = "3022582ade8049e6ccf18f358cedb996d6716945";
    sha256 = "0k3zhv2830z4bljcdvf6ciwjihk2zzcn9y23p49c6sba5hbsd6jb";
  };

  lib = import "${krops}/lib";
  pkgs = import "${krops}/pkgs" {};

  source = lib.evalSource [{
    nixpkgs.git = {
      ref = "4b4bbce199d3b3a8001ee93495604289b01aaad3";
      url = https://github.com/NixOS/nixpkgs;

    };
    nixos-config.file = toString (pkgs.writeText "nixos-config" ''
      { pkgs, ... }: {

        fileSystems."/" = { device = "/dev/sda1"; };
        boot.loader.systemd-boot.enable = true;
        services.openssh.enable = true;
        environment.systemPackages = [ pkgs.git ];
      }
    '');
  }];
in
  pkgs.krops.writeDeploy "deploy" {
    source = source;
    target = "root@192.168.56.101";
  }
```

and run `$(nix-build krops.nix)`. This results in a script which deploys the machine via ssh & rsync on the target machine.