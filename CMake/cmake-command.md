# 常用指令

1. `cmake -B build -G "MinGW Makefiles"`

2. `cmake --build build --config Release --target install`

3. Generate a Project Buildsystem
   - `cmake [<options>] <path-to-source>`
   - `cmake [<options>] <path-to-existing-build>`
   - `cmake [<options>] -S <path-to-source> -B <path-to-build>`

4. Build a Project
    - `cmake --build <dir> [<options>] [-- <build-tool-options>]`

5. Install a Project
    - `cmake --install <dir> [<options>]`

6. Open a Project
    - `cmake --open <dir>`

7. Run a Script
    - `cmake [{-D <var>=<value>}...] -P <cmake-script-file>`

8. Run a Command-Line Tool
    - `cmake -E <command> [<options>]`

9. Run the Find-Package Tool
    - `cmake --find-package [<options>]`

10. View Help
    - `cmake --help[-<topic>]`
