# What is this project?

I got tired of paying for Evernote and searched endlessly for a note system that was flexible and extremely fast. I wanted it all: plaintext markdown notes in simple folders with fulltext search. Yet everything out there seemed to compromise in one of these areas.

## Getting Started

This project contains only two simple things which are contained in this readme:

1. An append to vimrc that will install FZF and ripgrep for you
2. A shell alias and functions to wrap those tools into something usable for notes

## Installing

You must first install [ripgrep](https://github.com/BurntSushi/ripgrep) and [fzf](https://github.com/junegunn/fzf). Below is the easiest way to do that.

1. Paste this section at the top of your vimrc, which will cause vim to install ripgrep

```vim
""" Auto Installation credit to https://github.com/bag-man/dotfiles
if !empty(glob("~/.fzf/bin/fzf"))
    if empty(glob("~/.fzf/bin/rg"))
      silent !curl -fLo /tmp/rg.tar.gz
            \ https://github.com/BurntSushi/ripgrep/releases/download/0.7.1/ripgrep-0.7.1-x86_64-unknown-linux-musl.tar.gz
      silent !tar xzvf /tmp/rg.tar.gz --directory /tmp
      silent !cp /tmp/ripgrep-0.7.1-x86_64-unknown-linux-musl/rg ~/.fzf/bin/rg
    endif
endif
```

2. Now paste this into your plugin section to install fzf. Change the beginning to reflect which plugin system you are using (vundle etc). I am using Plug by the same author as fzf, JuneGunn.

```vim
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
```

3. Run your plugin updater, for example `vim :PlugUpdate` for Plug

4. Lastly, copy the following into your shell rc file (.zshrc, .bashrc etc...). I have the NOTESDIR as the default for [Syncthing](https://syncthing.net/), which is awesome.

```bash
export NOTESDIR="$HOME/Sync/notes"

nn() {
  local note_name="$*"
  local note_date="`date +%F`"
  local note_ext="md"  
  if [[ $note_name == "" ]]
  then
    note_name="$note_date.$note_ext"
  fi
  mkdir -p $NOTESDIR
  vim -c 'startinsert' "$NOTESDIR/$note_name"
}

ns() {
  local file
  [ -z "$1" ] && echo "No argument supplied - Enter a term to search" && return 1
  file="$(rg --files-with-matches --no-ignore --ignore-case --hidden --no-heading --no-messages $1 $NOTESDIR | fzf --preview "head -100 {}")"
  if [[ -n $file ]]
  then
    vim $file
  fi
}

nl() {
  local files
  files="$(rg --files $NOTESDIR | fzf --preview "head -100 {}")"
  if [[ -n $files ]]; then
    vim $files
  fi
}
```


## Usage

Congratulations! You now have three functions

* Typing `nn` (short for New Note) will trigger vim to open straight into insert mode ready to type a note with today's date as the filename.
* Typing `nn FILENAME` will do the same thing but with the filename you specify.
* Typing `ns TERM` (short for Note Search) will ripgrep *inside* your text files for a search term, then you will visually see the results and choose to open one in vim.
* Typing `nl` (short for Note List) will bring up a visual list of all your notes for you to open one in vim
