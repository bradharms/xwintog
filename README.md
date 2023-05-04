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

```bash
xwintog {APP_NAME}
```

## Dependencies

- The host system must have [xdotool](https://github.com/jordansissel/xdotool) installed.
- Not a strict dependency, but xwintog is designed to be used with [xbindkeys](https://www.nongnu.org/xbindkeys/) if you want to map it to keybindings.

## Installation Example

1. Clone this repo somewhere, such as your home folder:

        git clone https://github.com/bradharms/xwintog.git "$HOME/xwintog"

2. Add the binary to a PATH location:

        sudo ln -s "$HOME/xwintog/bin/xwintog" /usr/local/bin

3. Configure a local config file with entries for apps you use:

        # $HOME/.xwintogrc

        # Firefox
        window[firefox]="--classname ^Navigator$"
        command[firefox]="firefox"

        # Thunderbird
        window[thunderbird]="--classname ^Mail$"
        command[thunderbird]="thunderbird"

        # etc.

4. From the shell, test your configuration like this:

        # Open Firefox, or switch to it if it's already open
        xwintog firefox

        # Open Thunderbird, or switch to it if it's already open
        xwintog thunderbird

5. Map each app configuration into your xbindkeys configuration:

        # $HOME/.xbindkeysrc

        # Map Firefox to F1
        "xwintog firefox"
          F1

        # Map Thunderbird to F2
        "xwintog thunderbird"
          F2

        # etc.

    ...and then kill and restart xbindkeys.

## Configuration

Configuration is located at ~/.xwintogrc in the form of a bash source
include. Within the file, a set of associative arrays are used to configure
each application managed by xwintog, where the key of each array is a
symbolic name of the app being configured. These arrays are as follows,
where APP_NAME is substituted with the name of the app being configured:

### `window[APP_NAME]="STRING"`

Window identification string for an application, where STRING is the string.
The string is a set of arguments passed to the `xdotool search`
sub-command.

> **NOTE:** Getting the right string can be tricky. Xdotool search interprets
> names as regular expressions, which means that if you don't wrap the name
> inside of beginning and ending symbols (ie. `^like this$`) then the name can
> match any part of the window title or class. If you want to match an exact
> class name then you have to bookend it with `^` and `$`.

Example:

```bash
window[firefox]="--classname ^Navigator$"
```

### `filter[APP_NAME]="COMMAND"`

Optional filter for the output of `xdotool search`. This can be used to reduce
the the number of resulting window IDs down to 1 using `head` and `tail`.

Example:

```bash
filter[firefox]="tail -n 1"
```

### `command[APP_NAME]="COMMAND"`

Command to execute if no window is found for the application. This will
be executed via Bash.

## Tips

- This will only work with X11 based displays, not Wayland.

- The intended way to use this tool is in conjunction with [xbindkeys](https://www.nongnu.org/xbindkeys/), wherein
  you can map each app configured in xwintog to a keyboard key, combination
  of keys, or another gesture. However, it can theoretically be used anywhere
  else, too.

- The configuration file is just a normal bash file. You can create and set
  variables, perform logic, etc. and it will be invoked every time xwintog is
  called.

- It is recommended that you keep all parts of an app's configuration grouped
  together.

- Read the man page on `xdotool search` and then use
  [xprop](https://linux.die.net/man/1/xprop) or
  [xwininfo](https://linux.die.net/man/1/xwininfo) to figure out which window
  details are best for identifying the window. It's not always completely
  obvious. For example, Firefox requires the string `--classname ^Navigator$` as
  the string, because Firefox will create multiple windows that all have
  "firefox" in the name, but the browser window itself is called "Navigator".

- Some apps tend to have hidden background windows that will come up when
  searching for the app's name using xdotool search, but those background
  windows aren't the ones you probably want to reveal with xwintog, and trying
  to do so will often fail with X11 errors. The solution is to use xprop or
  xwinifo to find something unique in the name or class of the target window
  that distinguishes it from all the others, and use that in the window
  identification string or regex.

- If you can't find a way to identify a window using xdotool search alone, a
  way to "hack" the results is provided in the form of the `filter` option,
  which lets you use Linux's `head` or `tail` commands to do some post
  processing on the results of `xdotool search`:

        # Get only the FIRST matching window ID:
        window[APP_NAME]="--classname ^APP_NAME$"
        filter[APP_NAME]="head -n 1"
        command[APP_NAME]="APP_NAME"

        # Get only the LAST matching window ID:
        window[APP_NAME]="--classname ^APP_NAME$"
        filter[APP_NAME]="tail -n 1"
        command[APP_NAME]="APP_NAME"

- Many modern desktop apps have their own internal way of revealing existing
  windows when their associated CLI command is invoked, as is the case with the
  aforementioned Firefox. This can be used as a fallback in the event that you
  can't get xwintog to work. In this case you forgo using xwintog entirely and
  simply call that command directly from your xkeybindrc.

## See Also

- xbindkeys: <https://www.nongnu.org/xbindkeys/>
- xdotool: <https://github.com/jordansissel/xdotool>
- xprop: <https://linux.die.net/man/1/xprop>
- xwininfo: <https://linux.die.net/man/1/xwininfo>
