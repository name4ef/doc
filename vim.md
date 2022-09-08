### set target vimux panel for run commands
```vim
let g:VimuxRunnerIndex="1:0.1" " 1 - session; 0 - window; 1 - panel
```

### some notes of commands and other useful...
U   undo all changes in current line
:ls show list of edited files (active buffers)
:cc show message from compiler (after run :make)
:cn show next message from compiler

### remove comments in C/C++
```vim
:%! gcc -fpreprocessed -dD -E "%" 2>/dev/null
```

### remove blank lines
```vim
:g/^$/d
```
