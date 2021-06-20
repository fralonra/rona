---
title: "Build Skia on Archlinux"
date: 2021-06-20T22:18:36+08:00
author: "Fralonra"
cover: ""
tags: ["skia"]
keywords: ["skia", "build steps"]
description: "Build Skia on Archlinux"
showFullContent: false
---

### 1. Install depot-tools

`depot-tools` includes `gclient`, `git-cl`, and `Ninja`. They are essential to build `Skia`.

It's quite easy to install `depot-tools`, for we have `depot-tools` in AUR, and even its binary version in `archlinuxcn`'s [repo](https://github.com/archlinuxcn/repo)!

```bash
yay depot-tools
```

But notice that `depot-tools` is installed in `/opt/depot_tools/`. This directory is not in `$PATH` by default, so make sure to `export PATH=/opt/depot_tools:$PATH` before using it. Though in this way we should use it as root.

Another option is to install each components of `depot-tools` directly from the repo.

### 2. Fetch source code of Skia

```bash
git clone https://skia.googlesource.com/skia
```

### 3. Build Steps

Follow the official [guide](https://skia.org/docs/user/build).

TLDR:

```bash
python2 tools/git-sync-deps
# after the above command you would get bin/gn

# generate build files
bin/gn gen out/Static --args='is_official_build=true'

# run Ninja to compile and link Skia
ninja -C out/Static
```
