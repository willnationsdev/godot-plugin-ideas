godot-silk

Description: To design a micro-scale narrative scripting system similar to Twine. It would...

1. Allow users to create individual stories where a location in the story could be discretely saved.
2. Allow users to create individual passages within a story that can be easily linked together / scripted / tagged like in Twine.
3. Allow users to easily script Actor behaviors (who is talking, in what manner, etc.) as is done in YarnSpinner for Unity.
4. Provide "scripts" to users the same way "macros" are provided in Twine. Allow users to add their own scripts. - For this, we probably need to edit `RichTextLabel` to allow custom bbcode definitions.
5. Enable multiple simultaneous uses of any set of Stories during live gameplay (not like Twine where you follow only a single thread at a time). Users should be able to start a story, triggering a set of passages and their associated actions / dialogue boxes, and then start the same story again, creating an "echo" effect in the game world.
6. Allow users to have multiple entry points for their story content.

Purpose: Simplify the process of scripting narrative sequences in Godot.

Design Structure:

1. `add_custom_type`: `SilkEditor extends Node`.
    - Must work as a tool.
    - Provides a "system" wrapper for the Silk system.
    - Allows the user to easily swap between stories.
2. `add_custom_type`: `SilkStory extends GraphEdit`.
    - Provides a web in which to define the structure of your story.
3. `add_custom_type`: `SilkLabel extends RichTextLabel`.
    - Custom bbcodes allow for easy linking to other `SilkPassages`, e.g. `[silk text="" link="passage_name"]`, or even better `[text](passage_name)`.
    - Custom bbcodes allow for easy conditional logic and loops, e.g. `[if="condition"]`, `[elif="condition"]`, `[else]`, `[while="condition"]`.
    - Custom bbcodes allow for simple definition and manipulation of story variables, with SilkEditor-scope, e.g. `[my_var]`, `[my_var="val"]`, `[my_var+="10"]`.
3. `add_custom_type`: `SilkPassage extends GraphNode`.
    - Allows the user to enter story content using a `RichTextLabel`.
    - Custom bbcodes for the RichTextLabel should be defined
    - Adding a `SilkPassage` to a `SilkStory` would immediately update a `SilkEditor`-spanning "global" scope of passage names. This enables a link in one `SilkStory` to easily jump to a passage in another `SilkStory`. Links of this nature will need to be clearly communicated in the `SilkStory` UI. Perhaps have there be a dummy `SilkPassage` still appear in this story that, when clicked on, jumps you over to the associated `SilkStory` and `SilkPassage`.
    - Signals would exist to notify other game systems when a particular `SilkPassage` is entered/exited, and when individual lines of dialogue are advanced.
4. Users will save their game's stories by simply saving the scene with their `SilkEditor` at the root. After all, the content and structure is all saved in the contained `SilkStory` nodes.