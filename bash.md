### Using alias from script
Aliases are not expanded when the shell is not interactive, unless the
expand_aliases shell option is set using shopt (see the description of
shopt under SHELL BUILTIN COMMANDS below).

Example:
```bash
#!/usr/bin/env bash

shopt -s expand_aliases
eval python --version
alias python='/usr/bin/python2'
eval python --version
```
[1]: https://www.thegeekdiary.com/how-to-make-alias-command-work-in-bash-script-or-bashrc-file/
