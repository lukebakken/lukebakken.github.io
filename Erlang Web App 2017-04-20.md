<!-- vim:fo=tq:tw=0:syntax=off:sw=2:ts=2:
-->

# Day 3

## Overview

Today's goal is to [implement the tag entity](https://github.com/FriendsOfTheBluff/volunteer-mgr/issues/4) and complete the remaining CRUD operations for the initial version of the project.

### Person entity

* Update a person's information
* Retrieve all people
* Delete a person (both soft / hard delete)
* Optionally associate a person with a tag upon creation (via many-many associaton table)
* Update a person's tag list (via many-many associaton table)

### Tag entity

* Create / Retrieve tags
* Retrieve all tags
* Tags are immutable and can't be deleted (for now)

## Tag table

I added the following record and code to create a table consisting only of tag names:

```erlang
-record(volmgr_tags, {id :: atom()}).

VolmgrTagsOpts = [
            {attributes, record_info(fields, volmgr_tags)},
            {disc_copies, Nodes},
            {type, set}
          ],
{atomic, ok} = mnesia:create_table(volmgr_tags, VolmgrTagsOpts),
```

However, I got the following error when running tests:

```
{badmatch,{aborted,{bad_type,volmgr_tags,{attributes,[id]}}}}
```

Looks like the ["table must at least have one extra attribute in addition to the key"](http://erlang.org/doc/man/mnesia.html#create_table-2).

Once I added the `active` attribute, the test passed.
