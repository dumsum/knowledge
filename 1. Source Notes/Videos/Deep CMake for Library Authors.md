[Link](https://www.youtube.com/watch?v=m0DwB4OvDXk)
Craig Scott, CppCon 2019

## API Control
*What does the library provide?*

- don't expose things that aren't part of the API
- control with *symbol visibility*
- hide by default

```sh
-fvisibility=hidden
-fvilibility-inlines-hidden
```

```c++
__attribute__((visibility("default"))
```

```cmake
set(CMAKE_CXX_VISIBILITY_PRESET      hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN  YES)
```

## Library Consumers
*How might the library be used?*

## API Compatibility
*How does the library evolve?*

Consider [[Semantic Versioning]].
- MAJOR.MINOR.PATCH
- Shared Library symlinks to REAL LIBRARY:
	- NAME LINK for build-time linker (no version)
	- SONAME for run-time loader (MAJOR only)
	- REAL LIBRARY for humans, packages (full version)
```cmake
set_target_properties(
	Example   PROPERTIES
	SOVERSION 2
	VERSION   2.4.7
)
```

## Package Maintainers
*How might the library be packaged?*

