# multi

`net.lewisship.multi` is a complement to joker.tools.cli that allows a single tool, a Joker script, to 
contain multiple commands, each with its own command line options and arguments.

At the core is the `net.lewisship.multi/dispatch` macro; this identifies the name of the tool, and 
optionally, the namespaces to collect commands from.

Each command within the tool is provided via the `net.lewisship.multi/defcommand` macro.

A command option, `-h / --help`, is added to all commands automatically.

A `help` command is also added; it displays the list of commands available.
The help command displays the first line of each command's docstring, as a summary
of the command.

The `help` command displays a short summary of what the overall tool does; this is the docstring
of the first namespace provided to `dispatch` (again, if omitted, the current namespace
is used).

Whereas command option parsing is driven by the option names, command argument
parsing is positional. Each command option spec will consume one command line argument
(exception: the last spec may be repeatable).

command argument specs are similar to command option specs; each spec is a vector that starts
with one or two strings; the first string is always the label (by convention,
in upper case). The optional second string is the documentation of the argument.

Following that are key/value pairs:

* `:id` (keyword) - identifies which key is used in the arguments map; by default,
this is the label converted to a lower case keyword

* `:doc` (string) - documentation for the argument

* `:optional` (boolean) -- if true, the argument may be omitted if there isn't a
    command line argument to match

* `:repeatable` (boolean) -- if true, then any remaining command line arguments are processed
by the argument

* `:parse-fn` - passed the command line argument, returns a value, or throws an exception

* `:validate` - a vector of function/message pairs

* `:update-fn` - optional function used to update the (initially nil) entry for the argument in the :arguments map
 
* `:assoc-fn` - optional function used to update the arguments map; passed the map, the id, and the parsed value

* `:update-fn` and `:assoc-fn` are mutually exclusive.

For repeatable arguments, the default update function will construct a vector of values.
For non-repeatable arguments, the default update function simply sets the value.

# Using multi

```
#!/usr/bin/env joker

(ns-sources
  {"net.lewisship.multi" {:url "https://raw.githubusercontent.com/hlship/multi/v1.4.0/src/net/lewisship/multi.joke"}})

(ns example
  "System management tool"
  (:require [net.lewisship.multi :as multi]))

(multi/defcommand configure
  "Configures the system with keys and values"
  [verbose ["-v" "--verbose" "Enable verbose logging"]
   :args
   host ["HOST" "System configuration URL"
         :validate [#(re-matches #"https?://.+" %) "must be a URL"]]
   key-values ["DATA" "Data to configure as KEY=VALUE"
               :parse-fn (fn [s]
                           (when-let [[_ k v] (re-matches #"(.+)=(.+)" s)]
                             [(keyword k) v]))
               :update-fn (fn [m [k v]]
                            (assoc m k v))
               :repeatable true]]
  (prn :verbose verbose :host host :key-values key-values)

;; Execution:

(multi/dispatch {:tool-name "example"]})
```
If this file is saved as `bin/example`, it can be executed directly:

```
> bin/example configure --help
Usage: example configure [OPTIONS] HOST DATA+

Configures the system with keys and values

Options:
  -v, --verbose  Enable verbose logging
  -h, --help     This command summary


Arguments:
  HOST: System configuration URL
  DATA: Data to configure as KEY=VALUE

> bin/example configure http://localhost:9983 retries=3 alerts=on -v
:verbose true :host "http://localhost:9983" :key-values {:retries "3", :alerts "on"}
```

Notes:
* With `defcommand` a docstring is required
* After the docstring comes the _interface_, which defines options and arguments
* The interface initially expects to consume a symbol then an option spec
* The `:args` keyword switches over to consuming symbol and argument spec

## defcommand extras

## :as \<symbol\>

Inside the interface, you can request the _command map_ using `:as`.
This command map is used when invoking `net.lewisship.multi/show-summary`, 
which a command may wish to do to present errors to the user.

### :command-name \<string\>

You may also override the command name away from its default (the name of the function).

For example, if you had an existing `configure` function you didn't want to rename, 
then you could name the command function `configure-command`:

```
(multi/defcommand configure-command
  "Configures the system with keys and values"
  [verbose ["-v" "--verbose" "Enable verbose logging"]
   :args
   host ["HOST" "System configuration URL"
         :validate [#(re-matches #"https?://.+" %) "must be a URL"]]
   key-values ["DATA" "Data to configure as KEY=VALUE"
               :parse-fn (fn [s]
                           (when-let [[_ k v] (re-matches #"(.+)=(.+)" s)]
                             [(keyword k) v]))
               :update-fn (fn [m [k v]]
                            (assoc m k v))
               :repeatable true]
   :command-name "configure"]
  ...)
```

### Meta Data

Finally, you can specify additional meta data for the command by applying
it to the command's symbol.  For example:

```
(multi/defcommand ^{:command-name "conf"} configure
  ...
```

  ... though there's rarely a need to ever do this.

# Function Metadata

The `defcommand` macro works by adding the following meta-data to the Var it creates:

* `:command` (boolean) -- indicates the function is a command 

* `:command-name` (string, optional) -- overrides the name of the command, normally the symbol

* `:doc` (string) -- the command's docstring is the command description, used when printing help

* `:command-opts` - a list of option specs, passed to joker.tools.cli/parse-opts

* `:command-args` - a list of positional argument specs 

The command function is ultimately passed the command map containing two sub-maps:
The :options map contains the command options, and the :arguments
map contains a map of processed command line arguments;
`defcommand` adds a `let` to pull values out of the command map and into the 
option and argument symbols you define in the interface.

Other keys within the command map are private, used by multi.

Again, `defcommand` is the preferred way to define commands.

## License

`multi` is (c) 2019-present Howard M. Lewis Ship.

It is released under the terms of the Apache Software License, 2.0.