# What is this?
This is a system for multiplayer friendly systems or any system that needs to keep track of many entities ingame.
This can be used for ownership systems, rooms, etc..

Scoreboard ID systems allow implementing relationships (mainly one-to-one, many-to-one and one-to-many) when there are possibly multiple participating parties at each end.
Technique by Marmalade, discord: @marmalade_toast
# Assigning scoreboard IDs
Use the following command to assign random IDs to entities that don't have one yet.

```execute as @e unless score @s wiki:entity_id matches -2147483648.. run scoreboard players random @s wiki:entity_id -2147483648 2147483646```

The reason that you should use this command is simple: it can assign random IDs to *multiple* entities at once, which will come in handy in more advanced systems (when multiple entities may be spawned in a single tick). Also, the chance of an ID collision is pretty low (a birthday paradox calculator tells you that you need 77,164 simultaneously existing entities to have a 50% chance of an ID collision)

## Example of a situation where a scoreboard link is useful
Suppose players on some server have zero or more pet slimes. This is a **one-to-many relationship**: one player owns many slimes.

If there was one player on the server, this would be easy: we just teleport every slime to that player.

Suppose that there are now two players. Immediately, we run into a problem. How do we know which player to teleport each slime to? We could use tags like `player_1_pet`, `player_2_pet`, etc.... But, this would be an inefficient use of tags and command blocks, as teleporting entities with `player_x_tag` to player x would require one command block for every player.
The solution is to use a **scoreboard link**. We create a scoreboard for all entities, `wiki:entity_id`, holding the entity's ID in the system. If an entity in the world has a particular score *x*, we can associate an occurrence of *x* somewhere else with that entity. You may recognise this as the concept of a primary key in a relational database.
We create another scoreboard, `wiki:pet.owner_id`. Pet slimes have a score in this scoreboard matching a player's `wiki:entity_id`. A (player, slime) pair where the player's `wiki:entity_id` equals the slime's `wiki:pet.owner_id` represents a relationship: the player owns the slime.

Our teleport command requires that we can target the slimes (to teleport them) and access the player's position (so we know where to teleport them). **There is no single selector that allows selecting entities in this relationship.** For example, you cannot use `execute as @e[type=slime] run teleport @s @a[scores={wiki:entity_id = @s wiki:pet.owner_id}]`.
Instead, we take advantage of the fact that we only need the player's position to complete this task. In most cases, such as in this example, we only need the position of one entity in the relationship. Thus, we may implement our pet ownership as follows:
```execute as @e[type=slime] at @a if score @s wiki:pet.owner_id = @p wiki:entity_id run tp @s ~ ~ ~```
It is advisable to add extra protection when doing this, in case multiple entities are stacked on each other, to reduce the chance that `@p` (or `@e[c=1]` or `@n` in 1.21.100+)  selects the wrong entity, though this example does not benefit much. In general, with your `as` and `at` targets, try to be as specific as possible.
```execute as @e[type=slime,scores={wiki:pet.owner_id=-2147483648..}] at @a if score @s wiki:pet.owner_id = @p wiki:entity_id run tp @s ~ ~ ~```

The command `...` will run as entity A at the position of entity B. If you need to target entity B, use `@e[c=1,scores={objective_b=-2147483648..}]`. This is useful if entity B has a score in some objective that needs to be transferred to entity A, subject to a scoreboard link.

# General scoreboard link
If we wanted to run a command on each pair of entities (A, B) where entity A's score in `objective_a` matches entity B's score in `objective_b`, use ```execute as @e[scores={objective_a=-2147483648..}] at @e[scores={objective_b=-2147483648..}] if score @s objective_a = @e[c=1,scores={objective_b=-2147483648..}] objective_b run ...```
## Chaining scoreboard links (advanced)
It is of course possible to chain scoreboard links, though you **should consider if you need to**. I have personally never come across a legitimate need for chaining scoreboard links, as often a relation of the form (A, B) followed by a relation (B, C) can be simplified to a relation (A, C). As a rule of thumb, if you find yourself writing a command like ```execute as @e[scores={objective_a=-2147483648..}] at @e[scores={objective_b=-2147483648..}] if score @s objective_a = @e[c=1,scores={objective_b=-2147483648..}] objective_b as @e[scores={objective_c=-2147483648..}] if score @s objective_c = @e[c=1,scores={objective_b=-2147483648..}] run ...``` where the score of entity B being used to implement the relation (A, B) is the same score being used to implement the relation (B, C), you **do not need to chain scoreboard links.**\* Also, it is impossible to access entities A, B and C simultaneously.

*Note: there is perhaps one situation that would warrant this, and this is when the number of times the command runs on the entity pair matters. If the relationship (A, B) is one-to-many, and the relationship (B, C) is many-to-one, the command will run multiple times instead of just once when using the relationship (A, C).
