# -*- python -*-

# This file contains rules for the Bazel build system.
# See http://bazel.io/ .

package(default_visibility = ["//visibility:public"])

# In LCM's upstream build system lcm_export.h is generated by CMake and
# contains preprocessor conditionals for library decoration settings, based on
# what kind of compilation step is being run.  For our Bazel build of LCM, we
# don't want those values in a header file because we set them directly via our
# defines= parameter instead.  Here, we generate an empty header file to
# satisfy the #include "lcm_export.h" statements within the LCM source code.
genrule(
    name = "lcm_export",
    outs = ["lcm_export.h"],
    cmd = "echo '' > \"$@\"",
    visibility = [],
)

# These options are used when building all LCM C/C++ code.
# They do not flow downstream to LCM-using libraries.
LCM_COPTS = [
    "-Wno-all",
    "-Wno-deprecated-declarations",
    "-Wno-format-zero-length",
    "-std=gnu11",
]

cc_library(
    name = "lcm",
    srcs = [
        "lcm/eventlog.c",
        "lcm/lcm.c",
        "lcm/lcm_file.c",
        "lcm/lcm_memq.c",
        "lcm/lcm_mpudpm.c",
        "lcm/lcm_tcpq.c",
        "lcm/lcm_udpm.c",
        "lcm/lcmtypes/channel_port_map_update_t.c",
        "lcm/lcmtypes/channel_to_port_t.c",
        "lcm/ringbuffer.c",
        "lcm/udpm_util.c",
    ],
    hdrs = [
        "lcm/dbg.h",
        "lcm/eventlog.h",
        "lcm/ioutils.h",
        "lcm/lcm.h",
        "lcm/lcm-cpp.hpp",
        "lcm/lcm-cpp-impl.hpp",
        "lcm/lcm_coretypes.h",
        "lcm/lcm_internal.h",
        "lcm/lcm_version.h",
        "lcm/lcmtypes/channel_port_map_update_t.h",
        "lcm/lcmtypes/channel_to_port_t.h",
        "lcm/ringbuffer.h",
        "lcm/udpm_util.h",
        "lcm_export.h",  # N.B. This is from lcm_export genrule above.
    ],
    copts = LCM_COPTS,
    # In LCM's build system, these definitions are provided by a generated
    # lcm_export.h file.  For Bazel, we just set them directly as defines.
    defines = [
        "LCM_DEPRECATED=",
        "LCM_DEPRECATED_EXPORT=",
        "LCM_DEPRECATED_NO_EXPORT=",
        "LCM_EXPORT=",
        "LCM_NO_EXPORT=",
    ],
    includes = ["."],
    deps = ["@glib//:lib"],
)

cc_binary(
    name = "lcmgen",
    srcs = [
        "lcmgen/emit_c.c",
        "lcmgen/emit_cpp.c",
        "lcmgen/emit_csharp.c",
        "lcmgen/emit_java.c",
        "lcmgen/emit_lua.c",
        "lcmgen/emit_python.c",
        "lcmgen/getopt.c",
        "lcmgen/getopt.h",
        "lcmgen/lcmgen.c",
        "lcmgen/lcmgen.h",
        "lcmgen/main.c",
        "lcmgen/tokenize.c",
        "lcmgen/tokenize.h",
    ],
    copts = LCM_COPTS,
    includes = ["."],
    deps = [":lcm"],
)

cc_binary(
    name = "_lcm.so",
    srcs = [
        "lcm-python/module.c",
        "lcm-python/pyeventlog.c",
        "lcm-python/pylcm.c",
        "lcm-python/pylcm.h",
        "lcm-python/pylcm_subscription.c",
        "lcm-python/pylcm_subscription.h",
    ],
    copts = LCM_COPTS,
    linkshared = 1,
    linkstatic = 1,
    visibility = [],
    deps = [
        ":lcm",
        "@python2//:lib",
    ],
)

# Downstream users of lcm-python expect to say "import lcm".  However, in the
# sandbox the python package is located at lcm/lcm-python/lcm/__init__.py to
# match the source tree structure of LCM; without any special help the import
# would fail.
#
# Normally we'd add `imports = ["lcm-python"]` to establish a PYTHONPATH at the
# correct subdirectory, and that almost works.  However, because the external
# is named "lcm", Bazel's auto-generated empty "lcm/__init__.py" at the root of
# the sandbox is found first, and prevents the lcm-python subdirectory from
# ever being found.
#
# To repair this, we provide our own init file at the root of the sandbox that
# overrides the Bazel empty default.  Its implementation just delegates to the
# lcm-python init file.
genrule(
    name = "init_genrule",
    srcs = [],
    outs = ["__init__.py"],
    cmd = "echo 'execfile(__path__[0] + \"/lcm-python/lcm/__init__.py\")' > $@",
)

py_library(
    name = "lcm-python",
    srcs = [
        "__init__.py",  # Shim, from the genrule above.
        "lcm-python/lcm/__init__.py",  # Actual code from upstream.
    ],
    data = [":_lcm.so"],
)