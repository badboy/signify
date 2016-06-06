# Signify - Sign and Verify

[![Build Status](https://travis-ci.org/aperezdc/signify.svg?branch=master)](https://travis-ci.org/aperezdc/signify)

OpenBSD tool to signs and verify signatures on files. This is a portable
version which uses [libbsd](http://libbsd.freedesktop.org/wiki/) (version
0.7 or newer is required).

See http://www.tedunangst.com/flak/post/signify for more information.

## Build options

The following options can be passed to Make:

* `VERIFY_ONLY=1`

    Build only the verification code. Support for signing will not
    be available in the built `signify` binary. **Note that this is
    unsupported and compilation may not succeed.**

* `BOUNDS_CHECKING=1`

    Enables bounds-checking using `__attribute__((bounded))`. Your
    compiler must have support for this. Clang 3.4 is known to work.

* `BUNDLED_LIBBSD=1`

    Downloads and builds [libbsd](http://libbsd.freedesktop.org/wiki/),
    and links it statically into Signify. This can be used when the version
    installed in the system is an unsupported version, or when installing it
    in the system is not desirable.

    Enabling this options needs `wget`, `xz`, `tar`, and `gpg` installed in
    the build system. To avoid downloading files while building, there are
    two options:

      - `make BUNDLED_LIBBSD=1 libbsd-print-urls` will print the URLs
        of the files which need to be downloaded, one per line.
      - `make BUNDLED_LIBBSD=1 libbsd-download` will download the needed
        files.

    Pre-downloading the files will avoid hitting the network during build, but
    will still check the PGP signature of the `libbsd` tarball.

* `BUNDLED_LIBBSD_VERIFY_GPG=0`

    Disables checking the PGP signature of the bundled
    [libbsd](http://libbsd.freedesktop.org/wiki/) tarball. This avoids the
    need for GnuPG to be installed for building.

* `BUNDLED_LIBBSD_VERIFY_GPG=/path/to/gpg`

    Set the path to the `gpg` binary used for checking signatures. If not
    specified, the program is searched in the `$PATH`.

* `MUSL=1`

    Enable linking against the [Musl libc](http://www.musl-libc.org/). At the
    moment this needs a patched `libbsd`, so enabling this option will
    automatically set `BUNDLED_LIBBSD=1` and patch the locally-built version.

* `LTO=1`

    Perform Link-Time Optimizations. Both your compiler *and* linker
    must have support for this. Recent binutils and GCC/Clang are
    known to work.

* `PLEDGE=…`

    Choose among one of the alternative implementations of the
    [pledge()](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man2/pledge.2)
    system call. For the moment the only supported values is:

      - `noop` *(default)*: Uses an implementation which does nothing
      - `waive` *(Linux-only)*: Uses
        [libwaive](https://github.com/dimkr/libwaive), which itself uses
        [seccomp filters](https://en.wikipedia.org/wiki/Seccomp).

    To use your own implementation, use an empty value, and pass
    the needed flags for linking its code. For example:
    `make PLEDGE='' EXTRA_LDFLAGS=my-pledge.o`.

* `EXTRA_CFLAGS=…`, `EXTRA_LDFLAGS=…`

    Additional flags to be passed to the compiler and the linker,
    repectively.

For example, you can build a size-optimized version with:

    make EXTRA_CFLAGS='-Os -s' LTO=1


## Troubleshooting

* **Problem:** Undefined references to `clock_gettime`. <br>
  **Solution:** Your system has an old `glibc` version, you need to pass
  `LDLIBS=-lrt` to `make`.


## Other implementations

* [asignify](https://github.com/vstakhov/asignify) can read signatures
  generated by Signify (generating them is not yet implemented), and can be
  used as a library.
