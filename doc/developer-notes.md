Developer Notes
===============

Coding Style (General)
----------------------

* ❌NOT very consistent❌
  * Reason: 🧠MULTIPLE coding styles | history of the codebase🧠

* goal
  * 1! style

* | move-only commits,
  *❌NOT require mimic the style❌
* ❌NOT submit patches / ONLY modify the style❌

Coding Style (C++)
------------------

* TODO:
- **Indentation and whitespace rules** as specified in
[src/.clang-format](/src/.clang-format). You can use the provided
[clang-format-diff script](/contrib/devtools/README.md#clang-format-diffpy)
tool to clean up patches automatically before submission.
  - Braces on new lines for classes, functions, methods.
  - Braces on the same line for everything else.
  - 4 space indentation (no tabs) for every block except namespaces.
  - No indentation for `public`/`protected`/`private` or for `namespace`.
  - No extra spaces inside parenthesis; don't do `( this )`.
  - No space after function names; one space after `if`, `for` and `while`.
  - If an `if` only has a single-statement `then`-clause, it can appear
    on the same line as the `if`, without braces. In every other case,
    braces are required, and the `then` and `else` clauses must appear
    correctly indented on a new line.
  - There's no hard limit on line width, but prefer to keep lines to <100
    characters if doing so does not decrease readability. Break up long
    function declarations over multiple lines using the Clang Format
    [AlignAfterOpenBracket](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)
    style option.

- **Symbol naming conventions**. These are preferred in new code, but are not
required when doing so would need changes to significant pieces of existing
code.
  - Variable (including function arguments) and namespace names are all lowercase and may use `_` to
    separate words (snake_case).
    - Class member variables have a `m_` prefix.
    - Global variables have a `g_` prefix.
  - Constant names are all uppercase, and use `_` to separate words.
  - Enumerator constants may be `snake_case`, `PascalCase` or `ALL_CAPS`.
    This is a more tolerant policy than the [C++ Core
    Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-caps),
    which recommend using `snake_case`.  Please use what seems appropriate.
  - Class names, function names, and method names are UpperCamelCase
    (PascalCase). Do not prefix class names with `C`. See [Internal interface
    naming style](#internal-interface-naming-style) for an exception to this
    convention.

  - Test suite naming convention: The Boost test suite in file
    `src/test/foo_tests.cpp` should be named `foo_tests`. Test suite names
    must be unique.

- **Miscellaneous**
  - `++i` is preferred over `i++`.
  - `nullptr` is preferred over `NULL` or `(void*)0`.
  - `static_assert` is preferred over `assert` where possible. Generally; compile-time checking is preferred over run-time checking.
  - Use a named cast or functional cast, not a C-Style cast. When casting
    between integer types, use functional casts such as `int(x)` or `int{x}`
    instead of `(int) x`. When casting between more complex types, use `static_cast`.
    Use `reinterpret_cast` and `const_cast` as appropriate.
  - Prefer [`list initialization ({})`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-list) where possible.
    For example `int x{0};` instead of `int x = 0;` or `int x(0);`
  - Recursion is checked by clang-tidy and thus must be made explicit. Use
    `NOLINTNEXTLINE(misc-no-recursion)` to suppress the check.

For function calls a namespace should be specified explicitly, unless such functions have been declared within it.
Otherwise, [argument-dependent lookup](https://en.cppreference.com/w/cpp/language/adl), also known as ADL, could be
triggered that makes code harder to maintain and reason about:
```c++
#include <filesystem>

namespace fs {
class path : public std::filesystem::path
{
};
// The intention is to disallow this function.
bool exists(const fs::path& p) = delete;
} // namespace fs

int main()
{
    //fs::path p; // error
    std::filesystem::path p; // compiled
    exists(p); // ADL being used for unqualified name lookup
}
```

Block style example:
```c++
int g_count{0};

namespace foo {
class Class
{
    std::string m_name;

public:
    bool Function(const std::string& s, int n)
    {
        // Comment summarising what this section of code does
        for (int i = 0; i < n; ++i) {
            int total_sum{0};
            // When something fails, return early
            if (!Something()) return false;
            ...
            if (SomethingElse(i)) {
                total_sum += ComputeSomething(g_count);
            } else {
                DoSomething(m_name, total_sum);
            }
        }

        // Success return is usually at the end
        return true;
    }
}
} // namespace foo
```

Coding Style (C++ functions and methods)
--------------------

- When ordering function parameters, place input parameters first, then any
  in-out parameters, followed by any output parameters.

- *Rationale*: API consistency.

- Prefer returning values directly to using in-out or output parameters. Use
  `std::optional` where helpful for returning values.

- *Rationale*: Less error-prone (no need for assumptions about what the output
  is initialized to on failure), easier to read, and often the same or better
  performance.

- Generally, use `std::optional` to represent optional by-value inputs (and
  instead of a magic default value, if there is no real default). Non-optional
  input parameters should usually be values or const references, while
  non-optional in-out and output parameters should usually be references, as
  they cannot be null.

Coding Style (C++ named arguments)
------------------------------

When passing named arguments, use a format that clang-tidy understands. The
argument names can otherwise not be verified by clang-tidy.

For example:

```c++
void function(Addrman& addrman, bool clear);

int main()
{
    function(g_addrman, /*clear=*/false);
}
```

### Running clang-tidy

To run clang-tidy on Ubuntu/Debian, install the dependencies:

```sh
apt install clang-tidy clang
```

Configure with clang as the compiler:

```sh
cmake -B build -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

The output is denoised of errors from external dependencies.

To run clang-tidy on all source files:

```sh
( cd ./src/ && run-clang-tidy -p ../build -j $(nproc) )
```

To run clang-tidy on the changed source lines:

```sh
git diff | ( cd ./src/ && clang-tidy-diff -p2 -path ../build -j $(nproc) )
```

Coding Style (Python)
---------------------

Refer to [/test/functional/README.md#style-guidelines](/test/functional/README.md#style-guidelines).


Coding Style (Doxygen-compatible comments)
------------------------------------------

Bitcoin Core uses [Doxygen](https://www.doxygen.nl/) to generate its official documentation.

Use Doxygen-compatible comment blocks for functions, methods, and fields.

For example, to describe a function use:

```c++
/**
 * ... Description ...
 *
 * @param[in]  arg1 input description...
 * @param[in]  arg2 input description...
 * @param[out] arg3 output description...
 * @return Return cases...
 * @throws Error type and cases...
 * @pre  Pre-condition for function...
 * @post Post-condition for function...
 */
bool function(int arg1, const char *arg2, std::string& arg3)
```

A complete list of `@xxx` commands can be found at https://www.doxygen.nl/manual/commands.html.
As Doxygen recognizes the comments by the delimiters (`/**` and `*/` in this case), you don't
*need* to provide any commands for a comment to be valid; just a description text is fine.

To describe a class, use the same construct above the class definition:
```c++
/**
 * Alerts are for notifying old versions if they become too obsolete and
 * need to upgrade. The message is displayed in the status bar.
 * @see GetWarnings()
 */
class CAlert
```

To describe a member or variable use:
```c++
//! Description before the member
int var;
```

or
```c++
int var; //!< Description after the member
```

Also OK:
```c++
///
/// ... Description ...
///
bool function2(int arg1, const char *arg2)
```

Not picked up by Doxygen:
```c++
//
// ... Description ...
//
```

Also not picked up by Doxygen:
```c++
/*
 * ... Description ...
 */
```

A full list of comment syntaxes picked up by Doxygen can be found at https://www.doxygen.nl/manual/docblocks.html,
but the above styles are favored.

Recommendations:

- Avoiding duplicating type and input/output information in function
  descriptions.

- Use backticks (&#96;&#96;) to refer to `argument` names in function and
  parameter descriptions.

- Backticks aren't required when referring to functions Doxygen already knows
  about; it will build hyperlinks for these automatically. See
  https://www.doxygen.nl/manual/autolink.html for complete info.

- Avoid linking to external documentation; links can break.

- Javadoc and all valid Doxygen comments are stripped from Doxygen source code
  previews (`STRIP_CODE_COMMENTS = YES` in [Doxyfile.in](/doc/Doxyfile.in)). If
  you want a comment to be preserved, it must instead use `//` or `/* */`.

### Generating Documentation

Assuming the build directory is named `build`,
the documentation can be generated with `cmake --build build --target docs`.
The resulting files will be located in `build/doc/doxygen/html`;
open `index.html` in that directory to view the homepage.

Before building the `docs` target, you'll need to install these dependencies:

Linux: `sudo apt install doxygen graphviz`

MacOS: `brew install doxygen graphviz`

Development tips and tricks
---------------------------

### Compiling for debugging

* `cmake -B build`
  * == default build configuration
  * -> `-DCMAKE_BUILD_TYPE = RelWithDebInfo`
    * adds debug symbols
    * performs compiler optimizations / debugging trickier
      * Reason: 🧠code may NOT correspond to the source🧠

* if you need to build EXCLUSIVELY for debugging -> set `-DCMAKE_BUILD_TYPE = Debug`

* `ccmake build`
  * check EXISTING build's cmake build options  

### Show sources in debugging

* TODO: If you have ccache enabled, absolute paths are stripped from debug information
with the `-fdebug-prefix-map` and `-fmacro-prefix-map` options (if supported by the
compiler). This might break source file detection in case you move binaries
after compilation, debug from the directory other than the project root or use
an IDE that only supports absolute paths for debugging (e.g. it won't stop at breakpoints).

There are a few possible fixes:

1. Configure source file mapping.

For `gdb` create or append to [`.gdbinit` file](https://sourceware.org/gdb/current/onlinedocs/gdb#gdbinit-man):
```
set substitute-path ./src /path/to/project/root/src
```

For `lldb` create or append to [`.lldbinit` file](https://lldb.llvm.org/man/lldb.html#configuration-files):
```
settings set target.source-map ./src /path/to/project/root/src
```

2. Add a symlink to the `./src` directory:
```
ln -s /path/to/project/root/src src
```

3. Use `debugedit` to modify debug information in the binary.

4. If your IDE has an option for this, change your breakpoints to use the file name only.

### `debug.log`

* stored | data directory
* write
  * error & debugging messages 

* ways to enable
  * | startup,
    * -- via --
      * `-debug` (or `-debug=1`)
        * turn on ALL log categories 
      * `-loglevel` 
        * if `-loglevel=trace` -> turn on ALL log severity levels
  * | bitcoind is running,
    * toggle it -- via -- `logging` RPC

* Qt code
  * routes `qDebug()` output -- to -- "debug.log"'s category "qt"
    * if you want to see it -> run -- with -- `-debug=qt`

### Signet, testnet, and regtest modes

* TODO: If you are testing multi-machine code that needs to operate across the internet,
you can run with either the `-signet` or the `-testnet4` config option to test
with "play bitcoins" on a test network.

If you are testing something that can run on one machine, run with the
`-regtest` option.  In regression test mode, blocks can be created on demand;
see [test/functional/](/test/functional) for tests that run in `-regtest` mode.

### DEBUG_LOCKORDER

Bitcoin Core is a multi-threaded application, and deadlocks or other
multi-threading bugs can be very difficult to track down. The `-DCMAKE_BUILD_TYPE=Debug`
build option adds `-DDEBUG_LOCKORDER` to the compiler flags. This inserts
run-time checks to keep track of which locks are held and adds warnings to the
`debug.log` file if inconsistencies are detected.

### DEBUG_LOCKCONTENTION

Defining `DEBUG_LOCKCONTENTION` adds a "lock" logging category to the logging
RPC that, when enabled, logs the location and duration of each lock contention
to the `debug.log` file.

The `-DCMAKE_BUILD_TYPE=Debug` build option adds `-DDEBUG_LOCKCONTENTION` to the
compiler flags. You may also enable it manually by building with `-DDEBUG_LOCKCONTENTION`
added to your CPPFLAGS, i.e. `-DAPPEND_CPPFLAGS="-DDEBUG_LOCKCONTENTION"`.

You can then use the `-debug=lock` configuration option at bitcoind startup or
`bitcoin-cli logging '["lock"]'` at runtime to turn on lock contention logging.
It can be toggled off again with `bitcoin-cli logging [] '["lock"]'`.

### Assertions and Checks

The util file `src/util/check.h` offers helpers to protect against coding and
internal logic bugs. They must never be used to validate user, network or any
other input.

* `assert` or `Assert` should be used to document assumptions when any
  violation would mean that it is not safe to continue program execution. The
  code is always compiled with assertions enabled.
   - For example, a nullptr dereference or any other logic bug in validation
     code means the program code is faulty and must terminate immediately.
* `CHECK_NONFATAL` should be used for recoverable internal logic bugs. On
  failure, it will throw an exception, which can be caught to recover from the
  error.
   - For example, a nullptr dereference or any other logic bug in RPC code
     means that the RPC code is faulty and cannot be executed. However, the
     logic bug can be shown to the user and the program can continue to run.
* `Assume` should be used to document assumptions when program execution can
  safely continue even if the assumption is violated. In debug builds it
  behaves like `Assert`/`assert` to notify developers and testers about
  nonfatal errors. In production it doesn't warn or log anything, though the
  expression is always evaluated. However, if the compiler can prove that
  an expression inside `Assume` is side-effect-free, it may optimize the call away,
  skipping its evaluation in production. This enables a lower-cost way of
  making explicit statements about the code, aiding review.
   - For example it can be assumed that a variable is only initialized once,
     but a failed assumption does not result in a fatal bug. A failed
     assumption may or may not result in a slightly degraded user experience,
     but it is safe to continue program execution.

### Valgrind suppressions file

Valgrind is a programming tool for memory debugging, memory leak detection, and
profiling. The repo contains a Valgrind suppressions file
([`valgrind.supp`](https://github.com/bitcoin/bitcoin/blob/master/contrib/valgrind.supp))
which includes known Valgrind warnings in our dependencies that cannot be fixed
in-tree. Example use:

```shell
$ valgrind --suppressions=contrib/valgrind.supp build/bin/test_bitcoin
$ valgrind --suppressions=contrib/valgrind.supp --leak-check=full \
      --show-leak-kinds=all build/bin/test_bitcoin --log_level=test_suite
$ valgrind -v --leak-check=full build/bin/bitcoind -printtoconsole
$ ./build/test/functional/test_runner.py --valgrind
```

### Compiling for test coverage

#### Using LCOV

LCOV can be used to generate a test coverage report based upon `ctest`
execution. LCOV must be installed on your system (e.g. the `lcov` package
on Debian/Ubuntu).

To enable LCOV report generation during test runs:

```shell
cmake -B build -DCMAKE_BUILD_TYPE=Coverage
cmake --build build
cmake -P build/Coverage.cmake

# A coverage report will now be accessible at `./build/test_bitcoin.coverage/index.html`,
# which covers unit tests, and `./build/total.coverage/index.html`, which covers
# unit and functional tests.
```

Additional LCOV options can be specified using `LCOV_OPTS`, but may be dependent
on the version of LCOV. For example, when using LCOV `2.x`, branch coverage can be
enabled by setting `LCOV_OPTS="--rc branch_coverage=1"`:

```
cmake -DLCOV_OPTS="--rc branch_coverage=1" -P build/Coverage.cmake
```

To enable test parallelism:
```
cmake -DJOBS=$(nproc) -P build/Coverage.cmake
```

#### Using LLVM/Clang toolchain

The following generates a coverage report for unit tests and functional tests.

Configure the build with the following flags:

> Consider building with a clean state using `rm -rf build`

```shell
# MacOS may instead require `-DCMAKE_C_COMPILER="$(brew --prefix llvm)/bin/clang" -DCMAKE_CXX_COMPILER="$(brew --prefix llvm)/bin/clang++"`
cmake -B build -DCMAKE_C_COMPILER="clang" \
   -DCMAKE_CXX_COMPILER="clang++" \
   -DAPPEND_CFLAGS="-fprofile-instr-generate -fcoverage-mapping" \
   -DAPPEND_CXXFLAGS="-fprofile-instr-generate -fcoverage-mapping" \
   -DAPPEND_LDFLAGS="-fprofile-instr-generate -fcoverage-mapping"
cmake --build build # Use "-j N" here for N parallel jobs.
```

Generating the raw profile data based on `ctest` and functional tests execution:

```shell
# Create directory for raw profile data
mkdir -p build/raw_profile_data

# Run tests to generate profiles
LLVM_PROFILE_FILE="$(pwd)/build/raw_profile_data/%m_%p.profraw" ctest --test-dir build # Use "-j N" here for N parallel jobs.
LLVM_PROFILE_FILE="$(pwd)/build/raw_profile_data/%m_%p.profraw" build/test/functional/test_runner.py # Use "-j N" here for N parallel jobs

# Merge all the raw profile data into a single file
find build/raw_profile_data -name "*.profraw" | xargs llvm-profdata merge -o build/coverage.profdata
```

> **Note:** The "counter mismatch" warning can be safely ignored, though it can be resolved by updating to Clang 19.
> The warning occurs due to version mismatches but doesn't affect the coverage report generation.

Generating the coverage report:

```shell
llvm-cov show \
    --object=build/bin/test_bitcoin \
    --object=build/bin/bitcoind \
    -Xdemangler=llvm-cxxfilt \
    --instr-profile=build/coverage.profdata \
    --ignore-filename-regex="src/crc32c/|src/leveldb/|src/minisketch/|src/secp256k1/|src/test/" \
    --format=html \
    --show-instantiation-summary \
    --show-line-counts-or-regions \
    --show-expansions \
    --output-dir=build/coverage_report \
    --project-title="Bitcoin Core Coverage Report"
```

> **Note:** The "functions have mismatched data" warning can be safely ignored, the coverage report will still be generated correctly despite this warning.
> This warning occurs due to profdata mismatch created during the merge process for shared libraries.

The generated coverage report can be accessed at `build/coverage_report/index.html`.

#### Compiling for Fuzz Coverage

```shell
cmake -B build \
   -DCMAKE_C_COMPILER="clang" \
   -DCMAKE_CXX_COMPILER="clang++" \
   -DCMAKE_C_FLAGS="-fprofile-instr-generate -fcoverage-mapping" \
   -DCMAKE_CXX_FLAGS="-fprofile-instr-generate -fcoverage-mapping" \
   -DBUILD_FOR_FUZZING=ON
cmake --build build # Use "-j N" here for N parallel jobs.
```

Running fuzz tests with one or more targets

```shell
# For single target run with the target of choice
LLVM_PROFILE_FILE="$(pwd)/build/raw_profile_data/txorphan.profraw" ./build/test/fuzz/test_runner.py ../qa-assets/fuzz_corpora txorphan
# If running for multiple targets
LLVM_PROFILE_FILE="$(pwd)/build/raw_profile_data/%m_%p.profraw" ./build/test/fuzz/test_runner.py ../qa-assets/fuzz_corpora
# Merge profiles
llvm-profdata merge build/raw_profile_data/*.profraw -o build/coverage.profdata
```

Generate report:

```shell
llvm-cov show \
    --object=build/bin/fuzz \
    -Xdemangler=llvm-cxxfilt \
    --instr-profile=build/coverage.profdata \
    --ignore-filename-regex="src/crc32c/|src/leveldb/|src/minisketch/|src/secp256k1/|src/test/" \
    --format=html \
    --show-instantiation-summary \
    --show-line-counts-or-regions \
    --show-expansions \
    --output-dir=build/coverage_report \
    --project-title="Bitcoin Core Fuzz Coverage Report"
```

The generated coverage report can be accessed at `build/coverage_report/index.html`.

### Performance profiling with perf

Profiling is a good way to get a precise idea of where time is being spent in
code. One tool for doing profiling on Linux platforms is called
[`perf`](https://www.brendangregg.com/perf.html), and has been integrated into
the functional test framework. Perf can observe a running process and sample
(at some frequency) where its execution is.

Perf installation is contingent on which kernel version you're running; see
[this thread](https://askubuntu.com/questions/50145/how-to-install-perf-monitoring-tool)
for specific instructions.

Certain kernel parameters may need to be set for perf to be able to inspect the
running process's stack.

```sh
$ sudo sysctl -w kernel.perf_event_paranoid=-1
$ sudo sysctl -w kernel.kptr_restrict=0
```

Make sure you [understand the security
trade-offs](https://lwn.net/Articles/420403/) of setting these kernel
parameters.

To profile a running bitcoind process for 60 seconds, you could use an
invocation of `perf record` like this:

```sh
$ perf record \
    -g --call-graph dwarf --per-thread -F 140 \
    -p `pgrep bitcoind` -- sleep 60
```

You could then analyze the results by running:

```sh
perf report --stdio | c++filt | less
```

or using a graphical tool like [Hotspot](https://github.com/KDAB/hotspot).

See the functional test documentation for how to invoke perf within tests.


### Sanitizers

Bitcoin Core can be compiled with various "sanitizers" enabled, which add
instrumentation for issues regarding things like memory safety, thread race
conditions, or undefined behavior. This is controlled with the
`-DSANITIZERS` cmake build flag, which should be a comma separated list of
sanitizers to enable. The sanitizer list should correspond to supported
`-fsanitize=` options in your compiler. These sanitizers have runtime overhead,
so they are most useful when testing changes or producing debugging builds.

Some examples:

```bash
# Enable both the address sanitizer and the undefined behavior sanitizer
cmake -B build -DSANITIZERS=address,undefined

# Enable the thread sanitizer
cmake -B build -DSANITIZERS=thread
```

If you are compiling with GCC you will typically need to install corresponding
"san" libraries to actually compile with these flags, e.g. libasan for the
address sanitizer, libtsan for the thread sanitizer, and libubsan for the
undefined sanitizer. If you are missing required libraries, the build
will fail with a linker error when testing the sanitizer flags.

The test suite should pass cleanly with the `thread` and `undefined` sanitizers. You
may need to use a suppressions file, see `test/sanitizer_suppressions`. They may be
used as follows:
```bash
export LSAN_OPTIONS="suppressions=$(pwd)/test/sanitizer_suppressions/lsan"
export TSAN_OPTIONS="suppressions=$(pwd)/test/sanitizer_suppressions/tsan:halt_on_error=1:second_deadlock_stack=1"
export UBSAN_OPTIONS="suppressions=$(pwd)/test/sanitizer_suppressions/ubsan:print_stacktrace=1:halt_on_error=1:report_error_type=1"
```

See the CI config for more examples, and upstream documentation for more information
about any additional options.

Not all sanitizer options can be enabled at the same time, e.g. trying to build
with `-DSANITIZERS=address,thread` will fail in the build as
these sanitizers are mutually incompatible. Refer to your compiler manual to
learn more about these options and which sanitizers are supported by your
compiler.

Additional resources:

 * [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)
 * [LeakSanitizer](https://clang.llvm.org/docs/LeakSanitizer.html)
 * [MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html)
 * [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html)
 * [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)
 * [GCC Instrumentation Options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
 * [Google Sanitizers Wiki](https://github.com/google/sanitizers/wiki)

Locking/mutex usage notes
-------------------------

* code
  * 💡multi-threaded💡
  * macros protect data structures -- via -- mutexes & macros (`LOCK` & `TRY_LOCK`) 

* Deadlocks -- due to -- inconsistent lock ordering
  * == 
    * thread 1 locks `cs_main` & `cs_wallet`
    * thread 2 locks them | ⚠️opposite order⚠️
  * ⚠️problem⚠️
    * Reason:🧠EACH thread waits for the other to release its lock🧠
  * if you want lock order inconsistencies are reported | "debug.log" -> compile -- with -- 
    * `-DDEBUG_LOCKORDER` OR
    * `-DCMAKE_BUILD_TYPE=Debug` 

* goal
  * re-architecting the core code / 
    * BETTER-defined interfaces -- BETWEEN -- VARIOUS components
    * ANY necessary locking done -- by the -- components
      * _Example:_ `FillableSigningProvider` & `cs_KeyStore`

Threads
-------

- [Main thread (`bitcoind`)](https://doxygen.bitcoincore.org/bitcoind_8cpp.html#a0ddf1224851353fc92bfbff6f499fa97)
  - started -- from -- "bitcoind.cpp"'s `main()`
  - responsible for 
    - starting up the application
    - shutting down the application

- [Init load (`b-initload`)](https://doxygen.bitcoincore.org/namespacenode.html#ab4305679079866f0f420f7dbf278381d)
  - performs VARIOUS loading tasks /
    - part of init
    - should NOT block the node -- from -- being started
  - _Example:_
    - external block import,
    - reindex,
    - reindex-chainstate,
    - main chain activation,
    - spawn indexes background sync threads
    - mempool load

- [CCheckQueue::Loop (`b-scriptch.x`)](https://doxygen.bitcoincore.org/class_c_check_queue.html#a6e7fa51d3a25e7cb65446d4b50e6a987)
  - parallel script validation threads -- for -- transactions | blocks

- [ThreadHTTP (`b-http`)](https://doxygen.bitcoincore.org/httpserver_8cpp.html#abb9f6ea8819672bd9a62d3695070709c)
  - Libevent thread /
    - listen for connections
      - RPC
      - REST 

- [HTTP worker threads(`b-httpworker.x`)](https://doxygen.bitcoincore.org/httpserver_8cpp.html#aa6a7bc27265043bc0193220c5ae3a55f)
  - serve RPC & REST requests

- [Indexer threads (`b-txindex`, etc)](https://doxygen.bitcoincore.org/class_base_index.html#a96a7407421fbf877509248bbe64f8d87)
  - 1 thread / indexer

- [SchedulerThread (`b-scheduler`)](https://doxygen.bitcoincore.org/class_c_scheduler.html#a14d2800815da93577858ea078aed1fba)
  - makes asynchronous background tasks
  - _Example:_
    - dumping 
      - wallet contents,
      - addrman
    - running asynchronous validation interface callbacks

- [TorControlThread (`b-torcontrol`)](https://doxygen.bitcoincore.org/torcontrol_8cpp.html#a52a3efff23634500bb42c6474f306091)
  - Libevent thread -- for -- tor connections

- NET threads
  - [ThreadMessageHandler (`b-msghand`)](https://doxygen.bitcoincore.org/class_c_connman.html#aacdbb7148575a31bb33bc345e2bf22a9)
    - application level message handling (sending & receiving)
    - ALL net_processing & validation logic
  - [ThreadDNSAddressSeed (`b-dnsseed`)](https://doxygen.bitcoincore.org/class_c_connman.html#aa7c6970ed98a4a7bafbc071d24897d13)
    - loads addresses of peers -- from --- DNS
  - ThreadMapPort (`b-mapport`)
    - universal plug-and-play startup/shutdown.
  - [ThreadSocketHandler (`b-net`)](https://doxygen.bitcoincore.org/class_c_connman.html#a765597cbfe99c083d8fa3d61bb464e34)
    - sends/Receives data -- from -- peers | port 8333
  - [ThreadOpenAddedConnections (`b-addcon`)](https://doxygen.bitcoincore.org/class_c_connman.html#a0b787caf95e52a346a2b31a580d60a62)
    - opens network connections -- to -- added nodes
  - [ThreadOpenConnections (`b-opencon`)](https://doxygen.bitcoincore.org/class_c_connman.html#a55e9feafc3bab78e5c9d408c207faa45)
    - initiates NEW connections -- to -- peers
  - [ThreadI2PAcceptIncoming (`b-i2paccept`)](https://doxygen.bitcoincore.org/class_c_connman.html#a57787b4f9ac847d24065fbb0dd6e70f8)
    - I2P connections -- through the -- I2P SAM proxy
      - listens for 
      - accepts incoming 

Ignoring IDE/editor files
--------------------------

* | closed-source environments / everyone uses the SAME IDE,
  * recommended
    * add TEMPORARY files | ".gitignore"

* | Bitcoin Core,
  * NOT common to use them
    * Reason:🧠EACH one uses their OWN editors/IDE/tools🧠
  * recommendations
    * create your local ".gitignore" & add it
      * ways
          ```.gitconfig
          [core]
            excludesfile = /home/.../.gitignore_global
          ```
          ```
          git config --global core.excludesfile ~/.gitignore_global
          ```
    * create a excludes file ".git/info/exclude"
    * if set of tools is used by the build system OR scripts the repository -> add its files | `.gitignore`

Development guidelines
============================

General Bitcoin Core
----------------------

- NEW features
  - expose it | RPC
    - FIRSTLY
      - Reason: 🧠
        - better automatic testing
        - GUI's test suite: VERY limited🧠 
  - AVAILABLE | GUI

- | BEFORE merge PR, pass CI

- explanation
  - if the test suite is updated for a change -> it must be done first

Logging
-------

The macros `LogInfo`, `LogDebug`, `LogTrace`, `LogWarning` and `LogError` are available for
logging messages. They should be used as follows:

- `LogDebug(BCLog::CATEGORY, fmt, params...)` is what you want
  most of the time, and it should be used for log messages that are
  useful for debugging and can reasonably be enabled on a production
  system (that has sufficient free storage space). They will be logged
  if the program is started with `-debug=category` or `-debug=1`.

- `LogInfo(fmt, params...)` should only be used rarely, e.g. for startup
  messages or for infrequent and important events such as a new block tip
  being found or a new outbound connection being made. These log messages
  are unconditional, so care must be taken that they can't be used by an
  attacker to fill up storage. Note that `LogPrintf(fmt, params...)` is
  a deprecated alias for `LogInfo`.

- `LogError(fmt, params...)` should be used in place of `LogInfo` for
  severe problems that require the node (or a subsystem) to shut down
  entirely (e.g., insufficient storage space).

- `LogWarning(fmt, params...)` should be used in place of `LogInfo` for
  severe problems that the node admin should address, but are not
  severe enough to warrant shutting down the node (e.g., system time
  appears to be wrong, unknown soft fork appears to have activated).

- `LogTrace(BCLog::CATEGORY, fmt, params...)` should be used in place of
  `LogDebug` for log messages that would be unusable on a production
  system, e.g. due to being too noisy in normal use, or too resource
  intensive to process. These will be logged if the startup
  options `-debug=category -loglevel=category:trace` or `-debug=1
  -loglevel=trace` are selected.

Note that the format strings and parameters of `LogDebug` and `LogTrace`
are only evaluated if the logging category is enabled, so you must be
careful to avoid side-effects in those expressions.

Wallet
-------

- `-disablewallet`
  - == run-time option /
    - make sure NO crashes happen  

General C++
-------------

* [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/)
  * [passing (non-)fundamental types](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional)

- if you use `.h` -> link the `.cpp`
  - Reason:🧠if you do NOT link -> confusing🧠
- if you move functions FROM `.h` -- to -- `.cpp` -> should NOT result build errors
- if possible -> use RAII (Resource Acquisition Is Initialization)
  - _Example:_ use `unique_ptr` for allocations | function
  - Reason:🧠
    - avoids memory & resource leaks
    - ensures exception safety🧠

C++ data structures
--------------------

- | read from a map,
  - ❌NEVER use `std::map []`❌
  - use `.find()`
  - Reason:🧠 `[]`
    - caused 
      - memory leaks 
      - race conditions
    - write -- to a -- map

- ❌NOT compare data structure1's iterator vs data structure2's iterator❌
  - Reason:🧠 behavior is undefined 🧠

- watch out for out-of-bounds vector access
  - `&vch[vch.size()]`
    - illegal
    - include `&vch[0]` -- for -- EMPTY vector
  - recommendations
    - use 
      - `vch.data()`
      - `vch.data() + vch.size()`

- vector bounds checking
  - ONLY enabled | debug mode
  - ❌NOT rely on it❌

- | define NON-static class members, initialize ALL   
  - if it's skipped for a good reason -> add an explicit comment
  - Reason:🧠
    - avoid uninitialized values
    - easy to spot uninitialized ones🧠

  ```cpp
  class A
  {
      uint32_t m_count{0};
  }
  ```

- by default, declare constructors `explicit`
  - Reason:🧠avoid unintended [conversions](https://en.cppreference.com/w/cpp/language/converting_constructor)🧠

- use explicitly 
  - signed or unsigned `char`s OR
  - bare `char`
  -  | ONLY pass -- to a -- third-party API -> use  
  - `uint8_t` & `int8_t`

-  use explicit constructions vs implicit ones / rely on C++ behavior
  - Reason:🧠 easier to understand🧠

- Use `std::span` as function argument when it can operate on any range-like container.

  - *Rationale*: Compared to `Foo(const vector<int>&)` this avoids the need for a (potentially expensive)
    conversion to vector if the caller happens to have the input stored in another type of container.
    However, be aware of the pitfalls documented in [span.h](../src/span.h).

```cpp
void Foo(std::span<const int> data);

std::vector<int> vec{1,2,3};
Foo(vec);
```

- Prefer `enum class` (scoped enumerations) over `enum` (traditional enumerations) where possible.

  - *Rationale*: Scoped enumerations avoid two potential pitfalls/problems with traditional C++ enumerations: implicit conversions to `int`, and name clashes due to enumerators being exported to the surrounding scope.

- `switch` statement on an enumeration example:

```cpp
enum class Tabs {
    info,
    console,
    network_graph,
    peers
};

int GetInt(Tabs tab)
{
    switch (tab) {
    case Tabs::info: return 0;
    case Tabs::console: return 1;
    case Tabs::network_graph: return 2;
    case Tabs::peers: return 3;
    } // no default case, so the compiler can warn about missing cases
    assert(false);
}
```

*Rationale*: The comment documents skipping `default:` label, and it complies with `clang-format` rules. The assertion prevents firing of `-Wreturn-type` warning on some compilers.

Strings and formatting
------------------------

- Use `std::string`, avoid C string manipulation functions.

  - *Rationale*: C++ string handling is marginally safer, less scope for
    buffer overflows, and surprises with `\0` characters. Also, some C string manipulations
    tend to act differently depending on platform, or even the user locale.

- For `strprintf`, `LogInfo`, `LogDebug`, etc formatting characters don't need size specifiers.

  - *Rationale*: Bitcoin Core uses tinyformat, which is type safe. Leave them out to avoid confusion.

- Use `.c_str()` sparingly. Its only valid use is to pass C++ strings to C functions that take NULL-terminated
  strings.

  - Do not use it when passing a sized array (so along with `.size()`). Use `.data()` instead to get a pointer
    to the raw data.

    - *Rationale*: Although this is guaranteed to be safe starting with C++11, `.data()` communicates the intent better.

  - Do not use it when passing strings to `tfm::format`, `strprintf`, `LogInfo`, `LogDebug`, etc.

    - *Rationale*: This is redundant. Tinyformat handles strings.

  - Do not use it to convert to `QString`. Use `QString::fromStdString()`.

    - *Rationale*: Qt has built-in functionality for converting their string
      type from/to C++. No need to roll your own.

  - In cases where you do call `.c_str()`, you might want to additionally check that the string does not contain embedded '\0' characters, because
    it will (necessarily) truncate the string. This might be used to hide parts of the string from logging or to circumvent
    checks. If a use of strings is sensitive to this, take care to check the string for embedded NULL characters first
    and reject it if there are any (see `ParsePrechecks` in `strencodings.cpp` for an example).

Shadowing
--------------

Although the shadowing warning (`-Wshadow`) is not enabled by default (it prevents issues arising
from using a different variable with the same name),
please name variables so that their names do not shadow variables defined in the source code.

When using nested cycles, do not name the inner cycle variable the same as in
the outer cycle, etc.

Lifetimebound
--------------

The [Clang `lifetimebound`
attribute](https://clang.llvm.org/docs/AttributeReference.html#lifetimebound)
can be used to tell the compiler that a lifetime is bound to an object and
potentially see a compile-time warning if the object has a shorter lifetime from
the invalid use of a temporary. You can use the attribute by adding a `LIFETIMEBOUND`
annotation defined in `src/attributes.h`; please grep the codebase for examples.

Threads and synchronization
----------------------------

- Prefer `Mutex` type to `RecursiveMutex` one.

- Consistently use [Clang Thread Safety Analysis](https://clang.llvm.org/docs/ThreadSafetyAnalysis.html) annotations to
  get compile-time warnings about potential race conditions or deadlocks in code.

  - In functions that are declared separately from where they are defined, the
    thread safety annotations should be added exclusively to the function
    declaration. Annotations on the definition could lead to false positives
    (lack of compile failure) at call sites between the two.

  - Prefer locks that are in a class rather than global, and that are
    internal to a class (private or protected) rather than public.

  - Combine annotations in function declarations with run-time asserts in
    function definitions (`AssertLockNotHeld()` can be omitted if `LOCK()` is
    called unconditionally after it because `LOCK()` does the same check as
    `AssertLockNotHeld()` internally, for non-recursive mutexes):

```C++
// txmempool.h
class CTxMemPool
{
public:
    ...
    mutable RecursiveMutex cs;
    ...
    void UpdateTransactionsFromBlock(...) EXCLUSIVE_LOCKS_REQUIRED(::cs_main, cs);
    ...
}

// txmempool.cpp
void CTxMemPool::UpdateTransactionsFromBlock(...)
{
    AssertLockHeld(::cs_main);
    AssertLockHeld(cs);
    ...
}
```

```C++
// validation.h
class Chainstate
{
protected:
    ...
    Mutex m_chainstate_mutex;
    ...
public:
    ...
    bool ActivateBestChain(
        BlockValidationState& state,
        std::shared_ptr<const CBlock> pblock = nullptr)
        EXCLUSIVE_LOCKS_REQUIRED(!m_chainstate_mutex)
        LOCKS_EXCLUDED(::cs_main);
    ...
    bool PreciousBlock(BlockValidationState& state, CBlockIndex* pindex)
        EXCLUSIVE_LOCKS_REQUIRED(!m_chainstate_mutex)
        LOCKS_EXCLUDED(::cs_main);
    ...
}

// validation.cpp
bool Chainstate::PreciousBlock(BlockValidationState& state, CBlockIndex* pindex)
{
    AssertLockNotHeld(m_chainstate_mutex);
    AssertLockNotHeld(::cs_main);
    {
        LOCK(cs_main);
        ...
    }

    return ActivateBestChain(state, std::shared_ptr<const CBlock>());
}
```

- Build and run tests with `-DDEBUG_LOCKORDER` to verify that no potential
  deadlocks are introduced. This is defined by default when
  building with `-DCMAKE_BUILD_TYPE=Debug`.

- When using `LOCK`/`TRY_LOCK` be aware that the lock exists in the context of
  the current scope, so surround the statement and the code that needs the lock
  with braces.

  OK:
```c++
{
    TRY_LOCK(cs_vNodes, lockNodes);
    ...
}
```

  Wrong:
```c++
TRY_LOCK(cs_vNodes, lockNodes);
{
    ...
}
```

Scripts
--------------------------

Write scripts in Python rather than bash, when possible.

### Shebang

- Use `#!/usr/bin/env bash` instead of obsolete `#!/bin/bash`.

  - [*Rationale*](https://github.com/dylanaraps/pure-bash-bible#shebang):

    `#!/bin/bash` assumes it is always installed to /bin/ which can cause issues;

    `#!/usr/bin/env bash` searches the user's PATH to find the bash binary.

  OK:
```bash
#!/usr/bin/env bash
```

  Wrong:
```bash
#!/bin/bash
```

Source code organization
--------------------------

- Implementation code should go into the `.cpp` file and not the `.h`, unless necessary due to template usage or
  when performance due to inlining is critical.

  - *Rationale*: Shorter and simpler header files are easier to read and reduce compile time.

- Use only the lowercase alphanumerics (`a-z0-9`), underscore (`_`) and hyphen (`-`) in source code filenames.

  - *Rationale*: `grep`:ing and auto-completing filenames is easier when using a consistent
    naming pattern. Potential problems when building on case-insensitive filesystems are
    avoided when using only lowercase characters in source code filenames.

- Every `.cpp` and `.h` file should `#include` every header file it directly uses classes, functions or other
  definitions from, even if those headers are already included indirectly through other headers.

  - *Rationale*: Excluding headers because they are already indirectly included results in compilation
    failures when those indirect dependencies change. Furthermore, it obscures what the real code
    dependencies are.

- Don't import anything into the global namespace (`using namespace ...`). Use
  fully specified types such as `std::string`.

  - *Rationale*: Avoids symbol conflicts.

- Terminate namespaces with a comment (`// namespace mynamespace`). The comment
  should be placed on the same line as the brace closing the namespace, e.g.

```c++
namespace mynamespace {
...
} // namespace mynamespace

namespace {
...
} // namespace
```

  - *Rationale*: Avoids confusion about the namespace context.

- Use `#include <primitives/transaction.h>` bracket syntax instead of
  `#include "primitives/transactions.h"` quote syntax.

  - *Rationale*: Bracket syntax is less ambiguous because the preprocessor
    searches a fixed list of include directories without taking location of the
    source file into account. This allows quoted includes to stand out more when
    the location of the source file actually is relevant.

- Use include guards to avoid the problem of double inclusion. The header file
  `foo/bar.h` should use the include guard identifier `BITCOIN_FOO_BAR_H`, e.g.

```c++
#ifndef BITCOIN_FOO_BAR_H
#define BITCOIN_FOO_BAR_H
...
#endif // BITCOIN_FOO_BAR_H
```

GUI
-----

- Do not display or manipulate dialogs in model code (classes `*Model`).

  - *Rationale*: Model classes pass through events and data from the core, they
    should not interact with the user. That's where View classes come in. The converse also
    holds: try to not directly access core data structures from Views.

- Avoid adding slow or blocking code in the GUI thread. In particular, do not
  add new `interfaces::Node` and `interfaces::Wallet` method calls, even if they
  may be fast now, in case they are changed to lock or communicate across
  processes in the future.

  Prefer to offload work from the GUI thread to worker threads (see
  `RPCExecutor` in console code as an example) or take other steps (see
  https://doc.qt.io/archives/qq/qq27-responsive-guis.html) to keep the GUI
  responsive.

  - *Rationale*: Blocking the GUI thread can increase latency, and lead to
    hangs and deadlocks.

Subtrees
----------

Several parts of the repository are subtrees of software maintained elsewhere.

Normally, these are maintained by active developers of Bitcoin Core, in which case
changes should go directly upstream without being PRed directly against the project.
They will be merged back in the next subtree merge.

Others are external projects without a tight relationship with our project. Changes
to these should also be sent upstream, but bugfixes may also be prudent to PR against
a Bitcoin Core subtree, so that they can be integrated quickly. Cosmetic changes
should be taken upstream.

There is a tool in `test/lint/git-subtree-check.sh` ([instructions](../test/lint#git-subtree-checksh))
to check a subtree directory for consistency with its upstream repository.

The tool instructions also include a list of the subtrees managed by Bitcoin Core.

The ultimate upstream of the few externally managed subtrees are:

- src/leveldb
  - Upstream at https://github.com/google/leveldb ; maintained by Google. Open
    important PRs to the Bitcoin Core subtree to avoid delay.
  - **Note**: Follow the instructions in [Upgrading LevelDB](#upgrading-leveldb) when
    merging upstream changes to the LevelDB subtree.

- src/crc32c
  - Used by leveldb for hardware acceleration of CRC32C checksums for data integrity.
  - Upstream at https://github.com/google/crc32c ; maintained by Google.

Upgrading LevelDB
---------------------

Extra care must be taken when upgrading LevelDB. This section explains issues
you must be aware of.

### File Descriptor Counts

In most configurations, we use the default LevelDB value for `max_open_files`,
which is 1000 at the time of this writing. If LevelDB actually uses this many
file descriptors, it will cause problems with Bitcoin's `select()` loop, because
it may cause new sockets to be created where the fd value is >= 1024. For this
reason, on 64-bit Unix systems, we rely on an internal LevelDB optimization that
uses `mmap()` + `close()` to open table files without actually retaining
references to the table file descriptors. If you are upgrading LevelDB, you must
sanity check the changes to make sure that this assumption remains valid.

In addition to reviewing the upstream changes in `env_posix.cc`, you can use `lsof` to
check this. For example, on Linux this command will show open `.ldb` file counts:

```bash
$ lsof -p $(pidof bitcoind) |\
    awk 'BEGIN { fd=0; mem=0; } /ldb$/ { if ($4 == "mem") mem++; else fd++ } END { printf "mem = %s, fd = %s\n", mem, fd}'
mem = 119, fd = 0
```

The `mem` value shows how many files are mmap'ed, and the `fd` value shows you
many file descriptors these files are using. You should check that `fd` is a
small number (usually 0 on 64-bit hosts).

See the notes in the `SetMaxOpenFiles()` function in `dbwrapper.cc` for more
details.

### Consensus Compatibility

It is possible for LevelDB changes to inadvertently change consensus
compatibility between nodes. This happened in Bitcoin 0.8 (when LevelDB was
first introduced). When upgrading LevelDB, you should review the upstream changes
to check for issues affecting consensus compatibility.

For example, if LevelDB had a bug that accidentally prevented a key from being
returned in an edge case, and that bug was fixed upstream, the bug "fix" would
be an incompatible consensus change. In this situation, the correct behavior
would be to revert the upstream fix before applying the updates to Bitcoin's
copy of LevelDB. In general, you should be wary of any upstream changes affecting
what data is returned from LevelDB queries.

Scripted diffs
--------------

For reformatting and refactoring commits where the changes can be easily automated using a bash script, we use
scripted-diff commits. The bash script is included in the commit message and our CI job checks that
the result of the script is identical to the commit. This aids reviewers since they can verify that the script
does exactly what it is supposed to do. It is also helpful for rebasing (since the same script can just be re-run
on the new master commit).

To create a scripted-diff:

- start the commit message with `scripted-diff:` (and then a description of the diff on the same line)
- in the commit message include the bash script between lines containing just the following text:
    - `-BEGIN VERIFY SCRIPT-`
    - `-END VERIFY SCRIPT-`

The scripted-diff is verified by the tool `test/lint/commit-script-check.sh`. The tool's default behavior, when supplied
with a commit is to verify all scripted-diffs from the beginning of time up to said commit. Internally, the tool passes
the first supplied argument to `git rev-list --reverse` to determine which commits to verify script-diffs for, ignoring
commits that don't conform to the commit message format described above.

For development, it might be more convenient to verify all scripted-diffs in a range `A..B`, for example:

```bash
test/lint/commit-script-check.sh origin/master..HEAD
```

### Suggestions and examples

If you need to replace in multiple files, prefer `git ls-files` to `find` or globbing, and `git grep` to `grep`, to
avoid changing files that are not under version control.

For efficient replacement scripts, reduce the selection to the files that potentially need to be modified, so for
example, instead of a blanket `git ls-files src | xargs sed -i s/apple/orange/`, use
`git grep -l apple src | xargs sed -i s/apple/orange/`.

Also, it is good to keep the selection of files as specific as possible — for example, replace only in directories where
you expect replacements — because it reduces the risk that a rebase of your commit by re-running the script will
introduce accidental changes.

Some good examples of scripted-diff:

- [scripted-diff: Rename InitInterfaces to NodeContext](https://github.com/bitcoin/bitcoin/commit/301bd41a2e6765b185bd55f4c541f9e27aeea29d)
uses an elegant script to replace occurrences of multiple terms in all source files.

- [scripted-diff: Remove g_connman, g_banman globals](https://github.com/bitcoin/bitcoin/commit/8922d7f6b751a3e6b3b9f6fb7961c442877fb65a)
replaces specific terms in a list of specific source files.

- [scripted-diff: Replace fprintf with tfm::format](https://github.com/bitcoin/bitcoin/commit/fac03ec43a15ad547161e37e53ea82482cc508f9)
does a global replacement but excludes certain directories.

To find all previous uses of scripted diffs in the repository, do:

```
git log --grep="-BEGIN VERIFY SCRIPT-"
```

Release notes
-------------

Release notes should be written for any PR that:

- introduces a notable new feature
- fixes a significant bug
- changes an API or configuration model
- makes any other visible change to the end-user experience.

Release notes should be added to a PR-specific release note file at
`/doc/release-notes-<PR number>.md` to avoid conflicts between multiple PRs.
All `release-notes*` files are merged into a single `release-notes-<version>.md` file prior to the release.

RPC interface guidelines
--------------------------

A few guidelines for introducing and reviewing new RPC interfaces:

- Method naming: use consecutive lower-case names such as `getrawtransaction` and `submitblock`.

  - *Rationale*: Consistency with the existing interface.

- Argument and field naming: please consider whether there is already a naming
  style or spelling convention in the API for the type of object in question
  (`blockhash`, for example), and if so, try to use that. If not, use snake case
  `fee_delta` (and not, e.g. `feedelta` or camel case `feeDelta`).

  - *Rationale*: Consistency with the existing interface.

- Use the JSON parser for parsing, don't manually parse integers or strings from
  arguments unless absolutely necessary.

  - *Rationale*: Introduces hand-rolled string manipulation code at both the caller and callee sites,
    which is error-prone, and it is easy to get things such as escaping wrong.
    JSON already supports nested data structures, no need to re-invent the wheel.

  - *Exception*: AmountFromValue can parse amounts as string. This was introduced because many JSON
    parsers and formatters hard-code handling decimal numbers as floating-point
    values, resulting in potential loss of precision. This is unacceptable for
    monetary values. **Always** use `AmountFromValue` and `ValueFromAmount` when
    inputting or outputting monetary values. The only exceptions to this are
    `prioritisetransaction` and `getblocktemplate` because their interface
    is specified as-is in BIP22.

- Missing arguments and 'null' should be treated the same: as default values. If there is no
  default value, both cases should fail in the same way. The easiest way to follow this
  guideline is to detect unspecified arguments with `params[x].isNull()` instead of
  `params.size() <= x`. The former returns true if the argument is either null or missing,
  while the latter returns true if is missing, and false if it is null.

  - *Rationale*: Avoids surprises when switching to name-based arguments. Missing name-based arguments
  are passed as 'null'.

- Try not to overload methods on argument type. E.g. don't make `getblock(true)` and `getblock("hash")`
  do different things.

  - *Rationale*: This is impossible to use with `bitcoin-cli`, and can be surprising to users.

  - *Exception*: Some RPC calls can take both an `int` and `bool`, most notably when a bool was switched
    to a multi-value, or due to other historical reasons. **Always** have false map to 0 and
    true to 1 in this case.

- For new RPC methods, if implementing a `verbosity` argument, use integer verbosity rather than boolean.
  Disallow usage of boolean verbosity (see `ParseVerbosity()` in [util.h](/src/rpc/util.h)).

  - *Rationale*: Integer verbosity allows for multiple values. Undocumented boolean verbosity is deprecated
    and new RPC methods should prevent its use.

- Don't forget to fill in the argument names correctly in the RPC command table.

  - *Rationale*: If not, the call cannot be used with name-based arguments.

- Add every non-string RPC argument `(method, idx, name)` to the table `vRPCConvertParams` in `rpc/client.cpp`.

  - *Rationale*: `bitcoin-cli` and the GUI debug console use this table to determine how to
    convert a plaintext command line to JSON. If the types don't match, the method can be unusable
    from there.

- A RPC method must either be a wallet method or a non-wallet method. Do not
  introduce new methods that differ in behavior based on the presence of a wallet.

  - *Rationale*: As well as complicating the implementation and interfering
    with the introduction of multi-wallet, wallet and non-wallet code should be
    separated to avoid introducing circular dependencies between code units.

- Try to make the RPC response a JSON object.

  - *Rationale*: If a RPC response is not a JSON object, then it is harder to avoid API breakage if
    new data in the response is needed.

- Wallet RPCs call BlockUntilSyncedToCurrentChain to maintain consistency with
  `getblockchaininfo`'s state immediately prior to the call's execution. Wallet
  RPCs whose behavior does *not* depend on the current chainstate may omit this
  call.

  - *Rationale*: In previous versions of Bitcoin Core, the wallet was always
    in-sync with the chainstate (by virtue of them all being updated in the
    same cs_main lock). In order to maintain the behavior that wallet RPCs
    return results as of at least the highest best-known block an RPC
    client may be aware of prior to entering a wallet RPC call, we must block
    until the wallet is caught up to the chainstate as of the RPC call's entry.
    This also makes the API much easier for RPC clients to reason about.

- Be aware of RPC method aliases and generally avoid registering the same
  callback function pointer for different RPCs.

  - *Rationale*: RPC methods registered with the same function pointer will be
    considered aliases and only the first method name will show up in the
    `help` RPC command list.

  - *Exception*: Using RPC method aliases may be appropriate in cases where a
    new RPC is replacing a deprecated RPC, to avoid both RPCs confusingly
    showing up in the command list.

- Use *invalid* bech32 addresses (e.g. in the constant array `EXAMPLE_ADDRESS`) for
  `RPCExamples` help documentation.

  - *Rationale*: Prevent accidental transactions by users and encourage the use
    of bech32 addresses by default.

- Use the `UNIX_EPOCH_TIME` constant when describing UNIX epoch time or
  timestamps in the documentation.

  - *Rationale*: User-facing consistency.

- Use `fs::path::u8string()`/`fs::path::utf8string()` and `fs::u8path()` functions when converting path
  to JSON strings, not `fs::PathToString` and `fs::PathFromString`

  - *Rationale*: JSON strings are Unicode strings, not byte strings, and
    RFC8259 requires JSON to be encoded as UTF-8.

A few guidelines for modifying existing RPC interfaces:

- It's preferable to avoid changing an RPC in a backward-incompatible manner, but in that case, add an associated `-deprecatedrpc=` option to retain previous RPC behavior during the deprecation period. Backward-incompatible changes include: data type changes (e.g. from `{"warnings":""}` to `{"warnings":[]}`, changing a value from a string to a number, etc.), logical meaning changes of a value, or key name changes (e.g. `{"warning":""}` to `{"warnings":""}`). Adding a key to an object is generally considered backward-compatible. Include a release note that refers the user to the RPC help for details of feature deprecation and re-enabling previous behavior. [Example RPC help](https://github.com/bitcoin/bitcoin/blob/94f0adcc/src/rpc/blockchain.cpp#L1316-L1323).

  - *Rationale*: Changes in RPC JSON structure can break downstream application compatibility. Implementation of `deprecatedrpc` provides a grace period for downstream applications to migrate. Release notes provide notification to downstream users.

Internal interface guidelines
-----------------------------

Internal interfaces between parts of the codebase that are meant to be
independent (node, wallet, GUI), are defined in
[`src/interfaces/`](../src/interfaces/). The main interface classes defined
there are [`interfaces::Chain`](../src/interfaces/chain.h), used by wallet to
access the node's latest chain state,
[`interfaces::Node`](../src/interfaces/node.h), used by the GUI to control the
node, [`interfaces::Wallet`](../src/interfaces/wallet.h), used by the GUI
to control an individual wallet and [`interfaces::Mining`](../src/interfaces/mining.h),
used by RPC to generate block templates. There are also more specialized interface
types like [`interfaces::Handler`](../src/interfaces/handler.h)
[`interfaces::ChainClient`](../src/interfaces/chain.h) passed to and from
various interface methods.

Interface classes are written in a particular style so node, wallet, and GUI
code doesn't need to run in the same process, and so the class declarations
work more easily with tools and libraries supporting interprocess
communication:

- Interface classes should be abstract and have methods that are [pure
  virtual](https://en.cppreference.com/w/cpp/language/abstract_class). This
  allows multiple implementations to inherit from the same interface class,
  particularly so one implementation can execute functionality in the local
  process, and other implementations can forward calls to remote processes.

- Interface method definitions should wrap existing functionality instead of
  implementing new functionality. Any substantial new node or wallet
  functionality should be implemented in [`src/node/`](../src/node/) or
  [`src/wallet/`](../src/wallet/) and just exposed in
  [`src/interfaces/`](../src/interfaces/) instead of being implemented there,
  so it can be more modular and accessible to unit tests.

- Interface method parameter and return types should either be serializable or
  be other interface classes. Interface methods shouldn't pass references to
  objects that can't be serialized or accessed from another process.

  Examples:

  ```c++
  // Good: takes string argument and returns interface class pointer
  virtual unique_ptr<interfaces::Wallet> loadWallet(std::string filename) = 0;

  // Bad: returns CWallet reference that can't be used from another process
  virtual CWallet& loadWallet(std::string filename) = 0;
  ```

  ```c++
  // Good: accepts and returns primitive types
  virtual bool findBlock(const uint256& hash, int& out_height, int64_t& out_time) = 0;

  // Bad: returns pointer to internal node in a linked list inaccessible to
  // other processes
  virtual const CBlockIndex* findBlock(const uint256& hash) = 0;
  ```

  ```c++
  // Good: takes plain callback type and returns interface pointer
  using TipChangedFn = std::function<void(int block_height, int64_t block_time)>;
  virtual std::unique_ptr<interfaces::Handler> handleTipChanged(TipChangedFn fn) = 0;

  // Bad: returns boost connection specific to local process
  using TipChangedFn = std::function<void(int block_height, int64_t block_time)>;
  virtual boost::signals2::scoped_connection connectTipChanged(TipChangedFn fn) = 0;
  ```

- Interface methods should not be overloaded.

  *Rationale*: consistency and friendliness to code generation tools.

  Example:

  ```c++
  // Good: method names are unique
  virtual bool disconnectByAddress(const CNetAddr& net_addr) = 0;
  virtual bool disconnectById(NodeId id) = 0;

  // Bad: methods are overloaded by type
  virtual bool disconnect(const CNetAddr& net_addr) = 0;
  virtual bool disconnect(NodeId id) = 0;
  ```

### Internal interface naming style

- Interface method names should be `lowerCamelCase` and standalone function names should be
  `UpperCamelCase`.

  *Rationale*: consistency and friendliness to code generation tools.

  Examples:

  ```c++
  // Good: lowerCamelCase method name
  virtual void blockConnected(const CBlock& block, int height) = 0;

  // Bad: uppercase class method
  virtual void BlockConnected(const CBlock& block, int height) = 0;
  ```

  ```c++
  // Good: UpperCamelCase standalone function name
  std::unique_ptr<Node> MakeNode(LocalInit& init);

  // Bad: lowercase standalone function
  std::unique_ptr<Node> makeNode(LocalInit& init);
  ```

  Note: This last convention isn't generally followed outside of
  [`src/interfaces/`](../src/interfaces/), though it did come up for discussion
  before in [#14635](https://github.com/bitcoin/bitcoin/pull/14635).
