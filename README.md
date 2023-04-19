This repository illustrates two Python Bazel workspaces that each define a `core` package. Using
the standard approach to importing, eg `import core`, will fail due to shadowing (illustrated in
`shadowed_with_workspace/outer` by running `bazel run //core`.)

This has historically been fixable by using import paths prefixed with the workspace name. This is
illustated in `with_workspace/outer` by running `bazel run //core`.

However, with bzlmod there are two challenges that break this:

* The outer repository is now contained in a folder named `_main`, so a workspace can't simply
  import itself the module name as a prefix. As a module with Python code using a package that might
  be shadowed you have to choose between `import _main.core.impl` (allowing you to run code in your
  module but breaking people who depend on you, since when they run `_main` will refer to their
  module), `import dep.core.impl` (allowing people who depend on you to be able to import your code,
  but breaking yourself because when Bazel is run in your module it will be `_main` not `dep`), or
  always using relative paths (incredibly unwieldy and confusing if you have a large and deep
  repository.)
* `rules_python` ignores the generated `_repo_mapping` file for imports, and when
  `local_path_override` is used the folder is not `dep` but instead `dep~override`. This means that
  even if you resolve to always use relative paths for imports as a dependency, the outer module
  will never be able to even import you because `import dep~override` is not valid Python
  (nor would you even think to write that, you'd expect to need only `import dep.core` since the
  Bazel target is `@dep//core`)

