# xdgize

A tool that generates minimal wrapper/shim shell scripts around external commands to "XDG"-ize them.

## Usage

`$ xdgize COMMAND`

This will create the script `"$XDG_DATA_HOME"/xdgize/bin/COMMAND` with the following content :

```bash
#!/usr/bin/env bash
set -euo pipefail
<environment variables>
<pre-exec>
exec -a 'COMMAND' '/path/to/actual/COMMAND' <pre-args> "$@" <post-args>
```

The output directory, shebang, script options, environment variables, any pre-exec lines, and pre- and post-args on the exec line can all be configured in the main configuration file and `COMMAND`-specific "rules" files.

The output directory `"$XDG_DATA_HOME"/xdgize/bin` can be pre-pended to your `$PATH` in your shell of choice's configuration files, e.g.:

```sh
PATH="$XDG_DATA_HOME"/xdgize/bin:$PATH
```

When `COMMAND` is run, the shim will run the actual command with any defined environment variables, pre-/post-args, etc.

> **Caution on `sudo` usage** - `sudo` inherits the user's `$PATH` by default. This means `sudo COMMAND` will run the shim instead of the actual command if the destination directory is pre-pended to your `$PATH`. This may or may not be the desired behavior. For more information, see the [sudoers.5](https://jlk.fjfi.cvut.cz/arch/manpages/man/core/sudo/sudoers.5.en) man page on the `secure_path` option.

`COMMAND`-specific rules can be added/modified as needed, and the tool can then (re-)create the shim.

## Configuration

The config file `"$XDG_CONFIG_HOME"/xdgize/config` is a bash script that is sourced (if it exists). A sample file is included in this repo.

`COMMAND`-specific rules files in `<script location>/rules.d/COMMAND` and `"$XDG_CONFIG_HOME"/xdgize/rules.d/COMMAND` are also sourced bash scripts (if they exist).

From these, you can configure:
* `dest` - output directory (default: `"$XDG_DATA_HOME"/xdgize/bin`)
* `env` - associative array of environment variables
* `header` - shebang + script options line(s)
* `pre_exec` - array; any extra lines wanted before the exec line
* `pre_args` - array; any `COMMAND`-specific arguments before the `"$@"` token
* `post_args` - array; any `COMMAND`-specific arguments after the `"$@"` token
* `rules_dirs` - array; where to find any extra user-supplied rules directories
* `out_file`, `quiet`, `verbose` - flags

A few `COMMAND`-specific rules are included in this repo. (More to be added.)

## Examples

`$ xdgize wget`

```bash
#!/usr/bin/env bash
set -euo pipefail
[[ -z "${WGETRC:-}" ]] && WGETRC="${XDG_CONFIG_HOME:-$HOME/.config}"/wgetrc && export WGETRC
exec -a 'wget' '/usr/bin/wget' --hsts-file="${XDG_CACHE_HOME:-$HOME/.cache}"/wget-hsts "$@"
```

## Tested On
* Linux, bash 5.0.x
* Linux, bash 4.3.x

## Ideas for Improvement
* Rewrite in higher level language
* Handle the current manual process of moving/renaming existing config files and folders (such as ~/.wgetrc, ~/go, etc.)

## Miscellaneous
* You can make your $HOME directory read-only to prevent non-XDG compliant apps from creating files and directories there-- `chmod u-w ~`

## Reference
* [XDG Base Directory Spec](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
* [Arch Wiki](https://wiki.archlinux.org/index.php/XDG_Base_Directory#Support)
* [Super User question on sudo and $PATH](https://superuser.com/questions/927512/how-to-set-path-for-sudo-commands)
