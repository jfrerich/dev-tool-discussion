
# Zsh  

<!-- vim-markdown-toc GFM -->

* [CLI tools](#cli-tools)
* [Plugins](#plugins)
* [Aliases](#aliases)
* [zsh functions](#zsh-functions)

<!-- vim-markdown-toc -->

## CLI tools

| CLI Tool Name                | Description                    | 
| -                            | -                              | 
| exa                          | colorized ls                   | 
| bat                          | `cat` w/ syntax highlighting   | 
| fd                           | alternative to `find`          | 
| fzf                          | fuzzy finder                   | 
| coreutils                    | gnu command line tools         | 
| has                          |                                | 
| howdoi                       |                                | 
| htop                         | `top` replacement              | 
| httpie                       |                                | 
| hub                          |                                | 
| mycli                        |                                | 
| richgo                       | enrich `go test` output        | 
| the_silver_searcher          | much faster `grep` alternative | 
| tldr                         |                                | 
| zsh-history-substring-search |                                | 
| zsh-syntax-highlighting      |                                | 

## Plugins

| Plugin Name              | Description | 
| -                        | -           | 
| git                      |             | 
| history-substring-search |             | 
| zsh-autosuggestions      |             | 
| colored-man-pages        |             | 
| colorize                 |             | 
| dircycle                 |             | 
| zsh-dircolors-solarized  |             | 
| you-should-use           |             | 

## Aliases

```zsh
alias md='make deploy'
alias mm='cd $HOME/go/src/github.com/mattermost'
alias mmd='cd $HOME/go/src/github.com/mattermost/docs' # edit docs @ docs.mattermost.com/ 
alias mmdd='cd $HOME/Sites/mattermost-developer-documentation' # edit docs @ developers.mattermost.com/ 
alias mmm='cd $HOME/go/src/github.com/mattermost/mattermost-mobile'
alias mms='cd $HOME/go/src/github.com/mattermost/mattermost-server'
alias mmw='cd $HOME/go/src/github.com/mattermost/mattermost-webapp'
alias mmr='cd $HOME/go/src/github.com/mattermost/mattermost-redux'
alias mmj='cd $HOME/go/src/github.com/mattermost/mattermost-plugin-jira'
alias mmb='cd $HOME/go/src/github.com/mattermost/mattermost-plugin-bitbucket'
alias mmg='cd $HOME/go/src/github.com/mattermost/mattermost-plugin-github'
alias mme='cd $HOME/go/src/github.com/mattermost/enterprise'
```

## zsh functions

| | Function       | Description                      |  
| -        | -              | -                                | 
| fzf      | fbr()          | git checkout branch              | 
|          | fcoc()         | git checkout commit              | 
|          | fcoc_preview() | git checkout commit with preview | 
|          | fshow()        | git commit browser               | 
|          | fco()          | git checkout branch / tag        | 

```zsh
# fcoc - checkout git commit
fcoc() {
  local commits commit
  commits=$(git log --pretty=oneline --abbrev-commit --reverse) &&
  commit=$(echo "$commits" | fzf --tac +s +m -e) &&
  git checkout $(echo "$commit" | sed "s/ .*//")
}
```

```zsh
# fbr - checkout git branch
fbr() {
  local branches branch
  branches=$(git --no-pager branch -vv) &&
  branch=$(echo "$branches" | fzf +m) &&
  git checkout $(echo "$branch" | awk '{print $1}' | sed "s/.* //")
}
```

```zsh
# fbrr - checkout git branch (including remote branches)
fbrr() {
  local branches branch
  branches=$(git branch --all | grep -v HEAD) &&
  branch=$(echo "$branches" |
           fzf-tmux -d $(( 2 + $(wc -l <<< "$branches") )) +m) &&
  git checkout $(echo "$branch" | sed "s/.* //" | sed "s#remotes/[^/]*/##")
}
```

```zsh
# fshow - git commit browser
fshow() {
  git log --graph --color=always \
      --format="%C(auto)%h%d %s %C(black)%C(bold)%cr" "$@" |
  fzf --ansi --no-sort --reverse --tiebreak=index --bind=ctrl-s:toggle-sort \
      --bind "ctrl-m:execute:
                (grep -o '[a-f0-9]\{7\}' | head -1 |
                xargs -I % sh -c 'git show --color=always % | less -R') << 'FZF-EOF'
                {}
FZF-EOF"
}
```

```zsh
# fco - checkout git branch/tag
fco() {
  local tags branches target
  tags=$(
    git tag | awk '{print "\x1b[31;1mtag\x1b[m\t" $1}') || return
  branches=$(
    git branch --all | grep -v HEAD             |
    sed "s/.* //"    | sed "s#remotes/[^/]*/##" |
    sort -u          | awk '{print "\x1b[34;1mbranch\x1b[m\t" $1}') || return
  target=$(
    (echo "$tags"; echo "$branches") |
    fzf-tmux -l30 -- --no-hscroll --ansi +m -d "\t" -n 2) || return
  git checkout $(echo "$target" | awk '{print $2}')
}
```

```zsh
# fco_preview - checkout git branch/tag, with a preview showing the commits between the tag/branch and HEAD
fco_preview() {
  local tags branches target
  tags=$(
git tag | awk '{print "\x1b[31;1mtag\x1b[m\t" $1}') || return
  branches=$(
git branch --all | grep -v HEAD |
sed "s/.* //" | sed "s#remotes/[^/]*/##" |
sort -u | awk '{print "\x1b[34;1mbranch\x1b[m\t" $1}') || return
  target=$(
(echo "$tags"; echo "$branches") |
    fzf --no-hscroll --no-multi --delimiter="\t" -n 2 \
        --ansi --preview="git log -200 --pretty=format:%s $(echo {+2..} |  sed 's/$/../' )" ) || return
  git checkout $(echo "$target" | awk '{print $2}')
}
```

```zsh
alias glNoGraph='git log --color=always --format="%C(auto)%h%d %s %C(black)%C(bold)%cr% C(auto)%an" "$@"'
_gitLogLineToHash="echo {} | grep -o '[a-f0-9]\{7\}' | head -1"
_viewGitLogLine="$_gitLogLineToHash | xargs -I % sh -c 'git show --color=always % | diff-so-fancy'"
```

```zsh
# fcoc_preview - checkout git commit with previews
fcoc_preview() {
  local commit
  commit=$( glNoGraph |
    fzf --no-sort --reverse --tiebreak=index --no-multi \
        --ansi --preview="$_viewGitLogLine" ) &&
  git checkout $(echo "$commit" | sed "s/ .*//")
}
```

```zsh
# fshow_preview - git commit browser with previews
fshow_preview() {
    glNoGraph |
        fzf --no-sort --reverse --tiebreak=index --no-multi \
            --ansi --preview="$_viewGitLogLine" \
                --header "enter to view, alt-y to copy hash" \
                --bind "enter:execute:$_viewGitLogLine   | less -R" \
                --bind "alt-y:execute:$_gitLogLineToHash | xclip"
}
```

```zsh
# fstash - easier way to deal with stashes
# type fstash to get a list of your stashes
# enter shows you the contents of the stash
# ctrl-d shows a diff of the stash against your current HEAD
# ctrl-b checks the stash out as a branch, for easier merging
fstash() {
  local out q k sha
  while out=$(
    git stash list --pretty="%C(yellow)%h %>(14)%Cgreen%cr %C(blue)%gs" |
    fzf --ansi --no-sort --query="$q" --print-query \
        --expect=ctrl-d,ctrl-b);
  do
    mapfile -t out <<< "$out"
    q="${out[0]}"
    k="${out[1]}"
    sha="${out[-1]}"
    sha="${sha%% *}"
    [[ -z "$sha" ]] && continue
    if [[ "$k" == 'ctrl-d' ]]; then
      git diff $sha
    elif [[ "$k" == 'ctrl-b' ]]; then
      git stash branch "stash-$sha" $sha
      break;
    else
      git stash show -p $sha
    fi
  done
}
```
