# multi

`net.lewisship.multi` is a complement to joker.tools.cli that allows a single tool, a Joker script, to 
contain multiple commands, each with its own command line options and arguments.

`multi` makes use of metadata on the vars of one or
more namespaces to identify the available commands, and the command line options
and arguments allowed for each command.

multi uses the following meta data keys:

:command (boolean) -- indicates the function is a command 

:command-name (string, optional) -- overrides the name of the command, normally the symbol

:doc (string) -- the command's docstring is the command description, used when printing help

:command-opts - a list of option specs, passed to joker.tools.cli/parse-opts

:command-args - a list of positional argument specs 

A command option, -h / --help, is added to all commands automatically.

A help command is also added; it displays the list of commands available.

The help command displays the first line of each command's docstring, as a summary
of the command.

The command function is ultimately passed a map containing two sub-maps:
The :options map contains the command options, and the :arguments
map contains a map of processed command line arguments.

Whereas command option parsing is driven by the option names, command argument
parsing is positional. Each command option spec will consume one command line argument
(exception: the last spec may be repeatable).

command argument specs are similar to command option specs; each spec is a vector that starts
with one or two strings; the first string is always the label (by convention,
in upper case). The optional second string is the documentation of the argument.

Following that are key/value pairs:

:id (keyword) - identifies which key is used in the arguments map; by default,
this is the label converted to a lower case keyword

:doc (string) - documentation for the argument

:optional (boolean) -- if true, the argument may be omitted if there isn't a
    command line argument to match

:repeatable (boolean) -- if true, then any remaining command line arguments are processed
by the argument

:parse-fn - passed the command line argument, returns a value, or throws an exception

:validate - a vector of function/message pairs

:update-fn - optional function used to update the (initially nil) entry for the argument in the :arguments map
 
:assoc-fn - optional function used to update the arguments map; passed the map, the id, and the parsed value

:update-fn and :assoc-fn are mutually exclusive.

For repeatable arguments, the default update function will construct a vector of values.
For non-repeatable arguments, the default update function simply sets the value.

An example multi tool:

```
#!/usr/bin/env joker

(ns-sources
  {"net.lewisship.multi" {:url "https://raw.githubusercontent.com/hlship/multi/v1.0.0/src/net/lewisship/multi.joke"}})

(ns example
  (:require [net.lewisship.multi :as multi]))

(defn ^{:command true
        :command-opts [["-v" "--verbose" "Enable verbose logging"]]
        :command-args [["HOST" "System configuration URL"
                        :validate [#(re-matches #"https?://.+" %) "must be a URL"]]
                       ["DATA" "Data to configure as KEY=VALUE"
                        :id :key-values
                        :parse-fn (fn [s]
                                    (when-let [[_ k v] (re-matches #"(.+)=(.+)" s)]
                                      [(keyword k) v]))
                        :update-fn (fn [m [k v]]
                                     (assoc m k v))
                        :repeatable true]]}
  configure
  "Configures the system with keys and values"
  [parsed-opts]
  (pprint (select-keys parsed-opts [:options :arguments])))

;; Execution:

(multi/dispatch {:tool-name "example"
                 :namespaces ['example]]})
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
{:options {:verbose true},
 :arguments {:host "http://localhost:9983",
             :key-values {:retries "3",
                          :alerts "on"}}}
```

## License

`multi` is (c) 2019-present Howard M. Lewis Ship.
It is released under the terms of the Apache Software License, 2.0.