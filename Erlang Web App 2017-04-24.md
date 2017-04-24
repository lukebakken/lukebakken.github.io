<!-- vim:fo=tq:tw=0:syntax=off:sw=2:ts=2:
-->

# Mnesia

## Overview

This entry describes what I learned about using [Mnesia](http://erlang.org/doc/man/mnesia.html). Most importantly, how and when you should use `ram_copies` vs `disc_copies` for your tables, and when to set them up. It's a classic chicken-and-egg problem.
