Running the tests
-----------------

In order to run the tests, you will need to have the `grpcio`, `gevent`, and
`grpcio-tools` python packages installed. You can install them using
`pip`. It is recommended that you use a python virtualenv to do this.

```
$ virtualenv path/to/virtualenv # to create a virtualenv
$ . path/to/virtualenv/bin/activate # to use an existing virtualenv
$ pip install grpcio-tools gevent
$ pip install grpcio # Need to install grpcio-tools first to avoid a versioning problem
```

Building GRPC
-------------

In order to compile this project, and anything which depends on it, you will need a working installation
of the GRPC C core libraries. This library currently uses the 0.14 version range, so checkout an appropriate revision
of the repository, and install as follows:

```sh
git clone https://github.com/grpc/grpc.git
git checkout 2b223977c13975648bac2f422363e1ebf83506ce
cd grpc
git submodule update --init
make
sudo make install
```

Alternatively, using Nix, pass the following expression to `nix-build` and point Stack to the build products in the Nix store:

```nix
let pkgs = import <nixpkgs> {};
in  pkgs.stdenv.mkDerivation rec
    {   name = "grpc";
        src = pkgs.fetchgit
        { url    = "https://github.com/grpc/grpc.git";
          rev    = "2b223977c13975648bac2f422363e1ebf83506ce";
          sha256 = "0arxjdczgj6rbg14f6x24863mrz0xgpakmdfg54zp0xp7h2pghm6";
        };
        preInstall = "export prefix";
        buildInputs =
        [   pkgs.darwin.cctools
            pkgs.autoconf
            pkgs.automake
            pkgs.libtool
            pkgs.which
            pkgs.zlib

            pkgs.openssl
        ];
    }
```

Using the Library
-----------------

You must compile with `-threaded`, because we rely on being able to execute Haskell while blocking on foreign calls to the gRPC library. If not using code generation, the recommended place to start is in the `Network.GRPC.HighLevel.Server.Unregistered` module, where `serverLoop` provides a handler loop.