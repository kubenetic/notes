# How to customize 3rd party Nix derivation 

So I had to migrate an ancient MSSQL database with an application based on JDK17. The legacy DB communicated only via TLSv1.0 protocol, and an updating it was not an option. I'm not a JAVA guru, and the `-Djdk.tls.disabledAlgorithms` and `-Djdk.tls.client.protocols` arguments didn't work for me; the application constantly complained about the obsolete TLS version.

I tried to change the settings in the `java.security` configuration file, but I use NixOS nowadays. Thus, a config change is not so simple anymore. You cannot just go into the `JAVA_HOME` directory and change what you want because it's a read-only filesystem now. I tried to apply some overlays on the original `jdk17` package but I ran into some other issues with `chmod` which was indicated by the `substituteInPlace` command. 

Finally I found the solution: I created my own custom derivation, which used the `jdk17` installation that I wanted to use. I copied all the files from the default installation of that package, modify what I needed (remove the `TLSv1` and `TLSv1.1` algorithms from the disabled ones) and copied all these files into the output directory of the new derivation. 

```nix
{ pkgs ? import <nixpkgs> {} }:

pkgs.stdenv.mkDerivation {
  name = "custom-jdk17";

  buildInputs = [ pkgs.jdk17 ];

  src = "${pkgs.jdk17}";

  buildPhase = ''
    buildDir=$PWD/custom-jdk17
    mkdir $buildDir
    cp -r $src/lib/openjdk/* $buildDir/

    chmod -R +w $buildDir/
    sed --in-place \
          's/jdk.tls.disabledAlgorithms=SSLv3, TLSv1, TLSv1.1, /jdk.tls.disabledAlgorithms=/g' \
          $buildDir/conf/security/java.security
  '';

  installPhase = ''
    mkdir $out
    cp -r $buildDir/* $out/
  '';
}
```

Then I created a Nix Shell configuration to use the custom JDK:
```nix
{
  pkgs ? import <nixpkgs> { }
}:

let
  custom-jdk17 = pkgs.callPackage ./default.nix {};
in
pkgs.mkShell {
  buildInputs = [
    custom-jdk17
  ];

  shellHook = ''
    ...
  '';

}
```
