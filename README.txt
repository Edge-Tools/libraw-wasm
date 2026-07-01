libraw-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of libraw (license: LGPL-2.1-only) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  libraw.tar.gz
    https://www.libraw.org/data/LibRaw-0.21.4.tar.gz
    sha256 6be43f19397e43214ff56aab056bf3ff4925ca14012ce5a1538a172406a09e63
