# nix-index-database

Weekly updated [nix-index](https://github.com/bennofs/nix-index) database

This repository also provides nixos modules and home-manager modules that add a
`nix-index` wrapper to use the database from this repository.

The home-manager module also allows integration with the existing `command-not-found`
functionality.

## Demo

``` shell
$ nix run github:nix-community/nix-index-database bin/cntr
cntr.out                                        978,736 x /nix/store/09p2hys5bxcnzcaad3bknlnwsgdkznl1-cntr-1.5.1/bin/cntr
```

## Usage in NixOS

Include the nixos module in your configuration (requires 23.05 or nixos unstable)

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable-small";
  
    nix-index-database.url = "github:nix-community/nix-index-database";
    nix-index-database.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = { self, nixpkgs, nix-index-database, ... }: {
    nixosConfigurations = {
      my-nixos = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          ./configuration.nix
          nix-index-database.nixosModules.nix-index
          # optional to also wrap and install comma
          # { programs.nix-index-database.comma.enable = true; }
        ];
      };
    };
  };
}
```


## Usage in Home-manager

1. Follow the [manual](https://github.com/nix-community/home-manager/blob/master/docs/nix-flakes.adoc) to set up home-manager with flakes.
2. Include the home-manager module in your configuration:

```nix
{
  inputs = {
    # Specify the source of Home Manager and Nixpkgs.
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
    
    nix-index-database.url = "github:nix-community/nix-index-database";
    nix-index-database.inputs.nixpkgs.follows = "nixpkgs";
  };
  outputs = { nixpkgs, home-manager, nix-index-database, ... }:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations.jdoe = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;

        modules = [
          nix-index-database.hmModules.nix-index
          # optional to also wrap and install comma
          # { programs.nix-index-database.comma.enable = true; }
        ];
      };
    };

}
```

Additionally, if your shell is managed by home-manager, you can have `nix-index`
integrate with your shell's `command-not-found` functionality by
setting `programs.nix-index.enable = true`.


## Ad-hoc download

```shell
download_nixpkgs_cache_index () {
  filename="index-$(uname -m | sed 's/^arm64$/aarch64/')-$(uname | tr A-Z a-z)"
  mkdir -p ~/.cache/nix-index && cd ~/.cache/nix-index
  # -N will only download a new version if there is an update.
  wget -q -N https://github.com/nix-community/nix-index-database/releases/latest/download/$filename
  ln -f $filename files
}

download_nixpkgs_cache_index
```
