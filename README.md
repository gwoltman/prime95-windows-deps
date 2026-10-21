# prime95-windows-deps

Third-party libraries for building [prime95](https://github.com/gwoltman/prime95)
on Windows that can't simply be downloaded from their own sites any more.
They are published as release files, which prime95's CI downloads and checks
against SHA-256 hashes.

| File | What it is |
|---|---|
| `libgmp-gw1-6.2.0-win64.zip` | GMP 6.2.0 for 64-bit Windows: `gmp.h`, `libgmp-gw1.dll`, `libgmp-gw1.lib` (import library for MSVC) |
| `libgmp-gw1-6.2.0-win32.zip` | The same for 32-bit Windows |
| `gmp-6.2.0.tar.bz2` | The GMP source those DLLs were built from, unmodified from <https://gmplib.org/download/gmp/> (its signature by Niels Möller verifies) |
| `curl-8.5.0-win64-mingw.zip` | The official [curl-for-win](https://github.com/curl/curl-for-win) build of cURL 8.5.0 for 64-bit Windows, unmodified |
| `curl-8.5.0-win32-mingw.zip` | The same for 32-bit Windows |

prime95 also uses hwloc 2.8.0 and Boost 1.76.0; those are still available
from <https://www.open-mpi.org/projects/hwloc/> and <https://archives.boost.io>.

## Licenses

GMP is distributed under the GNU LGPL v3 or the GNU GPL v2; each libgmp-gw1
zip includes GMP's `COPYING` files, and the matching source is in the same
release.  prime95 links it as a separate DLL that can be replaced.

cURL is distributed under the curl license (`COPYING.txt` in each cURL zip).
The libraries curl-for-win bundles into libcurl are listed in its
`BUILD-MANIFEST.txt`, with their licenses under `dep/`.

## How libgmp-gw1 was built

The DLL is GMP's normal `libgmp-10.dll`, renamed to something unique.  GMP's
DLL ordinals can change with each source change, so a prime95 that loaded some
other program's `libgmp-10.dll` could break ("DLL hell").

### Setup

Install MSYS2 from <https://www.msys2.org> (see
<http://repo.msys2.org/distrib/i686/> for the October 2016 installer), then:

```
pacman -Syuu            (exit and repeat)
pacman -S tar
pacman -S gcc           (not needed?)
pacman -S m4
pacman -S make
pacman -S diffutils
pacman -S libtool
pacman -S mingw-w64-i686-gcc
pacman -S mingw-w64-x86_64-gcc
```

Download the GMP sources (the `.bz2` file, not the `.lz` file) and close the
MSYS shell.

### 32-bit

In a mingw32 shell:

```
tar xvf gmp-6.2.0.tar.bz2
cd gmp-6.2.0
./configure --prefix=/home/gmp_dir --enable-fat --disable-static --enable-shared
make
make check
```

It may be best if the prefix directory contains no spaces, though
single-quoting worked.  Then from a Windows command prompt with the 32-bit
Visual Studio environment (see prime95's `gwnum\c.bat`):

```
cd .libs
rename libgmp-10.dll libgmp-gw1.dll
rename libgmp-3.dll.def libgmp-gw1.def
lib /def:libgmp-gw1.def /out:libgmp-gw1.lib
```

### 64-bit

On a 64-bit Windows machine, in a mingw64 shell:

```
tar xvf gmp-6.2.0.tar.bz2
cd gmp-6.2.0
./configure --prefix=/home/gmp_dir64 --enable-fat --disable-static --enable-shared
make
make check
```

Then from a Windows command prompt with the 64-bit Visual Studio environment
(see prime95's `gwnum\c64.bat`):

```
cd .libs
rename libgmp-10.dll libgmp-gw1.dll
rename libgmp-3.dll.def libgmp-gw1.def
lib /def:libgmp-gw1.def /out:libgmp-gw1.lib /MACHINE:X64
```

### GMP on other platforms

These aren't in this repository, but for reference:

macOS (static library):

```
tar xvf gmp sources
./configure --prefix=/usr/local --enable-fat --enable-static --disable-shared
make
make check
sudo make install
```

Linux (shared library):

```
tar xvf gmp sources
./configure --prefix=/usr/local --enable-fat --disable-static --enable-shared
make
make check
su
make install
nano /etc/ld.so.conf        (add /usr/local/lib)
ldconfig -v
```

### Alternative: MPIR (not currently used)

```
pacman -S mingw-w64-i686-python3
pacman -S mingw-w64-i686-yasm
download mpir sources
```

In a mingw32 shell:

```
./configure --prefix=/home/mpir_dir --enable-gmpcompat --enable-fat --disable-static --enable-shared
make
make check          (fails if there are spaces in the path name)
make install        (fails if there are spaces in the path name)
```

From a Windows command prompt (see `c.bat`):

```
cd .libs
rename libmpir-3.dll.def libgmp-23.def
lib /def:libgmp-23.def /out:libgmp-23.lib
```

For 64-bit:

```
pacman -S mingw-w64-x86_64-python3
pacman -S mingw-w64-x86_64-yasm
```

In a mingw64 shell, the same `configure`, `make`, `make check` and
`make install` with `--prefix=/home/mpir_dir64`; then from a Windows command
prompt (see `c64.bat`):

```
cd .libs
rename libmpir-3.dll.def libgmp-23.def
lib /def:libgmp-23.def /out:libgmp-23.lib /MACHINE:X64
```
