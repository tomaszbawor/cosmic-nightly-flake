# COSMIC Nightly Flake for NixOS

This repository provides a Nix flake that overlays the official COSMIC packages in `nixpkgs-unstable` with the latest nightly builds directly from the Pop!_OS GitHub repositories.

This flake is for users who want to test the cutting-edge, pre-release versions of the COSMIC desktop environment on their NixOS systems (what's stability
good for, anyway?).

## Disclaimer and Contributions

This is a personal side project. I make no guarantees that it will work correctly or that it will be consistently updated. The primary goal is to provide a convenient way to test the latest COSMIC builds, but things may break and I have limited time allocated to keep it working.

**Contributions are welcome!** If you notice a package is missing or would like to help maintain the flake (e.g., by adding CI to automate hash updates), please feel free to open a Pull Request.

## Prerequisites

Before you begin, ensure that you have flakes enabled in your NixOS configuration.

## How to Use

Getting started involves editing your NixOS flake configuration to use the nightly COSMIC source and then enabling the desktop environment.

### 1. Add the Input and Apply the Overlay

In your system's `flake.nix` (e.g., in `/etc/nixos/flake.nix`), you need to perform two actions: add this repository to your `inputs` and then apply its `overlay` to your Nixpkgs configuration.

```nix
# /etc/nixos/flake.nix
{
  description = "My NixOS Configuration";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    # Add the COSMIC nightly flake
    cosmic-nightly.url = "github:busyboredom/cosmic-nightly-flake";
  };

  outputs = { self, nixpkgs, cosmic-nightly, ... }@inputs: {
    nixosConfigurations.your-hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      specialArgs = { inherit inputs; };
      modules = [
        # Apply the overlay here
        ({
          nixpkgs.overlays = [ cosmic-nightly.overlays.default ];
        })

        ./configuration.nix
      ];
    };
  };
}
````

### 2\. Enable the COSMIC Desktop

In your `configuration.nix`, enable the COSMIC desktop environment and a display manager. The `cosmic-greeter` is recommended for the most integrated experience.

```nix
# /etc/nixos/configuration.nix
{ config, pkgs, ... }:

{
  # ... other system settings

  # Enable the COSMIC Desktop Environment
  services.desktopManager.cosmic.enable = true;

  # Enable a display manager. cosmic-greeter is recommended.
  services.displayManager.cosmic-greeter.enable = true;
}
```

### 3\. Rebuild Your System

After saving your changes, apply the new configuration:

```sh
sudo nixos-rebuild switch --flake /etc/nixos#your-hostname
```

## Updating

```sh
nix flake update /etc/nixos
```

Then, run the `nixos-rebuild switch` command again to apply the updates.
