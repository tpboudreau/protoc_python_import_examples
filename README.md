##### This repo contains example usage scenarios for the proposed protoc replace_import_package Python option

Four scenarios in which you might want to use the new `replace_import_package` option include:

1. Your protobuf files are all in the same directory and at least one imports another; you would like the generated Python files to be created in a package directory.  Since there is no directory structure for the input files, protoc ordinarily produces Python import statements of the form `import M as m` (relying on the PYTHONPATH or implicit relative import), but you want to instruct it to produce import statements of the form `from . import M as m` (denoting explicit relative import enforced in Python3).

> *Example command:* `protoc --proto_path ./protobuf --python_out ./generated --python_opt replace_import_package protobuf/*.proto`

> *Note:* the default mapping string for the new `replace_import_package=package_name` option is "|.", so `--python_opt replace_import_package` (with no mapping string) is the same as `--python_opt 'replace_import_package=|.'`.

2. Your protobuf files (a) are all in the same directory, (b) import from each other, and (c) belong to the same package, named say, "a.b" (perhaps expressed through a protobuf package statement); you would like the generated Python code to reflect the intended package structure -- that is, to be placed in a directory structure like '$PATH/a/b' with working imports.  Since there is no directory structure for the input files from which to infer the package structure and protoc **must** ignore the package declaration in the protobuf files, protoc will again produce import statements of the form `import M as m`. Here, you wish to instruct protoc to produce import statements of the form `from a.b import M as m`.

> *Example command:* `protoc --proto_path ./protobuf --python_out ./a/b --python_opt 'replace_import_package=|a.b' protobuf/*.proto`

> *Notes:* (1) the preferred package name supplied with the `import_from_package` option is **not** verified against the package name in the protobuf file, any package name may be supplied; (2) the `replace_import_package` option does **not** affect the number or location of the output files produced: generated files are still 1:1 with input files and the `python_out` flag still controls the location of the generated code.
> 
> In other words, in order to achieve the result desired in this scenario, it is the user's responsibility to ensure (a) that the path supplied to the `python_out` option corresponds to any absolute preferred package name supplied to the `replace_import_package` option, and (b) that the package name supplied to the `replace_import_package` option matches the package declaration in the protobuf file.

3. Your protobuf files are in a set of nested directories and at least one imports from another; you wish to relocate the nested structure present in the input directory to inside a python package.  In this case, because there is some directory structure for the input files, protoc will produce import statements of the form `from d import M as m`, where "d" is derived from the directory path found in the protobuf import statement.  You wish to instruct protoc to prepend the name of your target package to the imported-from package (in order to avoid having to extend your runtime PYTHONPATH to reach into the package) so that the result has the form `from p.d import M as m`, where "p" is the name of the package you are creating.

> *Example command:* `protoc --proto_path ./protobuf --python_out ./p --python_opt 'replace_import_package=d|p.d' protobuf/*.proto`

4. Your protobuf files are all in the same logical package, (say "project"), at least one imports another message file within that package, but it also imports a standard message file (say, "google/protobuf/timestamp.proto").  You would like to place the protoc generated Python code in a directory that reflects its logical package and have the intra-package imports work, while leaving the standard message import statements (which work as generated) alone.

> *Example command:* `protoc --proto_path ./protobuf --python_out ./project --python_opt 'replace_import_package=|project' protobuf/*.proto`  

---
