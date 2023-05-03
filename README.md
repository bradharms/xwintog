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

```bash
command[APP_NAME]="COMMAND"
```

Command to execute if no window is found for the application. This will
be executed via Bash.

## Examples

### Firefox

From ~/.xwintogrc:

```bash
...
window[firefox]="--classname Navigator"
command[firefox]="firefox"
...
```

From the shell:

```bash
xwintog firefox
```

### Thunderbird

From ~/.xwintogrc:

```bash
...
window[thunderbird]="--classname thunderbird"
command[thunderbird]="thunderbird"
...
```

From the shell:

```bash
xwintog thunderbird
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
  Firefox requires the string "--classname Navigator" as the string, because
  "firefox" creates windows that aren't meant to be visible, and you may
  end up identifying the wrong one.

## See Also

- xdotool: <https://github.com/jordansissel/xdotool>
- xbindkeys: <https://www.nongnu.org/xbindkeys/>
- xprop: <https://linux.die.net/man/1/xprop>
