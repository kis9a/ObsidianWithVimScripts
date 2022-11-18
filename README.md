# vimsidian

![](https://img.shields.io/github/workflow/status/kis9a/vimsidian/test)&nbsp;&nbsp;<image height="18px" src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black"></image>&nbsp;&nbsp;<image height="18px" src="https://img.shields.io/badge/mac%20os-000000?style=for-the-badge&logo=apple&logoColor=white"></image>&nbsp;&nbsp;<image height="18px" src="https://www.vim.org/images/vim_shortcut.ico"></image>&nbsp;<image height="20px" src="https://obsidian.md/favicon.ico"></image></span>

Vim plugin to help edit [Obsidian](https://obsidian.md/) notes in Vim. Links, backlink resolution and jumps, search and completion and highlighting, daily notes. Even if you don't use [Obsidian](https://obsidian.md/), you can use it to manage your notes locally.

This plugin was made for me, but I hope it will be useful for those who want to easily edit [Obsidian](https://obsidian.md/) notes with vim as I do. If you have trouble using it, please post an [issues](https://github.com/kis9a/vimsidian/issues) below. Contributions, edits and distribution are also welcome.

<br/>
<image width="640px" src="https://raw.githubusercontent.com/kis9a/vimsidian/main/pictures/vimsidian.gif"></image>

## Motivation

In my earlier days, I used to divide notes in directories and manage note relationships by describing relative paths. However, I had trouble categorizing notes and spent a lot of time resolving note paths. I needed to achieve the following.

- Hierarchical structure is not suitable for classification of detailed personal knowledge.
- Create atomic notes and link notes to each other.
- Eliminate stress by unifying editing tasks and management of editing plugins in Vim.
- [[Link]] format to integrate into [Obsidian](https://obsidian.md/).

For me, [vimsidian](https://github.com/kis9a/vimsidian) is the plugin that solves these issues and complements my PKM (personal knowledge managment).

## Features

- Provide a completion function for note entry.
- Find and move the link under the cursor.
- Go to link before or afater current cursor.
- Create a note with the name of the link under the cursor.
- Search for notes and lines matching keywords.
- Display notes in the quickfix window containing the tag string under the cursor.
- Default syntax highlighting settings.
- Custom formatting of link spacing.
- Manage multiple `g:vimsidian_path` (Obsidian Vault).
- Daily note feature.
- Fewer dependencies.

## Initialization

### Requirements

- [ripgrep](https://github.com/BurntSushi/ripgrep) command
- [fd](https://github.com/sharkdp/fd) command

### Installation

Use your favorite plugin manager.

- Example: [vim-plug](https://github.com/junegunn/vim-plug)

```vim
Plug 'kis9a/vimsidian'
```

## Configuration

#### • Minimal

```vim
let g:vimsidian_path = $HOME . '/obsidian'
let g:vimsidian_enable_syntax_highlight = 1
let g:vimsidian_enable_complete_functions = 1
let g:vimsidian_complete_paths = [g:vimsidian_path . '/notes', g:vimsidian_path . '/images']
let $VIMSIDIAN_PATH_PATTERN = g:vimsidian_path . '/*.md'

function! s:vimsidianNewNoteAtNotesDirectory()
  execute ':VimsidianNewNote ' . g:vimsidian_path . '/notes'
endfunction

augroup vimsidian_augroup
  au!
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sl :VimsidianFdLinkedNotesByThisNote<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sg :VimsidianRgNotesLinkingThisNote<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> st :VimsidianRgTagMatches<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sm :VimsidianRgNotesWithMatchesInteractive<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> si :VimsidianRgLinesWithMatchesInteractive<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sF :VimsidianMoveToLink<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sk :VimsidianMoveToPreviousLink<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sj :VimsidianMoveToNextLink<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sN :call <SID>vimsidianNewNoteAtNotesDirectory()<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sO :VimsidianNewNoteInteractive<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sd :VimsidianDailyNote<CR>
  au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <buffer> sf :VimsidianFormatLink<CR>
augroup END
```

#### • Advance, ideas

<!--{{{ Multiple g:vimsidian_path (Vault) -->
<details open>
<summary>Multiple g:vimsidian_path</summary>

Multiple vimsidian_paths can be managed. The `$VIMSIDIAN_PATH_PATTERN` is the autocmd path-pattern (:h autocmd-pattern).
`g:vimsidian_path` variable is the path where notes and completion suggestions are searched.

```vim
let g:vimsidian_path_main = $HOME . '/obsidian'
let g:vimsidian_path_sub = $HOME . '/Nsidian'
let g:vimsidian_path = g:vimsidian_path_main
let $VIMSIDIAN_PATH_PATTERN = g:vimsidian_path_main . '/*.md,' . g:vimsidian_path_sub . '/*.md'

function! s:vimsidianSwitchVault()
  if stridx(expand('%:p'), g:vimsidian_path_main) !=# '-1'
    let g:vimsidian_path = g:vimsidian_path_main
    let g:vimsidian_complete_paths = [g:vimsidian_path_main . '/notes', g:vimsidian_path_main  . '/images']
  elseif stridx(expand('%:p'), g:vimsidian_path_sub) !=# '-1'
    let g:vimsidian_path = g:vimsidian_path_sub
    let g:vimsidian_complete_paths = [g:vimsidian_path_sub . '/Nnotes']
  endif
endfunction

augroup vimsidian_augroup
  au!
  au VimEnter,BufNewFile,BufReadPost *.md call s:vimsidianSwitchVault()
" ry ...
```

</details>
<!--}}}-->

<!--{{{ Change link open mode -->
<details close>
<summary>Change link open mode</summary>
<br/>

Change the way the buffer opens when opening a new note.

```vim
" default
let g:vimsidian_link_open_mode = 'e!'

" newtab
let g:vimsidian_link_open_mode = 'newtab'

" vsplit
let g:vimsidian_link_open_mode = 'vnew'

" hsplit
let g:vimsidian_link_open_mode = 'new'
```

</details>
<!--}}}-->

<!--{{{ Define the colors yourself -->
<details close>
<summary>Define the colors yourself</summary>
<br/>

```vim
let g:vimsidian_color_definition_use_default = 0

hi! def VimsidianLinkColor term=NONE ctermfg=47 guifg=#689d6a
hi! def VimsidianLinkMediaColor term=NONE ctermfg=142 guifg=#b8bb26
hi! def VimsidianLinkHeader term=NONE ctermfg=142 guifg=#b8bb26
hi! def VimsidianLinkBlock term=NONE ctermfg=142 guifg=#b8bb26
hi! def VimsidianTagColor term=NONE ctermfg=109 guifg=#076678
hi! def VimsidianPromptColor term=NONE ctermfg=109 guifg=#076678
```

</details>
<!--}}}-->

<!--{{{ Custom daily note template -->
<details close>
<summary>Custom daily note template</summary>
<br/>

```vim
let g:vimsidian_daily_note_path = g:vimsidian_path . "/daily/" . strftime("%Y-%m")
let g:vimsidian_daily_note_template_path = g:vimsidian_path . "/daily/Daily template.md"
```

The template file can use some parameters. (:h g:vimsidian_daily_note_template_path)

```
[[{{date}}]]

< [[{{previous_date}}]] | [[{{next_date}}]] >

[[{{year}}-{{month}}]] [[{{day}}]] [[{{day_of_week}}]]
```

</details>
<!--}}}-->

<!--{{{ Disable required commands checks to make plugins load a bit faster -->
<details close>
<summary>Disable required commands checks to make plugins load a bit faster</summary>
<br/>

```vim
" It is assumed that the following commands are already installed
" :echo g:vimsidian_required_commands
let g:vimsidian_check_required_commands_executable = 0
```

</details>
<!--}}}-->

<!--{{{ Refine complete_paths to speed up search for completions -->
<details close>
<summary>Refine complete_paths to speed up search for completions</summary>
<br/>

```vim
" vimsidian complete paths search use ls command
let g:vimsidian_complete_paths_search_use_fd = 0
let g:vimsidian_complete_paths = [g:vimsidian_path . '/notes/foo', g:vimsidian_path . '/notes/b']
```

</details>
<!--}}}-->

<!--{{{ Create new note same directory as current file -->
<details close>
<summary>Create new note same directory as current file</summary>
<br/>

```vim
function! s:vimsidianNewNoteSameDirectoryAsCurrentFile()
  execute ':VimsidianNewNote ' . fnamemodify(expand('%:p'), ':h')
endfunction

au BufNewFile,BufReadPost $VIMSIDIAN_PATH_PATTERN nn <silent> sC :call <SID>vimsidianNewNoteSameDirectoryAsCurrentFile()<CR>
```

</details>
<!--}}}-->

<!--{{{ Get link name under cursor -->
<details close>
<summary>Get link name under cursor</summary>
<br/>

```vim
function! s:getCurrentCursorLink()
  let link = vimsidian#unit#CursorLink()
  echo "Link name under cursor '" . link . "'"
endfunction
```

</details>
<!--}}}-->

## Help

See [Vim doc - Syntax highlight, Variables and Commands help](./doc/vimsidian.txt)

```vim
:h vimsidian
```

## Developments

If you contribute to this repository, please use the following tools for linting and testing.

### Linting

Use [vim-parser](https://github.com/ynkdir/vim-vimlparser), [vim-vimlint](https://github.com/syngan/vim-vimlint)

```
make init
make lint
```

When using [vint](https://github.com/Vimjas/vint)

```
make vint-int
make lint-vint
```

### Testing

Use [vim-themis](https://github.com/thinca/vim-themis/issues)

```
make init
make test
```

### CI

[.github/workflows/test.yml](./.github/workflows/test.yml)

## LICENSE

[WTFPL license - Do What The F\*ck You Want To Public License](./LICENSE.md)
