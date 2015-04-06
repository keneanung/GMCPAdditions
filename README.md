Module and message names are not case sensitive. JSON key names are case sensitive.

Supported modules
=================
- [Core](#core) - core functionality
- [Char](#char) - information about a character
- [Char.Skills](#charskills) - information about skills known by the player
- [Char.Items](#charitems) - information about items in inventory and room, with live updates
- [Char.Defences](#chardefences) - Sends information about defences of the character
- [Char.Afflictions](#charafflictions) - Sends information about afflictions of the character
- [Comm.Channel](#commchannel) - identification of communication channels and player lists
- [Room](#room) - various information about the current room
- [Redirect](#redirect) - redirect output to another window
- [IRE.Rift](#irerift) - IRE-specific, transmits information about the Rift contents
- [IRE.Composer](#irecomposer) - IRE-specific, used to edit bigger texts client-side
- [IRE.Tasks](#iretask) - used internally by the HTML5 client
- [IRE.Time](#iretime) - used internally by the HTML5 client
- [IRE.Wiz](#irewiz) - used internally by the Nexus client
- [IRE.Misc](#iremisc) - used internally by the HTML5, fMUD and Nexus clients
- [IRE.Display](#iredisplay) - used internally by the HTML5 and Nexus client
- [IRE.FileStore](#irefilestore) - used internally by the HTML5 and Nexus client
- [IRE.Sound](#iresound) - used internally by the HTML5 client
- [IRE.Target](#iretarget) - used internally by the HTML5 client, used to control server side targeting and sync it with client side as well as deliver some additional info


Supported messages by modules
=============================

Core
----

### Sent by client ###

#### Core.Hello ####

- needs to be the first message that the client sends, used to identify the client
- message body is an object with keys "client" and "version", containing the client's name and version
- example: `Core.Hello { "client": "Nexus", "version": "3.1.90" }`

#### Core.Supports.Set ####

- notifies the server about packages supported by the client
- if another Core.Supports.* package has been received earlier, the list is deleted and replaced with the new one
- message body is an array of strings, each consisting of the module name and version, separated by space
- module version is a positive non-zero integer
- most client implementations will only need to send Set once and won't need Add/Remove; exceptions are module implementations provided by plug-ins
- example: `Core.Supports.Set [ "Char 1", "Char.Skills 1", "Char.Items 1" ]`

#### Core.Supports.Add ####

- similar to Set, but appends the supported module list to the one sent earlier
- if no list was sent yet, the behaviour is identical to Set
- if the list includes module names that were already included earlier, the new version number takes precedence over the previously sent one, even if the newly sent number is lower
- message body format is identical to that of Set

#### Core.Supports.Remove #####

- removes specified modules from the list of supported modules
- message body format is similar to Set, except that module version numbers are optional and ignored if given
- example: `Core.Supports.Remove [ "Char", "Char.Skills", "Char.Items" ]`

#### Core.KeepAlive ####

- causes the server to reset the timeout for the logged character, no message body

#### Core.Ping ####

- causes the server to send a Core.Ping back
- message body is a number which indicates average ping time from previous requests, if available
- example: `Core.Ping 120`

### Sent by server ###

#### Core.Ping ####

- Sent in reply to Core.Ping. No body.

#### Core.Goodbye ####
  
- Sent by server immediately before terminating a connection
- Message body is a string to be shown to the user - it can explain the reason for the disconnect
- Example: `Core.Goodbye "Goodbye, adventurer"`



Char
----

### Sent by client ###

#### Char.Login ####

- Used to log in a character, only interpreted if no character is logged in for that connection
- Message body is an object with keys "name" and "password"
- Example: `Char.Login { "name": "somename", "password": "somepassword" }`

### Sent by server ###

#### Char.Name ####

- Contains the characters's name and the complete name including title.
- Message body in an object containing the variables `name` and `fullname`
- example: `Char.Name { "name": "Tecton", "fullname": "Tecton, the Terraformer"}`

#### Char.Vitals ####

- Basic character attributes such as health, mana, etc.
- Message body is an object containing several variables
- Additionally, each variable is included in a string, in the format name:cur/max
- Interpretation of the variables is game specific
- It is generally safe to assume that the values are numbers (even though encoded as strings)
- Example: `Char.Vitals { "hp": "4500", "maxhp": "4800", "mp": "1200", "maxmp": "2500", "ep": "15000", "maxep": "16000", "wp": "14000", "maxwp": "15000", "nl": "10", "string": "H:4500/4800 M:1200/2500 E:15000/16000 W:14000/15000 NL:10/100" }`
#### Char.StatusVars ####

- sent by server after a successful login or after the module is enabled
- contains a list of character variables (level, race, etc)
- message body is an object
- each object element is a name-caption pair, name is the internal name and caption the user-visible one
- example: `Char.StatusVars { "level": "Level", "race": "Race", "guild": "Guild" }`

#### Char.Status ####
  
- values of character values defined by StatusVars
- a full list is sent by server right after StatusVars, and changes are sent in subsequent messages as they occur
- with the exception of the initial Status message, messages only contain changed values; if a variable is not included, it has not changed since the previous Status message
- message body is an object
- each object element is a name-value pair, name is the internal name defined by the StatusVars message and value is the variable value
- example: `Char.Status { "level": "58", "guild": "Guild" }`




Char.Skills
-----------

### Sent by client ###

#### Char.Skills.Get ####

- Sent by client to request skill information
- message body is an object with keys "group" and "name"
- if both group and name is provided, the server will send Char.Skills.Info for the specified skill
- if group is provided but name is not, the server will send Char.Skills.List for that group
- otherwise the server will send Char.Skills.Groups
- example: `Char.Skills.Get { "group": "elemancy", "name": "firelash" }`

### Sent by server ###

#### Char.Skills.Groups ####
  
- groups of skills available to the character
- sent by server on request or at any time (usually if the list changes)
- for IRE games, groups are skills like Survival or Elemancy
- message body is an array of objects, each being one name and the skill rank
- example: `Char.Skills.Groups [ { "name":"Survival", "rank": "Transcendent" }, { "name": "Perception", "rank": "Trascendant" }, { "name": "Elemancy", "rank": "Transcendant"}, { "name": "Crystalism", "rank": "Expert"} ]`

#### Char.Skills.List ####

- list of skills in a group available to the character
- sent by server on request only
- for IRE games, this is the list visible on AB <groupname>
- message body is an object with keys "group", "desc" and "list", where group is the group name as a string
- the desc value is the description of the skill as seen in AB <groupname>
- the list value is an array of strings, each being the name of one skill
- example: `{ "group": "Elemancy", "desc": ["Cast light", "Make your skin hard as stone", "Cast a bold of fire"], "list": ["Light", "Stoneskin", "Firelash"] }`
- **Note** This will return all skills in the group, even not learned ones. Only learned abilities will return a Info.info text though (otherwise it's empty as it is for invalid skills)

#### Char.Skills.Info ####

- information about a single skill, only sent upon request
- message body is an object, keys are "group", "skill", and "info", values are strings
- group and skill identify the request, info is a description (usually multi-line) of the skill's functionality and usage
- Example: `Char.Skills.Info { "group": "Elemancy", "skill": "Firelash", "info": "blah blah" }`



Char.Items
----------

### Sent by client ###

#### Char.Items.Inv ####

- request for the server to send the list of items in player's inventory
- message body is empty
- causes the server to send back an appropriate Char.Items.List message
- Example: `Char.Items.Inv`
 
#### Char.Items.Contents ####

- request for the server to send the list of items located inside another item
- message body is a number identifying the item
- causes the server to send back an appropriate Char.Items.List message
- Example: `Char.Items.Contents 402879`

### Sent by server ###

#### Char.Items.List ####

- list of items at a specified location (room, inv, held container)
- message body is an object with keys "location" and "items"
- location value is a string, "inv", "room", or "repNUMBER" - the last one is container identification
- items value is an array, whose each item is an object with keys "id", "name" and optionally "attrib"
- id is a number identifying the item, name is a string containing a short player-visible item descrption
- attrib is a string consisting of characters describing item properties:
  - "w" = worn, 
  - "W" = wearable but not worn, 
  - "l" = wielded (left),
  - "L" = wielded (right)
  - "lL" = wielded (both) 
  - "g" = groupable, 
  - "c" = container
  - "r" = riftable
  - "f" = fluid
  - "e" = edible
  - "m" = mobile
  - "d" = dead
  - "t" = takeable
- icon: categorization of the item, name suggests icon type to use _(probably added for HTML5 client, not used yet)_
- example: `Char.Items.List { "location": "room", "items": [ {"id": 54685, "name": "an apple"}, {"id": 85462, "name": "a tiny worm"}] }`

#### Char.Items.Add ####
  
- informs the client about an item being added to the specified location
- message body is an object with keys "location" and "item"
- location is same as with List, item is an object with the same structure as one item from the items array of List
- example: `Char.Items.Add { "location": "room", "item": {"id": 123988, "name": "a cat"} }`

#### Char.Items.Update ####

- informs the client about an item's attributes being changed - only sent for inventory items
- message body syntax the same as with Add

#### Char.Items.Remove ####

- informs the client about an item being removed from the location
- message body is an object with the same structure as Add
- example: `Char.Items.Remove { "location": "room", "item": {"id": 123988, "name": "a cat"} }`



Char.Defences
-------------

### Sent by Server ###

#### Char.Defences.List ####

- Sends the list of current active defenses.
- Sent, whenever the client requests the `def` list.
- The body is a list of objects with the following keys
    - `name`: shows the name of the defense
    - `desc`: a short description of the defense
- example: `Char.Defences.List [ { "name": "selfishness", "desc": "Selfishness prevents you from giving away items." }, { "name": "lifevision", "desc": "Lifevison allows you to see traces of all living beings, piercing through illusions and hiding defences." }, { "name": "tekurastance", "desc": "Tekura stances enhance or detract from certain aspects of your attacks." }, { "name": "deafness", "desc": "Deafness can prevent some harmful effects, at the expense of your hearing." }, { "name": "blindness", "desc": "Blindness can prevent some harmful effects, at the expense of your sight." } ]`

#### Char.Defences.Add ####

- Shows newly acquired defenses
- The body has the same structure as the List message
- example: `Char.Defences.Add [ { "name": "selfishness", "desc": "Selfishness prevents you from giving away items." } ]`

#### Char.Defences.Remove ####

- Shows lost defenses
- The body is a list with the name of the lost defense in it
- example: `Char.Afflictions.Remove [ "blindness" ]`



Char.Afflictions
-------------

### Sent by Server ###

#### Char.Afflictions.List ####

- Sends the list of current afflictions.
- Sent, whenever the client requests the `diag` list.
- The body is a list of objects with the following keys
    - `name`: shows the name of the affliction
    - `cure`: shows the commands to cure this affliction
    - `desc`: a short description of the affliction
- example: `Char.Afflictions.List [ { "name": "insomnia", "cure": "EAT GOLDENSEAL", "desc": "Insomnia makes it very difficult to fall asleep." }, { "name": "deafness", "cure": "APPLY EPIDERMAL TO HEAD", "desc": "Deafness stops you from hearing anything." }, { "name": "blindness", "cure": "APPLY EPIDERMAL TO HEAD", "desc": "Blindness makes it impossible to see the world around you." } ]`

#### Char.Afflictions.Add ####

- Shows newly acquired afflictions
- The body has the same structure as the List message
- example: `Char.Afflictions.Add [ { "name": "insomnia", "cure": "EAT GOLDENSEAL", "desc": "Insomnia makes it very difficult to fall asleep." } ]`

#### Char.Afflictions.Remove ####

- Shows lost afflictions
- The body is a list with the name of the lost affliction in it
- example: `Char.Afflictions.Remove [ "blindness" ]`

 

Comm.Channel
------------

### Sent by client ###

#### Comm.Channel.Players ####

- request for the server to send Comm.Channel.Players
- no message body

#### Comm.Channel.Enable ####

- no info yet (found in http://client.achaea.com/includes/js/ui/channels.js)
  
### Sent by server ###

#### Comm.Channel.Players ####

- list of players and organizations (city, guild, ...) that they share with this player
- message body is an array with each element describing one player
- each element is an object with keys "name" and "channels", name is a string, channels is an array
- the channels array may be omitted if empty; if given, it is a list of organization names
- example: `Comm.Channel.Players [{"name": "Player1", "channels: ["Some city", "Some guild"]}, {"name": "Player2"}]`

#### Comm.Channel.List ####
 
- list of communication channels available to the player, sent on login/negotiation and if changed
- message body is an array of objects, each object representing one channel
- each object has keys "name", "caption" and "command" - name is internal name, caption is player-visible name, command is command used to communicate over this channel
- example: `Comm.Channel.List [{"name":"ct", "caption":"Some city", "command":"ct"}, {"name":"gt", "caption":"Some guild", "command":"gt"}]`

#### Comm.Channel.Start (*deprecated*) ####

- informs the client that text that follows is something said over a communication channel
- message body is a text containing the channel name
- for tells from/to another player, the channel name is "tell Name"
- Example: `Comm.Channel.Start "ct"`
- Example: `Comm.Channel.Start "tell Player1"`

#### Comm.Channel.End (*deprecated*) ####

- ends a channel text started by Comm.Channel.Start
- message body is a text containing the channel name

#### Comm.Channel.Text ####

- Complete information about a communication.
- The body is an object with the fields "talker", "channel" and "text" - talker is the person, who uses the channel, channel is the channel used, text is the complete text that is sent. This includes possibile ANSI colors and MXP commands
- example: `Comm.Channel.Text { "channel": "newbie", "talker": "Juliet", "text": "\u001b[0;1;32m(Newbie): Juliet says, \"You would simply slay Beku with Your lightning I imagine, Lady Aurora!\"\u001b[0;37m" }`



Room
----

### Sent by server ###

#### Room.Info ####
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
- Example: `Room.Info {"num": 12345, "name": "On a hill", "area": "Barren hills", "environment": "Hills", "coords": "45,5,4,3", "map": "www.imperian.com/itex/maps/clientmap.php?map=45&level=3 5 4", "exits": { "n": 12344, "se": 12336 }, "details": [ "shop", "bank" ] }`

#### Room.WrongDir ####

- Sent if the player attempts to move in a non-existant direction using the standard movement commands
- Upon receiving this message, the client can safely assume that the specified direction does not lead anywhere at this time
- Message body is a string with the name if the non-existant exit
- Example: `Room.WrongDir "ne"`

#### Room.Players ####

- A list of objects containing player details
- Each player object contains the keys `name` and `fullname`, which represent the actual name and the name including all titles, respectively.
- Example: `Room.Players [{"name": "Tecton", "fullname":"Tecton, the Terraformer"}, { "name": "Cardan", "fullname":"Cardan, the Curious"}]`

#### Room.AddPlayer ####

- Sent, if a player enters the room
- The message body is an object with the same structure of a single player in Room.Players
- Example: `Room.AddPlayer { "name": "Cardan", "fullname": "Cardan, the Curious" }`

#### Room.RemovePlayer ####

- Sent, when a player leaves a room
- Message body is a string with the name of the leaving player.
- Example: `Room.RemovePlayer "Cardan"`



Redirect
--------

### Sent by server ###

#### Redirect.Window ####

- Specifies a window to redirect further input to
- Message body is a string specifying the window to redirect to
- The main window is referred to as "main", and is the default if the message body is omitted or empty
- Example: `Redirect.Window "map"`




IRE.Rift
--------

### Sent by server ###

#### IRE.Rift.List ####

- contents of a Rift storage
- sent upon receiving the IRE.Rift.Request message
- message body is an array, with each element being an object containing three keys - "name" is item name, "amount" is a number holding the item's amount, and "desc" is user-visible description

#### IRE.Rift.Change ####

- sent whenever the item amount in a Rift changes
- message body is an object with the same structure as one element of an array sent with the IRE.Rift.List message


### Sent by client ###

#### IRE.Rift.Request ####
- asks the server to send the Rift contents using the IRE.Rift.List message
- no body



IRE.Composer
------------

### Sent by server ###

#### IRE.Composer.Edit ####

- sent by the server when the player enters an in-game editor
- Body is an object, with keys "title" and "text". Text contains the current buffer, title is a title that can be shown to the user.


### Sent by client ###

#### IRE.Composer.SetBuffer ####

- Sent by the client upon successfully editing a text which was sent to the client in an IRE.Composer.Edit message earlier
- Sending this message only changes the edit buffer, but does not end the editing session
- on IRE games, the client may send the command -`*save` to save a text, or command `*quit` to abort editing (IRE.Composer.SetBuffer is not sent in this case) - this behaviour is IRE-specific and is one of the reasons why the Composer module is in the IRE namespace



IRE.Tasks
---------

### Sent by client ###

#### IRE.Tasks.Request ####

- Sent by the client to request the current list of tasks.
- no body.

### Sent by server ###

#### IRE.Tasks.List ####

- Sent by the server to transmit a list of tasks the character has
- The body is an array of objects with the following fields:
  - "id": Unique ID of the task
  - "name": Name of the task
  - "desc": Describtion of the task, including details how to solve the task
  - "type": Type of the task. Known types are `Achievements`, `Quest`and `Task`
  - "cmd": Command to show the task via "normal" means
  - "status": : Status of the task. May be an empty string, "1" for completed tasks and "0" for uncompleted
  - "group": Group the task is organized into. On completion, tasks are moved to the "Completed" group.
- example: `IRE.Tasks.List [ { "id": "1", "name": "Break Free of Your Imprisonment", "desc": "Where are you? What's going on? There's no time to waste, you need to get out of here!\n\nPay close attention to the directions and tips on your screen and you'll be out of the dungeon in no time.", "type": "Task", "cmd": "NEWTASK 1 INFO", "status": "1", "group": "Completed" },  { "id": "122", "name": "Feed the Birds", "desc": "Paloma has asked you to scout around the city for stale bread, so she   \ncan continue feeding the pigeons in Artisan Plaza.                      ", "type": "Quest", "cmd": "QUEST 122 DETAILS", "status": "", "group": "Cyrene" } ]`

#### IRE.Tasks.Update ####
  
- Sent by the server to show that a task was changed.
- The body structure is the same as the one of IRE.Tasks.List



IRE.Time
--------

### Sent by Client ###

#### IRE.Time.Request ####
  
- Sent by the client to request the current IRE.Time.List
- no body


### Sent by Server ###

#### IRE.Time.List ####

- Sent by the server to transmit the current time and addtional information about the time.
- The body is am object with the following fields:
  - "time": The time of the day
  - "moonphase": phase of the moon
  - "day": current day of the month
  - "month": name of the current month
  - "mon": nummerical representation of the current month
  - "hour": current hour of the day
  - "year": current year
  - "daynight": *time of the day in a numerical representation, resets with the morning* (highly speculative)
- example: `IRE.Time.List { "day": "5", "mon": "9", "month": "Phaestian", "year": "648", "hour": "51", "time": "Dusk has overtaken the light in Achaea.", "moonphase": "Waxing Gibbous", "daynight": "122" }`

#### IRE.Time.Update ####

- sent by the server on time update
- body has the same stucture as IRE.Time.List
- only the changed fields are transmitted



IRE.Misc
--------

### Sent by server ###

#### IRE.Misc.RemindVote ####

- Sent by the server to remind the client to vote
- The body is a string that contains the url to vote
- example: `IRE.Misc.RemindVote "http://www.topmudsites.com/cgi-bin/topmuds/rankem.cgi?id=sarapis"`

#### IRE.Misc.Achievement ####

- Sent by the server when an achievement is met
- the body is an object with the name and value of the achievement
- example: `IRE.Misc.Achievement { "name": "AchievedLevel21", "value": "1" }`

#### IRE.Misc.OneTimePassword ####

- no info yet

#### IRE.Misc.JavaEnv ####

- no info yet

### Sent by client ###

#### IRE.Misc.OneTimePassword ####

- no info yet (found in http://client.achaea.com/includes/js/ui/dialogs.js)


IRE.Display
-----------

#### IRE.Display.Help ####

- Gating event for displaying help contents.
- Body is a string containing `start` to open the gate and `stop` to close it

#### IRE.Display.FixedFont (*speculative*) ####

- Used to order the client to use a fixed font (e.g. Monospace fonts) to display content
- Body is a string that may contain `start` to start the fixed part and `stop` to stop the part again.

#### IRE.Display.AutoFill ####

- Fill the input of the client with the given text.
- Body is an object with the following fields:
  - "command": The command to be put into the input line.
  - "highlight": string or bool (?) marking the line to be selected or not
- Example: `IRE.Display.AutoFill { "command": "Help help", "highlight": false }`

#### IRE.Display.HidePopup ####

- Message to tell the client to remove a popup from display
- Message body is an object with the key "id", showing the popup to be hidden

#### IRE.Display.HideAllPopups ####

- Hides all popups
- no body

#### IRE.Display.Popup ####

- Used to show a popup to the client
- The body is an object with the following fields:
  - "id": ID of the popup
  - "text": The text to display
  - "options": Options to use with the popup. Known options are display time (`time`, integer) and transparency (`transparent`, boolean)
  - "element": (HTML-ID of the) parent element of the popup.
  - "src": An image that should be displayed
  - "cmd": A command that should be run by the client upon clicking the popup. Known ones are `done`, `continue`, and `back` 
- Example: `IRE.Display.Popup {"id":"intro_step_4","text":"This window displays a map of your surroundings.","options":{"time":"20", "transparent":true},"element":"output"}`

#### IRE.Display.Ohmap ####

- Gating event for displaying the wilderness (overhead) map.
- Body is a string containing `start` immediately before the output starts or `stop` immidiately after the last line of the map
- Example: `IRE.Display.Ohmap "start"`



IRE.FileStore
-------------

### Sent by server

#### IRE.FileStore.Content ####
  
- Used to send server stored content to the client. The content is Bas64 encoded.
- For the HTML5 client, this is simple JavaScript, that seems to be run after receiving the message.

### Sent by client ###

#### IRE.FileStore.Request ####

- Used to access the reflex packages saved on the server. Depending on the request, different actions are taken.
- Body is an object with the keys "request" and "data". Those are filled in the following manner:

| request              | data                                                               | description                                       |
|----------------------|--------------------------------------------------------------------|-----------------------------------------------------------|
| clr                  | null                                                               | Clears the data on the server                             |
| add                  | Base64 encoded data to put on the server. Max length 8192 char (?) | Adds the data to the data already stored on the server.   |
| put *html5-reflexes* | null                                                               | Points the server to the storages space (?) where to previously `put` data should be stored. |


IRE.Sound
---------

#### IRE.Sound.Preload ####

- no info yet

#### IRE.Sound.Play ####

- no info yet

#### IRE.Sound.Stop ####

- no info yet

#### IRE.Sound.StopAll ####

- no info yet


IRE.Target
----------

### Sent by client ###

#### IRE.Target.Set ####

- Used to set the server side variable `&tar` and request target information about the target.
- The body is a string containing the ID of the target.
- Example: `IRE.Target.Set "12345"`

### Sent by Server ###

#### IRE.Target.Set ####

- Used to notify the client of a change of the `&tar` variable
- The body is a string containing the ID of the target or an empty string if unset.
- Example: `IRE.Target.Set "12345"`

#### IRE.Target.Info ####

- Used to send additional information about the current active server side target (contents of the `&tar`-variable).
- The body is an object the following fields:
  - `short_desc`: Contains the short description of the target
  - `hpperc`: Contains the current HP of the target as a percentage
  - `id`: The ID of the target.
- Example: `IRE.Target.Info {short_desc="a practice dummy",hpperc="100%",id="266744"}`

#### IRE.Target.Request ####

- Unconfirmed as not seen in real life yet
- Used by the server to re-request the target
- No body
