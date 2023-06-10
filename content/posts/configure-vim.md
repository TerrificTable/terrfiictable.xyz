---
title: "Configure Vim"
date: "2023-06-10"
tags: ["vim", "configuration"]
ShowToc: true
---

Vim is a simple terminal based text editor that can be configured.<br>
This config is also on [my github](https://github.com/TerrificTable/vim)
<br><br>

## Requirements
- vim
<br><br><br>

## Lets start
It is important to have a good configuration structure, so you can extend your configuration easily<br>

Vim uses a `.vimrc` file located in your home directory for configuration<br>
but using that file can be messy, especially when your configuration gets more and more complex<br>

So we will create a folder in `~/.config` called `vim` which we will store all our configurations in.<br>
to start, we should create a file called `init.vim` in this `~/.config/vim` directory<br>

But we have to import this file in vims configuration file, `.vimrc` like this:
```vim 
source ~/.config/vim/init.vim
```
This will "import" the file `~/.config/vim/init.vim`<br>


<br><br><br>
## Options

Now, lets set some simple options
```vim
" Set tab size to 2 spaces
set tabstop=2
set softtabstop=2
set shiftwidth=2
set expandtab

set number          " Enable line numbers
set relativenumber  " make line numbers relative to cursor

set autoindent      " Automatically indent
set smartindent     " Indent smartly
set nowrap          " dont wrap at end of line
set nobackup        " dont backup files (dont create .<filename>.swp files)
set scrolloff=10    " add 10 lines of padding before and after cursor

set hlsearch    " highlight searches

" lower priority for these file extensions when tab completing
set suffixes+=.info,.aux,.log,.dvi,.bbl,.out,.o,.lo

" enable syntax highlighting
syntax on
```
for clarity, create a new file `opts.vim` for these options, and "import" this file in the `init.vim` file like this: 
```vim
source ~/.config/vim/opts.vim
```

<br><br><br>
### Mappings

now lets create some new mappings, again in a new file `remap.vim` which is imported in `init.vim`
```vim
" Move selected lines up/down
vnoremap J :m '>+1<CR>gv=gv
vnoremap K :m '<-2<CR>gv=gv

" open new buffer/file
nnoremap <Leader>n :enew<CR>

" Escape terminal using ESC key
tnoremap <Esc> <C-\><C-n>

" :W to sudo save
command! W w !sudo tee % > /dev/null
```

<br>
we have set some options, and created some new keybinds<br>
there are a few more things we can do, lets start with plugins
<br><br><br>

## Plugins
I will use [vim-plug](https://github.com/junegunn/vim-plug) for plugins<br>
it can be installed using this command (from [vim-plug readme](https://github.com/junegunn/vim-plug#vim)):
```sh
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

we have installed `vim-plug`, now we can actually install plugins,<br>
for that i will create a new file `plugs.vim` that i will also import in init.vim<br>

to ensure `vim-plug` is installed i will use this snippet (also from vim-plug readme): 
```vim
let data_dir = has('nvim') ? stdpath('data') . '/site' : '~/.vim'
if empty(glob(data_dir . '/autoload/plug.vim'))
  silent execute '!curl -fLo '.data_dir.'/autoload/plug.vim --create-dirs  https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
  autocmd VimEnter * PlugInstall --sync | source $MYVIMRC
endif
```

we can install plugins now by using `call plug#begin()` and `call plug#end()`<br>
between these two calls we can install plugins using `Plug 'example/whatever'`<br>

### Theme

i will install the tokyonight theme like this: 
```vim
Plug 'ghifarit53/tokyonight-vim'
```

### tpope

I will also install some of tpope's plugins:
```vim
Plug 'tpope/vim-commentary' " Comment stuff
Plug 'tpope/vim-surround'   " Surround stuff
Plug 'tpope/vim-dispatch'   " Async Make
Plug 'tpope/vim-fugitive'   " Git
```
<br>


### FZF

and i will also install [fzf.vim](https://github.com/junegunn/fzf.vim)<br>
this requires [fzf](https://github.com/junegunn/fzf) to be installed on your system<br>

```vim
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } } " Fuzzy Finder
Plug 'junegunn/fzf.vim'                             " More fuzzy stuff
```
You will notice, there is a `{ 'do': { -> fzf#install() } }` this thing in the Plug call,<br>
but what does that actually do?<br>
it will run `fzf#install()` when the plugin is installed or updated<br>

### Languages
I also want some plugins for languages i use<br>
these are:
```vim
Plug 'fatih/vim-go' " GoLang support
Plug 'nsf/gocode', { 'rtp': 'vim' }
Plug 'Blackrush/vim-gocode'
Plug 'dgryski/vim-godef'
```

### UI
Until now, we've only installed plugins to make working with vim nicer<br>
now we will install some plugins for the ui<br>

```vim
Plug 'preservim/nerdtree'                " File Explorer
Plug 'itchyny/lightline.vim'             " Status bar
Plug 'mengelbrecht/lightline-bufferline' " Bufferline
Plug 'yggdroot/indentline'               " Indicate indents
```
I use [NERDTree](https://github.com/preservim/nerdtree) as a file explorer,<br>
[lightline.vim](https://github.com/itchyny/lightline.vim) as a status bar,<br>
[lightline-bufferline](https//.github.com/mengelbretch/lightline-bufferline) as a bufferline,<br>
and [indentline](https://github.com/yggdroot/indentline) to highlight indents<br>

### LSP
Now lets get some plugins for autocompleteion

```vim
Plug 'prabirshrestha/vim-lsp'              " Lsp
Plug 'mattn/vim-lsp-settings'              " More Lsp
Plug 'prabirshrestha/asyncomplete.vim'     " Autocomplete
Plug 'prabirshrestha/asyncomplete-lsp.vim' " AutoComplete with LSP
Plug 'dense-analysis/ale'                  " ALE
```


### Other
I also use some other plugins for snippets and automatically closing pairs
```vim
Plug 'hrsh7th/vim-vsnip'       " Snippets
Plug 'hrsh7th/vim-vsnip-integ' " Snippets integrations for lsp
Plug 'jiangmiao/auto-pairs'
```


## Plugin Configuration
But, these plugins have to be configured, i will create a new folder called `plugins` which will contain all my configs for plugins<br>

### LSP
lets start with configuring lsp:
```vim
function! s:on_lsp_buffer_enabled() abort
    setlocal omnifunc=lsp#complete
    setlocal signcolumn=yes
    if exists('+tagfunc') | setlocal tagfunc=lsp#tagfunc | endif
    nmap <buffer> gd <plug>(lsp-definition)
    nmap <buffer> gs <plug>(lsp-document-symbol-search)
    nmap <buffer> gS <plug>(lsp-workspace-symbol-search)
    nmap <buffer> gr <plug>(lsp-references)
    nmap <buffer> gi <plug>(lsp-implementation)
    nmap <buffer> gt <plug>(lsp-type-definition)
    nmap <buffer> <leader>rn <plug>(lsp-rename)
    nmap <buffer> [g <plug>(lsp-previous-diagnostic)
    nmap <buffer> ]g <plug>(lsp-next-diagnostic)
    nmap <buffer> K <plug>(lsp-hover)
    nnoremap <buffer> <expr><c-f> lsp#scroll(+4)
    nnoremap <buffer> <expr><c-d> lsp#scroll(-4)

    let g:lsp_format_sync_timeout = 1000
    autocmd! BufWritePre *.rs,*.go call execute('LspDocumentFormatSync')

    " refer to doc to add more commands
endfunction

augroup lsp_install
    au!
    " call s:on_lsp_buffer_enabled only for languages that has the server registered.
    autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
```


### Go
and the go plugins:
```vim
let g:go_highlight_types = 1

au FileType go nmap <leader>d <Plug>(go-def)
```
This is a very simple configuration, since it just sets `<leader>g` to call `go-def` when the filetype is go<br>
and making vim-go highlight types

### FZF
Lets configure fzf<br>
I will start with a small if statement to install fzf
```vim
if empty(glob("~/.fzf/install"))
  echo "Hello"
  execute '!git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install'
endif
```

and i will also make fzf display commits in a specific format
```vim
let g:fzf_commits_log_options = '--graph --color=always --format="%C(auto)%h%d %s %C(black)%C(bold)%cr"'
```

### Bufferline
lets configure our bufferline next
```vim
let g:lightline = {
      \ 'colorscheme': 'tokyonight',
      \ 'active': {
      \   'left': [ [ 'mode', 'paste' ], [ 'readonly', 'filename', 'modified' ] ]
      \ },
      \ 'tabline': {
      \   'left': [ ['buffers'] ],
      \   'right': [ ['close'] ]
      \ },
      \ 'component_expand': {
      \   'buffers': 'lightline#bufferline#buffers'
      \ },
      \ 'component_type': {
      \   'buffers': 'tabsel'
      \ }
      \ }
autocmd BufWritePost,TextChanged,TextChangedI * call lightline#update()
```

<br>and some keybinds to move between buffers
```vim
nmap <Leader><Tab>   <Plug>lightline#bufferline#go_next()
nmap <Leader><S-Tab> <Plug>lightline#bufferline#go_previous()
```

### Autocomplete
And finally lets configure the autocompleteion
```vim
" Tab to autocomplete
inoremap <expr> <Tab>   pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"
" inoremap <expr> <cr>    pumvisible() ? asyncomplete#close_popup() : "\<cr>"

" Enter key always insert new line
inoremap <expr> <cr> pumvisible() ? asyncomplete#close_popup() . "\<cr>" : "\<cr>"
```
