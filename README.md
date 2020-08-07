[![Downloads](http://pepy.tech/badge/opencv-python)](http://pepy.tech/project/opencv-python)

## OpenCV on Wheels

**Unofficial** pre-built OpenCV packages for Python.

### Installation and Usage

1. If you have previous/other manually installed (= not installed via ``pip``) version of OpenCV installed (e.g. cv2 module in the root of Python's site-packages), remove it before installation to avoid conflicts.
2. Select the correct package for your environment:

    There are four different packages and you should **select only one of them**. Do not install multiple different packages in the same environment. There is no plugin architecture: all the packages use the same namespace (`cv2`). If you installed multiple different packages in the same environment, uninstall them all with ``pip uninstall`` and reinstall only one package.

    **a.** Packages for standard desktop environments (Windows, macOS, almost any GNU/Linux distribution)

    - run ``pip install opencv-python`` if you need only main modules
    - run ``pip install opencv-contrib-python`` if you need both main and contrib modules (check extra modules listing from [OpenCV documentation](https://docs.opencv.org/master/))

    **b.** Packages for server (headless) environments

    These packages do not contain any GUI functionality. They are smaller and suitable for more restricted environments.

    - run ``pip install opencv-python-headless`` if you need only main modules
    - run ``pip install opencv-contrib-python-headless`` if you need both main and contrib modules (check extra modules listing from [OpenCV documentation](https://docs.opencv.org/master/))

3. Import the package:

    ``import cv2``

    All packages contain haarcascade files. ``cv2.data.haarcascades`` can be used as a shortcut to the data folder. For example:

    ``cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")``

5. Read [OpenCV documentation](https://docs.opencv.org/master/)

6. Before opening a new issue, read the FAQ below and have a look at the other issues which are already open.

Frequently Asked Questions
--------------------------

**Q: Do I need to install also OpenCV separately?**

A: No, the packages are special wheel binary packages and they already contain statically built OpenCV binaries.

**Q: Pip fails with ``Could not find a version that satisfies the requirement ...``?**

A: Most likely the issue is related to too old pip and can be fixed by running ``pip install --upgrade pip``. Note that the wheel (especially manylinux) format does not currently support properly ARM architecture so there are no packages for ARM based platforms in PyPI. However, ``opencv-python`` packages for Raspberry Pi can be found from https://www.piwheels.org/.

**Q: Import fails on Windows: ``ImportError: DLL load failed: The specified module could not be found.``?**

A: If the import fails on Windows, make sure you have [Visual C++ redistributable 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145) installed. If you are using older Windows version than Windows 10 and latest system updates are not installed, [Universal C Runtime](https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows) might be also required.

Windows N and KN editions do not include Media Feature Pack which is required by OpenCV. If you are using Windows N or KN edition, please install also [Windows Media Feature Pack](https://support.microsoft.com/en-us/help/3145500/media-feature-pack-list-for-windows-n-editions).

If you have Windows Server 2012+, media DLLs are probably missing too; please install the Feature called "Media Foundation" in the Server Manager. Beware, some posts advise to install "Windows Server Essentials Media Pack", but this one requires the "Windows Server Essentials Experience" role, and this role will deeply affect your Windows Server configuration (by enforcing active directory integration etc.); so just installing the "Media Foundation" should be a safer choice.

If the above does not help, check if you are using Anaconda. Old Anaconda versions have a bug which causes the error, see [this issue](https://github.com/skvark/opencv-python/issues/36) for a manual fix.

If you still encounter the error after you have checked all the previous solutions, download [Dependencies](https://github.com/lucasg/Dependencies) and open the ``cv2.pyd`` (located usually at ``C:\Users\username\AppData\Local\Programs\Python\PythonXX\Lib\site-packages\cv2``) file with it to debug missing DLL issues.

**Q: I have some other import errors?**

A: Make sure you have removed old manual installations of OpenCV Python bindings (cv2.so or cv2.pyd in site-packages).

**Q: Why the packages do not include non-free algorithms?**

A: Non-free algorithms such as SURF are not included in these packages because they are patented / non-free and therefore cannot be distributed as built binaries. Note that SIFT is included in the builds due to patent expiration since OpenCV versions 4.3.0 and 3.4.10. See this issue for more info: https://github.com/skvark/opencv-python/issues/126

**Q: Why the package and import are different (opencv-python vs. cv2)?**

A: It's easier for users to understand ``opencv-python`` than ``cv2`` and it makes it easier to find the package with search engines. `cv2` (old interface in old OpenCV versions was named as `cv`) is the name that OpenCV developers chose when they created the binding generators. This is kept as the import name to be consistent with different kind of tutorials around the internet. Changing the import name or behaviour would be also confusing to experienced users who are accustomed to the ``import cv2``.

## Documentation for opencv-python

[![AppVeyor CI test status (Windows)](https://img.shields.io/appveyor/ci/skvark/opencv-python.svg?maxAge=3600&label=Windows)](https://ci.appveyor.com/project/skvark/opencv-python)
[![Travis CI test status (Linux and macOS)](https://img.shields.io/travis/skvark/opencv-python.svg?maxAge=3600&label=Linux+macOS)](https://travis-ci.org/skvark/opencv-python)

The aim of this repository is to provide means to package each new [OpenCV release](https://github.com/opencv/opencv/releases) for the most used Python versions and platforms.

### CI build process

The project is structured like a normal Python package with a standard ``setup.py`` file.
The build process for a single entry in the build matrices is as follows (see for example ``appveyor.yml`` file):

0. In Linux and MacOS build: get OpenCV's optional C dependencies that we compile against

1. Checkout repository and submodules

   -  OpenCV is included as submodule and the version is updated
      manually by maintainers when a new OpenCV release has been made
   -  Contrib modules are also included as a submodule

2. Find OpenCV version from the sources

3. Build OpenCV

   -  tests are disabled, otherwise build time increases too much
   -  there are 4 build matrix entries for each build combination: with and without contrib modules, with and without GUI (headless)
   -  Linux builds run in manylinux Docker containers (CentOS 5)
   -  source distributions are separate entries in the build matrix 

4. Rearrange OpenCV's build result, add our custom files and generate wheel

5. Linux and macOS wheels are transformed with auditwheel and delocate, correspondingly

6. Install the generated wheel
7. Test that Python can import the library and run some sanity checks
8. Use twine to upload the generated wheel to PyPI (only in release builds)

Steps 1--4 are handled by ``pip wheel``.

The build can be customized with environment variables. In addition to any variables that OpenCV's build accepts, we recognize:

- ``CI_BUILD``. Set to ``1`` to emulate the CI environment build behaviour. Used only in CI builds to force certain build flags on in ``setup.py``. Do not use this unless you know what you are doing.
- ``ENABLE_CONTRIB`` and ``ENABLE_HEADLESS``. Set to ``1`` to build the contrib and/or headless version
- ``ENABLE_JAVA``, Set to ``1`` to enable the Java client build.  This is disabled by default.
- ``CMAKE_ARGS``. Additional arguments for OpenCV's CMake invocation. You can use this to make a custom build. 

See the next section for more info about manual builds outside the CI environment.

### Manual builds

If some dependency is not enabled in the pre-built wheels, you can also run the build locally to create a custom wheel.

1. Clone this repository: `git clone --recursive https://github.com/skvark/opencv-python.git`
2. ``cd opencv-python``
3. Add custom Cmake flags if needed, for example: `export CMAKE_ARGS="-DSOME_FLAG=ON -DSOME_OTHER_FLAG=OFF"` (in Windows you need to set environment variables differently depending on Command Line or PowerShell)
4. Select the version which you wish to build with `ENABLE_CONTRIB` and `ENABLE_HEADLESS`: i.e. `export ENABLE_CONTRIB=1` if you wish to build `opencv-contrib-python`
5. Run ``pip wheel . --verbose``. NOTE: make sure you have the latest ``pip``, the ``pip wheel`` command replaces the old ``python setup.py bdist_wheel`` command which does not support ``pyproject.toml``.
     - Optional: on Linux use the `manylinux` images as a build hosts if maximum portability is needed and run `auditwheel` for the wheel after build
     - Optional: on macOS use ``delocate`` (same as ``auditwheel`` but for macOS)
6. You'll have the wheel file in the `dist` folder and you can do with that whatever you wish

#### Source distributions

Since OpenCV version 4.3.0, also source distributions are provided in PyPI. This means that if your system is not compatible with any of the wheels in PyPI, ``pip`` will attempt to build OpenCV from sources.

You can also force ``pip`` to build the wheels from the source distribution. Some examples: 

- ``pip install --no-binary opencv-python opencv-python``
- ``pip install --no-binary :all: opencv-python``

If you need contrib modules or headless version, just change the package name (step 4 in the previous section is not needed). However, any additional CMake flags can be provided via environment variables as described in step 3 of the manual build section. If none are provided, OpenCV's CMake scripts will attempt to find and enable any suitable dependencies. Headless distributions have hard coded CMake flags which disable all possible GUI dependencies. 

Please note that build tools and ``numpy`` are required for the build to succeed. On slow systems such as Raspberry Pi the full build may take several hours. On a 8-core Ryzen 7 3700X the build takes about 6 minutes. 

### Licensing

Opencv-python package (scripts in this repository) is available under MIT license.

OpenCV itself is available under [3-clause BSD License](https://github.com/opencv/opencv/blob/master/LICENSE).

Third party package licenses are at [LICENSE-3RD-PARTY.txt](https://github.com/skvark/opencv-python/blob/master/LICENSE-3RD-PARTY.txt).

All wheels ship with [FFmpeg](http://ffmpeg.org) licensed under the [LGPLv2.1](http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html).

Non-headless Linux and MacOS wheels ship with [Qt 5](http://doc.qt.io/qt-5/lgpl.html) licensed under the [LGPLv3](http://www.gnu.org/licenses/lgpl-3.0.html).

The packages include also other binaries. Full list of licenses can be found from [LICENSE-3RD-PARTY.txt](https://github.com/skvark/opencv-python/blob/master/LICENSE-3RD-PARTY.txt).

### Versioning

``find_version.py`` script searches for the version information from OpenCV sources and appends also a revision number specific to this repository to the version string. It saves the version information to ``version.py`` file under ``cv2`` in addition to some other flags.

### Releases

A release is made and uploaded to PyPI when a new tag is pushed to master branch. These tags differentiate packages (this repo might have modifications but OpenCV version stays same) and should be incremented sequentially. In practice, release version numbers look like this:

``cv_major.cv_minor.cv_revision.package_revision`` e.g. ``3.1.0.0``

The master branch follows OpenCV master branch releases. 3.4 branch follows OpenCV 3.4 bugfix releases.

### Development builds

Every commit to the master branch of this repo will be built. Possible build artifacts use local version identifiers:

``cv_major.cv_minor.cv_revision+git_hash_of_this_repo`` e.g. ``3.1.0+14a8d39``

These artifacts can't be and will not be uploaded to PyPI.

### Manylinux wheels

Linux wheels are built using [manylinux](https://github.com/pypa/python-manylinux-demo). These wheels should work out of the box for most of the distros (which use GNU C standard library) out there since they are built against an old version of glibc.

The default ``manylinux`` images have been extended with some OpenCV dependencies. See [Docker folder](https://github.com/skvark/opencv-python/tree/master/docker) for more info.

### Supported Python versions

Python 3.x releases are provided for officially supported versions (not in EOL).

Currently, builds for following Python versions are provided:

- 3.5 (EOL in 2020-09-13, builds for 3.5 will not be provided after this)
- 3.6
- 3.7
- 3.8

### Backward compatibility

Starting from 4.2.0 and 3.4.9 builds the macOS Travis build environment was updated to XCode 9.4. The change effectively dropped support for older than 10.13 macOS versions.

Starting from 4.3.0 and 3.4.10 builds the Linux build environment was updated from `manylinux1` to `manylinux2014`. This dropped support for old Linux distributions.

