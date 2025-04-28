# PCAP Analysis
Analyse PCAP files to identify deployment artifact types in a data center network. Main programming language is Python, virtual environments rely on [automatic shell activation](https://devenv.sh/automatic-shell-activation/) using devenv.sh and direnv.

## Initial Setup

It is recommended to prepare a python repository on github and clone the rope into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Add the python environment parameter to `devenv.nix`

```nix
  # Python environment
  languages.python = {
    enable = true;
    # https://devenv.sh/languages/python/
    # packages = [ pkgs.python39Packages.pip ];
    venv.enable = true;
    venv.requirements = ''
      requests
      # torch
    '';
    # venv.packages = [ pkgs.python39Packages.pip ];
    # venv.packages = [ pkgs.python39Packages.pip ];
    uv.enable = true;
  };
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
```

We add the dotfiles for the virtual environment to `.gitignore`

```sh
echo ".envrc" >> .gitignore
```

## Start analyzing

Before loading data, we initilaise the virtual environment

```sh
uv init && uv venv
source .devenv/state/venv/bin/activate
```



## Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
