# Objective

Create a neovim based IDE environment using built-in features only. No plugin manager nor plugins other than `lspconfig`.

# Knowledge needed

Nothing except know how to use vim/nvim as an end user.

I didn't have written any Lua code until this project.

# History

I have used vim for years doing kernel and btrfs-progs development, but it's not always as easy as nowadays.

## Ctags era

In the old days before LSP, it was pretty bad, code lookup was all done through [ctags](https://en.wikipedia.org/wiki/Ctags).
But ctags is not compiler, it can not understand a lot of macros.

E.g. function `btrfs_set_file_extent_disk_bytenr()` inside `btrfs_insert_hole_extent()` function from `fs/btrfs/file-item.c` file.

If using ctags, it will return no declaration nor definition.


## Early days with LSP

If my memory is still correct, [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) and [vim-lsp](https://github.com/prabirshrestha/vim-lsp)
were the two early vim LSP clients I tried.

I didn't have a good experience with YCM either, I guess it was related to my incorrect setup at the time, but IIRC
at the early days it was not even a full LSP client, and still a very complex system to setup, involving a lot of
scripts (python?) out of vim.

Vim-lsp is much better experience, but at that time I still need to do a per-project LSP server setup, so that different projects
can use their own `compile_commands.json` file.
Furthermore at that time, not a lot of projects are ignoring `compile_commands.json`.
E.g. kernel gitignore for `compile_commands.json` was only [added](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=26c4c71bcd9a9f2baf8334995b31f718854f7f42) in 2019.

I had to maintain a centralized `compile_commands.d`
directory for all projects, and manually add entries for each new projects.

But I still remember the joy when `C-]` properly jumps to the `BTRFS_SETGET_FUNCS()` declaring accessors for `btrfs_set_file_extent_disk_bytes()`.

## Problems showing up
However good days never last, for large structures (e.g. `btrfs_fs_info`) the members of the structure is either truncated
during auto-completion, or can take a long time for certain LSP parsing.

It was more or less acceptable when developing locally with a beefy PC, but everything changed when I got an RPI (2?) SBC.
Now arm is new the hot platform, trying to run Archlinux with upstream kernel is my latest experiment.

And I didn't know why but I choose to edit the code on RPI (2 or 3?) and compile the kernel natively, suddenly I got no
LSP support.

When setting up the environment again, the old problems just show up again:

- Even slower completion 
- Centralized `compile_commands.d` directory
- Mim config that depends on all kinds of plugins
- Various regressions between different plugins

  Now my pc and sbc are having different versions (hey, who will
  try to update their plugins when they all work fine?) and behaviors.

And all above problems are also preventing me from spread the enlightenment of LSP.

## The new hope
Then the project neovim came, with Lua as the main script language. Well, the script language doesn't make that much difference,
I can not read vim script nor lua, so they are the same to me mostly.

But the important thing is, neovim has built-in LSP client support, and with the help of Lua it's also way faster parsing LSP info.

Then I jumped the ship of Vim and changed to neovim.

But unfortunately there are still problems:

- Centralized `compile_commands.d` directory

  This time it's mostly on me, I was passing other clangd options to work around various
  quirks anway, thus I keept the per-project clangd setup and point the search path to the
  old `compile_commands.d` directory, even I know this is not expandable.

- Still all kinds of nvim plugins, along with less vim plugins

  The biggest reason for the extra plugins were the missing of snippet engine.
  At that time I was using [LuaSnip](https://github.com/L3MON4D3/LuaSnip) alone with
  [cmp](https://github.com/hrsh7th/nvim-cmp) and all kinds of extra plugins from [hrsh7th](https://github.com/hrsh7th).

  I think all those plugins are just to make auto-completion work, because that's the only
  working setup to an end user without Lua/Vim-script experience.
  The only changes that I can do is to change the keymap, and the keymay change didn't even always work.

Hey, at least the setup worked (and worked for a long time), and it is way faster than the old setup.
And I haven't touched it for quite some time.

## The simplest one to rule them all

Now neovim 0.11 has integrated everything I needed for simple LSP based auto-completion:

- LSP client
- Snippet engine
- Auto-completion

Finally we're at the beginning of the adventure. To build the simplest neovim IDE, with the minimal amount of
init.lua.
And this time, NO MORE DAMN PLUGINS!! (*)

(*): Except [lspconfig](https://github.com/neovim/nvim-lspconfig)
