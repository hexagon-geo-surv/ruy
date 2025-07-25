# Ruy is not BLAS

load("@bazel_skylib//:bzl_library.bzl", "bzl_library")
load("@bazel_skylib//lib:selects.bzl", "selects")
load("//third_party/bazel_rules/rules_cc/cc:cc_library.bzl", "cc_library")
load("//third_party/bazel_rules/rules_cc/cc:cc_test.bzl", "cc_test")
load(":build_defs.bzl", "ruy_copts", "ruy_copts_avx", "ruy_copts_avx2_fma", "ruy_copts_avx512")
load(":build_defs.oss.bzl", "ruy_linkopts_thread_standard_library")
load(":ruy_test.bzl", "ruy_benchmark", "ruy_test")
load(":ruy_test_ext.oss.bzl", "ruy_test_ext_defines", "ruy_test_ext_deps")

package(
    default_applicable_licenses = ["//third_party/ruy:license"],
    licenses = ["notice"],  # Apache 2.0
)

# Detect ARM 32-bit targets where we are going to just assume NEON support.
config_setting(
    name = "arm32_assuming_neon",
    constraint_values = [
        "@platforms//cpu:armv7",
    ],
)

config_setting(
    name = "x86_64_and_not_msvc",
    constraint_values = [
        "@platforms//cpu:x86_64",
        "@platforms//os:linux",
    ],
)

config_setting(
    name = "windows_msvc",
    constraint_values = [
        "@platforms//os:windows",
    ],
    flag_values = {
        "//tools/cpp:compiler": "msvc",
    },
)

config_setting(
    name = "ppc",
    constraint_values = [
        "@platforms//cpu:ppc",
    ],
)

config_setting(
    name = "s390x",
    constraint_values = [
        "@platforms//cpu:s390x",
    ],
)

config_setting(
    name = "fuchsia",
    constraint_values = [
        "@platforms//os:fuchsia",
    ],
)

config_setting(
    name = "dbg_build",
    values = {
        "compilation_mode": "dbg",
    },
)

config_setting(
    name = "fastbuild_build",
    values = {
        "compilation_mode": "fastbuild",
    },
)

selects.config_setting_group(
    name = "do_not_want_O3",
    match_any = [
        ":windows_msvc",
        ":dbg_build",
        ":fastbuild_build",
    ],
)

cc_library(
    name = "trace",
    hdrs = ["trace.h"],
    copts = ruy_copts(),
    deps = [
        ":mat",
        ":matrix",
        ":path",
        ":platform",
        ":side_pair",
    ],
)

cc_library(
    name = "platform",
    hdrs = ["platform.h"],
    copts = ruy_copts(),
)

cc_library(
    name = "gtest_wrapper",
    testonly = True,
    hdrs = ["gtest_wrapper.h"],
    visibility = [":__subpackages__"],
    deps = ["@com_google_googletest//:gtest"],
)

cc_library(
    name = "check_macros",
    hdrs = ["check_macros.h"],
    copts = ruy_copts(),
)

cc_test(
    name = "check_macros_test",
    srcs = ["check_macros_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":gtest_wrapper",
    ],
)

cc_library(
    name = "opt_set",
    hdrs = ["opt_set.h"],
    copts = ruy_copts(),
)

cc_library(
    name = "time",
    hdrs = ["time.h"],
    copts = ruy_copts(),
)

cc_library(
    name = "wait",
    srcs = ["wait.cc"],
    hdrs = ["wait.h"],
    copts = ruy_copts(),
    linkopts = ruy_linkopts_thread_standard_library(),
    deps = [":time"],
)

cc_test(
    name = "wait_test",
    srcs = ["wait_test.cc"],
    copts = ruy_copts(),
    linkopts = ruy_linkopts_thread_standard_library(),
    deps = [
        ":gtest_wrapper",
        ":platform",
        ":wait",
    ],
)

cc_library(
    name = "size_util",
    hdrs = ["size_util.h"],
    copts = ruy_copts(),
    deps = [":check_macros"],
)

cc_test(
    name = "size_util_test",
    srcs = ["size_util_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":gtest_wrapper",
        ":size_util",
    ],
)

cc_library(
    name = "tune",
    srcs = [
        "tune.cc",
    ],
    hdrs = [
        "tune.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":cpu_cache_params",
        ":cpuinfo",
        ":opt_set",
        ":platform",
        ":time",
    ],
)

cc_library(
    name = "system_aligned_alloc",
    srcs = [
        "system_aligned_alloc.cc",
    ],
    hdrs = [
        "system_aligned_alloc.h",
    ],
    copts = ruy_copts(),
)

cc_library(
    name = "prepacked_cache",
    srcs = [
        "prepacked_cache.cc",
    ],
    hdrs = [
        "prepacked_cache.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":mat",
        ":system_aligned_alloc",
        "//ruy/profiler:instrumentation",
    ],
)

cc_test(
    name = "tune_test",
    srcs = ["tune_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":cpuinfo",
        ":gtest_wrapper",
        ":tune",
    ],
)

cc_test(
    name = "prepacked_cache_test",
    srcs = ["prepacked_cache_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":context",
        ":context_get_ctx",
        ":ctx",
        ":gtest_wrapper",
        ":mat",
        ":matrix",
        ":prepacked_cache",
        ":ruy",
        ":time",
    ],
)

cc_library(
    name = "allocator",
    srcs = [
        "allocator.cc",
    ],
    hdrs = [
        "allocator.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":opt_set",
        ":size_util",
        ":system_aligned_alloc",
    ],
)

cc_test(
    name = "allocator_test",
    srcs = ["allocator_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":gtest_wrapper",
    ],
)

cc_library(
    name = "side_pair",
    hdrs = ["side_pair.h"],
    copts = ruy_copts(),
    deps = [":check_macros"],
)

cc_library(
    name = "block_map",
    srcs = [
        "block_map.cc",
    ],
    hdrs = [
        "block_map.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":cpu_cache_params",
        ":opt_set",
        ":side_pair",
        ":size_util",
        ":trace",
        "//ruy/profiler:instrumentation",
    ],
)

cc_test(
    name = "block_map_test",
    srcs = ["block_map_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":block_map",
        ":cpu_cache_params",
        ":gtest_wrapper",
        ":path",
        ":platform",
        ":side_pair",
    ],
)

cc_library(
    name = "blocking_counter",
    srcs = [
        "blocking_counter.cc",
    ],
    hdrs = [
        "blocking_counter.h",
    ],
    copts = ruy_copts(),
    linkopts = ruy_linkopts_thread_standard_library(),
    deps = [
        ":check_macros",
        ":time",
        ":wait",
    ],
)

cc_library(
    name = "thread_pool",
    srcs = [
        "thread_pool.cc",
    ],
    hdrs = [
        "thread_pool.h",
    ],
    copts = ruy_copts(),
    linkopts = ruy_linkopts_thread_standard_library(),
    visibility = ["//visibility:public"],
    deps = [
        ":blocking_counter",
        ":check_macros",
        ":denormal",
        ":time",
        ":trace",
        ":wait",
    ],
)

cc_library(
    name = "cpu_cache_params",
    hdrs = ["cpu_cache_params.h"],
    copts = ruy_copts(),
)

cc_library(
    name = "cpuinfo",
    srcs = [
        "cpuinfo.cc",
    ],
    hdrs = [
        "cpuinfo.h",
    ],
    copts = ruy_copts() +
            select({
                "@platforms//os:windows": [],
                "//conditions:default": [
                    # ruy_copts contains -Wundef, but cpuinfo's header warns with that.
                    "-Wno-undef",
                ],
            }) + select({
        # This select must match the similar select in `deps`.
        # We intentionally define this token in the BUILD
        # file so that ports to other build-systems do not
        # use cpuinfo by default - they need to port the
        # cpuinfo BUILD first, then can define this token.
        ":ppc": [],
        ":s390x": [],
        ":fuchsia": [],
        "//conditions:default": ["-DRUY_HAVE_CPUINFO"],
    }),
    deps = [
        ":check_macros",
        ":cpu_cache_params",
        ":platform",
    ] + select({
        # This select must match the similar select in `copts`
        ":ppc": [],
        ":s390x": [],
        ":fuchsia": [],
        "//conditions:default": ["@cpuinfo"],
    }),
)

cc_library(
    name = "path",
    hdrs = ["path.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":platform",
        ":size_util",
    ],
)

cc_library(
    name = "denormal",
    srcs = ["denormal.cc"],
    hdrs = ["denormal.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
)

cc_library(
    name = "performance_advisory",
    hdrs = ["performance_advisory.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
)

cc_library(
    name = "strategy_controls",
    hdrs = ["strategy_controls.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
)

cc_library(
    name = "matrix",
    hdrs = ["matrix.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [":check_macros"],
)

cc_test(
    name = "matrix_test",
    srcs = ["matrix_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":gtest_wrapper",
        ":matrix",
    ],
)

cc_library(
    name = "mul_params",
    hdrs = ["mul_params.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":check_macros",
        ":size_util",
    ],
)

cc_test(
    name = "mul_params_test",
    srcs = ["mul_params_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":gtest_wrapper",
        ":mul_params",
    ],
)

cc_library(
    name = "mat",
    hdrs = ["mat.h"],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":matrix",
        ":size_util",
    ],
)

cc_library(
    name = "asm_helpers",
    hdrs = [
        "asm_helpers.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":opt_set",
    ],
)

cc_library(
    name = "apply_multiplier",
    srcs = ["apply_multiplier.cc"],
    hdrs = ["apply_multiplier.h"],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":mul_params",
    ],
)

cc_test(
    name = "apply_multiplier_test",
    srcs = ["apply_multiplier_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":apply_multiplier",
        ":gtest_wrapper",
        ":mul_params",
    ],
)

cc_library(
    name = "kernel_common",
    hdrs = [
        "kernel_common.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":apply_multiplier",
        ":check_macros",
        ":mat",
        ":matrix",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":side_pair",
        ":size_util",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack_common",
    hdrs = [
        "pack_common.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":mat",
        ":matrix",
        ":opt_set",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "kernel_arm",
    srcs = [
        "kernel_arm32.cc",
        "kernel_arm64.cc",
    ],
    hdrs = ["kernel_arm.h"],
    copts = ruy_copts(),
    deps = [
        ":asm_helpers",
        ":check_macros",
        ":kernel_common",
        ":mat",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":side_pair",
        ":size_util",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack_arm",
    srcs = [
        "pack_arm.cc",
    ],
    hdrs = [
        "pack_arm.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":asm_helpers",
        ":check_macros",
        ":mat",
        ":opt_set",
        ":pack_common",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "kernel_avx512",
    srcs = [
        "kernel_avx512.cc",
    ],
    hdrs = [
        "kernel_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx512(),
    deps = [
        ":check_macros",
        ":kernel_common",
        ":mat",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack_avx512",
    srcs = [
        "pack_avx512.cc",
    ],
    hdrs = [
        "pack_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx512(),
    deps = [
        ":check_macros",
        ":mat",
        ":opt_set",
        ":pack_common",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "have_built_path_for_avx512",
    srcs = [
        "have_built_path_for_avx512.cc",
    ],
    hdrs = [
        "have_built_path_for.h",
    ],
    copts = ruy_copts() + ruy_copts_avx512(),
    deps = [
        ":opt_set",
        ":platform",
    ],
)

cc_library(
    name = "kernel_avx2_fma",
    srcs = [
        "kernel_avx2_fma.cc",
    ],
    hdrs = [
        "kernel_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx2_fma(),
    deps = [
        ":check_macros",
        ":kernel_common",
        ":mat",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack_avx2_fma",
    srcs = [
        "pack_avx2_fma.cc",
    ],
    hdrs = [
        "pack_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx2_fma(),
    deps = [
        ":check_macros",
        ":mat",
        ":opt_set",
        ":pack_common",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "have_built_path_for_avx2_fma",
    srcs = [
        "have_built_path_for_avx2_fma.cc",
    ],
    hdrs = [
        "have_built_path_for.h",
    ],
    copts = ruy_copts() + ruy_copts_avx2_fma(),
    deps = [
        ":opt_set",
        ":platform",
    ],
)

cc_library(
    name = "kernel_avx",
    srcs = [
        "kernel_avx.cc",
    ],
    hdrs = [
        "kernel_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx(),
    deps = [
        ":check_macros",
        ":kernel_common",
        ":mat",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack_avx",
    srcs = [
        "pack_avx.cc",
    ],
    hdrs = [
        "pack_x86.h",
    ],
    copts = ruy_copts() + ruy_copts_avx(),
    deps = [
        ":check_macros",
        ":mat",
        ":opt_set",
        ":pack_common",
        ":path",
        ":platform",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "have_built_path_for_avx",
    srcs = [
        "have_built_path_for_avx.cc",
    ],
    hdrs = [
        "have_built_path_for.h",
    ],
    copts = ruy_copts() + ruy_copts_avx(),
    deps = [
        ":opt_set",
        ":platform",
    ],
)

cc_library(
    name = "kernel",
    hdrs = [
        "kernel.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":apply_multiplier",
        ":check_macros",
        ":kernel_arm",  # fixdeps: keep
        ":kernel_avx",
        ":kernel_avx2_fma",  # fixdeps: keep
        ":kernel_avx512",  # fixdeps: keep
        ":kernel_common",
        ":mat",
        ":matrix",
        ":mul_params",
        ":opt_set",
        ":path",
        ":platform",
        ":side_pair",
        ":size_util",
        ":trace",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "pack",
    hdrs = [
        "pack.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":mat",
        ":matrix",
        ":opt_set",
        ":pack_arm",  # fixdeps: keep
        ":pack_avx",  # fixdeps: keep
        ":pack_avx2_fma",  # fixdeps: keep
        ":pack_avx512",  # fixdeps: keep
        ":pack_common",
        ":path",
        ":platform",
        ":trace",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "have_built_path_for",
    hdrs = [
        "have_built_path_for.h",
    ],
    deps = [
        ":have_built_path_for_avx",
        ":have_built_path_for_avx2_fma",
        ":have_built_path_for_avx512",
        ":platform",
    ],
)

cc_library(
    name = "context",
    srcs = ["context.cc"],
    hdrs = [
        "context.h",
    ],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":allocator",
        ":check_macros",
        ":ctx",
        ":path",
        ":performance_advisory",
        ":platform",
        ":prepacked_cache",
        ":strategy_controls",
        ":thread_pool",
        ":tune",
    ],
)

cc_test(
    name = "context_test",
    srcs = ["context_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":context",
        ":gtest_wrapper",
        ":path",
        ":platform",
        ":prepacked_cache",
        ":strategy_controls",
        ":tune",
    ],
)

cc_library(
    name = "ctx_header_only_should_not_include_other_ruy_headers",
    testonly = True,
    hdrs = [
        "ctx.h",
    ],
    # Intentionally no deps. This will cause the stand-alone build of ctx.h to
    # fail if ctx.h #includes any other ruy header.
)

cc_library(
    name = "ctx",
    srcs = [
        "ctx.cc",
    ],
    hdrs = [
        "ctx.h",
        "ctx_impl.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":check_macros",
        ":cpuinfo",
        ":have_built_path_for",
        ":path",
        ":performance_advisory",
        ":platform",
        ":prepacked_cache",
        ":strategy_controls",
        ":thread_pool",
        ":trace",
        ":tune",
    ],
)

cc_library(
    name = "context_get_ctx",
    srcs = [
        "context_get_ctx.cc",
    ],
    hdrs = [
        "context_get_ctx.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":context",
        ":ctx",
    ],
)

cc_test(
    name = "ctx_test",
    srcs = ["ctx_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":ctx",
        ":gtest_wrapper",
        ":path",
        ":platform",
        ":strategy_controls",
    ],
)

cc_library(
    name = "trmul_params",
    hdrs = ["trmul_params.h"],
    copts = ruy_copts(),
    deps = [
        ":mat",
        ":mul_params",
        ":path",
        ":side_pair",
        ":tune",
    ],
)

cc_library(
    name = "trmul",
    srcs = ["trmul.cc"],
    hdrs = ["trmul.h"],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":block_map",
        ":check_macros",
        ":cpu_cache_params",
        ":cpuinfo",
        ":ctx",
        ":denormal",
        ":mat",
        ":matrix",
        ":mul_params",
        ":opt_set",
        ":side_pair",
        ":size_util",
        ":strategy_controls",
        ":thread_pool",
        ":trace",
        ":trmul_params",
        ":tune",
        "//ruy/profiler:instrumentation",
    ],
)

cc_library(
    name = "prepare_packed_matrices",
    srcs = ["prepare_packed_matrices.cc"],
    hdrs = ["prepare_packed_matrices.h"],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":ctx",
        ":matrix",
        ":prepacked_cache",
        ":side_pair",
        ":trace",
        ":trmul_params",
    ],
)

cc_library(
    name = "create_trmul_params",
    hdrs = ["create_trmul_params.h"],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":check_macros",
        ":ctx",
        ":kernel",
        ":mat",
        ":mul_params",
        ":pack",
        ":path",
        ":performance_advisory",
        ":platform",
        ":side_pair",
        ":trace",
        ":trmul_params",
    ],
)

cc_library(
    name = "validate",
    hdrs = ["validate.h"],
    copts = ruy_copts(),
    deps = [
        ":check_macros",
        ":mat",
        ":mul_params",
        ":side_pair",
    ],
)

cc_library(
    name = "frontend",
    srcs = [
        "frontend.cc",
    ],
    hdrs = [
        "frontend.h",
    ],
    copts = ruy_copts(),
    deps = [
        ":allocator",
        ":create_trmul_params",
        ":ctx",
        ":mat",
        ":mul_params",
        ":prepare_packed_matrices",
        ":trace",
        ":trmul",
        ":trmul_params",
        ":validate",
        "//ruy/profiler:instrumentation",
    ],
)

# The main library.
cc_library(
    name = "ruy",
    hdrs = [
        "context.h",
        "matrix.h",
        "mul_params.h",
        "path.h",
        "ruy.h",
    ],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":check_macros",
        ":context",
        ":context_get_ctx",
        ":frontend",
        ":mat",
        ":matrix",
        ":mul_params",
        ":path",
        ":platform",
        ":size_util",
        ":trace",
    ],
)

cc_test(
    name = "perchannel_buffers_reallocation_test",
    srcs = ["perchannel_buffers_reallocation_test.cc"],
    copts = ruy_copts(),
    deps = [
        ":context",
        ":gtest_wrapper",
        ":kernel",
        ":matrix",
        ":path",
        ":performance_advisory",
        ":ruy",
    ],
)

# Small library to query PMU counters, for benchmark only
cc_library(
    name = "pmu",
    testonly = True,
    srcs = ["pmu.cc"],
    hdrs = ["pmu.h"],
    copts = ruy_copts(),
    deps = [":check_macros"],
)

cc_library(
    name = "reference_mul",
    hdrs = ["reference_mul.h"],
    copts = ruy_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":apply_multiplier",
        ":matrix",
        ":mul_params",
    ],
)

# Testing framework.
cc_library(
    name = "test_lib",
    testonly = True,
    hdrs = ["test.h"],
    copts = ruy_copts(),
    # need defines, not copts, because it's controlling a header, test.h
    defines = ruy_test_ext_defines(),
    linkopts = select({
        "@platforms//os:windows": [],
        "//conditions:default": ["-lm"],
    }),
    deps = [
        ":allocator",
        ":context",
        ":context_get_ctx",
        ":ctx",
        ":gtest_wrapper",
        ":matrix",
        ":mul_params",
        ":pack_common",
        ":platform",
        ":pmu",
        ":reference_mul",
        ":ruy",
        ":size_util",
        ":time",
        "//ruy/profiler",
    ] + ruy_test_ext_deps(),
)

ruy_benchmark(
    name = "benchmark",
    srcs = ["benchmark.cc"],
    copts = ruy_copts(),
    lhs_rhs_accum_dst = [
        ("f32", "f32", "f32", "f32"),
        ("u8", "u8", "i32", "u8"),
        ("i8", "i8", "i32", "u8"),
        ("i8", "i8", "i32", "i8"),
        ("u8", "u8", "i32", "i16"),
        ("i8", "i8", "i32", "i32"),
        ("i8", "i16", "i32", "i16"),
        ("i16", "i8", "i32", "i16"),
    ],
    deps = [
        ":test_lib",
        "//ruy/profiler:instrumentation",
    ],
)

ruy_test(
    name = "test_fast",
    srcs = ["test_fast.cc"],
    copts = ruy_copts(),
    lhs_rhs_accum_dst = [
        ("f32", "f32", "f32", "f32"),
        ("f64", "f32", "f64", "f32"),
        ("f32", "f64", "f64", "f64"),
        ("u8", "u8", "i32", "u8"),
        ("i8", "i8", "i32", "i8"),
        ("i8", "u8", "i32", "i8"),
        ("u8", "u8", "i32", "i16"),
        ("i8", "i8", "i32", "i32"),
        ("i8", "u8", "i32", "i32"),
        ("i8", "i16", "i32", "i16"),
        ("i16", "i8", "i32", "i16"),
    ],
    deps = [
        ":test_lib",
        "@com_google_googletest//:gtest_main",
    ],
)

ruy_test(
    name = "test_slow",
    srcs = ["test_slow.cc"],
    copts = ruy_copts(),
    lhs_rhs_accum_dst = [
        ("f32", "f32", "f32", "f32"),
        ("u8", "u8", "i32", "u8"),
        ("i8", "i8", "i32", "i8"),
        ("u8", "u8", "i32", "i16"),
        ("i8", "i8", "i32", "i32"),
        ("i8", "i16", "i32", "i16"),
        ("i16", "i8", "i32", "i16"),
    ],
    tags = ["slow"],
    deps = [
        ":test_lib",
        "@com_google_googletest//:gtest_main",
    ],
)

cc_test(
    name = "test_overflow_dst_zero_point",
    srcs = [
        "test_overflow_dst_zero_point.cc",
    ],
    copts = ruy_copts(),
    deps = [
        ":gtest_wrapper",
        ":matrix",
        ":path",
        ":ruy",
        ":test_lib",
        ":tune",
    ],
)

bzl_library(
    name = "ruy_test_ext.oss_bzl",
    srcs = ["ruy_test_ext.oss.bzl"],
    visibility = ["//visibility:private"],
)

bzl_library(
    name = "ruy_test_bzl",
    srcs = ["ruy_test.bzl"],
    visibility = ["//visibility:private"],
)

bzl_library(
    name = "build_defs.oss_bzl",
    srcs = ["build_defs.oss.bzl"],
    visibility = ["//visibility:private"],
)

bzl_library(
    name = "build_defs_bzl",
    srcs = ["build_defs.bzl"],
    visibility = ["//visibility:private"],
)
