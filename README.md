## Tube.vim

**v0.3.0**

This plugin provides a tiny interface for sending commands from MacVim to a
separate iTerm or Terminal window without leaving MacVim.


## Requirements

* Mac OS X 10.6+ (note that this plugin has been tested only on Mac OS X 10.6
  but should work even with successive versions)
* iTerm2 or Terminal installed
* MacVim compiled with python 2.x+


## Installation

Extract the content of the folder into `$HOME/.vim` or use your favourite
package manager.

To complete the installation you need to set at least the following variable in
your `.vimrc` file:

```
let g:tube_terminal = 'iterm'      " if you use iTerm.app
```

or

```
let g:tube_terminal = 'terminal'   " if you use Terminal.app
```


## Visual tour

### A simple example
```
                 focus remains here
 MacVim         /                                     Terminal
---------------°---------------------                -------------------------------------
| # hello_world.py                  |                | ...                               |
|                                   |                | $ ls                              |
| print "Hello World!"              |                | hello_world.py                    |
|                                   |        ------> | $ python hello_world.py           |
|                                   |       |        | Hello World!                      |
|___________________________________|       |        |                                   |
|:Tube python %                     |-------'        |                                   |
--------------°----------------------                -------------------------------------
               \
                The % character stands for the current buffer name. If you want
                no expansion at all, just escape it with another % character (%%).
                Note the absence of quotes around the command.
```

### Selection injection
```
                 focus remains here
 MacVim         /                                     Terminal
---------------°---------------------                -------------------------------------
| # hello_world.py                  |                | ...                               |
|                                   |                | $ python                          |
| print "this is a selected line"   |        ------> | >>> print "this is a selected.. " |
|                                   |       |        | this is a selected line           |
|                                   |       |        |                                   |
|___________________________________|       |        |                                   |
|:'<,'>Tube @                       |-------'        |                                   |
-------------°-----------------------                -------------------------------------
              \
               The @ character stand for the current selection. If you just happen to be
               on a line in normal mode then the @ character stands for the current
               line (in this case you'll use the plain :Tube @). If the selection spans
               multiple lines they are passed to the terminal as they are, that is,
               whitespaces are preserved.
```

### Function injection
```
                       focus remains here
 MacVim               /                               MacVim (invisible state)
---------------------°---------------                ....................................
|                                   |                .                                  .
|                                   |                .                                  .
| // beautifully crafted code       |                . // beautifully crafted code      .
|                                   | -------------> .                                  .
|                                   |                .                                  .
|___________________________________|                ....................................
|:Tube cd #{Foo(1,'@')} && do sth   |          _____ |:Tube cd project_root && do sth   |
--------------|---°------------------         |      ....................................
              |    \_____________________     |
 Your .vimrc  |                          |    |       Terminal
--------------|----------------------    |    |      ------------------------------------
|                                   |    |    `----> | $ cd project_root && do sth      |
| fu! Foo(arg1, arg2)               |    |           | ...                              |
|  // really heavy computation      |    |           |                                  |
|  return "project_root"            |    |           |                                  |
| endfu                             |    |           |                                  |
|                                   |    |           |                                  |
-------------------------------------    |           ------------------------------------
                                         |
              __________________________/ \___________
             /                                        \
   In this example we used the special            As you can see only string arguments require
   character @ as one of the arguments.           quotes. Also, you do not have to bother about
   Doing so we pass the selection right           escaping yourself the string since it's done
   into the function as a normal argument         automatically for you. Note however that all
   (note the quotes). This might be useful        arguments are passed to the function as strings.
   if you need to perform some kind of 
   formatting on the selection before sending 
   it to the terminal.
```

### Aliasing
```
                       focus remains here
 MacVim               /                               MacVim (invisible state)
---------------------°---------------                ....................................
| // a very                         |                . // a very                        .
| // long long                      |                . // long long                     .
| // paragraph                      |                . // paragraph                     .
| // is selected                    | -------------> . // is selected                   .
| ...                               |                . ...                              .
|___________________________________|                ....................................
|:'<,'>TubeAlias cmd                |          _____ |:Tube make etc                    |
---------------|--°------------------         |      ....................................
               |   \_____________________     |
 Your .vimrc   |                         |    |       Terminal
---------------|---------------------    |    |      ------------------------------------
|                                   |    |    `----> | $ make etc                        |
| let g:tube_aliases = {            |    |           | ...                               |
|  \'cmd':'#{Format("@")} | do sth' |    |           |                                   |
|  \}                               |    |           |                                   |
|                                   |    |           |                                   |
--------°-----------°----------------    |           -------------------------------------
        |            \____________________\
        |                                   Selection, function and buffer injection
      You can define aliases in your        still work with aliasing.
      .vimrc file or at runtime. Keep
      in mind that in the latter case
      you'll lose those aliases once 
      you quit MacVim.
```


## Commands


### Tube
```
arguments: a string of any length (the command)
accept selection: yes
```

Execute the command in the separate iTerm (or Terminal) window. If the that
window does not exist yet, then it is created. If no command is given the
window is simply created, or cleared if it already exists.  By default the
window focus remains on MacVim but you can customize this behavior with
the g:tube_run_command_background setting. Note that you do'nt have to wrap
the command into quotes.

Some character as a special meaning inside the command. Those chracters are
`%`, `#{..}`, `@` and inform **Tube** that it has to inject some kind of
information into the command:

* `%`: inject the current buffer name
* `@`: inject the current selection or the current line if there is no selected
  text. Note that block selection is not supported.
* `#{FunctionName(arg1, .., argn)}`: inject the return value of the user function
  named FunctionName. String arguments need to be wrapped with single or double
  quotes but you don't need to bother about escaping quotes in your string:
  it's done automatically for you. Another nicety is that you can use the special 
  characters `%` and `@` even as arguments of the function. Just remember to
  wrap the with quotes too.

  **NOTE**: if you need a plain `%` or `@` character in your command just append
  the same character twice, respectively `%%` and `@@`


### TubeClear
```
arguments: a string of any length (the command)
accept selection: yes
```

As the `Tube` command but force the terminal to clear its screen before
executing the command. Under the hood it appends a `clear` command before
the main command.


### TubeLastCommand
```
arguments: no
```

Execute the last executed command.


### TubeInterruptCommand
```
arguments: no
```

Interrupt the current running command in the terminal window. Under the hood it calls
the Ctrl-C command.


### TubeCd
```
arguments: no
```

Execute a `cd 'current buffer name'` command in the terminal window.


### TubeClose
```
arguments: no
```

Close the terminal window.


### TubeAlias
```
arguments: a string of any length (the alias name)
accept selection: yes
```

Execute the command associated with the given alias name. The alias might be
defined in the `.vimrc` file via the `g:tube_aliases` setting or at run time
via the `TubeAddAlias` command.


### TubeAliasClear
```
arguments: a string of any length (the alias name)
accept selection: yes
```

As the `TubeAlias` command but force the terminal to clear its screen before
executing the command associated with the alias.


### TubeRemoveAlias
```
arguments: a string of any length (the alias name)
e.g. TubeRemoveAlias my_alias
```

Remove the command associated with the given alias.


### TubeAddAlias
```
arguments: at least two tokens of any length.
e.g. TubeAddAlias myalias cd into/that & rm all
```

Associate the alias with the given command.


### TubeAliases
```
arguments: no
```

Show all defined aliases.


### TubeRemoveAllAliases
```
arguments: no
```

Remove all defined aliases. This affect only the current vim session for aliases
defined at runtime.


### TubeReloadAliases
```
arguments: no
```

Reload the g:tube_aliases vim variable. This might be needed when the user
change that variable at runtime.


### TubeToggleClearScreen
```
arguments: no
```

Toggle the g:tube_always_clear_screen setting.


### TubeToggleRunBackground
```
arguments: no
```

Toggle the g:tube_run_command_background setting.


### TubeToggleBufnameExp
```
arguments: no
```

Toggle the g:tube_bufname_expansion setting.


### TubeToggleFunctionExp
```
arguments: no
```

Toggle the g:tube_function_expansion setting.


### TubeToggleSelectionExp
```
arguments: no
```

Toggle the g:tube_selection_expansion setting.



## Settings


### g:tube_terminal
```
values: 'iterm' or 'terminal'
default: 'terminal'
```

Use this setting to set the terminal emulator of your choice. At the moment
only iTerm and Terminal are supported.


### g:tube_always_clear_screen
```
values: 1 or 0
default: 0
```

Setting this to 0 forces the terminal to clear its screen whenever
a command is executed. You can toggle this setting on or off with the
TubeToggleClearScreen command.


### g:tube_run_command_background
```
values: 1 or 0
default: 1
```

Set this variable to 1 to mantain the focus on the MacVim window whenever a
command is executed. You can toggle this setting on or off with the
TubeToggleRunBackground command.


### g:tube_aliases
```
values: a dictionary {'alias': 'command', ...}
default: {}
```

With this dictionary you can set your own aliases for commands. Just use the alias
name as the dictionary key and the string command as the value. Be sure to have
unique aliases. Special characters (`%`, `@` and `#{..}`) are supported.


### g:tube_bufname_expansion
```
values: 0 or 1
default: 1
```

Set this variable to 1 and every `%` character in your commands will be replaced with
the current buffer name (its absolute path). If you need the just the `%`
character use the `%%` sequence. You can toggle the setting on or off with the
TubeToggleBufnameExp command.


### g:tube_function_expansion
```
values: 0 or 1
default: 1
```

Set this variable to 1 to enable function expansion. Every `#{FunctionName(arg1, .., argn)}`
string inside commands will be replaced with the return value of `FunctionName(arg1, .., argn)`
function defined by the user.


### g:tube_selection_expansion
```
values: 0 or 1
default: 1
```

Set this variable to 1 to enable selection expansion. Every `@` character inside
commands will be replaced with the current selection. In order to get the
current selection you must use a Tube command the way you usually do with vim
commands: `:'<,'>Tube command`. If no selection is found, then the current 
line is taken.


### g:tube_enable_shortcuts
```
values: 0 or 1
default: 0
```

Set this to 1 to to enable shortcuts for the most important commands.

* `T`: Tube
* `Tc`: TubeClear
* `Tl`: TubeLastCommand
* `Ti`: TubeInterruptCommand
* `Tcd`: TubeCd
* `Ta`: TubeAlias
* `Tac`: TubeAliasClear
* `Tar`: TubeRemoveAlias
* `Tad`: TubeAddAlias
* `Tall`: TubeAliases


## Changelog

### v0.3.0
* new feature: selection injection into the command with the @ character.
* new feature: injected functions accept arguments.
* g:tube_at_character_expansion setting renamed to g:tube_bufname_expansion.
* TubeToggleExpandPercent command renamed to TubeToggleBufnameExp.
* TubeToggleExpandFunction command renamed to TubeToggleFunctionExp.
* added shortcuts for most important commands (disabled by default).
* added g:tube_enable_shortcuts setting.
* added g:tube_selection_expansion setting.
* added TubeToggleSelectionExp command.
* fixed backslash escaping in commands.
* minor bug fixes.

### v0.2.1
* fix plugin feedback

### v0.2.0
* new feature: the result of a custom vim function can be injected into the command with the special notation #{CustomFunction}.
* minor bug fixes.

### v0.1.0
* first release
