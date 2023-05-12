# xwintog

A Linux command line tool that either focuses a running X11 application or runs
a command according to a configuration file.

This is used to simulate the effect of windowing systems that use a dock to
conditionally either start an app on the dock or else switch focus onto it,
depending on whether the app is already running or not.

This is designed to be used in conjunction with a global keyboard shortcut
manager such as xbindkeys to create hot keys that can be used to open or
switch to apps.

## Command Line Usage

> **NOTE:** [Setup](#setup) is required before this will work.

```bash
xwintog <APP_NAME>
```

## Dependencies

- The host system must have [xdotool](https://github.com/jordansissel/xdotool) installed.
- Not a strict dependency, but xwintog is designed to be used with [xbindkeys](https://www.nongnu.org/xbindkeys/) if you want to map it to keybindings.

## Setup

1. Clone this repo somewhere, such as your home folder:

        git clone https://github.com/bradharms/xwintog.git "$HOME/xwintog"

2. Add the binary to a PATH location:

        sudo ln -s "$HOME/xwintog/bin/xwintog" /usr/local/bin

3. Configure a local config file with entries for apps you use by using a bash
   `case` statement on the global `$APP` variable :

        # $HOME/.xwintogrc

        case $APP in

          firefox)
            APP_WINDOW_SEARCH="--classname ^Navigator$"
            APP_COMMAND="firefox"
            ;;

          thunderbird)
            APP_WINDOW_SEARCH="--classname ^Mail$"
            APP_COMMAND="thunderbird"
            ;;

          # (and so on)

        esac

4. From the shell, test your configuration like this:

        # Open Firefox, or switch to it if it's already open
        xwintog firefox

        # Open Thunderbird, or switch to it if it's already open
        xwintog thunderbird

5. Optionally, map each app configuration into your xbindkeys configuration:

        # $HOME/.xbindkeysrc

        # Map Firefox to F1
        "xwintog firefox"
          F1

        # Map Thunderbird to F2
        "xwintog thunderbird"
          F2

        # etc.

    ...and then restart xbindkeys.

## Configuration

Configuration is located at ~/.xwintogrc in the form of a bash source
include. Within the file, a set of global variables must be defined to configure
each application managed by xwintog. As shown in the previous example, you
can determine what to set the variables to by looking at the `$APP` variable,
which will be set by xwintog beforehand.

Recognized variables are as follows:

### `APP_WINDOW_SEARCH="<STRING>"`

Window identification string for an application, where `<STRING>` is the string.
The string is a set of arguments passed to the `xdotool search`
sub-command.

**NOTE:** Getting the right string can be tricky. Xdotool search interprets
names as regular expressions, which means that if you don't wrap the name
inside of beginning and ending symbols (ie. `^like this$`) then the name can
match any part of the window title or class. If you want to match an exact
class name then you have to bookend it with `^` and `$`.

Example:

```bash
case $APP in

  firefox)
    APP_WINDOW_SEARCH="--classname ^Navigator$"
    ;;

esac
```

### `APP_WINDOW_SEARCH_FILTER="<COMMAND>"`

Optional filter for the output of `xdotool search`. This can be used to reduce
the the number of resulting window IDs down to 1 using `head` and `tail`.

Example:

```bash
case $APP in

  firefox)
    APP_WINDOW_SEARCH_FILTER="tail -n 1"
    ;;

esac
```

### `APP_COMMAND="<COMMAND>"`

Command to execute if no window is found for the application. This will
be executed via Bash.

```bash
case $APP in

  firefox)
    APP_COMMAND="firefox"
    ;;

esac
```

# Behavior

This section describes the behavior of the xwintog script in more detail:

1. The script will start by checking for system dependencies and terminate with
   an error if they're not present.
2. Next it will ensure that an app name was passed, and fail with an error if
   not.
3. The user's `.xwintogrc` config file is sourced. This script must set the
   required variables by looking at the global variable `$APP` that was set by
   the xwintog in the previous step.
4. We check that all required variables (ie. `APP_COMMAND` and
   `APP_WINDOW_SEARCH`) were set by the config file, and fail with an error if
   not.
5. Proceed to look for windows matching the search string specified by
   `APP_WINDOW_SEARCH`.
6. If no windows were found, we execute `APP_COMMAND` in a detached shell and
   then exit.
7. If any windows were found, we check to see if any of them are currently
   focused. If so, we minimize that window and then exit.
8. If none of the windows were focused, we attempt to focus and reveal each one
   sequentially. (Any failures will be ignored.)

The last two steps have the effect of cycling through windows that match the
search string by way of iteratively minimizing each one to reveal the next one
in the stack. This behavior is somewhat unintentional but could potentially be
useful, so it is being retained for now.

## Tips

- This will only work with X11 based displays, not Wayland.

- The intended way to use this tool is in conjunction with
  [xbindkeys](https://www.nongnu.org/xbindkeys/), wherein
  you can map each app configured in xwintog to a keyboard key, combination
  of keys, or another gesture. However, it can theoretically be used anywhere
  else, too.

- Read the man page on `xdotool search` and then use
  [xprop](https://linux.die.net/man/1/xprop) or
  [xwininfo](https://linux.die.net/man/1/xwininfo) to figure out which window
  details are best for identifying the window. It's not always completely
  obvious. For example, Firefox requires the string `--classname ^Navigator$` as
  the string, because Firefox will create multiple windows that all have
  "firefox" in the name, but the browser window itself is called "Navigator".

- If you want to identify only a single window but can't find a way to use an
  xdotool search string to do this, a way to "hack" the results is provided in
  the form of the `APP_WINDOW_SEARCH_FILTER` option, which lets you use Linux's
  `head` or `tail` commands to do some post processing on the results of
  `xdotool search`:

        case $APP in

          APP_NAME_1)
            APP_WINDOW_SEARCH="--classname ^APP_NAME$"
            # Get only the FIRST matching window ID:
            APP_WINDOW_SEARCH_FILTER="head -n 1"
            APP_COMMAND="APP_NAME"
            ;;

          APP_NAME_2)
            APP_WINDOW_SEARCH="--classname ^APP_NAME$"
            # Get only the LAST matching window ID:
            APP_WINDOW_SEARCH_FILTER="tail -n 1"
            APP_COMMAND="APP_NAME"
            ;;

        esac

- The configuration file is just a normal bash file. You can create and set
  variables, perform logic, etc. and it will be invoked every time xwintog is
  called. The only requirement for it is that the variables `APP_WINDOW_SEARCH`
  and `APP_COMMAND` get set before it ends.

## See Also

- xbindkeys: <https://www.nongnu.org/xbindkeys/>
- xdotool: <https://github.com/jordansissel/xdotool>
- xprop: <https://linux.die.net/man/1/xprop>
- xwininfo: <https://linux.die.net/man/1/xwininfo>
