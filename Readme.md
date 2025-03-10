# Lushwal.nvim

This plugin uses [lush.nvim](https://github.com/rktjmp/lush.nvim) to generate a Neovim colorscheme based on your current [pywal](https://github.com/dylanaraps/pywal) colors. Given that it reads a JSON file each time you start, the plugin can also use [shipwright.nvim](https://github.com/rktjmp/shipwright.nvim) to compile a Vimscript version of your colorscheme for faster startup.


## Installation

### `packer.nvim`

```lua
use({
	"oncomouse/lushwal",
	requires = { { "rktjmp/lush.nvim", opt = true }, { "rktjmp/shipwright.nvim", opt = true } },
})
```

You can use `opt=true` because this plugin calls `packadd` when it needs lush and/or shipwright.

### `lazy.nvim`

```lua
{
	"oncomouse/lushwal.nvim",
	cmd = { "LushwalCompile" },
	dependencies = {
		{ "rktjmp/lush.nvim" },
		{ "rktjmp/shipwright.nvim" },
	},
	lazy = false,
}
```

The `cmd` key is necessary so that lazy.nvim knows to recognize the `:LushwalCompile` user command.

## Usage

Set `colorscheme lushwal` somewhere in your `init.lua` or `init.vim`.

If you are using the caching feature (on by default), [shipwright.nvim](https://github.com/rktjmp/shipwright.nvim) will take care of everything else.

### With pywal

`lushwal.nvim` detects changes to the pywal theme both at startup and while Neovim is running.

To synchronize between any Neovim processes that might be running on the system, `lushwal.nvim` needs a perl interpreter in order to [flock](https://linux.die.net/man/2/flock) the new colorscheme file. If `perl` isn't found, `lushwal.nvim` tries its best, but the theme can get out of sync.

### With other plugins

Use `require("lushwal").add_reload_hook(hook)` to add callbacks that `lushwal.nvim` will run when new colorschemes are generated. The argument (`hook`) passed to `add_reload_hook` can be either a string (evaluated with `vim.cmd`) or a function.

## Configuration

Set `vim.g.lushwal_configuration` (in Lua) or `g:lushwal_configuration` (in Vimscript) to override any of the following default values:

~~~lua
{
	transparent_background = false,
	compile_to_vimscript = true,
	terminal_colors = false,
	color_overrides = nil,
	wal_path = vim.fn.expand("XDG_CACHE_HOME/wal/colors.json"),
	addons = {
		ale = false,
		barbar = false,
		bufferline_nvim = false,
		coc_nvim = false,
		dashboard_nvim = false,
		fern_vim = false,
		gina = false,
		gitsigns_nvim = false,
		hop_nvim = false,
		hydra_nvim = false,
		indent_blankline_nvim = false,
		lightspeed_nvim = false,
		lspsaga_nvim = false,
		lsp_trouble_nvim = false,
		lualine = false,
		markdown = false,
		mini_nvim = false,
		native_lsp = true,
		neogit = false,
		neomake = false,
		nerdtree = false,
		nvim_cmp = false,
		nvim_tree_lua = false,
		nvim_ts_rainbow = false,
		semshi = false,
		telescope_nvim = false,
		treesitter = true,
		vim_dirvish = false,
		vim_gitgutter = false,
		vim_signify = false,
		vim_sneak = false,
		which_key_nvim = false,
	}
}
~~~

### Addons

lushwal supports a variety of popular plugins but most are disabled by default. The addons section contains a list of plugins that can be enabled. If you change your lushwal configuration, you will need to re-run `:LushwalCompile`.

By default, treesitter and native_lsp support are enabled.

### Color Overrides

Some color palettes may end up with strange tones for the 15 colors pywal defines (for instance, I use [Ayu Mirage](https://github.com/dempfi/ayu), which defines red as an orange tone and magenta as a red tone). If you would like to change any of these, you can set the `color_overrides` key in `vim.g.lushwal_configuration` to a function or a table.

Functions will receive the color object generated by lushwal.nvim and may return it changed however you like.

#### Example

As I mentioned, Ayu Mirage uses an orange tone for terminal red and a red tone for terminal magenta. To override Lushwal's settings, I use the following:

```lua
vim.g.lushwal_configuration = {
	color_overrides = function(colors)
		local overrides = {
			red = colors.color5,
			orange = colors.color1,
		}
		return vim.tbl_extend("force", colors, overrides)
	end,
	-- ...
}
```

#### Additional Colors

In addition to the standard ANSI terminal colors, Lushwal uses some custom colors: `grey`, `br_grey`, `orange`, `purple`, `pink`, `amaranth`, and `brown`. These are generated by Lushwal using Lush's color transformation API. You can use those in your `color_overrides` function, too. For instance, in the above example, I wanted to redefine `amaranth`, because `red` is also being redefined. The full version of the function, with `amaranth` redefined, looks like this:

```lua
vim.g.lushwal_configuration = {
	color_overrides = function(colors)
		local overrides = {
			red = colors.color5,
			orange = colors.color1,
			amaranth = colors.color5.mix(colors.color4, 34).saturate(46).darken(5),
		}
		return vim.tbl_extend("force", colors, overrides)
	end,
	-- ...
}
```

The defaults for these generated colors look pretty good in the pywal themes I tried, but if you want to override them, here are their definitions:

```lua
{
	grey = color8.mix(color7, 30), -- Darker mid-grey
	br_grey = color8.mix(color7, 65), -- Mid-grey
	orange = color1.mix(color3, 50),
	purple = color4.rotate(65).li(45), -- Purple
	pink = color4.rotate(65).li(45).mix(color5, 50), -- Pink
	amaranth = color1.mix(color4, 34).saturate(46).darken(5),
	brown = color1.mix(color5, 15), -- Brown
}
```

## Source Material

Plugins I don't use (which are most of the supported plugins) are adapted from [catppuccin](https://github.com/catppuccin/nvim), which has great plugin support and uses a configuration system similar to the one used by pywal.

If you use Lushwal and use one of the plugins I've sourced from catppuccin looks weird, please submit a PR.
