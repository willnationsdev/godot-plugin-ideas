godot-skills

Description: To design a generic skill system that...

1. Can be used for a variety of game projects.
2. Takes advantage of Godot's node system.
3. Maximizes re-usability of skill effects and forms.

Purpose: Simplify the process of designing intricate and varied interactions in gameplay. Apply composition to the design of interactions.

Design Structure:

1. `add_custom_type`: `Effect extends Node`
    - `Effect`s are functors used to functionally map a change to some target.
    - `Effect`s implement the Command pattern and can execute undo/redo operations on their targets.
    - `Effect`s extended with scripting may use their own instance properties in their implementation. For example, a `DamageEffect` may have `amount` and `type` properties.
    - `Effect`s have `func apply(p_source, p_target)` where `p_source` is the one using the skill, and `p_target` is the thing operated on by the `Effect` (could be a `SkillUser` or even another `Skill`).
    - An `Effect`s effect may even be to spawn another `Skill` instance. This allows for easy `Skill` chaining for things like the common "Chain Lightning" ability in RPGs.
2. `add_custom_type`: `TargetingWorld extends Node`
    - `TargetingWorld` is responsible for processing all static `Targeter`s requests for targeting.
    - It is recommended that only one `TargetingWorld` is used and that it be an Autoload Singleton.
    - `Targeter`s maintain a list of `Targeter`s that they process.
3. `add_custom_type`: `Targeter extends Node`
    - `Targeter`s maintain a static array ( export(StringArray) ) of NodePaths to all `TargetingWorld`s they wish to work with. This is mainly so that a common NodePath variable can be shared between all `Targeter` instances (won't have to hard-code it in a static function's return value).
    - When a `Targeter` is added to a scene, it will search for all `TargetingWorld`s it knows about and will add itself to the `TargetingWorld`s list of `Targeter`s.
    - When a `Targeter` is added to a scene, the `TargetingWorld` will grab the `Targeter`'s parent and check to see if any of the currently stored `Targeter`s .canTarget(...) the node.
    - `Targeter`s also have `export(NodePath) var target_path = ""`. If `target_path` is non-empty, the node at that path will be included in the list of targets.
4. `add_custom_type`: `Skill extends Node`
    - `Skill`s are named collections of `Effect`s that use `Targeter`s to automatically map their child `Effect`s to targets. Note that this allows static saving of effects-to-targets via scene design.
    - `Skill`s need at least one `Targeter` or `Targeter2D` child and at least one `Effect` child to operate properly.
    - `Skill`s maintain an internal array of both their child `Effect`s and child `Targeter`s/`Targeter2D`s. `Effect`s and `Targeter`s/`Targeter2D`s consequently will add or remove themselves from their parent `Skill`s corresponding array when `NOTIFICATION_PARENTED` and `NOTIFICATION_UNPARENTED` are received in `_notification(int)`.
    - `Skill`s implement the Command pattern and can execute undo/redo operations on their targets.
    - `Skill`s can "test" themselves on their targets. Need to have a `test_properties` method that applies the effects on the ACTUAL target(s), collects the named properties on those targets, and then reverts those targets to return them to their previous state, returning a Dictionary of the targets' NodePaths mapped to the dictionary of property name/values. Also need a `generate_test_result` method that creates copies of all of the targets, applies the effects to them, and then returns a Dictionary of those targets' NodePaths mapped to the associated copy. Documentation would need to highlight how the latter is much more expensive.
5. `add_custom_type`: `SkillCondition extends Reference`
    - associates a `Skill` with some condition for its becoming "inactive". Could be a `Timer`, could be a limited-use thing. Doesn't matter. Just need a loose way of coupling a `Skill` to *some* means of deactivating it.
6. `add_custom_type`: `SkillFilter extends Reference`
    - Stores `Skill`s paired with `SkillCondition`s. When a `SkillCondition` triggers a signal notifying that the `Skill` is done, it removes the `Skill` and itself from the `SkillFilter`'s `Skill`-`SkillCondition` `Dictionary`.
    - Applies stored `Skill`s to any passing through `Skill`s. `Effect`s can therefore be applied to a `SkillUser` that add `Skill`s matched with a `SkillCondition` to these `SkillFilter`s. Think skills that add or remove status effects from a target.
7. `add_custom_type`: `SkillUser extends Node`
    - `SkillUser`s maintain a list of `Skill` names that are currently owned by them.
    - Individual owned `Skill`s can be enabled or disabled on the `SkillUser`.
    - `SkillUser`s have two `SkillFilter`s, one for incoming `Skill`s and one for outgoing `Skill`s.
    - `SkillUser`s can `use` or `revert` (do/redo) a `Skill` by name with an optional target override, e.g. `my_skill_user.use("HealSelf", null)` or `my_skill_user.use("Fireball", $targeter.get_targets())`.
8. Add an optional inventory system that allows people to attach `Skill`s to inventory items (consumables)