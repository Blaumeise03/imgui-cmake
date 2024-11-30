Dear ImGUI CMake Config
=======================

This is a CMake configuration for the [Dear ImGUI](https://github.com/ocornut/imgui) for easy installation.

Usage
```bash
git clone --recursive --shallow-submodules https://github.com/Blaumeise03/imgui-cmake
cd imgui-cmake
cmake -S . -B build -DIMGUI_BACKEND_GLFW=ON -DIMGUI_BACKEND_VULKAN=ON
cmake --build build --config release --target install
```

After that, CMake should be able to find the library
```cmake
find_package(imgui REQUIRED)
target_link_libraries(main PRIVATE imgui::imgui)
```