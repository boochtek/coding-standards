
# Shell Coding Standards

These are our coding standards for shell scripts.
In general, that will mean Bash.
But we may encounter the occasional system that has some other POSIX-compliant shell.
And we have Zsh startup scripts to maintain as well.

- Unless there's a compelling reason, use Bash.
- If you can rely on the system having Bash, use it.
    - Use `#!/bin/bash` if you can.
        - You can in macOS and most Linux distros.
    - Most of these rules assume Bash.
    - If it's macOS, you can only assume Bash 3.2.
        - Be sure not to use newer Bash features.
    - If it's Linux, you can assume Bash 5.0+.
- Prefer long versions of option names.
    - Don't make readers look up what the options do.
    - Exceptions can be made for very common options.
- Use my variant of the unofficial Bash strict mode.
- Indent 4 spaces; no tabs.
- Files end with a newline.
- Use [Shellcheck](https://github.com/koalaman/shellcheck) to auto-format.
- Use `trap` to ensure the computer is always in a known good state.

~~~bash
# Ensure we always switch back to the original branch.
trap 'git switch "$original_branch"' EXIT

# Ensure we don't leave the temp branch around.
trap 'git br -D "$merge_into_qa_branch"' EXIT
~~~

- Use output to make it clear what's happening.
- Use proper error exit codes.
- Consider prompting the user before making destructive changes.

~~~ bash
read -p "Force-pushing ${branch}. Press ENTER to continue, Ctrl+C to exit." -n 1 -r
echo
~~~

- Comment liberally.
- Validate assumptions
    - Echo (to `stderr`) and bail out (`exit 1`) if they're not met.
- Prefer single-quoted strings.
    - I might change this to prefer double-quoted strings.
        - To match my Ruby preferred style.
        - Because it's more common to have an apostrophe in text than a double-quote.
- Double-quote anything with `$`, including variables, command expansion, etc.
    - Don't forget that glob expansion won't work inside double-quotes.
    - Don't forget that tilde expansion won't work inside double-quotes.
- Never use positional parameters (`$1` etc) directly; put them in well-named variables.
    - There's no need to name [other special shell variables](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html) (`$@`, `$$`, `$?`, etc).
- Use `"$@"` unless you have a specific reason to use `$*`.
- Prefer `[[ … ]]` over `[ … ]` and `test`.
    - Unless you're not able to assume Bash (or Zsh or some other modern shell).

~~~ bash
[[ "${my_var}" == 'some string' ]]
[[ -z "${my_var}" ]]
[[ "${my_var}" -gt 3 ]]
~~~

- It’s a lot safer to expand wildcards with `./*` instead of `*`.
    - Because filenames can begin with a `-`.
- Use `readonly` or `declare -r` to ensure a variable can't be changed.
- Use `local` to declare function-specific variables.

~~~ bash
local my_var="$1"
~~~

- Use shell variable substitution and other similar constructs.

~~~ bash
# Don't use `sed` for this.
substitution="${string/#foo/bar}"

# Example, getting a branch name like `merge_into_qa/1234-xyz` from `feature/1234-xyz`:
merge_into_qa_branch="${branch/feature/merge_into_qa}"
~~~

## Useful patterns

~~~ bash
# Determine whether the current file has been sourced (as opposed to executed).
# See https://stackoverflow.com/a/28776166/26311 for details, or more shells.
# NOTE: This will not work if it's run within a function.
(
  [[ -n $ZSH_VERSION && $ZSH_EVAL_CONTEXT =~ :file$ ]] ||
  [[ -n $BASH_VERSION ]] && (return 0 2>/dev/null)
) && sourced=1 || sourced=0

my_command() {
    # Do the work.
}

# If we're sourced, then we just want to define the command; otherwise we want to run it.
$sourced || my_command


# Post-checkout git hook.
PREVIOUS_HEAD="$1"
NEW_HEAD="$2"
NEW_BRANCH="$(git branch --show-current)"
[[ "$3" == 1 ]] && CHECKOUT_TYPE='branch' || CHECKOUT_TYPE='file'

if [[ "$CHECKOUT_TYPE" == 'branch' ]]; then
    docker-compose run --rm api rails db:migrate
fi
~~~

## Useful functions

~~~ bash
command_exists() {
    command -v "$1" >/dev/null 2>&1
}
~~~

## Other shell style guides

- [Google](https://google.github.io/styleguide/shellguide.html)
