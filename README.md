# luducat-innoextract

luducat-innoextract unpacks installers made with [Inno Setup](https://jrsoftware.org/isinfo.php) without running them, so you do not need Windows or Wine for it. It reads installers from Inno Setup 1.2.10 up to 7.1, including GOG.com's game installers with their multi-part and GOG Galaxy layouts.

This is a fork of innoextract, maintained by the [luducat](https://luducat.org) project. luducat uses it to unpack the DRM-free game installers you own, so it gets the fixes those installers need. The tool works the same way as upstream innoextract and is useful on its own.

## What this fork changes

Compared with crazy-max/innoextract 1.13.0:

* **Local file times.** Inno Setup stores file times as the local time of the machine that built the installer. With `--timestamps local`, extracted files now show exactly that time. Before, every date was read as standard time, so files dated in summer showed one hour later than stored.
* **Inno Setup 6.6.0 development builds.** Some installers were built with a pre-release of Inno Setup 6.6.0 from late September to early October 2025. They carry the same data ID as the release but store their language entries in the older layout, and innoextract stopped with a stream error on them. They now unpack. Development builds from other weeks used other layouts again; those will be added when installers made with them turn up.
* **Unit tests** for the setup header, language entries and local time conversion. Run them with `-DBUILD_TESTS=ON` and `make check`.
* **Version and bug reports.** Releases carry an `-lc` suffix (`1.13.0-lc1`), and bug reports go to [this repository](https://github.com/luducat/luducat-innoextract/issues), not to upstream.

The full list is in [CHANGELOG](CHANGELOG). Fixes that help every user are offered back to crazy-max/innoextract.

## Where it comes from

* **[dscharrer/innoextract](https://github.com/dscharrer/innoextract)** by Daniel Scharrer, the original project, written from 2011 on. Its last release is 1.9 (2020), and its last change dates from February 2025.
* **Johannes Schindelin** added support for Inno Setup 6.4.2 through 7.0.2 and the 64-bit data layouts (upstream pull request #210).
* **[crazy-max/innoextract](https://github.com/crazy-max/innoextract)** by CrazyMax gathered that work and several open upstream pull requests into releases 1.10 to 1.13, with CI, release binaries and end-to-end tests up to Inno Setup 7.1.0. Contributions there came from Stephen Kitt, Patryk Miś, XIE Zhibang and others.
* **luducat-innoextract** builds on crazy-max/innoextract 1.13.0.

The git history keeps every author's commits.

## Building

### Dependencies

* **[Boost](https://www.boost.org/) 1.37** or newer
* **liblzma** from [xz-utils](https://tukaani.org/xz/) *(optional)*
* **iconv** (*optional*, either as part of the system libc, as is the case with [glibc](https://www.gnu.org/software/libc/) and [uClibc](https://uclibc.org/), or as a separate [libiconv](https://www.gnu.org/software/libiconv/))

For Boost you will need the headers as well as the `iostreams`, `filesystem`, `date_time` and `program_options` libraries. Older Boost versions may work but are not actively supported. The boost `iostreams` library needs to be built with zlib and bzip2 support.

While innoextract can be built without liblzma by manually setting `-DUSE_LZMA=OFF`, it is highly recommended and you won't be able to extract most installers created by newer Inno Setup versions without it.

To build innoextract you will also need **[CMake](https://cmake.org/) 2.8** and a working C++ compiler, as well as the development headers for liblzma and boost.

See the Website for [operating system-specific instructions](https://constexpr.org/innoextract/install).

### Compile and install

To compile innoextract, run:

    $ mkdir -p build && cd build
    $ cmake ..
    $ make

To install the binaries system-wide, run as root:

    # make install

The default build settings are tuned for users - if you plan to make changes to innoextract you should append the `-DDEVELOPER=1` option to the `cmake` command to enable debug output and fast incremental builds.

### Build options

| Option                    | Default   | Description                                                                                                                                                                                     |
|---------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `BUILD_DECRYPTION`        | `ON`      | Build decryption support.                                                                                                                                                                       |
| `USE_LZMA`                | `ON`      | Use `liblzma`.                                                                                                                                                                                  |
| `WITH_CONV`               | *not set* | The charset conversion library to use. Valid values are `iconv`, `win32` and `builtin`¹. If not set, a library appropriate for the target platform will be chosen.                              |
| `CMAKE_BUILD_TYPE`        | `Release` | Set to `Debug` to enable debug output.                                                                                                                                                          |
| `DEBUG`                   | `OFF`²    | Enable debug output and runtime checks.                                                                                                                                                         |
| `DEBUG_EXTRA`             | `OFF`     | Expensive debug options.                                                                                                                                                                        |
| `SET_WARNING_FLAGS`       | `ON`      | Adjust compiler warning flags. This should not affect the produced binaries but is useful to catch potential problems.                                                                          |
| `SET_NOISY_WARNING_FLAGS` | `OFF`     | Enable warnings with false positives many cases that still need to be fixed.                                                                                                                    |
| `SET_OPTIMIZATION_FLAGS`  | `ON`      | Adjust compiler optimization flags.                                                                                                                                                             |
| `CXX_STD_VERSION`         | `2017`    | Maximum C++ standard version to enable.                                                                                                                                                         |
| `USE_DYNAMIC_UTIMENSAT`   | `OFF`     | Dynamically load utimensat(2) if not available at compile time.                                                                                                                                 |
| `USE_STATIC_LIBS`         | `OFF`³    | Turns on static linking for all libraries, using `-static` with MinGW and `-static-libgcc` / `-static-libstdc++` with other GCC-like toolchains. You can also use the individual options below: |
| `LZMA_USE_STATIC_LIBS`    | `OFF`⁴    | Statically link `liblzma`.                                                                                                                                                                      |
| `Boost_USE_STATIC_LIBS`   | `OFF`⁴    | Statically link Boost. See also `FindBoost.cmake`.                                                                                                                                              |
| `ZLIB_USE_STATIC_LIBS`    | `OFF`⁴    | Statically link `libz`. (used via Boost)                                                                                                                                                        |
| `BZip2_USE_STATIC_LIBS`   | `OFF`⁴    | Statically link `libbz2`. (used via Boost)                                                                                                                                                      |
| `iconv_USE_STATIC_LIBS`   | `OFF`⁴    | Statically link `libiconv`.                                                                                                                                                                     |
| `STRICT_USE`              | `OFF`     | Abort if there are missing optional dependencies.                                                                                                                                               |
| `DEVELOPER`               | `OFF`     | Enable build options suitable for developers⁵.                                                                                                                                                  |
| `FASTLINK`                | `OFF`⁶    | Optimize for link speed.                                                                                                                                                                        |
| `USE_LTO`                 | `ON`²     | Use link-time code generation.                                                                                                                                                                  |
| `USE_LD`                  | `best`⁸   | Linker to use - `default`, `mold`, `lld`, `gold`, `bfd` or `best`                                                                                                                               |
| `BUILD_TESTS`             | `OFF`⁶    | Build unit tests that can be run using `make check`                                                                                                                                             |
| `RUN_TESTS`               | `OFF`⁷    | Automatically run tests                                                                                                                                                                         |
| `RUN_TARGET`              | (none)    | Wrapper to run binaries produced in the build process                                                                                                                                           |

1. The builtin charset conversion only supports Windows-1252 and UTF-16LE. This is normally enough for filenames, but custom message strings (which can be included in filenames) may use arbitrary encodings.
2. Enabled automatically if `CMAKE_BUILD_TYPE` is set to `Debug`.
3. Under Windows, the default is `ON`.
4. Default is `ON` if `USE_STATIC_LIBS` is enabled.
5. Currently this and enables `DEBUG`, `BUILD_TESTS`, `RUN_TESTS` and `FASTLINK` for faster incremental builds and improved debug output, unless those options have been explicitly specified by the user.
6. Enabled automatically if `DEVELOPER` is enabled.
7. Enabled automatically if `DEVELOPER` is enabled unless cross-compiling without `RUN_TARGET` set
8. Disabled automatically (set to `default`) if both `SET_OPTIMIZATION_FLAGS` and `FASTLINK` are disabled. `best` will select the most suited linker based on availability and other settings such as `USE_LTO`.

Install options:

| Option                      | Default              | Description                                   |
|-----------------------------|----------------------|-----------------------------------------------|
| `CMAKE_INSTALL_PREFIX`      | `/usr/local`         | Where to install innoextract.                 |
| `CMAKE_INSTALL_BINDIR`      | `bin`                | Location for binaries (relative to prefix).   |
| `CMAKE_INSTALL_DATAROOTDIR` | `share`              | Location for data files (relative to prefix). |
| `CMAKE_INSTALL_MANDIR`      | `${DATAROOTDIR}/man` | Location for man pages (relative to prefix).  |

Set options by passing `-D<option>=<value>` to cmake.

## Run

To extract a setup file to the current directory run:

    $ innoextract <file>

A list of available options can be retrieved using

    $ innoextract --help

Documentation is also available as a man page:

    $ man 1 innoextract

## Limitations

* There is no support for extracting individual components and limited support for filtering by name.
* Included scripts and checks are not executed.
* The mapping from Inno Setup variables like the application directory to subdirectories is hard-coded.
* Names for data slice/disk files in multi-file installers must follow the standard naming scheme.

A perhaps more complete, but Windows-only, tool to extract Inno Setup files is [innounp](https://github.com/jrathlev/InnoUnpacker-Windows-GUI).

Extracting Windows installer executables created by programs other than Inno Setup is out of the scope of this project. Some of these can be unpacked by [cabextract](https://cabextract.org.uk/) or [unshield](https://github.com/twogood/unshield).

## License

luducat-innoextract is available under the zlib license, like innoextract; see [LICENSE](LICENSE). It is an altered version of innoextract and not the original software. Code from other projects in the source tree, and the libraries it links against, are listed in [THIRD_PARTY_LICENSES.txt](THIRD_PARTY_LICENSES.txt).

## Disclaimer

This project is in no way associated with Inno Setup or [jrsoftware.org](https://jrsoftware.org/). It is not the original innoextract by Daniel Scharrer, and not crazy-max/innoextract; please report problems with this fork here.

---
luducat-innoextract has been written with the help of tooled assistance, but has been
designed, reviewed and thoroughly tested and signed off by its developer.
