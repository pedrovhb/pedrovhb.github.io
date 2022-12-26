---
title: "Building JetBrains IDEs with Nix"
date: 2022-12-26T16:09:00-03:00
draft: false
---

This one is mostly a note for myself, and is quite possibly not the cleanest solution ever (still not confident in my Nix skills).


NixOS sometimes lags behind the latest versions of the JetBrains IDEs, so I managed to scrape together a file which can be used to build the latest versions of the IDEs.
I found the base for this file in a GitHub discussion about building JetBrains Gateway, which incidentally also works, if you substitute in the right links.
For PyCharm, I had to try building a few times and adding missing dependencies to `buildInputs` as I went along by using `nix-locate` to find missing shared objects.


```nix
with import <nixpkgs> {};
{
  my-pycharm = pkgs.stdenv.mkDerivation rec {
    pname = "jetbrains-pycharm";
    version = "2022.3";
  
    src = builtins.fetchurl {
      url = "https://download.jetbrains.com/python/pycharm-professional-2022.3.tar.gz";
    };
  
    nativeBuildInputs = [ pkgs.autoPatchelfHook pkgs.makeWrapper ];
    buildInputs = [
      pkgs.jetbrains.jdk
      pkgs.zlib
      pkgs.glib
      pkgs.glibc
      pkgs.libxcrypt
      pkgs.musl
      pkgs.xorg.libX11
      pkgs.xorg.libXi
      pkgs.xorg.libXrender
      pkgs.freetype
      pkgs.alsa-lib
      pkgs.xorg.libXtst
      pkgs.glibc
      pkgs.linux-pam
      pkgs.cups
    ];
  
    installPhase = ''
      mkdir -p $out/share
      cp -r . $out/share
      rm -r $out/share/jbr
  
      makeWrapper \
        $out/share/bin/pycharm.sh \
        $out/bin/jetbrains-pycharm \
        --prefix LD_LIBRARY_PATH : $out/lib \
        --set GATEWAY_JDK "${pkgs.jdk}" \
        --set JETBRAINSCLIENT_JDK "${pkgs.jetbrains.jdk.home}"
    '';
  };
  
}
```


Building is as simple as dropping the above into a file called `default.nix` and running `nix-build` in the same directory.
That'll build the thing and make a symlink to the built IDE path called `result` (the IDE itself is in the Nix store), so it can then be run with `./result/bin/jetbrains-pycharm` from the same directory. 

Enjoy!
