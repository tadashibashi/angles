# ANGLES

Precompiled ANGLE libs and headers wrapped in a convenient CMake interface library.

For special platforms like Emscripten, compile options will be set to accommodate OpenGL ES3
without ANGLE.

Intended to help make development builds convenient.
It may be good to compile ANGLE from source for builds tailored for production.

Development builds supported for:
- macOS arm64/x86_64
- Windows x64
- Emscripten
- iOS
- Android

## Usage

```cmake
# Link to the interface library
# Relevent include directories and libraries are added to your exe/lib
target_link_libraries(my_exe PRIVATE angles)

# Call this to copy the relevant libraries into the current binary directory
angles_copy_libs()
```

From C/C++
```C++
#include <angles.h>

// Call OpenGL ES3 code here

```

## Notes

### Angles Headers
The main angles header only includes ES3.0 for the sake of compatibility with WebGL. 
But other platforms may use higher versions, and this repo contains other version headers for convenience: 
e.g. `#include <GLES3/gl32.h>`

### Mobile

Android and iOS both fallback to OpenGL ES3. Ee should add linkage to pre-built ANGLE libraries at some point,
especially since OpenGLES is deprecated on iOS.
On an iOS simulator, performance drops significantly, but on hardware it appears to be fine.
