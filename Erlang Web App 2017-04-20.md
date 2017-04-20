<!-- vim:fo=tq:tw=0:syntax=off:sw=2:ts=2:
-->

# Day 3

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
