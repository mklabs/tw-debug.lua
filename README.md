# tw-debug

A tiny LUA debugging utility designed for Total War modding, inspired by [visionmedia/debug](https://github.com/visionmedia/debug).

It is in fact a direct port to LUA and Total War modding environment.

Tested heavily in the context of Troy modding, but should be supported as well in other Total War games.

## Installation

Simply clone or download this repository and copy the content of the `script` directory to your mod.

## Usage

`debug` exposes a function, simply pass this function the name of your mod, and it will return a decorated version of `out()` for you to pass debug statements to. This will allow you to toggle the debug output for different parts of your mod as well as the mod as a whole.

Example `mod.lua`

 ```lua
local debug = require("tw-debug")("mymod")

debug("Init module")

require("mod/stuff")
```

Example `mod/stuff.lua`

```lua
local a = require("tw-debug")("mymod:stuff:a")
local b = require("tw-debug")("mymod:stuff:b")

local function worka()
    a('doing lots of uninteresting work');
    cm:callback(worka, math.random())
end

local function workb()
    b('doing some work');
    cm:callback(workb, math.random())
end

worka()
workb()
```

The `DEBUG` ini variable is then used to enable these based on space delimited names.

By default, `debug` will write those debug logs to `debug.txt` file, in the current working directory (usually the game base directory, where the .exe is located).

```
mymod Init module +0ms
mymod:stuff:a doing lots of uninteresting work +0ms
mymod:stuff:b doing some work +0ms
mymod:stuff:a doing lots of uninteresting work +488ms
mymod:stuff:a doing lots of uninteresting work +101ms
mymod:stuff:b doing some work +590ms
mymod:stuff:b doing some work +197ms
mymod:stuff:a doing lots of uninteresting work +707ms
mymod:stuff:b doing some work +908ms
mymod:stuff:b doing some work +200ms
mymod:stuff:a doing lots of uninteresting work +903ms
mymod:stuff:b doing some work +502ms
mymod:stuff:a doing lots of uninteresting work +503ms
mymod:stuff:b doing some work +403ms
mymod:stuff:b doing some work +707ms
mymod:stuff:a doing lots of uninteresting work +812ms
mymod:stuff:a doing lots of uninteresting work +97ms
mymod:stuff:a doing lots of uninteresting work +454ms
mymod:stuff:b doing some work +856ms
mymod:stuff:b doing some work +404ms
mymod:stuff:b doing some work +203ms
mymod:stuff:a doing lots of uninteresting work +914ms
mymod:stuff:a doing lots of uninteresting work +496ms
mymod:stuff:b doing some work +602ms
```

## Configuration

Configuration is done through an ini file `debug.ini`, in the current working directory (usually the game base directory, where the .exe is located).

```ini
# Enables/disables specific debugging namespaces.
DEBUG=*

# Example
# DEBUG=mymod mymod:*
```

`debug` won't log anything unless this file is present, and the debugging namespaces matche one of your debugger names.

## Millisecond diff

When actively developing a mod it can be useful to see when the time spent between one `debug()` call and the next. Suppose for example you invoke `debug()` before invoking a callback, and after as well, the "+NNNms" will show you how much time was spent between calls.

```
mk:temples:mod init +0ms
mk:temples:ui init +0ms
mk:temples:ui buildButtonUIC +4ms
mk:temples:ui registerDropdownButtonsListeners +4ms
mk:temples:ui { "tab_factions", "tab_regions", "tab_units", "tab_events", "tab_missions" } +4ms
mk:temples:ui Register listener for tab_factions +4ms
mk:temples:ui Register listener for tab_regions +4ms
mk:temples:ui Register listener for tab_units +4ms
mk:temples:ui Register listener for tab_events +4ms
mk:temples:ui Register listener for tab_missions +4ms
mk:temples:mod done +41ms
mk:temples:ui onButtonClick +5s
mk:temples:ui buildTemplesDropdownUIC +3ms
mk:temples:data getTemplesData for 47 regions +0ms
```

## Pretty print

The [inspect](https://github.com/kikito/inspect.lua) library is used to pretty print every argument you pass to `debug()`

```lua
local debug = require("tw-debug")("mod:ui")
local buttons = { "tab_factions", "tab_regions", "tab_units", "tab_events", "tab_missions" }
debug(buttons)

debug("==>", { "a", "b", "c", {
  foo = {
    bar = "bar"
  }
}});

--[[ Output

mod:ui { "tab_factions", "tab_regions", "tab_units", "tab_events", "tab_missions" } +0ms
mod:ui ==> { "a", "b", "c", {
    foo = {
      bar = "bar"
    }
  } } +4ms

]]
```

## Conventions

If you're using this in one or more of your mods, you _should_ use the name of your mod so that developers may toggle debugging as desired without guessing names. If you have more than one debuggers you _should_ prefix them with your mod name and use ":" to separate features.

## Wildcards

The `*` character may be used as a wildcard. Suppose for example your mod has debuggers named "mod:ui", "mod:utils", "mod:factions", instead of listing all three with `DEBUG=mod:ui mod:utils mod:factions`, you may simply do `DEBUG=mod:*`, or to run everything using this library simply use `DEBUG=*`.

You can also exclude specific debuggers by prefixing them with a "-" character. For example, `DEBUG=* -mod:*` would include all debuggers except those
starting with "mod:".

## Formatters

Debug uses [printf-style](https://wikipedia.org/wiki/Printf_format_string) formatting, through the use of `string.format()`

Below are the officially supported formatters:

| Formatter | Representation |
|-----------|----------------|
| `%s`      | String. |
| `%q`      | String (quoted). |
| `%d`      | Number (integer). |
| `%f`      | Number (float). |
| `%%`      | Single percent sign ('%'). This does not consume an argument. |

[`string.format()`](http://lua-users.org/wiki/StringLibraryTutorial) has more options you can use (like , `%g`)

Between the `%` and the letter, a directive can include other options, which control the details of the format, such as the number of decimal digits of a floating-point number:

```lua
string.format("pi = %.4f", math.pi)     --> pi = 3.1416
```

## Output logs

By default `debug` will log to `debug.txt`, however this can be configured per-namespace by overriding the `log` method.

```lua
local debug = require("tw-debug")("test")
debug.log = function(env, ...)
    --[[ env
    {
        -- the current debug timestamp (as returned by os.clock())
        curr = 3150.4162597656,

        -- the humanized diff between the previous debug call and this one
        diff = "0ms",

        -- the configured namespace for this debugger
        namespace = "test"
    }
    ]]
    
    -- remaining arguments can be retrived with the arg table (use unpack(arg) to call a function with them)
    out("outputs stuff " .. arg[1])
end

-- now goes to script_log.txt
debug("goes to script_log.txt via out()")
```

## Output files

Similarly, if you just want to change the file to which logs are written instead of the default `debug.txt`, this can be configured per-namespace by setting the `file` property.

```lua
local debug = require("tw-debug")("test")
debug.file = "mod_logs.txt"

-- now goes to mod_logs.txt
debug("goes to mod_logs.txt")
```


## Checking whether a debug target is enabled

After you've created a debug instance, you can determine whether or not it is
enabled by checking the `enabled` property:

```lua
local debug = require('tw-debug')('mymod');

if debug.enabled then
  -- do stuff...
end
```

You can also manually toggle this property to force the debug instance to be enabled or disabled.

## Credits

[TJ Holowaychuk](https://github.com/visionmedia) for the original [debug](https://github.com/visionmedia/debug) library

[Enrique García Cota](https://github.com/kikito) for the [inspect](https://github.com/kikito/inspect.lua) library, used to pretty print every argument you pass.

[Carreras Nicolas](https://github.com/Dynodzzo) for the [Lua_INI_Parser](https://github.com/Dynodzzo/Lua_INI_Parser), used to read configuration variable from `debug.ini`.

## License

MIT License

Copyright (c) 2020 Mickael Daniel

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
