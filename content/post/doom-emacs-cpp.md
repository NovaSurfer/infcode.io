---
title: "Doom emacs for C++"
date: 2023-01-17T02:42:31+01:00
categories:
  - C++
tags:
  - c++
---

For C++ I mainly use CLion but sometimes, especially when using a laptop I want something simpler.
So I tried to setup and configure Doom Emacs.  
My needs:
- Vim mode
- LSP support
- Interactive debugger

1. Install [Doom Emacs (see prerequirements)](https://github.com/doomemacs/doomemacs)
2. In `init.el` under `:tools` use `(debugger +lsp)` and under `:lang` set `(cc +lsp)`
3. In `config.el` add `(require 'dap-cpptools)`

That's all for the emacs configuration.
Now let's configure C++ project for auto-completion, reference findings, etc.

1. Generate [`compile_commands.json`](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html)  
   1.1 `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` for CMake  
   2.2 or by using [Bear](https://github.com/rizsotto/Bear)  
2. Add project by pressing **SPC-p-a**, open it by **SPC-p-p**, select file  
  There should be a message under modeline in minibuffer:
  ```
LSP :: Connected to [clangd:56087/starting].
LSP :: clangd:56087 initialized successfully in folders: (/home/max/src/scarecrow2d)
  ```
3. Run `M-x dap-cpptools-setup` **needed only once** (That's because I'm using vscode-cpptools, but there are other options listed [here](https://emacs-lsp.github.io/dap-mode/page/configuration/#native-debug-gdblldb))
4. Setup debug configuration  
   4.1 `M-x  dap-edit-debug-template`, here is template for my project
```
(dap-register-debug-template
  "cpptools::Run Configuration"
  (list :type "cppdbg"
        :request "launch"
        :name "cpptools::Run Configuration"
        :MIMode "gdb"
        :program "${workspaceFolder}/build/sc2d_client/sc2d_client"
        :cwd "${workspaceFolder}/build/"))
```  
   4.2 Eval buffer with `M-x eval-buffer`  
5. Open debugger `M-x SPC-o-d`

#### Things to discover
- dap-hydra
- add binds for debugging
- save debug-template in separate `.el` file
- resize debug windows
- setup build commands / cmake
   
#### Links
- [Doom emacs](https://github.com/doomemacs/doomemacs)
- [Lsp-mode](https://emacs-lsp.github.io/lsp-mode/)
- [Dap-mode](https://emacs-lsp.github.io/dap-mode/)
- [MS Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/)
- [MS Language Server Protocol](https://microsoft.github.io/language-server-protocol/)

