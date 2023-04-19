load("@rules_python//python:defs.bzl", "py_binary")

py_binary(
    name = "core",
    main = "outer.py",
    srcs = glob(["*.py"]),
    deps = ["@dep//core"],
)

