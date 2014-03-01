Module and message names are not case sensitive. JSON key names are case sensitive.

Supported modules
=================
- [Core](#core) - core functionality
- [Char](#char) - information about a character
- [Char.Skills](#charskills) - information about skills known by the player
- [Char.Items](#charitems) - information about items in inventory and room, with live updates
- [Comm.Channel](#commchannel) - identification of communication channels and player lists
- [Room](#room) - various information about the current room
- [Redirect](#redirect) - redirect outpot to another window
- [IRE.Rift](#irerift) - IRE-specific, transmits information about the Rift contents
- [IRE.Composer](#irecomposer) - IRE-specific, used to edit bigger texts client-side
- [IRE.Wiz](#irewiz) - used internally by the Nexus client
- [IRE.FileStore](#irefilestore) - used internally by the Nexus client
- [IRE.Misc](#iremisc) - used internally by the fMUD and Nexus clients


Supported messages by modules
=============================

Core
----

### Sent by client ###

- Core.Hello
  - needs to be the first message that the client sends, used to identify the client
  - message body is an object with keys "client" and "version", containing the client's name and version
  - example: `Core.Hello { "client": "Nexus", "version": "3.1.90" }`
- Core.Supports.Set
  - notifies the server about packages supported by the client
  - if another Core.Supports.* package has been received earlier, the list is deleted and replaced with the new one
  - message body is an array of strings, each consisting of the module name and version, separated by space
  - module version is a positive non-zero integer
  - most client implementations will only need to send Set once and won't need Add/Remove; exceptions are module implementations provided by plug-ins
  - example: `Core.Supports.Set [ "Char 1", "Char.Skills 1", "Char.Items 1" ]`
- Core.Supports.Add
  - similar to Set, but appends the supported module list to the one sent earlier
  - if no list was sent yet, the behaviour is identical to Set
  - if the list includes module names that were already included earlier, the new version number takes precedence over the previously sent one, even if the newly sent number is lower
  - message body format is identical to that of Set
- Core.Supports.Remove
  - removes specified modules from the list of supported modules
  - message body format is similar to Set, except that module version numbers are optional and ignored if given
  - example: `Core.Supports.Remove [ "Char", "Char.Skills", "Char.Items" ]`
- Core.KeepAlive
  - causes the server to reset the timeout for the logged character, no message body
- Core.Ping
  - causes the server to send a Core.Ping back
  - message body is a number which indicates average ping time from previous requests, if available
  - example: `Core.Ping 120`

### Sent by server ###
- Core.Ping
  - Sent in reply to Core.Ping. No body.
- Core.Goodbye
  - Sent by server immediately before terminating a connection
  - Message body is a string to be shown to the user - it can explain the reason for the disconnect
  - Example: `Core.Goodbye "Goodbye, adventurer"`



Char
----

### Sent by client ###

- Char.Login
  - Used to log in a character, only interpreted if no character is logged in for that connection
  - Message body is an object with keys "name" and "password"
  - Example: `Char.Login { "name": "somename", "password": "somepassword" }`

### Sent by server ###


- Char.Name
  - Contains the characters's name and the complete name including title.
  - Message body in an object containing the variables `name` and `fullname`
  - example: `Char.Name { "name": "Tecton", "fullname": "Tecton, the Terraformer"}`
- Char.Vitals
  - Basic character attributes such as health, mana, etc.
  - Message body is an object containing several variables
  - Additionally, each variable is included in a string, in the format name:cur/max
  - Interpretation of the variables is game specific
  - It is generally safe to assume that the values are numbers (even though encoded as strings)
  - Example: `Char.Vitals { "hp": "4500", "maxhp": "4800", "mp": "1200", "maxmp": "2500", "ep": "15000", "maxep": "16000", "wp": "14000", "maxwp": "15000", "nl": "10", "string": "H:4500/4800 M:1200/2500 E:15000/16000 W:14000/15000 NL:10/100" }`
- Char.StatusVars
  - sent by server after a successful login or after the module is enabled
  - contains a list of character variables (level, race, etc)
  - message body is an object
  - each object element is a name-caption pair, name is the internal name and caption the user-visible one
  - example: `Char.StatusVars { "level": "Level", "race": "Race", "guild": "Guild" }`
- Char.Status
  - values of character values defined by StatusVars
  - a full list is sent by server right after StatusVars, and changes are sent in subsequent messages as they occur
  - with the exception of the initial Status message, messages only contain changed values; if a variable is not included, it has not changed since the previous Status message
  - message body is an object
  - each object element is a name-value pair, name is the internal name defined by the StatusVars message and value is the variable value
  - example: `Char.Status { "level": "58", "guild": "Guild" }`




Char.Skills
-----------

### Sent by client ###

- Char.Skills.Get
  - Sent by client to request skill information
  - message body is an object with keys "group" and "name"
  - if both group and name is provided, the server will send Char.Skills.Info for the specified skill
  - if group is provided but name is not, the server will send Char.Skills.List for that group
  - otherwise the server will send Char.Skills.Groups
  - example: `Char.Skills.Get { "group": "elemancy", "name": "firelash" }`

### Sent by server ###

- Char.Skills.Groups
  - groups of skills available to the character
  - sent by server on request or at any time (usually if the list changes)
  - for IRE games, groups are skills like Survival or Elemancy
  - message body is an array of strings, each being one name
  - example: `Char.Skills.Groups [ "Survival", "Perception", "Elemancy", "Crystalism" ]`
- Char.Skills.List
  - list of skills in a group available to the character
  - sent by server on request only
  - for IRE games, this is the list visible on AB <skillname>
  - message body is an object with keys "group" and "list", where group is the group name as a string
  - the list value is an array of strings, each being the name of one skill
  - example: `{ "group": "Elemancy", "list": ["Light", "Stoneskin", "Firelash"] }`
- Char.Skills.Info
  - information about a single skill, only sent upon request
  - message body is an object, keys are "group", "skill", and "info", values are strings
  - group and skill identify the request, info is a description (usually multi-line) of the skill's functionality and usage
  - Example: `Char.Skills.Info { "group": "Elemancy", "skill": "Firelash", "blah blah" }`



Char.Items
----------

### Sent by client ###

- Char.Items.Inv
  - request for the server to send the list of items in player's inventory
  - message body is empty
  - causes the server to send back an appropriate Char.Items.List message
- Char.Items.Contents
  - request for the server to send the list of items located inside another item
  - message body is a number identifying the item
  - causes the server to send back an appropriate Char.Items.List message

### Sent by server ###

- Char.Items.List
  - list of items at a specified location (room, inv, held container)
  - message body is an object with keys "location" and "items"
  - location value is a string, "inv", "room", or "repNUMBER" - the last one is container identification
  - items value is an array, whose each item is an object with keys "id", "name" and optionally "attrib"
  - id is a number identifying the item, name is a string containing a short player-visible item descrption
  - attrib is a string consisting of characters describing item properties:
    - "w" = worn, 
    - "W" = wearable but not worn, 
    - "l" = wielded, 
    - "g" = groupable, 
    - "c" = container
    - "r" = riftable
    - "f" = fluid
    - "e" = edible
    - "m" = mobile
    - "d" = dead
    - "t" = takeable
  - example: `Char.Items.List { "location": "room", "items": [ {"id": 54685, "name": "an apple"}, {"id": 85462, "name": "a tiny worm"}] }`
- Char.Items.Add
  - informs the client about an item being added to the specified location
  - message body is an object with keys "location" and "item"
  - location is same as with List, item is an object with the same structure as one item from the items array of List
  - example: `Char.Items.Add { "location": "room", "item": {"id": 123988, "name": "a cat"} }`
- Char.Items.Update
  - informs the client about an item's attributes being changed - only sent for inventory items
  - message body syntax the same as with Add
- Char.Items.Remove
  - informs the client about an item being removed from the location
  - message body is an object with keys "location" and "item"
  - location is same as with List, item is an integer value identifying the item
  - example: `Char.Items.Remove { "location": "room", "item": 123988 }`

Comm.Channel
------------

### Sent by client ###

- Comm.Channel.Players
  - request for the server to send Comm.Channel.Players
  - no message body
  
### Sent by server ###

- Comm.Channel.Players
  - list of players and organizations (city, guild, ...) that they share with this player
  - message body is an array with each element describing one player
  - each element is an object with keys "name" and "channels", name is a string, channels is an array
  - the channels array may be omitted if empty; if given, it is a list of organization names
  - example: `Comm.Channel.Players [{"name": "Player1", "channels: ["Some city", "Some guild"]}, {"name": "Player2"}]`
- Comm.Channel.List
  - list of communication channels available to the player, sent on login/negotiation and if changed
  - message body is an array of objects, each object representing one channel
  - each object has keys "name", "caption" and "command" - name is internal name, caption is player-visible name, command is command used to communicate over this channel
  - example: `Comm.Channel.List [{"name":"ct", "caption":"Some city", "command":"ct"}, {"name":"gt", "caption":"Some guild", "command":"gt"}]`
- Comm.Channel.Start
  - informs the client that text that follows is something said over a communication channel
  - message body is a text containing the channel name
  - for tells from/to another player, the channel name is "tell Name"
  - Example: `Comm.Channel.Start "ct"`
  - Example: `Comm.Channel.Start "tell Player1"`
- Comm.Channel.End
  - ends a channel text started by Comm.Channel.Start
  - message body is a text containing the channel name
- Comm.Channel.Text
  - Complete information about a communication.
  - The body is an object with the fields "talker", "channel" and "text" - talker is the person, who uses the channel, channel is the channel used, text is the complete text that is sent. This includes possibile ANSI colors and maybe even MXP (the latter is unconfirmed)
  - example: `Comm.Channel.Text { "channel": "newbie", "talker": "Juliet", "text": "\u001b[0;1;32m(Newbie): Juliet says, \"You would simply slay Beku with Your lightning I imagine, Lady Aurora!\"\u001b[0;37m" }`



Room
----

### Sent by server ###

- Room.Info
  - Contains information about the room that the player is in. Some of these may be IRE-specific
  - Message body is an object with the following keys
    - "num" - number identifying the room
    - "name" - string containing the brief description
    - "area" - string containing area name
    - "environment" - string containing environment type ("Hills", "Ocean", ...)
    - "coords" - room coordinates (string of numbers separated by commas - area,X,Y,X,building, building is optional
    - "map" - map information - URL pointing to a map image, followed by X and Y room (not pixel) coordinates on the map
    - "details" - array holding further information about a room - shop,bank,...
    - "exits" - object containing exits, each key is a direction and each value is the number identifying the target room
  - `Example: Room.Info {"num": 12345, "name": "On a hill", "area": "Barren hills", "environment": "Hills", "coords": "45,5,4,3", "map": "www.imperian.com/itex/maps/clientmap.php?map=45&level=3 5 4", "exits": { "n": 12344, "se": 12336 }, "details": [ "shop", "bank" ] }`
- Room.WrongDir
  - Sent if the player attempts to move in a non-existant direction using the standard movement commands
  - Upon receiving this message, the client can safely assume that the specified direction does not lead anywhere at this time
  - Message body is a string with the name if the non-existant exit
  - Example: `Room.WrongDir "ne"`



Redirect
--------

### Sent by server ###

- Redirect.Window
  - Specifies a window to redirect further input to
  - Message body is a string specifying the window to redirect to
  - The main window is referred to as "main", and is the default if the message body is omitted or empty
  - Example: `Redirect.Window "map"`




IRE.Rift
--------

### Sent by server ###

- IRE.Rift.List
  - contents of a Rift storage
  - sent upon receiving the IRE.Rift.Request message
  - message body is an array, with each element being an object containing three keys - "name" is item name, "amount" is a number holding the item's amount, and "desc" is user-visible description
- IRE.Rift.Change
  - sent whenever the item amount in a Rift changes
  - message body is an object with the same structure as one element of an array sent with the IRE.Rift.List message

### Sent by client ###

- IRE.Rift.Request
  - asks the server to send the Rift contents using the IRE.Rift.List message



IRE.Composer
------------

### Sent by server ###

- IRE.Composer.Edit
  - sent by the server when the player enters an in-game editor. Body is an object, with keys "title" and "text". Text contains the current buffer, title is a title that can be shown to the user.

### Sent by client ###

- IRE.Composer.SetBuffer
  - Sent by the client upon successfully editing a text which was sent to the client in an IRE.Composer.Edit message earlier
  - Sending this message only changes the edit buffer, but does not end the editing session
  - on IRE games, the client may send the command -`*save` to save a text, or command `*quit` to abort editing (IRE.Composer.SetBuffer is not sent in this case) - this behaviour is IRE-specific and is one of the reasons why the Composer module is in the IRE namespace


