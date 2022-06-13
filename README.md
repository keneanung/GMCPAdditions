# Additional documentation of GMCP modules #

Module and message names are not case sensitive. JSON key names are case sensitive.

Source: <http://nexus.ironrealms.com/GMCP> and <http://play.achaea.com> sourcecode

The examples in this document show a formatted example message body. For the full GMCP message, they need to be prefixed with the message name. For example the full `Char.Items.Update` message would look like this: `Char.Items.Update { "location": "room", "item": {"id": 123988, "name": "a cat"} }`.

## Supported modules ##

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
- [IRE.Tasks](#iretasks) - used internally by the HTML5 client
- [IRE.Time](#iretime) - used internally by the HTML5 client
- [IRE.Wiz](#irewiz) - used internally by the Nexus client
- [IRE.Misc](#iremisc) - used internally by the HTML5, fMUD and Nexus clients
- [IRE.Display](#iredisplay) - used internally by the HTML5 and Nexus client
- [IRE.FileStore](#irefilestore) - used internally by the HTML5 and Nexus client
- [IRE.Sound](#iresound) - used internally by the HTML5 client
- [IRE.Target](#iretarget) - used internally by the HTML5 client, used to control server side targeting and sync it with client side as well as deliver some additional info
- [Client](#client) - used to transmit game enhancing client side modules
- [External.Discord](#externaldiscord) - used to set Discord rich presence information

## Supported messages by modules ##

### Core ###

#### Sent by client ####

##### Core.Hello #####

- Needs to be the first message that the client sends, used to identify the client
- Message body is an object with keys "client" and "version", containing the client's name and version
- Example:

```json
{
  "client": "Nexus",
  "version": "3.1.90"
}
```

##### Core.Supports.Set #####

- Notifies the server about packages supported by the client
- If another Core.Supports.* package has been received earlier, the list is deleted and replaced with the new one
- Message body is an array of strings, each consisting of the module name and version, separated by space
- Module version is a positive non-zero integer
- Most client implementations will only need to send Set once and won't need Add/Remove; exceptions are module implementations provided by plug-ins
- Example:

```json
[
  "Char 1",
  "Char.Skills 1",
  "Char.Items 1"
]
```

##### Core.Supports.Add #####

- Similar to Set, but appends the supported module list to the one sent earlier
- If no list was sent yet, the behaviour is identical to Set
- If the list includes module names that were already included earlier, the new version number takes precedence over the previously sent one, even if the newly sent number is lower
- Message body format is identical to that of Set

##### Core.Supports.Remove ######

- Removes specified modules from the list of supported modules
- Message body format is similar to Set, except that module version numbers are optional and ignored if given
- Example:

```json
[
  "Char",
  "Char.Skills",
  "Char.Items"
]
```

##### Core.KeepAlive #####

- Causes the server to reset the timeout for the logged character, no message body

##### Core.Ping #####

- Causes the server to send a Core.Ping back
- Message body is a number which indicates average ping time from previous requests, if available
- Example:

```json
120
```

#### Sent by server ####

##### Core.Ping #####

- Sent in reply to Core.Ping. No body.

##### Core.Goodbye #####

- Sent by server immediately before terminating a connection
- Message body is a string to be shown to the user - it can explain the reason for the disconnect
- Example:

```json
"Goodbye, adventurer"
```

### Char ###

#### Sent by client ####

##### Char.Login #####

- Used to log in a character, only interpreted if no character is logged in for that connection
- Message body is an object with keys "name" and "password"
- Example:

```json
{
  "name": "somename",
  "password": "somepassword"
}
```

#### Sent by server ####

##### Char.Name #####

- Contains the characters's name and the complete name including title.
- Message body in an object containing the variables `name` and `fullname`
- Example:

```json
{
  "name": "Tecton",
  "fullname": "Tecton, the Terraformer"
}
```

##### Char.Vitals #####

- Basic character attributes such as health, mana, etc.
- Message body is an object containing several variables
- Additionally, each variable is included in a string, in the format name:cur/max
- Interpretation of the variables is game specific
- It is generally safe to assume that the values are numbers (even though encoded as strings)
- Possible values (these are specific for the IRE game Achaea):
  - `hp`: current health points
  - `maxhp`: maximum health points
  - `mp`: current mana points
  - `maxmp`: maximum mana points
  - `ep`: current endurance points
  - `maxep`: maximum endurance points
  - `wp`: current willpower points
  - `maxwp`: maximum willpower points
  - `nl`: current percentage on the way to the next level
  - `string` all of the former in the form `current/max` as a single string
  - `charstats`: An array of class specific values (+ bleeding) in a string per value. Format: `var: value`
- Example:

```json
{
  "hp": "4500",
  "maxhp": "4800",
  "mp": "1200",
  "maxmp": "2500",
  "ep": "15000",
  "maxep": "16000",
  "wp": "14000",
  "maxwp": "15000",
  "nl": "10",
  "string": "H:4500/4800 M:1200/2500 E:15000/16000 W:14000/15000 NL:10/100",
  "charstats": [
    "Bleed: 0",
    "Kai: 0%",
    "Stance: None"
  ]
}
```

##### Char.StatusVars #####

- Sent by server after a successful login or after the module is enabled
- Contains a list of character variables (level, race, etc)
- Message body is an object
- Each object element is a name-caption pair, name is the internal name and caption the user-visible one
- Example:

```json
{
  "level": "Level",
  "race": "Race",
  "guild": "Guild"
}
```

##### Char.Status #####

- Values of character values defined by StatusVars
- A full list is sent by server right after StatusVars, and changes are sent in subsequent messages as they occur
- With the exception of the initial Status message, messages only contain changed values; if a variable is not included, it has not changed since the previous Status message
- Message body is an object
- Each object element is a name-value pair, name is the internal name defined by the StatusVars message and value is the variable value
- Example:

```json
{
  "level": "58",
  "guild": "Guild"
}
```

### Char.Skills ###

#### Sent by client ####

##### Char.Skills.Get #####

- Sent by client to request skill information
- Message body is an object with keys "group" and "name"
- If both group and name is provided, the server will send Char.Skills.Info for the specified skill
- If group is provided but name is not, the server will send Char.Skills.List for that group
- Otherwise the server will send Char.Skills.Groups
- Example:

```json
{
  "group": "elemancy",
  "name": "firelash"
}
```

#### Sent by server ####

##### Char.Skills.Groups #####

- Groups of skills available to the character
- Sent by server on request or at any time (usually if the list changes)
- For IRE games, groups are skills like `Survival` or `Elemancy`
- Message body is an array of objects, each being one name and the skill rank
- Example:

```json
[
  {
    "name": "Perception",
    "rank": "Transcendent (100%)"
  },
  {
    "name": "Survival",
    "rank": "Inept (0%)"
  },
  {
    "name": "Weaponry",
    "rank": "Inept (0%)"
  },
  {
    "name": "Tattoos",
    "rank": "Inept (0%)"
  },
  {
    "name": "Evasion",
    "rank": "Inept (0%)"
  },
  {
    "name": "Engineering",
    "rank": "Inept (0%)"
  },
  {
    "name": "Taming",
    "rank": "Inept (0%)"
  },
  {
    "name": "Concoctions",
    "rank": "Inept (0%)"
  },
  {
    "name": "Toxins",
    "rank": "Inept (0%)"
  },
  {
    "name": "Smithing",
    "rank": "Inept (0%)"
  },
  {
    "name": "Malignosis",
    "rank": "Adept (1%)"
  },
  {
    "name": "Necromancy",
    "rank": "Inept (0%)"
  },
  {
    "name": "Evileye",
    "rank": "Adept (40%)"
  }
]
```

##### Char.Skills.List #####

- List of skills in a group available to the character
- Sent by server on request only
- For IRE games, this is the list visible on `AB <groupname>`
- Message body is an object with keys `group`, `desc` and `list`, where group is the group name as a string
- The desc value is the description of the skill as seen in `AB <groupname>`
- The list value is an array of strings, each being the name of one skill
- **Note for IRE games** This will return all skills in the group, even not learned ones.
- Example:

```json
{
  "group": "Elemancy",
  "desc": [
    "Cast light",
    "Make your skin hard as stone",
    "Cast a bold of fire"
  ],
  "list": [
    "Light",
    "Stoneskin",
    "Firelash"
  ]
}
```

##### Char.Skills.Info #####

- Information about a single skill, only sent upon request
- Message body is an object, keys are `group`, `skill`, and `info`, values are strings
- Group and skill identify the request, info is a description (usually multi-line) of the skill's functionality and usage
- **Note for IRE games** Unlearned abilities will have `*** You have not yet learned this ability ***` in their info field in addition to the description
- Example:

```json
{
  "group": "Elemancy",
  "skill": "Firelash",
  "info": "blah blah"
}
```

### Char.Items ###

#### Sent by client ####

##### Char.Items.Inv #####

- Request for the server to send the list of items in player's inventory
- Message body is empty
- Causes the server to send back an appropriate Char.Items.List message

##### Char.Items.Contents #####

- Request for the server to send the list of items located inside another item
- Message body is a number identifying the item
- Causes the server to send back an appropriate Char.Items.List message
- Example:

```json
402879
```

##### Char.Items.Room #####

- Request for the server to send the list of items in the current room
- Message body is empty
- Causes the server to send back an appropriate Char.Items.List message

#### Sent by server ####

##### Char.Items.List #####

- List of items at a specified location (room, inv, held container)
- Message body is an object with keys `location` and `items`
- `location` value is a string, `inv`, `room`, or `repNUMBER` - the last one is container identification
- `items` value is an array, whose each item is an object with keys `id`, `name`, `icon` and optionally `attrib`
- `id` is a number identifying the item, `name` is a string containing a short player-visible item descrption
- `attrib` is a string consisting of characters describing item properties:
  - `w` = worn,
  - `W` = wearable but not worn,
  - `l` = wielded (left),
  - `L` = wielded (right)
  - `lL` = wielded (both)
  - `g` = groupable,
  - `c` = container
  - `r` = riftable
  - `f` = fluid
  - `e` = edible
  - `m` = monster
  - `d` = dead monster
  - `t` = takeable
  - `x` = should not be targeted (loyal to city, player....)
- `icon`: categorization of the item, for example which icon type to use
- Example:

```json
{
  "location": "room",
  "items": [
    {
      "id": 54685,
      "name": "an apple"
    },
    {
      "id": 85462,
      "name": "a tiny worm"
    }
  ]
}
```

##### Char.Items.Add #####

- Informs the client about an item being added to the specified location
- Message body is an object with keys "location" and "item"
- Location is same as with List, item is an object with the same structure as one item from the items array of List
- Example:

```json
{
  "location": "room",
  "item": {
    "id": 123988,
    "name": "a cat"
  }
}
```

##### Char.Items.Update #####

- Informs the client about an item's attributes being changed - only sent for inventory items
- Message body is an object with the same structure as Add

##### Char.Items.Remove #####

- Informs the client about an item being removed from the location
- Message body is an object with the same structure as Add

### Char.Defences ###

#### Sent by server ####

##### Char.Defences.List #####

- Sends the list of current active defenses.
- Sent, whenever the client requests the `def` list.
- The body is a list of objects with the following keys
  - `name`: shows the name of the defense
  - `desc`: a short description of the defense
- Example:

```json
[
  {
    "name": "selfishness",
    "desc": "Selfishness prevents you from giving away items."
  },
  {
    "name": "lifevision",
    "desc": "Lifevison allows you to see traces of all living beings, piercing through illusions and hiding defences."
  },
  {
    "name": "tekurastance",
    "desc": "Tekura stances enhance or detract from certain aspects of your attacks."
  },
  {
    "name": "deafness",
    "desc": "Deafness can prevent some harmful effects, at the expense of your hearing."
  },
  {
    "name": "blindness",
    "desc": "Blindness can prevent some harmful effects, at the expense of your sight."
  }
]
```

##### Char.Defences.Add #####

- Shows newly acquired defenses
- The body is a single defense object as in the `Char.Defences.List`.
- Example:

```json
{
  "name": "selfishness",
  "desc": "Selfishness prevents you from giving away items."
}
```

##### Char.Defences.Remove #####

- Shows lost defenses
- The body is a list with the name of the lost defense in it
- Example:

```json
[
  "blindness"
]
```

### Char.Afflictions ###

#### Sent by Server ####

##### Char.Afflictions.List #####

- Sends the list of current afflictions.
- Sent, whenever the client requests the `diag` list.
- The body is a list of objects with the following keys
  - `name`: shows the name of the affliction
  - `cure`: shows the commands to cure this affliction
  - `desc`: a short description of the affliction
- Example:

```json
[
  {
    "name": "insomnia",
    "cure": "EAT GOLDENSEAL",
    "desc": "Insomnia makes it very difficult to fall asleep."
  },
  {
    "name": "deafness",
    "cure": "APPLY EPIDERMAL TO HEAD",
    "desc": "Deafness stops you from hearing anything."
  },
  {
    "name": "blindness",
    "cure": "APPLY EPIDERMAL TO HEAD",
    "desc": "Blindness makes it impossible to see the world around you."
  }
]
```

##### Char.Afflictions.Add #####

- Shows newly acquired afflictions
- The body has the same structure as the List message

##### Char.Afflictions.Remove #####

- Shows lost afflictions
- The body is a list with the name of the lost affliction in it
- Example:

```json
[
  "blindness"
]
```

### Comm.Channel ###

#### Sent by client ####

##### Comm.Channel.Players #####

- Request for the server to send Comm.Channel.Players
- No message body

##### Comm.Channel.Enable #####

- Used to tell the game to turn on a character channel without typing in a command line command.
- Example:

```json
"newbie"
```

#### Sent by server ####

##### Comm.Channel.Players #####

- List of players and organizations (city, guild, ...) that they share with this player
- Message body is an array with each element describing one player
- Each element is an object with keys "name" and "channels", name is a string, channels is an array
- The channels array may be omitted if empty; if given, it is a list of organization names
- Example:

```json
[
  {
    "name": "Player1",
    "channels": [
      "Some city",
      "Some guild"
    ]
  },
  {
    "name": "Player2"
  }
]
```

##### Comm.Channel.List #####

- List of communication channels available to the player, sent on login/negotiation and if changed
- Message body is an array of objects, each object representing one channel
- Each object has keys `name`, `caption` and `command`:
  - `name` is the internal name
  - `caption` is player-visible name
  - `command` is command used to communicate over this channel
- Example:

```json
[
  {
    "name": "ct",
    "caption": "Some city",
    "command":"ct"
  },
  {
    "name": "gt",
    "caption":"Some guild",
    "command":"gt"
  }
]
```

##### Comm.Channel.Start (*deprecated*) #####

- Informs the client that text that follows is something said over a communication channel
- Message body is a text containing the channel name
- For tells from/to another player, the channel name is "tell Name"
- Example:

```json
"ct"
```

##### Comm.Channel.End (*deprecated*) #####

- Ends a channel text started by Comm.Channel.Start
- Message body is a text containing the channel name (same as `Comm.Channel.Start`)

##### Comm.Channel.Text #####

- Complete information about a communication.
- The body is an object with the fields `talker`, `channel` and `text`:
  - `talker` is the person, who uses the channel
  - `channel` is the channel used
  - `text` is the complete text that is sent. This includes possibile ANSI colors and MXP commands
- Example:

```json
{
  "channel": "newbie",
  "talker": "Juliet",
  "text": "\u001b[0;1;32m(Newbie): Juliet says, \"You would simply slay Beku with Your lightning I imagine, Lady Aurora!\"\u001b[0;37m"
}
```

### Room ###

#### Sent by server ####

##### Room.Info #####

- Contains information about the room that the player is in. Some of these may be IRE-specific
- Message body is an object with the following keys
  - `num` - number identifying the room
  - `name` - string containing the brief description
  - `area` - string containing area name
  - `environment` - string containing environment type ("Hills", "Ocean", ...)
  - `coords` - room coordinates (string of numbers separated by commas - area number,X,Y,X,building, building is optional)
  - `map` - map information - URL pointing to a map image, followed by X and Y room (not pixel) coordinates on the map
  - `details` - array holding further information about a room - shop,bank,...
  - `exits` - object containing exits, each key is a direction and each value is the number identifying the target room
- Example:

```json
{
  "num": 12345,
  "name": "On a hill",
  "area": "Barren hills",
  "environment": "Hills",
  "coords": "45,5,4,3",
  "map": "www.imperian.com/itex/maps/clientmap.php?map=45&level=3 5 4",
  "exits": {
    "n": 12344,
    "se": 12336
  },
  "details": [
    "shop",
    "bank"
  ]
}
```

##### Room.WrongDir #####

- Sent if the player attempts to move in a non-existant direction using the standard movement commands
- Upon receiving this message, the client can safely assume that the specified direction does not lead anywhere at this time
- Message body is a string with the name if the non-existant exit
- Example:

```json
"ne"
```

##### Room.Players #####

- A list of objects containing player details
- Each player object contains the keys `name` and `fullname`, which represent the actual name and the name including all titles, respectively.
- Example:

```json
[
  {
    "name": "Tecton",
    "fullname": "Tecton, the Terraformer"
  },
  {
    "name": "Cardan",
    "fullname": "Cardan, the Curious"
  }
]
```

##### Room.AddPlayer #####

- Sent if a player enters the room
- The message body is an object with the same structure of a single player in Room.Players
- Example:

```json
{
  "name": "Cardan",
  "fullname": "Cardan, the Curious"
}
```

##### Room.RemovePlayer #####

- Sent, when a player leaves a room
- Message body is a string with the name of the leaving player.
- Example:

```json
"Cardan"
```

### Redirect ###

#### Sent by server ####

##### Redirect.Window #####

- Specifies a window to redirect further input to
- Message body is a string specifying the window to redirect to
- The main window is referred to as `main`, and is the default if the message body is omitted or empty
- Example:

```json
"map"
```

### IRE.Rift ###

#### Sent by server ####

##### IRE.Rift.List #####

- Contents of a Rift storage
- Sent upon receiving the IRE.Rift.Request message
- Message body is an array, with each element being an object containing three keys:
  - `name` is item name
  - `amount` is a number holding the item's amount
  - `desc` is user-visible description
- Example:

```json
[
  {
    "name": "rawstone",
    "amount": "1",
    "desc": "rawstone"
  },
  {
    "name": "moss",
    "amount": "421",
    "desc": "irid"
  },
  {
    "name": "asilver",
    "amount": "793",
    "desc": "alchemical silver"
  },
  {
    "name": "argentum",
    "amount": "721",
    "desc": "argentum"
  }
]
```

##### IRE.Rift.Change #####

- Sent whenever the item amount in a Rift changes
- Message body is an object with the same structure as one element of the array sent with the IRE.Rift.List message

#### Sent by client ####

##### IRE.Rift.Request #####

- Asks the server to send the Rift contents using the IRE.Rift.List message
- No body

### IRE.Composer ###

#### Sent by server ####

##### IRE.Composer.Edit #####

- Sent by the server when the player enters an in-game editor
- Body is an object, with keys "title" and "text". Text contains the current buffer, title is a title that can be shown to the user.
- Example:

```json
{
  "title": "Journal",
  "Content": "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.\n\nDuis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi. Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat."
}
```

#### Sent by client #####

##### IRE.Composer.SetBuffer #####

- Sent by the client upon successfully editing a text which was sent to the client in an IRE.Composer.Edit message earlier
- Sending this message only changes the edit buffer, but does not end the editing session
- on IRE games, the client may send the command -`*save` to save a text, or command `*quit` to abort editing (IRE.Composer.SetBuffer is not sent in this case) - this behaviour is IRE-specific and is one of the reasons why the Composer module is in the IRE namespace
- Example:

```json
"Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.\n\nDuis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi. Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat."
```

### IRE.Tasks ###

#### Sent by client ####

##### IRE.Tasks.Request #####

- Sent by the client to request the current list of tasks.
- no body.

#### Sent by server ####

##### IRE.Tasks.List #####

- This is used to send a list of quest, tasks, and achievements, depending on what the game supports.
- The body is an array of objects with the following fields:
  - `id`: Unique ID of the task
  - `name`: Name of the task
  - `desc`: Describtion of the task, including details how to solve the task
  - `type`: Type of the task. Known types are `Achievements`, `Quest`and `Task`
  - `cmd`: Command to show the task via "normal" means
  - `status`: : Status of the task. May be an empty string, "1" for completed tasks and "0" for uncompleted
  - `group`: Group the task is organized into. On completion, tasks are moved to the "Completed" group.
- Example:

```json
[
  {
    "id": "1",
    "name": "Break Free of Your Imprisonment",
    "desc": "Where are you? What's going on? There's no time to waste, you need to get out of here!\n\nPay close attention to the directions and tips on your screen and you'll be out of the dungeon in no time.",
    "type": "Task",
    "cmd": "NEWTASK 1 INFO",
    "status": "1",
    "group": "Completed"
  },
  {
    "id": "122",
    "name": "Feed the Birds",
    "desc": "Paloma has asked you to scout around the city for stale bread, so she   \ncan continue feeding the pigeons in Artisan Plaza.                      ",
    "type": "Quest",
    "cmd": "QUEST 122 DETAILS",
    "status": "",
    "group": "Cyrene"
  }
]
```

##### IRE.Tasks.Update #####

- Sent by the server to show that a task was changed.
- The body structure is the same as the one of IRE.Tasks.List

##### IRE.Tasks.Completed #####

- no info yet

### IRE.Time ###

#### Sent by Client ####

##### IRE.Time.Request #####

- Sent by the client to request the current IRE.Time.List
- no body

#### Sent by Server ####

##### IRE.Time.List #####

- Sent by the server to transmit the current time and addtional information about the time.
- The body is am object with the following fields:
  - `time`: The time of the day
  - `moonphase`: phase of the moon
  - `day`: current day of the month
  - `month`: name of the current month
  - `mon`: nummerical representation of the current month
  - `hour`: current hour of the day
  - `year`: current year
  - `daynight`: *time of the day in a numerical representation, resets with the morning* (highly speculative)
- Example:

```json
{
  "day": "5",
  "mon": "9",
  "month": "Phaestian",
  "year": "648",
  "hour": "51",
  "time": "Dusk has overtaken the light in Achaea.",
  "moonphase": "Waxing Gibbous",
  "daynight": "122"
}
```

##### IRE.Time.Update #####

- sent by the server on time update
- body has the same stucture as IRE.Time.List
- only the changed fields are transmitted

### IRE.Misc ###

#### Sent by server ####

##### IRE.Misc.RemindVote #####

- Sent by the server to remind the client to vote
- The body is a string that contains the url to vote
- Example:

```json
"http://www.topmudsites.com/cgi-bin/topmuds/rankem.cgi?id=sarapis"`
```

##### IRE.Misc.Achievement #####

- Sent by the server when an achievement is met
- The body is an object with the name and value of the achievement
- Example:

```json
{
  "name": "AchievedLevel21",
  "value": "1"
}
```

##### IRE.Misc.URL #####

- Sends a url to the client to open in a window when clicked.
- Message body is a list with an object. The object's keys are `url` and `window`.
- `url` is the URL to open when clicked.
- `window` is the window where the content should be shown
- Example:

```json
[
  {
    "url": "http://www.imperian.com/tos",
    "window": "ire_game_tos"
  }
]
```

##### IRE.MISC.Tip #####

- Sends a line of text to the client.
- Example:

```json
"This is a tip!"
```

##### IRE.Misc.OneTimePassword #####

- no info yet

##### IRE.Misc.JavaEnv #####

- no info yet

#### Sent by client ####

##### IRE.Misc.OneTimePassword #####

- no info yet (found in <http://client.achaea.com/includes/js/ui/dialogs.js>)

##### IRE.Misc.Voted #####

- Informs the game a vote button was clicked.
- No body

### IRE.Display ###

#### Sent by server ####

##### IRE.Display.Help #####

- Gating event for displaying help contents.
- Body is a string containing `start` to open the gate and `stop` to close it
- Example:

```json
"stop"
```

##### IRE.Display.FixedFont (*speculative*) #####

- Used to order the client to use a fixed font (e.g. Monospace fonts) to display content
- Body is a string that may contain `start` to start the fixed part and `stop` to stop the part again.
- Example:

```json
"stop"
```

##### IRE.Display.AutoFill #####

- Fill the input of the client with the given text.
- Body is an object with the following fields:
  - `command`: The command to be put into the input line.
  - `highlight`: string or bool (?) marking the line to be selected or not
- Example:

```json
{
  "command": "Help help",
  "highlight": false
}
```

##### IRE.Display.HidePopup #####

- Message to tell the client to remove a popup from display
- Message body is an object with the key `id`, showing the popup to be hidden
- Example:

```json
{
  "id": "intro_step_4"
}
```

##### IRE.Display.HideAllPopups #####

- Hides all popups
- No body

##### IRE.Display.Popup #####

- Used to show a popup to the client
- The body is an object with the following fields:
  - `id`: ID of the popup
  - `text`: The text to display
  - `options`: Options to use with the popup. Known options are display time (`time`, integer) and transparency (`transparent`, boolean)
  - `element`: (HTML-ID of the) parent element of the popup.
  - `src`: An image that should be displayed
  - `cmd`: A command that should be run by the client upon clicking the popup. Known ones are `done`, `continue`, and `back`
- Example:

```json
{
  "id":"intro_step_4",
  "text":"This window displays a map of your surroundings.",
  "options": {
    "time":"20",
    "transparent":true
  },
  "element":"output"
}
```

##### IRE.Display.Ohmap #####

- Gating event for displaying the wilderness (overhead) map.
- Body is a string containing `start` immediately before the output starts or `stop` immidiately after the last line of the map
- Example:

```json
"start"
```

##### IRE.Display.ButtonActions #####

- Available from module version 3 (IRE.Display 3)
- Used by the HTML5 client to change the default buttons.
- The body is an object with a descriptive object for each Button. Property names are Button1, Button2 ...
- Each descriptive object contains:
  - `text`: the text to be shown
  - `commands`: the commands to be sent to the game
  - `highlight`: 0 for no highlight, 1 for highlight
- Example:

```json
{
  "button1": {
    "text":"Combo",
    "commands":"COMBO @tar",
    "highlight":0
  },
  "button2": {
    "text":"Spinningbackfist",
    "commands":"sbp @tar",
    "highlight":1
  },
  "button3": {
    "text":"Scramble",
    "commands":"mind scramble @tar",
    "highlight":0
  },
  "button4": {
    "text":"Splinterkick",
    "commands":"spk @tar",
    "highlight":1
  },
  "button5": {
    "text":"Tornadokick",
    "commands":"tnk @tar",
    "highlight":1
  },
  "button6": {
    "text":"Mindblast",
    "commands":"mind blast @tar",
    "highlight":0
  },
  "button7": {
    "text":"Ripplestrike",
    "commands":"rpst @tar",
    "highlight":0
  }
}
```

##### IRE.Display.Window #####

- Gating event to move output to a different window
- used by the Achaean "window" command to move the command output to a different window
- Body is an object containing:
  - `start`: 1 to start redirecting, 0 to stop redirecting
  - `cmd`: the command the output belongs to
- Example:

```json
{
  "start":"1",
  "cmd":"sit"
}
```

### IRE.FileStore ###

#### Sent by server ####

##### IRE.FileStore.Content #####

- Used to send server stored content to the client. The content is Bas64 encoded.
- For the HTML5 client, this is simple JavaScript, that seems to be run after receiving the message.

#### Sent by client ####

##### IRE.FileStore.Request #####

- Used to access the reflex packages saved on the server. Depending on the request, different actions are taken.
- Body is an object with the keys "request" and "data". Those are filled in the following manner:

| request              | data                                                               | description                                       |
|----------------------|--------------------------------------------------------------------|-----------------------------------------------------------|
| clr                  | null                                                               | Clears the data on the server                             |
| add                  | Base64 encoded data to put on the server. Max length 8192 char (?) | Adds the data to the data already stored on the server.   |
| put *html5-reflexes* | null                                                               | Points the server to the storages space (?) where to previously `put` data should be stored. |

### IRE.Sound ###

#### Sent by server ####

##### IRE.Sound.Preload #####

- no info yet

##### IRE.Sound.Play #####

- no info yet

##### IRE.Sound.Stop #####

- no info yet

##### IRE.Sound.StopAll #####

- no info yet

### IRE.Target ###

#### Sent by client ####

##### IRE.Target.Set #####

- Used to set the server side variable `&tar` and request target information.
- The body is a string containing the ID of the target.
- Example:

```json
"12345"
```

#### Sent by Server ####

##### IRE.Target.Set #####

- Used to notify the client of a change of the `&tar` variable unless the client requested by sending `IRE.Target.Set` itself.
- The body is a string containing the ID of the target or an empty string if unset.
- Example:

```json
"12345"
```

##### IRE.Target.Info #####

- Used to send additional information about the current active server side target (contents of the `&tar`-variable).
- The body is an object with the following fields:
  - `short_desc`: Contains the short description of the target
  - `hpperc`: Contains the current HP of the target as a percentage
  - `id`: The ID of the target.
- Example:

```json
{
  "short_desc": "a practice dummy",
  "hpperc": "100%",
  "id": "266744"
}
```

##### IRE.Target.Request #####

- Unconfirmed as not seen in real life yet
- Used by the server to re-request the target
- No body

### Client ###

It is usually not necessary to enable this module as it is enabled by default. Servers may send different client specific content in these messages to provide a customized experience specific to the client.

#### Sent by server ####

##### Client.Map #####

- Used to send the location of a server provided map to the client.
- The body is an object with the following fields:
  - `url`: Contains the URL where to find and download a server provided map. This might be a map in the MMP or a client specific format.
- Example:

```json
{
  "url": "http://www.achaea.com/maps/map.xml"
}
```

### External.Discord ###

Additional information about this module can be found at <https://wiki.mudlet.org/w/Standards:Discord_GMCP>

#### Sent by server ####

##### External.Discord.Info #####

- Sent in response to a `External.Discord.Hello` message
- The body is an object that is either empty or has the following fields:
  - `inviteurl`: the URL that can be used to join a game specific Discord server.
  - `applicationid`: the application ID to be used by the client when connecting to the Discord Rich Presence service. That way the game will show up on its own and can use custom assets.
- Example:

```json
{
  "inviteurl": "https://discord.gg/kuYvMQ9",
  "applicationid": "165132135465411234567890"
}
```

##### External.Discord.Status #####

- Sent in response to a `External.Discord.Get` message or whenever the status changes
- The body is an object with the following optional fields:
  - `smallimage`: Array of Discord image resource names that should be used as the small Discord image. Ordered by order of preference. Pick one.
  - `smallimagetext`: Text that should appear on mouseover on the small image
  - `largeimage`: Array of Discord image resource names that should be used as the large Discord image. Ordered by order of preference. Pick one.
  - `largeimagetext`: Text that should appear on mouseover on the large image
  - `details`: Second line in the Discord rich presence popup
  - `state`: Third line in the Discord rich presence popup
  - `partysize`: Number of people in the player party
  - `partymax`: Maximum number of people in the player party
  - `game`: Game that is played
  - `starttime`: Time when the current session was started as a unix timestamp (seconds since epoch). Discord will convert this to a counter
  - `endtime`: Time when the current session will end as a unix timestamp (seconds since epoch). Discord will convert this to a countdown
- Example:

```json
{
  "smallimage": [
    "iconname",
    "iconname2",
    "iconname3"
  ],
  "smallimagetext": "Icon hover text",
  "details": "Details String",
  "state": "State String",
  "partysize": 0,
  "partymax": 10,
  "game": "Achaea",
  "starttime": "1563354350",
  "endtime": "1563355350"
}
```

#### Sent by client ####

##### External.Discord.Hello #####

- Sent to announce the player Discord information to the server. The server might use it for bot integration or publishing contact information or other usage.
- The body is an object with the following fields:
  - `user`: Discord unique user name
  - `private`: Boolean to opt out of publishing this data. This should not disable bot usage. Servers MUST comply with this request.
- Example:

```json
{
  "user": "person#1234",
  "private": true
}
```

##### External.Discord.Get #####

- Sent to manually retrieve `External.Discord.Status` from the server
- The body is empty
