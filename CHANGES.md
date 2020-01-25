# 1.4.0 - 25 Jan 2020

Tools can now provide a description of the tool as the docstring of the first namespace;
this is displayed by the built-in `help` command.

Command names may be abbreviated to a unique prefix.

Added `net.lewisship.multi/collect`, which is useful for repeatable options or arguments;
it collects each value into a vector.

# 1.3.0 - 24 Jan 2020

Fixed a bug w/ `:validate` in argument specs.

Fixed a bug where it was not possible to have a function literal in a symbol referenced by
an option or argument spec.

Can now use `:command-name <string>` inside `defcommand`.

# 1.2.0 -- 23 Jan 2020

Now uses joker.better-cond namespace, so requires Joker v0.14.1 or later.

# 1.1.0 -- 17 Jan 2020

Add `defcommand` macro, inspired by Boot.

# 1.0.0 -- 2 Dec 2019

Initial release.
