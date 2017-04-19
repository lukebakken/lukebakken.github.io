<!-- vim:fo=tq:tw=0:syntax=off:sw=2:ts=2:
-->

# Day 1

The first day of a greenfield project requires that you do a bunch of housekeeping. Here's an outline of what I did on the first day of this project:

# GitHub

Every project needs a home, and [GitHub](https://github.com) is that place. I originally created the project in [my user organization](https://github.com/lukebakken/volunteer-mgr) but have since moved it to an organization I created for the end-user. Note that GitHub helpfully implements redirects when you transfer repositories between organizations.

# Continuous Integration

I am very familiar with [Travis CI](https://travisci.org), having used it as the CI provider for several Riak client libraries ([like this](https://travis-ci.org/basho/riak-go-client/builds)). I decided to try out one of the new kids on the block, [CircleCI](https://circleci.org), and set up the continuous integration builds [here](https://circleci.com/gh/FriendsOfTheBluff/volunteer-mgr/tree/master). The entire configuration for this build is found in [this file](https://github.com/FriendsOfTheBluff/volunteer-mgr/blob/master/.circleci/config.yml). So far, CircleCI is working well and appears to be as good a solution as Travis CI.

# Project Skeleton

My former co-worker, [Shane Utt](https://github.com/shaneutt), provided a [`rebar3` template](https://github.com/shaneutt/cowboy_erlydtl_rebar3_template) that will build out a bare-bones skeleton project using [Cowboy](https://github.com/ninenines/cowboy) and [Erlydtl](https://github.com/erlydtl/erlydtl). I used this to bootstrap my project, and in the process made some changes to the template that I then provided back in [this PR](https://github.com/shaneutt/cowboy_erlydtl_rebar3_template/pull/1). We'll see if Shane likes them.

# Storage

To keep things simple and learn some "new" technology, I'm implementing this project's storage using [Mnesia](http://learnyousomeerlang.com/mnesia), the built-in KV store for Erlang. I figure, why not use the simplest thing for this project? Also, Mnesia has been around forever and appears to be plenty robust for this project's requirements.

# Code

You can check out [my commits from the first day](https://github.com/FriendsOfTheBluff/volunteer-mgr/commits/master). Not a lot accomplished as I familiarized myself again with Common Test and learned me some Mnesia from scratch. Going forward I may implement features on a per-branch basis rather than just pushing to `master`.
