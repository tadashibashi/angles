# ANGLES

Precompiled ANGLE libs and headers wrapped in a convenient CMake interface library.

For special platforms like Emscripten, compile options will be set to accommodate OpenGL ES3
without ANGLE.

Intended to help make development builds convenient.
It may be good to compile ANGLE from source for builds tailored for production.

Current supported platforms:
- MacOS 15 Silicon
- Emscripten
