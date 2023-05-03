# xwintog

## Description

Focus or start an X11 application based on a configuration file.

This is used to simulate the effect of windowing systems that use a dock to
conditionally either start an app on the dock or else switch focus onto it,
depending on whether the app is already running or not.

This is designed to be used in conjunction with a global keyboard shortcut
manager such as xbindkeys to create hot keys that can be used to open or
switch to apps.

## Usage

```bash
xwintog {APP_NAME}
```

## Dependencies

The host system must have xdotool installed.

See: <https://github.com/jordansissel/xdotool>

## Configuration

Configuration is located at ~/.xwintogrc in the form of a bash source
include. Within the file, a set of associative arrays are used to configure
each application managed by xwintog, where the key of each array is a
symbolic name of the app being configured. These arrays are as follows,
where APP_NAME is substituted with the name of the app being configured:

```bash
window[APP_NAME]="STRING"
```

Window identification string for an application, where STRING is the string.
The string is a set of arguments passed to the `xdotool search`
sub-command.

> **NOTE:** Getting the right string can be tricky. Xdotool search interprets
> names as regular expressions, which means that if you don't wrap the name
> inside of beginning and ending symbols (ie. `^like this$`) then the name can
> match any part of the window title or class. If you want to match an exact
> class name then you have to bookend it with `^` and `$`.

```bash
command[APP_NAME]="COMMAND"
```

Command to execute if no window is found for the application. This will
be executed via Bash.

## Examples

From the shell:

```bash
# Open Firefox, or switch to it if it's already open
xwintog firefox

# Open Thunderbird, or switch to it if it's already open
xwintog thunderbird
```

To make this work, write a config file like this:

```bash
# ~/.xwintogrc:

# Firefox
window[firefox]="--classname ^Navigator$"
command[firefox]="firefox"

# Thunderbird
window[thunderbird]="--classname ^Thunderbird$"
command[thunderbird]="thunderbird"
```

## Tips

- This will only work with X11 based displays, not Wayland.

- The intended way to use this tool is in conjunction with xbindkeys, wherein
  you can map each app configured in xwintog to a keyboard key, combination
  of keys, or another gesture. However, it can theoretically be used anywhere
  else, too.

- The configuration file is just a normal bash file. You can create and set
  variables, perform logic, etc.

- It is recommended that you keep all parts of an app's configuration grouped
  into blocks.

- Familiarize yourself with the way xdotool search works when identifying
  windows, and use xprop to figure out which window details are best for
  identifying the window. It's not always completely obvious. For example,
  Firefox requires the string "--classname ^Navigator$" as the string, because
  Firefox will create multiple winodws that all have "firefox" in the name,
  but the browser window itself is called "Navigator". Furthermore, you
  have to ensure you

## See Also

- xdotool: <https://github.com/jordansissel/xdotool>
- xbindkeys: <https://www.nongnu.org/xbindkeys/>
- xprop: <https://linux.die.net/man/1/xprop>
