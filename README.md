<h1 align='center'>Code_Runner</h1>

<h4 align='center'>🔥 Code Runner for Neovim written in pure lua 🔥</h4>

<p align='center'><img src='https://i.ibb.co/1njTRTL/ezgif-com-video-to-gif.gif'></p>

## Introduction

When I was still in college it was common to try multiple programming languages, at that time I used vscode that with a single plugin allowed me to run many programming languages, I left the ballast that are electron apps and switched to neovim, I searched the Internet and finally i found a lot of plugins, but none of them i liked (maybe i didn't search well), so i started adding autocmds like i don't have a tomorrow, this worked fine but this is lazy (maybe it will work for you, if you only programs in one or three languages maximum). So I decided to make this plugin and since the migration of my commands was very fast, it was just copy and paste and everything worked. Currently I don't test many languages anymore and work in the professional environment, but this plugin is still my swiss army knife.

## Requirements

- Neovim (>= 0.8)

## Install

- With [lazy.nvim](https://github.com/folke/lazy.nvim)

```lua
require("lazy").setup({
  { "CRAG666/code_runner.nvim", config = true },
}
```

- With [packer.nvim](https://github.com/wbthomason/packer.nvim)

```lua
use 'CRAG666/code_runner.nvim'
```

- With [paq-nvim](https://github.com/savq/paq-nvim)

```lua
require "paq"{ 'CRAG666/code_runner.nvim'; }
```
Consider using [CRAG666/betterTerm.nvim](https://github.com/CRAG666/betterTerm.nvim)

## Features

> **Note**  
> If you want implement a new feature open an issue to know if it is worth implementing it and if there are people interested.

- Toggle runner
- Reload task on the fly
- Run code in a Float window
- Run code in a tab
- Run code in a split
- Run code with toggleTerm
- Assign commands to projects without files in the root of the project
- Run project commands in different modes for a per projects base

## Setup

This plugin can be configured either in lua, with the `setup` function, or with json files for interopability between this plugin and the [original code runner](https://github.com/formulahendry/vscode-code-runner) vscode plugin.

See also:
* [options](#options): options that can be passed to setup function;
* [filetype](#setup-filetypes): filetype specific commands;
* [project](#setup-projects): project specific commands.
  * [project-parameters](#projects-parameters): project specific commands.

### Minimal example

#### Lua

```lua
require('code_runner').setup({
  filetype = {
    java = {
      "cd $dir &&",
      "javac $fileName &&",
      "java $fileNameWithoutExt"
    },
    python = "python3 -u",
    typescript = "deno run",
    rust = {
      "cd $dir &&",
      "rustc $fileName &&",
      "$dir/$fileNameWithoutExt"
    },
  },
})
```

#### Json

> **Warning**  
> A common mistake is using relative paths instead of absolute paths in . Use absolute paths in configurations or else the plugin won't work, in case you like to use short or relative paths you can use something like this `vim.fn.expand('~/.config/nvim/project_manager.json')`

> **Note**
> If you want to change were the code is displayed you need to specify the [`mode`]() attribute in the setup `function`

```lua
-- this is a config example
require('code_runner').setup {
  filetype_path = vim.fn.expand('~/.config/nvim/code_runner.json'),
  project_path = vim.fn.expand('~/.config/nvim/project_manager.json')
}
```

## Commands

> **Note**  
> To check what modes ore supported see [mode parameter](#setup-1).

All run commands allow restart. So, for example, if you use a command that does not have hot reload, you can call a command again and it will close the previous one and start again.

- `:RunCode`: Runs based on file type, first checking if belongs to project, then if filetype mapping exists
- `:RunCode <A_key_here>`: Execute command from its key in current directory.
- `:RunFile <mode>`: Run the current file (optionally you can select an opening mode).
- `:RunProject <mode>`: Run the current project(If you are in a project otherwise you will not do anything,).
- `:RunClose`: Close runner
- `:CRFiletype` - Open json with supported files(Use only if you configured with json files).
- `:CRProjects` - Open json with list of projects(Use only if you configured with json files).

Recommended mappings:
```lua
vim.keymap.set('n', '<leader>r', ':RunCode<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>rf', ':RunFile<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>rft', ':RunFile tab<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>rp', ':RunProject<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>rc', ':RunClose<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>crf', ':CRFiletype<CR>', { noremap = true, silent = false })
vim.keymap.set('n', '<leader>crp', ':CRProjects<CR>', { noremap = true, silent = false })
```
### Variables

> **Note**  
> If you don't want to use the plugin specific variables you can use vim [filename-modifiers](https://neovim.io/doc/user/cmdline.html#filename-modifiers).

This uses some special keyword to that means different things. This is do mainly for be compatible with the original vscode plugin.

The available variables are the following:
- `file`: path to current open file (e.g. `/home/user/current_dir/current_file.ext`
- `fileName`: filename of current open file (e.g. `current_file.ext`)
- `fileNameWithoutExt`: filename without extension of current file (e.g. `current_file`)
- `dir`: path to directory of current open file (e.g. `/home/user/current_dir`)
- `end`: finish the command (it is useful for commands that do not require final autocompletion)

## Setup Filetypes

> **Note**  
> The commands are runned in a shell. This means that you can't run neovim commands with [this](https://github.com/CRAG666/code_runner.nvim/issues/59).

### Lua

The filetype table can take either a `string`, a `table` or a `function`.

```lua
-- in setup function
filetype = {
  java = { "cd $dir &&", "javac $fileName &&", "java $fileNameWithoutExt" },
  python = "python3 -u",
  typescript = "deno run",
  rust = { "cd $dir &&",
    "rustc $fileName &&",
    "$dir/$fileNameWithoutExt"
  },
  cs = function(...)
    local root_dir = require("lspconfig").util.root_pattern "*.csproj"(vim.loop.cwd())
    return "cd " .. root_dir .. " && dotnet run$end"
  end,
},
```

If you want to add some other language or some other command follow this structure `key = commans`.

### Json

> **Note**  
> In Json you can only pass the commands as a string

The equivalent for your json filetype file is:
```json
{
  "java": "cd $dir && javac $fileName && java $fileNameWithoutExt",
  "python": "python3 -u",
  "typescript": "deno run",
  "rust": "cd $dir && rustc $fileName && $dir/$fileNameWithoutExt"
}
```

If you want to add some other language or some other command follow this structure `key: commans`.

## Setup Projects

There are 3 main ways to configure the execution of a project (found in the example.)

1. Use the default command defined in the filetypes file (see `:CRFiletype`or check your config). In order to do that it is necessary to define file_name.
2. Use a different command than the one set in `CRFiletype` or your config. In this case, the file_name and command must be provided.
3. Use a command to run the project. It is only necessary to define command (You do not need to write navigate to the root of the project, because automatically the plugin is located in the root of the project).

Also see [project parameters](#projects) to set correctly your project commands.

#### Lua

```lua
project = {
  ["~/python/intel_2021_1"] = {
    name = "Intel Course 2021",
    description = "Simple python project",
    file_name = "POO/main.py"
  },
  ["~/deno/example"] = {
    name = "ExapleDeno",
    description = "Project with deno using other command",
    file_name = "http/main.ts",
    command = "deno run --allow-net"
  },
  ["~/cpp/example"] = {
    name = "ExapleCpp",
    description = "Project with make file",
    command = "make buid && cd buid/ && ./compiled_file"
  }
},
```

#### Json

```json
{
  "~/python/intel_2021_1": {
    "name": "Intel Course 2021",
    "description": "Simple python project",
    "file_name": "POO/main.py"
  },
  "~/deno/example": {
    "name": "ExapleDeno",
    "description": "Project with deno using other command",
    "file_name": "http/main.ts",
    "command": "deno run --allow-net"
  },
  "~/cpp/example": {
    "name": "ExapleCpp",
    "description": "Project with make file",
    "command": "make buid && cd buid/ && ./compiled_file"
  }
}
```

## Parameters

### Setup

This are the the configuration option you can pass to the `setup` function. To see the default values see: [`code_runner.nvim/lua/code_runner/options`](https://github.com/CRAG666/code_runner.nvim/blob/main/lua/code_runner/options.lua).

Parameters:
- `mode`: Mode in which you want to run. Are supported: "toggle", "float", "tab", "toggleterm" (type: `bool`)
- `focus`: Focus on runner window. Only works on toggle, term and tab mode (type: `bool`)
- `startinsert`: init in insert mode (type: `bool`)
- `term`: Configurations for the integrated terminal
  - `position`: terminal position consult `:h windows` for options (type: `string`)
  - `size`: Size of the terminal window (type: `uint` | `float`)
- `float`: Configurations for the float win
  - `border`: Window border see `:h nvim_open_win` (type: `string`)
  - `height`
  - `width`
  - `x`
  - `y`
  - `border_hl`: (type: `string`)
- `before_run_filetype`: Execute before executing a file (type: `func`)
- `filetype`: If you prefer to use lua instead of json files, you can add your settings by file type here (type: `table`)
- `filetype_path`: Absolute path to json file config (type: `absolute paths`)
- `project`: If you prefer to use lua instead of json files, you can add your settings by project here (type: `table`)
- `project_path`: Absolute path to json file config (type: `absolute paths`)

### Projects

> **Warning**  
> Avoid using all the parameters at the same time. The correct way to use them is shown in the example and described above.

> **Note**  
> Don't forget to name your projects because if you don't do so code runner will fail as it uses the name for the buffer name

Parameters:
- `name`: Project name
- `description`: Project description
- `file_name`: Filename relative to root path
- `command`: Command to run the project. It is possible to use variables exactly the same as we would in [`CRFiletype`](#commands).

## Integration with other plugins

### API

These functions could be useful if you intend to create plugins around code_runner, currently only the file type and current project commands can be accessed respectively

```lua
require("code_runner.commands").get_filetype_command() -- get the current command for this filetype
require("code_runner.commands").get_project_command() -- get the current command for this project
```

### Harpoon

You can directly integrate this plugin with [ThePrimeagen/harpoon](https://github.com/ThePrimeagen/harpoon) the way to do it is through command queries, harpoon allows the command to be sent to a terminal, below it is shown how to use harpoon term together with code_runner.nvim:

```lua
require("harpoon.term").sendCommand(1, require("code_runner.commands").get_filetype_command() .. "\n")
```

### betterTerm

You can directly integrate this plugin with [CRAG666/betterTerm.nvim](https://github.com/CRAG666/betterTerm.nvim) the way to do it is through command queries, `betterTerm` allows the command to be sent to a terminal, below it is shown how to use `betterTerm` term together with code_runner.nvim:

```lua
-- Send command to term one
require("betterTerm").send(require("code_runner.commands").get_filetype_command(), 1)
```

## Inspirations and thanks

- The idea of this project comes from the vscode plugin [code_runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner) You can even copy your configuration and pass it to this plugin, as they are the same in the way of defining commands associated with [filetypes](https://github.com/CRAG666/code_runner.nvim#add-support-for-more-file-types)
- [jaq-nvim](https://github.com/is0n/jaq-nvim) some ideas of how to execute commands were taken from this plugin, thank you very much.
- [FTerm.nvim](https://github.com/numToStr/FTerm.nvim) Much of how this README.md is structured was blatantly stolen from this plugin, thank you very much
- Thanks to all current and future collaborators, without their contributions this plugin would not be what it is today

## Screenshots

<p align='center'><img src='https://i.ibb.co/JCg3tNd/ezgif-com-video-to-gif.gif'></p>
<p align='center'><img src='https://i.ibb.co/1njTRTL/ezgif-com-video-to-gif.gif'></p>
<p align='center'><img src='https://i.ibb.co/gFRhLgr/screen-1628272271.png'></p>

## Contributing

> **Note**  
> If you have any ideas to improve this project, do not hesitate to make a request, if problems arise, try to solve them and publish them. Don't be so picky I did this in one afternoon

Your help is needed to make this plugin the best of its kind, be free to contribute, criticize (don't be soft) or contribute ideas. All PRs are welcome.

# LICENCE

---

[MIT](https://github.com/CRAG666/code_runner.nvim/blob/main/LICENSE)
