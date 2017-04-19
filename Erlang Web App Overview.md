<!-- vim:fo=tq:tw=0:syntax=off:sw=2:ts=2:
-->

# Overview

This series will document creation of an [Erlang web application](https://github.com/FriendsOfTheBluff/volunteer-mgr) from the ground up. The goal of this application is to provide basic tracking of volunteers for a [local non-profit group](http://www.friendsofthebluff.org/), based loosely on [this project](http://sos.sourceforge.net/).

# Goals

* Erlang!
* `rebar3` as the build system. I am familiar with `rebar` and `erlang.mk`, so may as well use something different.
* As a learning exercise, use minimal helper libraries to force me to implement things like session state and other features normally provided by frameworks.
* Follow TDD development process as much as possible.
* Nothing fancy in the UI - just HTML, CSS and HTTP
