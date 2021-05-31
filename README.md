Example SaC Project
===================

This repo provides a very basic setup for compiling SaC modules and programs,
and can be used as a base for your own SaC project.

Note that most of the 'tricky' action happens in `src/CMakeLists.txt`, as here
we need to handle the interdependencies between the module and program.

The project use `cmake-common`, and so upon cloning this repo it is important
you call this before running `cmake`:
```sh
$ git submodule update --init
```
