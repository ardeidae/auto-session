# 🗒️ AutoSession

AutoSession takes advantage of Neovim's existing session management capabilities to provide seamless automatic session management.

<img src="https://github.com/rmagatti/readme-assets/blob/main/auto-session-new-example.gif" width="800" />

[<img alt="GitHub Actions Workflow Status" src="https://img.shields.io/github/actions/workflow/status/rmagatti/auto-session/tests.yml?style=for-the-badge&label=tests">](https://github.com/rmagatti/auto-session/actions/workflows/tests.yml)

## 📦 Installation

[Lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
{
  'rmagatti/auto-session',
  lazy = false,
  dependencies = {
    'nvim-telescope/telescope.nvim', -- Only needed if you want to use session lens
  },
  opts = {
    suppressed_dirs = { '~/', '~/Projects', '~/Downloads', '/' },
    -- log_level = 'debug',
  }
}
```

[Packer.nvim](https://github.com/wbthomason/packer.nvim):

```lua
use {
  'rmagatti/auto-session',
  config = function()
    require("auto-session").setup {
      suppressed_dirs = { "~/", "~/Projects", "~/Downloads", "/"},
    }
  end
}
```

## 💡 Behaviour

1. When starting `nvim` with no arguments, AutoSession will try to restore an existing session for the current `cwd` if one exists.
2. When starting `nvim .` (or another directory), AutoSession will try to restore the session for that directory.
3. When starting `nvim some_file.txt` (or multiple files), by default, AutoSession won't do anything. See [argument handling](#argument-handling) for more details.
4. Even after starting `nvim` with a file argument, a session can still be manually restored by running `:SessionRestore`.
5. Any session saving and restoration takes into consideration the current working directory `cwd`.
6. When piping to `nvim`, e.g: `cat myfile | nvim`, AutoSession won't do anything.

:warning: Please note that if there are errors in your config, restoring the session might fail, if that happens, auto session will then disable auto saving for the current session.
Manually saving a session can still be done by calling `:SessionSave`.

## ⚙️ Configuration

Here are the default settings:

```lua
{
  enabled = true, -- Enables/disables auto creating, saving and restoring
  root_dir = vim.fn.stdpath "data" .. "/sessions/", -- Root dir where sessions will be stored
  auto_save = true, -- Enables/disables auto saving session on exit
  auto_restore = true, -- Enables/disables auto restoring session on start
  auto_create = true, -- Enables/disables auto creating new session files. Can take a function that should return true/false if a new session file should be created or not
  suppressed_dirs = nil, -- Suppress session restore/create in certain directories
  alloweded_dirs = nil, -- Allow session restore/create in certain directories
  auto_restore_last_session = false, -- On startup, loads the last saved session if session for cwd does not exist
  use_git_branch = false, -- Include git branch name in session name
  lazy_support = true, -- Automatically detect if Lazy.nvim is being used and wait until Lazy is done to make sure session is restored correctly. Does nothing if Lazy isn't being used. Can be disabled if a problem is suspected or for debugging
  bypass_save_filetypes = nil, -- List of file types to bypass auto save when the only buffer open is one of the file types listed, useful to ignore dashboards
  close_unsupported_windows = true, -- Close windows that aren't backed by normal file before autosaving a session
  args_allow_single_directory = true, -- Follow normal sesion save/load logic if launched with a single directory as the only argument
  args_allow_files_auto_save = false, -- Allow saving a session even when launched with a file argument (or multiple files/dirs). It does not load any existing session first. While you can just set this to true, you probably want to set it to a function that decides when to save a session when launched with file args. See documentation for more detail
  continue_restore_on_error = true, -- Keep loading the session even if there's an error. Set to false to get the line number of an error when loading a session
  cwd_change_handling = false, -- Follow cwd changes, saving a session before change and restoring after
  log_level = "error", -- Sets the log level of the plugin (debug, info, warn, error).

  session_lens = {
    load_on_setup = true, -- Initialize on startup (requires Telescope)
    theme_conf = { -- Pass through for Telescope theme options
      -- layout_config = { -- As one example, can change width/height of picker
      --   width = 0.8,    -- percent of window
      --   height = 0.5,
      -- },
    },
    previewer = false, -- File preview for session picker

    mappings = {
      -- Mode can be a string or a table, e.g. {"i", "n"} for both insert and normal mode
      delete_session = { "i", "<C-D>" },
      alternate_session = { "i", "<C-S>" },
    },

    session_control = {
      control_dir = vim.fn.stdpath "data" .. "/auto_session/", -- Auto session control dir, for control files, like alternating between two sessions with session-lens
      control_filename = "session_control.json", -- File name of the session control file
    },
  },
}
```

NOTE: Older configuration names are still currently supported and will be automatically translated to the names above. If you want to update your config to the new names, `:checkhealth auto-session` will show you your config using the new names.

#### Recommended sessionoptions config

For a better experience with the plugin overall using this config for `sessionoptions` is recommended.

**Lua**

```lua
vim.o.sessionoptions="blank,buffers,curdir,folds,help,tabpages,winsize,winpos,terminal,localoptions"
```

**VimL**

```viml
set sessionoptions+=winpos,terminal,folds
```

:warning: if you use [packer.nvim](https://github.com/wbthomason/packer.nvim)'s lazy loading feature, and you have the `options` value in your `sessionoptions` beware it might lead to weird behaviour with the lazy loading, especially around key-based lazy loading where keymaps are kept and thus the lazy loading mapping packer creates never gets set again.

## 📢 Commands

AutoSession exposes the following commands that can be used or mapped to any keybindings for manually saving and restoring sessions.

```viml
:SessionSave " saves a session based on the `cwd` in `auto_session_root_dir`
:SessionSave my_session " saves a session called `my_session` in `auto_session_root_dir`

:SessionRestore " restores a session based on the `cwd` from `auto_session_root_dir`
:SessionRestore my_session " restores `my_session` from `auto_session_root_dir`

:SessionDelete " deletes a session based on the `cwd` from `auto_session_root_dir`
:SessionDelete my_session " deletes `my_sesion` from `auto_session_root_dir`

:SessionDisableAutoSave " disables autosave
:SessionDisableAutoSave! " enables autosave (still does all checks in the config)
:SessionToggleAutoSave " toggles autosave

:SessionPurgeOrphaned " removes all orphaned sessions with no working directory left.

:SessionSearch " open a session picker, uses Telescope if installed, vim.ui.select otherwise

:Autosession search " open a vim.ui.select picker to choose a session to load.
:Autosession delete " open a vim.ui.select picker to choose a session to delete.
```

## 📖 More Configuration Details

### 🔭 Session Lens

You can use Telescope to see, load, and delete your sessions. It's enabled by default if you have Telescope, but here's the Lazy config that shows the configuration options:

```lua

{
  'rmagatti/auto-session',
  lazy = false,
  dependencies = {
    'nvim-telescope/telescope.nvim',
  },
  keys = {
    -- Will use Telescope if installed or a vim.ui.select picker otherwise
    { '<leader>wr', '<cmd>SessionSearch<CR>', desc = 'Session search' },
    { '<leader>ws', '<cmd>SessionSave<CR>', desc = 'Save session' },
    { '<leader>wa', '<cmd>SessionToggleAutoSave<CR>', desc = 'Toggle autosave' },
  },
  opts = {
    -- ⚠️ This will only work if Telescope.nvim is installed
    -- The following are already the default values, no need to provide them if these are already the settings you want.
    session_lens = {
      -- If load_on_setup is false, make sure you use `:SessionSearch` to open the picker as it will initialize everything first
      load_on_setup = true,
      previewer = false,
      mappings = {
        -- Mode can be a string or a table, e.g. {"i", "n"} for both insert and normal mode
        delete_session = { "i", "<C-D>" },
        alternate_session = { "i", "<C-S>" },
      },
      -- Can also set some Telescope picker options
      theme_conf = {
        border = true,
        -- layout_config = {
        --   width = 0.8, -- Can set width and height as percent of window
        --   height = 0.5,
        -- },
      },
    },
  }
}
```

You can use `:SessionSearch` to launch the session picker. If `load_on_setup = false`, `:SessionSearch` will initialize the Telescope extension when called. You can also use
`:Telescope session-lens` to launch the session picker but only if `load_on_setup = true` or you've previously called `SessionSearch`.

The following default keymaps are available when the session-lens picker is open:

- `<CR>` loads the currently highlighted session.
- `<C-S>` swaps to the previously opened session. This can give you a nice flow if you're constantly switching between two projects.
- `<C-D>` will delete the currently highlighted session. This makes it easy to keep the session list clean.

NOTE: If you previously installed `rmagatti/session-lens`, you should remove it from your config as it is no longer necessary.

### Preview

<img src="https://github.com/rmagatti/readme-assets/blob/main/session-lens.gif" width=800 />

### 📁 Current Working Directory

AutoSession can track `cwd` changes!

It's disabled by default, but when enabled it works as follows:

- DirChangedPre (before the cwd actually changes):
  - Save the current session
  - Clear all buffers `%bd!`. This guarantees buffers don't bleed to the
    next session.
  - Clear jumps. Also done so there is no bleeding between sessions.
  - Run the `pre_cwd_changed` hook
- DirChanged (after the cwd has changed):
  - Restore session using new cwd
  - Run the `post_cwd_changed` hook

Now when you changes the cwd with `:cd some/new/dir` AutoSession handles it gracefully, saving the current session so there aren't losses and loading the session for the upcoming cwd if it exists.

Hooks are available for custom actions _before_ and _after_ the `cwd` is changed. Here's the config for tracking cwd and a hook example:

```lua
require('auto-session').setup({
  suppressed_dirs = { '~/', '~/Projects', '~/Downloads', '/' },

  cwd_change_handling = true

  pre_cwd_changed_cmds = {
    "tabdo NERDTreeClose" -- Close NERDTree before saving session
  }

  post_cwd_changed_cmds = {
    function()
      require("lualine").refresh() -- example refreshing the lualine status line _after_ the cwd changes
    end
  }

})
```

### 🖥️Dashboards

If you use a dashboard, you probably don't want to try and save a session when just the dashboard is open. To avoid that, add your dashboard filetype to the bypass list as follows:

```lua
require('auto-session').setup({
  bypass_save_filetypes = { 'alpha', 'dashboard' } -- or whatever dashboard you use
})

```

### 🪝 Command Hooks

#### Command hooks are a list of commands that get executed at different stages of the session management lifecycle.

Command hooks exist in the format: {hook_name}

- `{pre_save}`: executes _before_ a session is saved
- `{save_extra}`: executes _after_ a session is saved, return string will save to `*x.vim`, reference `:help mks`
- `{post_save}`: executes _after_ a session is saved
- `{pre_restore}`: executes _before_ a session is restored
- `{post_restore}`: executes _after_ a session is restored
- `{pre_delete}`: executes _before_ a session is deleted
- `{post_delete}`: executes _after_ a session is deleted
- `{no_restore}`: executes _at_ `VimEnter` _when_ no session is restored
- `{pre_cwd_changed}`: executes _before_ a directory is changed (if `cwd_change_handling` is enabled)
- `{post_cwd_changed}`: executes _after_ a directory is changed (if `cwd_change_handling` is enabled)

Each hook is a table of vim commands or lua functions (or a mix of both):

```lua
require('auto-session').setup {
  -- {hook_name}_cmds = {"{hook_command1}", "{hook_command2}"}

  pre_save_cmds = {
    "tabdo NERDTreeClose" -- Close NERDTree before saving session
  },

  post_restore_cmds = {
    "someOtherVimCommand",
    function()
      -- Restore nvim-tree after a session is restored
      local nvim_tree_api = require('nvim-tree.api')
      nvim_tree_api.tree.open()
      nvim_tree_api.tree.change_root(vim.fn.getcwd())
      nvim_tree_api.tree.reload()
    end
  },

  save_extra_cmds = {
    function()
      return [[echo "hello world"]]
    end
  }
}
```

### ➖ Statusline

You can show the current session name in the statusline by using the function `current_session_name()`. With no arguments, it will return the full session name. For automatically created sessions that will be the path where the session was saved. If you only want the last directory in the path, you can call `current_session_name(true)`.

Here's an example using [Lualine](https://github.com/nvim-lualine/lualine.nvim):

```lua
require('lualine').setup{
  options = {
    theme = 'tokyonight',
  },
  sections = {
    lualine_c = {
      function()
        return require('auto-session.lib').current_session_name(true)
      end
    }
  }
}
```

<img width="1904" alt="Screen Shot 2021-10-30 at 3 58 57 PM" src="https://user-images.githubusercontent.com/2881382/139559478-8edefdb8-8254-42e7-a0f3-babd3dfd6ff2.png">

### ⏮️ Last Session

This optional feature enables the keeping track and loading of the last session.
The last session is only loaded at startup if there isn't already a session for the current working directory.
This feature can come in handy when starting Neovim from a GUI for example.

:warning: If the directory does not exist, default directory will be used and an error message will be printed.  
:warning: This feature is still experimental and as of right now it interferes with the plugin's ability to auto create new sessions when opening Neovim in a new directory.

```lua
require('auto-session').setup {
    auto_restore_last_session = true,
}
```

A quick workaround for inability to auto create new sessions is to conditionally enable last session.

```lua
require('auto-session').setup {
    auto_restore_last_session = vim.loop.cwd() == vim.loop.os_homedir(),
}
```

Now last session will be restored only when Neovim is launched in the home directory, which is usually right after launching the terminal or Neovim GUI clients.

### Conditionally creating a session

`auto_create` doesn't just have to be a boolean, it can also take a function that returns if a session should be created or not as part of auto saving. As one example, you could use this to only create new session files for git projects:

```lua

require('auto-session').setup({
  auto_create = function()
    local cmd = 'git rev-parse --is-inside-work-tree'
    return vim.fn.system(cmd) == 'true\n'
  end,
})

```

### 🗃️ Argument Handling

By default, when `nvim` is run with a single directory argument, AutoSession will try to restore the session for that directory. If `nvim` is run with multiple directories or any file arguments, AutoSession won't try to restore a session and won't auto-save a session on exit (if enabled). Those behaviors can be changed with these config parameters:

```lua
  args_allow_single_directory = true, -- boolean Follow normal sesion save/load logic if launched with a single directory as the only argument
  args_allow_files_auto_save = false, -- boolean|function Allow saving a session even when launched with a file argument (or multiple files/dirs). It does not load any existing session first. While you can just set this to true, you probably want to set it to a function that decides when to save a session when launched with file args. See documentation for more detail
```

For `args_allow_single_directory`, if you frequently use `netrw` to look at directories, you might want to add it to `bypass_save_filetypes` if you don't want to create a session for each directory you look at:

```lua
      bypass_save_filetypes = { 'netrw' }
```

If `args_allow_files_auto_save` is true, AutoSession won't load any session when `nvim` is launched with file argument(s) but it will save on exit. What's probably more useful is to set `args_allow_files_auto_save` to a function that returns true if a session should be saved and false otherwise. AutoSession will call that function on auto save when run with arguments. Here's one example config where it will save the session if at least two buffers are open after being launched with arguments:

```lua
require('auto-session').setup({
  args_allow_files_auto_save = function()
    local supported = 0

    local buffers = vim.api.nvim_list_bufs()
    for _, buf in ipairs(buffers) do
      -- Check if the buffer is valid and loaded
      if vim.api.nvim_buf_is_valid(buf) and vim.api.nvim_buf_is_loaded(buf) then
        local path = vim.api.nvim_buf_get_name(buf)
        if vim.fn.filereadable(path) ~= 0 then supported = supported + 1 end
      end
    end

    -- If we have more 2 or more supported buffers, save the session
    return supported >= 2
  end,
})
```

Another possibility is to only save the session if there are at least two windows with buffers backed by normal files:

```lua
require('auto-session').setup({
  args_allow_files_auto_save = function()
    local supported = 0

    local tabpages = vim.api.nvim_list_tabpages()
    for _, tabpage in ipairs(tabpages) do
      local windows = vim.api.nvim_tabpage_list_wins(tabpage)
      for _, window in ipairs(windows) do
        local buffer = vim.api.nvim_win_get_buf(window)
        local file_name = vim.api.nvim_buf_get_name(buffer)
        if vim.fn.filereadable(file_name) ~= 0 then supported = supported + 1 end
      end
    end

    -- If we have 2 or more windows with supported buffers, save the session
    return supported >= 2
  end,
})
```

### 🚫 Disabling the plugin

You might run into issues with Firenvim or another plugin and want to disable `auto_session` altogether based on some condition.
For this example, as to not try and save sessions for Firenvim, we disable the plugin if the `started_by_firenvim` variable is set.

```viml
if exists('g:started_by_firenvim')
  let g:auto_session_enabled = v:false
endif
```

One can also disable the plugin by setting the `enabled` option to false at startup.

```sh
nvim --cmd "let g:auto_session_enabled = v:false"
```

## 🚧 Troubleshooting

For troubleshooting refer to the [wiki page](https://github.com/rmagatti/auto-session/wiki/Troubleshooting).

# Compatibility

Neovim > 0.7

Tested with:

```
NVIM v0.7.2 - NVIM 0.10.1
```
