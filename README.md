# MiniMIP

MiniMIP is an open source, machine learning oriented Mixed-Integer Programming (MIP) solver.
We provide a range of interfaces for all aspects of solving MIPs (e.g. heuristics, cut generators, LP solvers), 
supplying users with a constant view of the internal state and allowing them to propose modifications that are integrated into the global state internally.

This project is still under heavy development and is not ready for practical application.

# Build system

## Bazel
The `bazel` build environment composes of:
* The file `WORKSPACE` that tells from where to fetch all the dependencies
* The file `.bazelrc` that defines commonly used options while building
(e.g., MiniMIP compiles with C++17)
* The build rules for all targets, which are stated explicitly in each
(sub-)directory's `BUILD.bazel` file.

Use **Bazel 7.1.1**, the version configured in `.gitlab-ci.yml`. You can download
it from https://github.com/bazelbuild/bazel/releases/tag/7.1.1.
This project uses `WORKSPACE` dependencies and disables Bzlmod in `.bazelrc`;
Bazel 9 is not supported because it removed `WORKSPACE` support. An empty,
automatically generated `MODULE.bazel` does not migrate these dependencies.

### Linux prerequisites

Before building on Debian/Ubuntu, install the system prerequisites:
```
sudo apt update
sudo apt install clang build-essential libboost-all-dev
```

The Linux configuration in `.bazelrc` explicitly selects `clang`, so it must be
available on `PATH`. `build-essential` supplies standard build tools and C/C++
development files; Boost is needed for SoPlex's LP file support and tests.
Bazel does not install these system packages.

Verify the compiler is available:
```
clang --version
```

If Bazel reports `Cannot find gcc or CC (clang)`, install the prerequisites above
or correct your `PATH`, then retry the build.

### Build and test

To build the solver library, run:
```
bazel build //minimip:solver
```

To build all MiniMIP targets, as in CI, run:
```
bazel build //minimip/...
```

To run a single test target:
```
bazel test //unit_tests:solver_test
```

To run all unit tests, as in CI:
```
bazel test //unit_tests/...
```

Bazel puts all artifacts generated during compilation in the top directory in
directories that start with `bazel-`. Test executables are kept under `bazel-bin`;
for example, after building `//unit_tests:solver_test`, you can run it directly:
```
bazel-bin/unit_tests/solver_test
```

There is currently no `minimip/examples` package or standalone example executable.

Bazel by default builds in "fast-build" mode. To maximize binary performance
(e.g., for benchmarking purposes) compile in "opt" mode:
```
bazel build -c opt //minimip:solver
```
To include symbols allowing for debugging (with `gdb`) compile in `dbg` mode:
```
bazel build -c dbg //minimip:solver
```
See the [bazel doc](https://docs.bazel.build/versions/master/user-manual.html#flag--compilation_mode)
for more details.

## CMake
We also provide a `CMakeLists.txt` file for building the project with CMake.
CMake **3.25 or newer**, a C++17 compiler, and Git are required. On Debian/Ubuntu,
install the build tools and check the available CMake version:
```
sudo apt update
sudo apt install cmake build-essential git
cmake --version
```

From the repository root, configure, build, and run the tests:
```
cmake -S . -B build &&
cmake --build build -v -j 12 &&
ctest --test-dir build --output-on-failure
```
The `&&` operators stop the sequence if a step fails. Reduce `-j 12` if the build
uses too much memory.

The first configuration downloads dependencies using CMake's `FetchContent`.
Cloning large repositories such as OR-Tools can take several minutes; subsequent
configurations reuse the downloads in the build directory.

## Dependencies
See `WORKSPACE` or `CMakeLists.txt` file for a list of dependencies and extra information. These
dependencies are automatically downloaded by Bazel or CMake during configuration.

**Important**
Boost still needs to be installed on the machine for SoPlex to read LP files and be tested.
On Debian machines, the package `libboost-all-dev` covers it.

# Style guide
Google C++ style guide is available at:
https://google.github.io/styleguide/cppguide.html

To automatically format the syntax you can run:
`clang-format -i PATH_TO_FILES/*.h PATH_TO_FILES/*.cc` which will pick up the style specified in `.clang-format`.

Be careful not run clang-format on BUILD files. To format bazel's BUILD files
automatically, run `buildifier` (see https://github.com/bazelbuild/buildtools).

# VSCode setup
There is an official VSCode plugin for bazel called `vscode-bazel`. While it allows context highlights when editing Bazel files (`BUILD`, `WORKSPACE`, etc.) it doesn't configure IntelliSense for code files compiled using Bazel. Instead, use the following steps to get code suggestions using `clangd`. Tested on Linux Mint 20 (Ubuntu 20.04 derivative) on 2022-10-04.

1. Make sure `clangd` is installed on your system
```
sudo apt install clangd
```
2. Download the [Bazel Compilation Database](https://github.com/grailbio/bazel-compilation-database) tool.
3. Go to MiniMIP's root directory (where the `WORKSPACE` file is located) and run the `generate.py` script you just downloaded. When the script has completed (may take several minutes), you should see a `compile_commands.json` file.
4. Install the `vscode-clangd` extension for VSCode.

    Note: The clangd extension is incompatible with Microsoft's IntelliSense, and will ask you to disable it. If you want to control this manually, the setting is found under C/C++: Intelli Sense Engine.
5. In your VSCode settings, find Clangd: Arguments and add `--compile-commands-dir=${workspaceFolder}/` and make sure Clangd: Check Updates is enabled.
6. Restart VSCode.

Your MiniMIP files should now give you contextual highlighting, symbol lookup, clang-tidy warnings and more. You may need to occasionally repeat step 3 above if the information in `compile_commands.json` becomes stale, e.g. if you add a new target or introduce new dependencies.
