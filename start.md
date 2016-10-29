# Introduction to Cerbero

Cerbero is a Python-based packaging system for downloading, compiling, installing, and packaging GStreamer and all the libraries it uses. Instructions on how to do this for each component are codified in `recipe` files which are shipped with Cerbero itself. Cerbero can be used on Linux, OS X, and Windows, and can cross-compile for Linux, Windows, Android, and iOS. See the full [list of architectures and platforms](#supported-platforms).

The first step is to fetch the latest Cerbero from git:

    $ git clone git://anongit.freedesktop.org/gstreamer/cerbero cerbero.git
    $ cd cerbero.git

By default, this will build the latest git version of GStreamer. If you want to, for example, build the 1.8 stable release, you can checkout the `1.8` branch like so:

    $ git checkout 1.8

If you have multiple remotes, you will need to specify the pathspec and the branch name explicitly:

    $ git checkout -b 1.8 origin/1.8
    
You can switch back by doing:

    $ git checkout master

And so on.

# Table of Contents
 * [Directory Structure](#dir-structure)
 * [Command-Line Interface](#cli-usage)
 * [Supported Architectures and Platforms](#supported-platforms)
 * [Recipes](#recipe-format)
 * [Fetching and Extracting](#fetch-extract)
 * [Building and Logging](#building-logging)
 * [Packages](#package-format)
 * [Generate Visual Studio projects](#vs-projects)
 * [Development Workflow](#dev-workflow)

TODO: configuration, home directory structure (build tools, dist, etc)

## Directory Structure <a id="dir-structure"></a>

Out of all the files and directories inside the Cerbero git directory, the following are the important ones:

    cerbero.git/
      cerbero/
      cerbero-uninstalled
      config/
      packages/
      recipes/

`cerbero-uninstalled` is the [interface to Cerbero](#cli-usage).

`cerbero` contains the Python code that constitutes Cerbero.

`config` has the various "configurations" ([combinations of platforms and architectures](#supported-platforms)) on and for which Cerbero can be run.

`recipes` is where you'll find the `.recipe` files that fetch, build, and install the sources for a piece of software. These are in the form of a Python derived class that overrides methods for fetching, building, and installing. Please see the [recipe documentation](#recipe-format) for details.

`packages` has `.package` files which list the files that each package will contain. Packages can be `.rpm`, `.deb`, `.framework`, `.msi` and `.msm`, just `.tar.bz2`, and so on. Each [package file](#package-format) is supposed to list all the files necessary for making a particular set of libraries work. This will create two sets of packages: one contains runtime libraries and executables and the other will have development libraries and headers.

## Command-Line Interface <a id="cli-usage"></a>

Although Cerbero can be installed on your system using the bundled `Makefile`, the most common (and recommended) way to use it is by directly running it from the git repository as shown in the following commands.

Let's start with building GStreamer 1.x for the native platform and architecture. First, we must bootstrap the Cerbero build environment.

    $ python2 cerbero-uninstalled bootstrap

Next, we can build a specific recipe. Let's say, we only want the core GStreamer libraries and GStreamer base plugins

    $ python2 cerbero-uninstalled build gst-plugins-base-1.0

This will build all the recipes that are needed by `gst-plugins-base-1.0.recipe` (found in the `recipes` directory) and then build the recipe itself.

We can also build an entire package in one go. Such as the set of core GStreamer libraries and plugins:

    $ python2 cerbero-uninstalled package gstreamer-1.0-core

Or the entire set of GStreamer libraries and plugins:

    $ python2 cerbero-uninstalled package gstreamer-1.0

While on Linux, to cross-compile for Windows, you just need to add a `-c` argument specifying the appropriate configuration file to use:

    $ python2 cerbero-uninstalled -c config/cross-win32.cbc package gstreamer-1.0

You can also cross-compile for 32-bit Linux when building on 64-bit Linux:

    $ python2 cerbero-uninstalled -c config/cross-lin-x86.cbc package gstreamer-1.0

While on Windows, to build for 32-bit x86 using MinGW and the MSVC toolchain (wherever possible), the config file to use is:

    $ python2 cerbero-uninstalled -c config/win32-mixed-msvc.cbc package gstreamer-1.0

## Supported Architectures and Platforms <a id="supported-platforms"></a>

Currently Cerbero supports the following architecture, platform, and toolchain combinations:

<table>
  <tr>
    <th>Platforms</th>
    <th>Toolchain</th>
    <th>Architecture</th>
    <th>Configuration</th>
  </tr>
  <tr>
    <td>Linux</td>
    <td>GCC</td>
    <td>any</td>
    <td>(native, none)</td>
  </tr>
  <tr>
    <td rowspan="4">Windows</td>
    <td rowspan="2">MinGW/GCC</td>
    <td>x86</td>
    <td>win32.cbc</td>
  </tr>
  <tr>
    <td>x86_64</td>
    <td>win64.cbc</td>
  </tr>
  <tr>
    <td rowspan="2">MSVC</td>
    <td>x86</td>
    <td>win32-mixed-msvc</td>
  </tr>
  <tr>
    <td>x86_64</td>
    <td>win64-mixed-msvc</td>
  </tr>
  <tr>
    <td rowspan="2">OS X</td>
    <td rowspan="2">Clang</td>
    <td>x86_64</td>
    <td>osx-x86-64.cbc</td>
  </tr>
  <tr>
    <td>universal</td>
    <td>osx-universal.cbc</td>
  </tr>
  <tr>
    <td rowspan="4">iOS (cross)</td>
    <td rowspan="4">Clang</td>
    <td>armv7</td>
    <td>cross-ios-armv7.cbc</td>
  </tr>
  <tr>
    <td>arm64</td>
    <td>cross-ios-arm64.cbc</td>
  </tr>
  <tr>
    <td>x86_64</td>
    <td>cross-ios-x86-64.cbc</td>
  </tr>
  <tr>
    <td>universal</td>
    <td>cross-ios-universal.cbc</td>
  </tr>
  <tr>
    <td rowspan="4">Android (cross)</td>
    <td rowspan="4">GCC</td>
    <td>armv7</td>
    <td>cross-android-armv7.cbc</td>
  </tr>
  <tr>
    <td>arm64</td>
    <td>cross-android-arm64.cbc</td>
  </tr>
  <tr>
    <td>x86</td>
    <td>cross-android-x86.cbc</td>
  </tr>
  <tr>
    <td>x86_64</td>
    <td>cross-android-x86-64.cbc</td>
  </tr>
  <tr>
    <td rowspan="2">Windows (cross)</td>
    <td rowspan="2">MinGW/GCC</td>
    <td>x86</td>
    <td>cross-win32.cbc</td>
  </tr>
  <tr>
    <td>x86_64</td>
    <td>cross-win64.cbc</td>
  </tr>
  <tr>
    <td rowspan="2">Linux (cross)</td>
    <td rowspan="2">GCC</td>
    <td>x86</td>
    <td>cross-lin-x86.cbc</td>
  </tr>
  <tr>
    <td>arm</td>
    <td>cross-lin-arm.cbc</td>
  </tr>
</table>


Support for LLVM/Clang on Linux and for GCC on OS X is easy to add if needed.

Cross-compilation targetting Windows is currently only available on Linux.

Of particular note is that while building natively on Windows with the use of Meson build files, Cerbero is able to build GStreamer and its core dependencies entirely with the Microsoft Visual C++ compiler. The full list of recipes that can use it are:

| Recipes that can use MSVC |
|---------------------------|
| bzip2.recipe |
| orc.recipe |
| libffi.recipe (only 32-bit) |
| glib.recipe |
| gstreamer-1.0.recipe |
| gst-plugins-base-1.0.recipe |
| gst-plugins-good-1.0.recipe |
| gst-plugins-bad-1.0.recipe |
| gst-plugins-ugly-1.0.recipe |

This feature is enabled by the use of `config/win32-mixed-msvc.cbc` as demonstrated in the previous section. By default if no configuration is specified the Meson build files use MinGW on Windows.

## Recipes <a id="recipe-format"></a>

A recipe describes the way a specific project (repository or source tarball) is configured, built, and installed. It specifies the URI for the source tree (http, git, etc), the build type (Autotools, CMake, Meson, Custom), the version number (only used while packaging), the dependencies (other recipes needed), patches, and so on.

#### Sample Recipe

```python
class Recipe(recipe.Recipe):
    name = 'json-glib'
    version = '1.0.4'
    licenses = [License.LGPLv2_1Plus]
    btype = BuildType.AUTOTOOLS
    stype = SourceType.TARBALL
    url = 'http://download.gnome.org/sources/{0}/1.0/{0}-{1}.tar.xz'.format(name, version)
    patches = ['json-glib/0001-Don-t-override-our-own-ACLOCAL_FLAGS-but-append-them.patch']
    deps = ['glib']
```

As you can see, the syntax is similar to that of Python. In fact, it *is* Python. It's a simple class definition with attributes attached.

In technical terms, each recipe in the `recipes/` directory just extends the `recipe.Recipe` class defined in `cerbero/build/recipe.py`. The `metaclass` of the `recipe.Recipe` adds attributes based on the `btype` variable from classes in `cerbero/build/build.py` and based on the `stype` variable from classes in `cerbero/build/source.py`. There is more to it, but these are the two most important interactions.

#### Basic Attributes

`name` is the name of the recipe. The filename of the recipe does not matter at all. This variable is used as the name and it must be unique.

`version` is only used in build paths and has no other effect. It is still a good idea to keep it in sync with the project version.

`stype` is the source type to be fetched. The most common values are `TARBALL`, `GIT`, `SVN`, and `CUSTOM` (no sources to download).

`btype` is the build system type that will be used. Valid values are `MAKEFILE`, `AUTOTOOLS`, `CMAKE`, `MESON`, and `CUSTOM`. The default if unspecified is `AUTOTOOLS`.

`licenses` is a list of licenses. The full list of available licenses is in `cerbero/enums.py`.

`deps` is a list of recipe names that are needed for this recipe to be built.

Besides these basic attributes, there are other sourcetype-specific and buildtype-specific attributes. We will also look at the function attributes that can be defined in the recipe.

#### Extended Sample Recipe

```python
class Recipe(recipe.Recipe):
    name = 'json-glib'
    version = '1.0.4'
    licenses = [License.LGPLv2_1Plus]
    btype = BuildType.AUTOTOOLS
    stype = SourceType.TARBALL
    url = 'http://download.gnome.org/sources/{0}/1.0/{0}-{1}.tar.xz'.format(name, version)
    patches = ['json-glib/0001-Don-t-override-our-own-ACLOCAL_FLAGS-but-append-them.patch']
    deps = ['glib']

    autoreconf = True
    #autoreconf_sh = './autogen.sh --noconfigure'
    configure_options = '--enable-static'
    #config_sh = './configure'

    files_bins = ['json-glib-validate', 'json-glib-format']
    files_libs = ['libjson-glib-1.0']
    files_devel  = ['include/json-glib-1.0', 'lib/pkgconfig/json-glib-1.0.pc']
    files_typelibs = ['Json-1.0']

    def prepare(self):
        self.append_env['CFLAGS'] = ' -Wno-error '
        if self.config.target_platform == Platform.WINDOWS:
            # Don't need autoreconf on MinGW so don't do it since it's very slow
            self.autoreconf = False
        if self.config.target_arch == Architecture.X86_64:
            self.append_env['CFLAGS'] += '-fno-omit-pointer '
```

#### More Attributes

`url` is only used when the `stype` is `TARBALL` and it defines a single HTTP(S) URL to download and extract for use as the source tarball. The supported compression formats are `tar` (`gzip`, `bzip2`, `xz`) and `zip`.

When the `stype` is GIT, two separate variables are used instead: `remotes` and `commit`. `remotes` is a `dict` of one or more git URLs from which to fetch the source. `commit` is a revision specification that points to something that can be checked out. This can be a branch, a tag, a specific commit, etc. For example:

```python
remotes = {'origin' : 'git://anongit.freedesktop.org/gstreamer/gstreamer',
           'github: 'https://github.com/myorg/gstreamer.git'}
#commit = 'origin/master' (original value)
#commit = '1ed65e56' (a commit)
commit = 'github/my-work-branch'
```

`patches` is a list of patches that will be applied to the source tree after extraction. The path is relative to the recipe's directory.

`autoreconf` is used when `btype` is `AUTOTOOLS` and forces an autoreconf when set to `True`. Optionally, you can also specify a specific script to run for doing the autoreconf.

`configure_options` is used when `btype` is `AUTOTOOLS`, `MAKEFILE`, `CMAKE`, and `MESON`. Optionally, you can also specify the script that runs configure and takes these arguments with `config_sh`.

`files_*` variables define the executables (`files_bins`), libraries (`files_libs`), headers and pkg-config files and so on (`files_devel`), and typelibs `files_typelibs`. These control which files are copied into the packages (generated with the `package` command), and into which package (development or runtime). Besides these, there are several other file-related variables that are quite self-explanatory. Please see [gstreamer-1.0.recipe](https://cgit.freedesktop.org/gstreamer/cerbero/tree/recipes/gstreamer-1.0.recipe#n30) and [gst-plugins-bad-1.0.recipe](https://cgit.freedesktop.org/gstreamer/cerbero/tree/recipes/gst-plugins-bad-1.0.recipe#n37). For example, for specifying and categorising plugins, we use the `files_plugins_*` variables.

#### Recipe Methods

The `prepare` method on the class is run when the recipe is parsed and the recipe object has been created. This happens for all recipes known to Cerbero every time a command is run. It can set attributes on the recipe such as the environment to be used for it (`self.append_env`, it can modify the values of the instance attributes described above, and so on. It does not have access to the sources or the build result.

In the unlikely case that you need to intervene during the build process itself, you can override or extend the `configure`, `compile`, and `install` methods. The default implementations of these are defined in `cerbero/build/build.py` and depend on the `btype` of the recipe. The best way to understand how these work is by looking at examples such as [openh264.recipe](https://cgit.freedesktop.org/gstreamer/cerbero/tree/recipes/openh264.recipe).

If you want to make changes to the installed files after install, you can implement a method called `post_install`. The default implementation is to do nothing.

## Fetching and Extraction <a id="fetch-extract"></a>

For this entire section, it will be assumed that your Cerbero home is set to `~/cerbero`. You can change this value by adding `home_dir = '/some/path/to/dir'` to `~/.cerbero/cerbero.cbc`.

#### Fetch

If `stype` is `TARBALL`, either `wget` or `curl` is used to download the specified URL. If `stype` is `GIT`, `git` is used to fetch all the specified `remotes` and then the tree-ish specified in `commit` is checked out. All this is stored in `~/cerbero/sources/local`. The contents of this directory are shared between all architectures and configurations since it contains unmodified sources downloaded from remote servers.

#### Extract

The next step is to extract and patch these sources in a recipe-specific subdir inside a configuration-specific directory. This can be, for instance, `~/cerbero/sources/windows_x86/json-glib-1.0.4`. This same directory is later used for `configure`, `compile`, etc.

All this is done as part of the `fetch` and `extract` methods in the corresponding class in `cerbero/build/source.py`.

## Building and Logging <a id="building-logging"></a>

The standard build steps every project goes through are:

#### `configure`

For Autotools this might involve running `autoreconf` (if `self.autoreconf = True`), followed by running `./configure` (or the value of `config_sh`).

For Meson, this means running `meson` which will generate the `build.ninja` file to actually build the project.

#### `compile`

For Autotools this runs `make` (with or without a jobs `-j` argument depending on the configuration)

For Meson this runs `ninja` (or `ninja-build`)

#### `install`

For Autotools this runs `make install` with the `DESTDIR` environment variable set.

For Meson this runs `ninja install` with the `DESTDIR` environment variable set.

In both cases, the `DESTDIR` env variable points to a configuration-specific directory inside the Cerbero home dir. This can be, for instance, `~/cerbero/dist/windows_x86` or `~/cerbero/dist/linux_x86_64`, and so on.

#### `post_install`

By default, this method does nothing at all. If you wish to manipulate the installed files, you can implement it in your recipe.

#### `gen_library_file`

When building for Windows, an extra step is run after `post_install`. This step generates import libraries.

When `btype` is `MESON` and the toolchain used is MSVC, this step creates `.dll.a` import libraries that are used by the MinGW toolchain for linking to the generated DLLs.

When `btype` is anything else, this step creates `.lib` import libraries that are needed by the MSVC toolchain for linking to the generated DLLs.

Note: MinGW can consume MSVC-style `.lib` import libraries too, but it is sometimes unable to resolve variables defined in them, so we always generate `.dll.a` import libraries since those don't have this problem.

#### Logging

Cerbero always logs all stdout and stderr output to `~/cerbero/logs`. The logging for each build step described above is stored inside a configuration-specific directory with a flat namespace.

For instance, the configure log for cross-compiling glib on 32-bit Windows on Linux would be stored at `~/cerbero/logs/windows_x86/glib-configure.log`.

The steps that are logged here are `fetch`, `extract`, `configure`, `compile`, `install`, and for Windows `gen_library_file`.

When building with Meson, one might need to look at the internal logging stored by Meson itself. That can be found in the source build directory. For example, `~/cerbero/sources/linux_x86_64/gstreamer-1.0-1.8/cerbero-build-dir/meson-logs/meson-log.txt`.

## Packages <a id="package-format"></a>

Cerbero also has the concept of 'packages' each of which is essentially a manifest with metadata. Similar to a `.recipe`, a `.package` file is just a Python class with various attributes that define the class and also has a `prepare` method for runtime definition of variables. Unlike recipes, packages do not have any other methods.

A package file lists one or many libraries, binaries, development files (headers, import libraries, pkg-config files, and so on), GStreamer plugins, translations, GIRs, typelibs, etc, from one or more recipes.

These lists are used for creating packages for various OSes such as Debian `.deb` packages, Fedora `.rpm` packages, Windows `.msi` installers, and many more.

For instance, the `gstreamer-1.0-core.package` looks like this:

```python
class Package(package.Package):

    name = 'gstreamer-1.0-core'
    shortdesc = 'GStreamer 1.0 core'
    longdesc = 'GStreamer 1.0 core'
    url = "http://gstreamer.freedesktop.org"
    version = '1.8.0'
    codename = 'Congo'
    license = License.LGPL
    vendor = 'GStreamer Project'
    org = 'org.freedesktop.gstreamer'
    uuid = 'd9626750-e8b7-4e40-944d-98b67ed0c6bf'
    deps = ['base-system-1.0']

    files = ['gstreamer-1.0', 'gst-plugins-base-1.0:bins:libs:core:lang:typelibs',
            'gst-shell',
            'gst-plugins-good-1.0:plugins_core:lang',
            'gst-plugins-bad-1.0:plugins_core:lang:libs:typelibs',
            'gst-plugins-ugly-1.0:plugins_core:lang']
    files_devel = ['gstreamer-1.0-static', 'gst-plugins-base-1.0-static:plugins_core_devel',
                   'gst-plugins-good-1.0-static:plugins_core_devel',
                   'gst-plugins-bad-1.0-static:plugins_core_devel']
    platform_files = {
            Platform.DARWIN: ['gstreamer-1.0-osx-framework'],
            Platform.IOS: ['gstreamer-ios-templates'],
            Platform.ANDROID: ['gst-android-1.0'],
            }
```

You can see that many of the attributes are self-explanatory. Besides those,

`deps` is a list of *other* packages that this package depends on

`files_*` are variables defining files from other **recipes** that this package will contain

The syntax `gst-plugins-bad-1.0:plugins_core:lang:libs:typelibs` means 'from the recipe gst-plugins-bad-1.0, include the core plugins (`files_plugins_core` in the recipe), the translations (`files_lang` in the recipe), the libraries (`files_libs` in the recipe) and GObject Introspection typelibs (`files_typelibs` in the recipe). You can think of the `:` separating the file types in the syntax as commas in a list.

The development files syntax is exactly the same. For example, `plugins_core_devel` pulls in the core static plugins from the specified recipe.

Each package file gets mapped to two 'units' of installation. For Linux distros, this means that each package file yields two `.deb` or `.rpm` files. One for runtime files (the `files` attribute above) and one for development files (the `files_devel` attribute above). Packages that don't list any files but have dependencies on other packages simply yield meta-rpms or meta-debs.

For Windows, the selected package and all the dependency packages combined yield exactly two `.msi` installers. One for runtime files (`files`) and one for development filesi (`files_devel`). Each package then becomes a `feature` in the installer and the inter-package dependencies become inter-feature dependencies.

## Generating Visual Studio Projects and Solutions <a id="vs-projects"></a>

On Windows, when building recipes that use Meson and support building with MSVC, Cerbero can use the Meson Visual Studio backend to generate Visual Studio 2015 projects and a solution for a specific recipe.

**IMPORTANT:** Before doing this, you must ensure that the recipe has been built once by Cerbero for the selected platform.

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc build glib

To generate `vcxproj`s and a `sln` for the same:

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc genvssln glib

This will run `meson --backend=vs2015` inside the Cerbero environment which will embed all the necessary information (libraries, library paths, include paths, cflags, etc) inside the project files. It will then print the path to the solution file. Note that this means the project files will refer to the libraries *built by Cerbero* for external dependencies.

You can also use the `--open` argument to open the directory in `explorer` so you can open and build the solution in Visual Studio.

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc genvssln --open glib

By default, these commands will create Visual Studio files in the Cerbero platform-specific `sources` directory. In this case, that would be `~/cerbero/sources/windows_x86_64/glib-*/` (replace `*` with the version). The solution and all related build files will be in the `vs-build-dir` sub-directory.

You can also specify any other directory as the source tree for generating the Visual Studio files from with the `-s/--source-dir` option.

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc genvssln -s C:/projects/glib.git glib

Make sure that you always use forward slashes `/` when specifying the path because `bash` reserves `\` as an escape character. You must also be careful that the sources in the directory you are using match the recipe name that you are specifying.

You can now open the generated solution in Visual Studio and build it normally. The outputs will be in the same directory as the `vcxproj` for each output. So, for instance, for `glib`, `gio-2.0-0.dll`, `gio-2.0.lib`, and `gio-2.0-0.pdb` will be found in the `gio` subdirectory.

## Development Workflow <a id="dev-workflow"></a>

Cerbero was initially developed as a B&I system for use on build boxes/VMs and was not intended for development use since the default Linux environment provides everything you need to do GStreamer development out of the box. However, on OS X and Windows, the most convenient way to work on GStreamer itself is to setup Cerbero and use it to build GStreamer with any changes that you have made.

There are three different ways to use Cerbero as a development environment. We will give the example of Windows in many of these explanations because that platform has the most number of exceptions or non-standard behaviour compared to any other platform.

#### Custom Development Repositories

The cleanest and simplest way to build GStreamer with your own changes is to create your own copies of the repositories you want to make changes to, and change the corresponding recipes to point to your repositories instead. To do that for specific repository, you can follow these steps:

Let's say we want to make changes to `gst-plugins-base-1.0`. First we look at `recipes/gst-plugins-base-1.0.recipe` and look for the `remotes` and `commit` variables. Let's say they look like this:

```python
remotes = {'origin': 'git://anongit.freedesktop.org/gstreamer/gst-plugins-base'}
commit = 'origin/1.8'
```

This means you want to make changes to the `1.8` branch of `origin`. So we clone that remote and create our own branch based on `1.8`.


    $ cd projects
    $ git clone git://anongit.freedesktop.org/gstreamer/gst-plugins-base
    $ git checkout -b my-1.8 origin/1.8

Next, we can make any changes we need to make and optionally push it to our own Git server

    $ git remote add my-remote git@git.myserver.net:projects/gst-plugins-base
    $ git push -u my-remote my-1.8

Now we edit the recipe to refer to our Git server's repository

```python
remotes = {'origin': 'git://anongit.freedesktop.org/gstreamer/gst-plugins-base',
           'my-remote': 'git@git.myserver.net:projects/gst-plugins-base',}
commit = 'my-remote/my-1.8'
```

**If you don't want to use a git server**, you can also just use your local git repository

```python
remotes = {'origin': 'git://anongit.freedesktop.org/gstreamer/gst-plugins-base',
           'local': 'C:/projects/gst-plugins-base',}
commit = 'local/my-1.8'
```

That's it! Now you can work on your local repository, commit any changes (and push them if you use your git server as the remote) and build them using Cerbero:

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc buildone gst-plugins-base

You need to use `buildone` here instead of `build` so that the recipe status is reset and Cerbero fetches any new commits from the remotes that you have specified. With just `build` it will assume that everything is up-to-date and happily exit.

If you base your entire workflow around Visual Studio, you can skip the `buildone` command from above and just run `genvssln --fetch`. Just make sure that you exit Visual Studio before running this since it will try to wipe the build dir and that will fail if Visual Studio is still open.

    $ python2 cerbero-uninstalled -c config/win32-mixed-msvc.cbc genvssln --fetch --open gst-plugins-base

Of course if you use a custom source tree with `genvssln`, you shouldn't use `--fetch` and in fact it will be ignored if `-s/--source-dir` is specified.

If you want to fetch any changes done upstream and integrate them with your own changes in your branch, you can do a merge.

```sh
# Ensure you're on your work branch
$ git checkout my-1.8
# Fetch all upstream changes
$ git fetch origin
# Merge upstream's changes with your own
$ git merge origin/1.8
# ... resolve any merge conflicts, commit the result ...
# Push the merged branch
$ git push my-remote my-1.8
```

If you have not pushed your work anywhere else (i.e., you do not use any git servers), you can do a rebase instead of a merge.

```sh
# Ensure you're on your work branch
$ git checkout my-1.8
# Fetch all upstream changes
$ git fetch origin
# Rebase your changes on top of upstream's changes
$ git rebase origin/1.8
# ... resolve any rebase conflicts; follow the instructions given by git ...
```

#### Development Using Recipe Patches

Another method of doing GStreamer development using Cerbero is by adding patches to the recipe that you want to make changes to by appending to (or creating a new) `patches` recipe attribute. Let's say you have the following recipe.

```python
class Recipe(recipe.Recipe):
    name = 'gst-plugins-base-1.0'
    version = '1.8'
    licenses = [License.LGPLv2Plus]
    configure_options = "--enable-static --program-prefix= --disable-examples "
    remotes = {'origin': 'git://anongit.freedesktop.org/gstreamer/gst-plugins-base'}
    commit = 'origin/master'
    stype = SourceType.GIT
    btype = BuildType.MESON
```

Note that it has no patches right now. The next step is to create a directory called `gst-plugins-base-1.0` (same as the `name` attribute) inside the `recipes` directory.

    $ mkdir recipes/gst-plugins-base-1.0

Now you put your patches (they must be git-formatted) inside this directory and list them in a new `patches` attribute

```python
class Recipe(recipe.Recipe):
    name = 'gst-plugins-base-1.0'
    version = '1.8'
    licenses = [License.LGPLv2Plus]
    configure_options = "--enable-static --program-prefix= --disable-examples "
    remotes = {'origin': 'git://anongit.freedesktop.org/gstreamer/gst-plugins-base'}
    commit = 'origin/master'
    stype = SourceType.GIT
    btype = BuildType.MESON
    patches = ['gst-plugins-base-1.0/0001-Fix-some-silly-bug.patch',
               'gst-plugins-base-1.0/0002-Add-new-feature-foo.patch']
```

The recipe is now ready for building!

    $ python2 cerbero-uninstalled -c config/win64-mixed-msvc.cbc build gst-plugins-base

These two methods are very clean and will never cause strange Cerbero issues because they do not touch Cerbero's internal source trees or any other internal state at all. The next one does, and hence it's not a recommended method. Please don't use it unless you know exactly what you are doing. If you break anything, you get to keep the pieces.

#### Development Using Cerbero's Source Repositories

This method is a short-cut and will work most of the time, but if you mess up, you will lose all your changes. Also, if you don't understand how Cerbero works, you will likely install garbage to your install prefix and will have to `wipe` everything. The basic idea is that you can directly make changes to the Cerbero source repositories, invoke `make` or `ninja` manually, and check if your patches actually work. Here's a series of commands that show how to do this for a recipe that uses Meson and Ninja.

```sh
$ cd ~/cerbero/sources/windows_x86/gstreamer-1.0-1.8
# ... make quick changes ...
$ cd cerbero-build-dir
# This will invoke `meson` automatically if you changed the build files
$ ninja -v
$ GST_PLUGIN_PATH_1_0=$PWD/plugins/elements ./tools/gst-inspect-1.0.exe coreelements
```

Be careful while running `ninja install` or `make install` since recipes sometimes have custom arguments, environment variables, `post_install` hooks, `gen_library_file`, etc that need to be done afterwards.

Note that whatever changes you make here will be overwritten the next time you build this recipe. Hence, this method is not appropriate for long-term development and is actually cumbersome if you have several changes you need to apply. The first method is the best one in that case.
