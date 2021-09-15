# conda

The stuff Conda writes to your shell configuration file:

```
$ cat ~/.bash_profile
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/Users/jhalverson/software/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/Users/jhalverson/software/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

The definition of `__conda_setup` is accomplished by calling a Python script:

```
$ cat /Users/jhalverson/software/anaconda3/bin/conda
#!/Users/jhalverson/software/anaconda3/bin/python
# -*- coding: utf-8 -*-
import sys
# Before any more imports, leave cwd out of sys.path for internal 'conda shell.*' commands.
# see https://github.com/conda/conda/issues/6549
if len(sys.argv) > 1 and sys.argv[1].startswith('shell.') and sys.path and sys.path[0] == '':
    # The standard first entry in sys.path is an empty string,
    # and os.path.abspath('') expands to os.getcwd().
    del sys.path[0]

if __name__ == '__main__':
    from conda.cli import main
    sys.exit(main())
```

Below is the result of the command being executed:

```
$ __conda_setup="$('/Users/jhalverson/software/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
$ echo $__conda_setup
export CONDA_EXE='/Users/jhalverson/software/anaconda3/bin/conda' export _CE_M='' export _CE_CONDA='' export CONDA_PYTHON_EXE='/Users/jhalverson/software/anaconda3/bin/python' # Copyright (C) 2012 Anaconda, Inc # SPDX-License-Identifier: BSD-3-Clause __add_sys_prefix_to_path() { # In dev-mode CONDA_EXE is python.exe and on Windows # it is in a different relative location to condabin. if [ -n "${_CE_CONDA}" ] && [ -n "${WINDIR+x}" ]; then SYSP=$(\dirname "${CONDA_EXE}") else SYSP=$(\dirname "${CONDA_EXE}") SYSP=$(\dirname "${SYSP}") fi if [ -n "${WINDIR+x}" ]; then PATH="${SYSP}/bin:${PATH}" PATH="${SYSP}/Scripts:${PATH}" PATH="${SYSP}/Library/bin:${PATH}" PATH="${SYSP}/Library/usr/bin:${PATH}" PATH="${SYSP}/Library/mingw-w64/bin:${PATH}" PATH="${SYSP}:${PATH}" else PATH="${SYSP}/bin:${PATH}" fi \export PATH } __conda_exe() ( __add_sys_prefix_to_path "$CONDA_EXE" $_CE_M $_CE_CONDA "$@" ) __conda_hashr() { if [ -n "${ZSH_VERSION:+x}" ]; then \rehash elif [ -n "${POSH_VERSION:+x}" ]; then : # pass else \hash -r fi } __conda_activate() { if [ -n "${CONDA_PS1_BACKUP:+x}" ]; then # Handle transition from shell activated with conda <= 4.3 to a subsequent activation # after conda updated to >= 4.4. See issue #6173. PS1="$CONDA_PS1_BACKUP" \unset CONDA_PS1_BACKUP fi \local ask_conda ask_conda="$(PS1="${PS1:-}" __conda_exe shell.posix "$@")" || \return \eval "$ask_conda" __conda_hashr } __conda_reactivate() { \local ask_conda ask_conda="$(PS1="${PS1:-}" __conda_exe shell.posix reactivate)" || \return \eval "$ask_conda" __conda_hashr } conda() { \local cmd="${1-__missing__}" case "$cmd" in activate|deactivate) __conda_activate "$@" ;; install|update|upgrade|remove|uninstall) __conda_exe "$@" || \return __conda_reactivate ;; *) __conda_exe "$@" ;; esac } if [ -z "${CONDA_SHLVL+x}" ]; then \export CONDA_SHLVL=0 # In dev-mode CONDA_EXE is python.exe and on Windows # it is in a different relative location to condabin. if [ -n "${_CE_CONDA:+x}" ] && [ -n "${WINDIR+x}" ]; then PATH="$(\dirname "$CONDA_EXE")/condabin${PATH:+":${PATH}"}" else PATH="$(\dirname "$(\dirname "$CONDA_EXE")")/condabin${PATH:+":${PATH}"}" fi \export PATH # We're not allowing PS1 to be unbound. It must at least be set. # However, we're not exporting it, which can cause problems when starting a second shell # via a first shell (i.e. starting zsh from bash). if [ -z "${PS1+x}" ]; then PS1= fi fi conda activate base
```

On Mac, `$?` evaluates to 0 so `eval "$__conda_setup"` is ran. This is saying, if the previous command ran successfully then evaluate the output.


## Add return statement to ignore the `conda init` stuff

```
return
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/Users/jhalverson/software/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/Users/jhalverson/software/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

When then create a new shell and update `PATH`:

```
$ export PATH="/Users/jhalverson/software/anaconda3/bin:$PATH"
$ conda info --envs
# conda environments:
#
broken-links-env         /Users/jhalverson/Downloads/CONDA/envs/broken-links-env
base                  *  /Users/jhalverson/software/anaconda3

$ conda activate broken-links-env

CommandNotFoundError: Your shell has not been properly configured to use 'conda activate'.
To initialize your shell, run

    $ conda init <SHELL_NAME>

Currently supported shells are:
  - bash
  - fish
  - tcsh
  - xonsh
  - zsh
  - powershell

See 'conda init --help' for more information and options.

IMPORTANT: You may need to close and restart your shell after running 'conda init'.
```

### sourcing conda.sh

Here we also use the return statement to skip initialization except now we source the conda.sh script:

```
$ which conda
$
$ . "/Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh"
$ which conda
/Users/jhalverson/software/anaconda3/condabin/conda
$ which python
/usr/bin/python
$ cat /Users/jhalverson/software/anaconda3/etc/profile.d/conda.sh | wc -l
     100
jhalverson@~$ conda info --envs
# conda environments:
#
broken-links-env         /Users/jhalverson/Downloads/CONDA/envs/broken-links-env
base                  *  /Users/jhalverson/software/anaconda3

$ conda activate broken-links-env
(broken-links-env) $ 
```

We see there is quite a difference between sourcing conda.sh and just setting the PATH.
