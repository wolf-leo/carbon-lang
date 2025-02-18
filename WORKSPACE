# Part of the Carbon Language project, under the Apache License v2.0 with LLVM
# Exceptions. See /LICENSE for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

workspace(name = "carbon")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

###############################################################################
# Python rules
###############################################################################

rules_python_version = "0.8.1"

# Add Bazel's python rules and set up pip.
http_archive(
    name = "rules_python",
    sha256 = "cdf6b84084aad8f10bf20b46b77cb48d83c319ebe6458a18e9d2cebf57807cdd",
    strip_prefix = "rules_python-%s" % rules_python_version,
    url = "https://github.com/bazelbuild/rules_python/archive/refs/tags/%s.tar.gz" % rules_python_version,
)

load("@rules_python//python:pip.bzl", "pip_install")

# Create a central repo that knows about the pip dependencies.
pip_install(
    name = "py_deps",
    requirements = "//github_tools:requirements.txt",
)

###############################################################################
# C++ rules
###############################################################################

# Configure the bootstrapped Clang and LLVM toolchain for Bazel.
load(
    "//bazel/cc_toolchains:clang_configuration.bzl",
    "configure_clang_toolchain",
)

configure_clang_toolchain(name = "bazel_cc_toolchain")

###############################################################################
# Abseil libraries
###############################################################################

# Head as of 2022-09-13.
abseil_version = "530cd52f585c9d31b2b28cea7e53915af7a878e3"

http_archive(
    name = "com_google_absl",
    sha256 = "f8a6789514a3b109111252af92da41d6e64f90efca9fb70515d86debee57dc24",
    strip_prefix = "abseil-cpp-%s" % abseil_version,
    urls = ["https://github.com/abseil/abseil-cpp/archive/%s.tar.gz" % abseil_version],
)

###############################################################################
# RE2 libraries
###############################################################################

# Head as of 2022-09-14.
re2_version = "cc1c9db8bf5155d89d10d65998cdb226f676492c"

http_archive(
    name = "com_googlesource_code_re2",
    sha256 = "8ef976c79a300f8c5e880535665bd4ba146fb09fb6d2342f8f1a02d9af29f365",
    strip_prefix = "re2-%s" % re2_version,
    urls = ["https://github.com/google/re2/archive/%s.tar.gz" % re2_version],
)

###############################################################################
# GoogleTest libraries
###############################################################################

# Head as of 2022-09-14.
googletest_version = "1336c4b6d1a6f4bc6beebccb920e5ff858889292"

http_archive(
    name = "com_google_googletest",
    sha256 = "d701aaeb9a258afba27210d746d971042be96c371ddc5a49f1e8914d9ea17e3c",
    strip_prefix = "googletest-%s" % googletest_version,
    urls = ["https://github.com/google/googletest/archive/%s.tar.gz" % googletest_version],
)

###############################################################################
# Google Benchmark libraries
###############################################################################

benchmark_version = "1.6.1"

http_archive(
    name = "com_github_google_benchmark",
    sha256 = "6132883bc8c9b0df5375b16ab520fac1a85dc9e4cf5be59480448ece74b278d4",
    strip_prefix = "benchmark-%s" % benchmark_version,
    urls = ["https://github.com/google/benchmark/archive/refs/tags/v%s.tar.gz" % benchmark_version],
)

###############################################################################
# LLVM libraries
###############################################################################

# We pin to specific upstream commits and try to track top-of-tree reasonably
# closely rather than pinning to a specific release.
llvm_version = "6fa65f8a98967a5d2d2a6863e0f67a40d2961905"

http_archive(
    name = "llvm-raw",
    build_file_content = "# empty",
    patch_args = ["-p1"],
    patches = [
        "@carbon//bazel/llvm-patches:0001-Patch-for-mallinfo2-when-using-Bazel-build-system.patch",
        "@carbon//bazel/llvm-patches:0002-Added-Bazel-build-for-compiler-rt-fuzzer.patch",
    ],
    sha256 = "0a3929c5f2fe756820277be7b10e95f7480e7cb7f297ec574d3e9ddeac9068d7",
    strip_prefix = "llvm-project-%s" % llvm_version,
    urls = ["https://github.com/llvm/llvm-project/archive/%s.tar.gz" % llvm_version],
)

load("@llvm-raw//utils/bazel:configure.bzl", "llvm_configure")

llvm_configure(
    name = "llvm-project",
    repo_mapping = {"@llvm_zlib": "@zlib"},
    targets = [
        "AArch64",
        "X86",
    ],
)

load("@llvm-raw//utils/bazel:terminfo.bzl", "llvm_terminfo_system")

# We require successful detection and use of a system terminfo library.
llvm_terminfo_system(name = "llvm_terminfo")

load("@llvm-raw//utils/bazel:zlib.bzl", "llvm_zlib_system")

# We require successful detection and use of a system zlib library.
llvm_zlib_system(name = "zlib")

###############################################################################
# Flex/Bison rules
###############################################################################

rules_m4_version = "0.2.1"

http_archive(
    name = "rules_m4",
    sha256 = "eaa674cd84546038ecbcc49cdd346134a20961a41fa1a541e80d8bf4b470c34d",
    strip_prefix = "rules_m4-%s" % rules_m4_version,
    urls = ["https://github.com/jmillikin/rules_m4/archive/v%s.tar.gz" %
            rules_m4_version],
)

load("@rules_m4//m4:m4.bzl", "m4_register_toolchains")

# When building M4, disable all compiler warnings as we can't realistically fix
# them anyways.
m4_register_toolchains(extra_copts = ["-w"])

# TODO: Can switch to a normal release version when it includes:
# https://github.com/jmillikin/rules_flex/commit/1f1d9c306c2b4b8be2cb899a3364b84302124e77
rules_flex_version = "1f1d9c306c2b4b8be2cb899a3364b84302124e77"

http_archive(
    name = "rules_flex",
    sha256 = "a4e99a0a241c8a5aa238e81724ea3529722522c3702fd3aa674add5eb9807002",
    strip_prefix = "rules_flex-%s" % rules_flex_version,
    urls = ["https://github.com/jmillikin/rules_flex/archive/%s.tar.gz" %
            rules_flex_version],
)

load("@rules_flex//flex:flex.bzl", "flex_register_toolchains")

# When building Flex, disable all compiler warnings as we can't realistically
# fix them anyways.
flex_register_toolchains(extra_copts = ["-w"])

# TODO: Can switch to a normal release version when it includes:
# https://github.com/jmillikin/rules_bison/commit/478079b28605a38000eaf83719568d756b3383a0
rules_bison_version = "478079b28605a38000eaf83719568d756b3383a0"

http_archive(
    name = "rules_bison",
    sha256 = "6bc2d382e4ffccd66e60a74521c24722fc8fdfe9af49ff182f79bb5994fa1ba4",
    strip_prefix = "rules_bison-%s" % rules_bison_version,
    urls = ["https://github.com/jmillikin/rules_bison/archive/%s.tar.gz" %
            rules_bison_version],
)

load("@rules_bison//bison:bison.bzl", "bison_register_toolchains")

# When building Bison, disable all compiler warnings as we can't realistically
# fix them anyways.
bison_register_toolchains(extra_copts = ["-w"])

###############################################################################
# Protocol buffers - for structured fuzzer testing.
###############################################################################

# TODO: `rules_proto` pulls in a version of `rules_cc` with a frozenset bug.
rules_cc_version = "0.0.1"

http_archive(
    name = "rules_cc",
    sha256 = "4dccbfd22c0def164c8f47458bd50e0c7148f3d92002cdb459c2a96a68498241",
    urls = ["https://github.com/bazelbuild/rules_cc/releases/download/%s/rules_cc-%s.tar.gz" % (rules_cc_version, rules_cc_version)],
)

rules_proto_version = "4.0.0-3.20.0"

http_archive(
    name = "rules_proto",
    sha256 = "e017528fd1c91c5a33f15493e3a398181a9e821a804eb7ff5acdd1d2d6c2b18d",
    strip_prefix = "rules_proto-%s" % rules_proto_version,
    urls = [
        "https://github.com/bazelbuild/rules_proto/archive/refs/tags/%s.tar.gz" % rules_proto_version,
    ],
)

load("@rules_proto//proto:repositories.bzl", "rules_proto_dependencies", "rules_proto_toolchains")

rules_proto_dependencies()

rules_proto_toolchains()

###############################################################################
# libprotobuf_mutator - for structured fuzzer testing.
###############################################################################

# Head as of 2022-09-13.
libprotobuf_mutator_version = "a304ec48dcf15d942607032151f7e9ee504b5dcf"

http_archive(
    name = "com_google_libprotobuf_mutator",
    build_file = "@//:third_party/libprotobuf_mutator/BUILD.txt",
    sha256 = "0ce80217393fe6b01dac9818127e664801d865fefd708b98183181c0ed457878",
    strip_prefix = "libprotobuf-mutator-%s" % libprotobuf_mutator_version,
    urls = ["https://github.com/google/libprotobuf-mutator/archive/%s.tar.gz" % libprotobuf_mutator_version],
)

###############################################################################
# Example conversion repositories
###############################################################################

local_repository(
    name = "brotli",
    path = "third_party/examples/brotli/original",
)

new_local_repository(
    name = "woff2",
    build_file = "third_party/examples/woff2/BUILD.original",
    path = "third_party/examples/woff2/original",
    workspace_file = "third_party/examples/woff2/WORKSPACE.original",
)

local_repository(
    name = "woff2_carbon",
    path = "third_party/examples/woff2/carbon",
)
