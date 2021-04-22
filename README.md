# Castle-of-Bedrock-Text-Game
C# Text-based game

This project is a game called Castle of Bedrock. First, you choose your character class, then can choose your name and/or exit character select with exit. After exiting you arrive in a central room. Then, you can move around the castle with go “direction”, pickup items, drop items, equip and unequip equippable items, open, close, and unlock doors, start battles and attack in them, level up, check your inventory, go back a room until you arrive at your starting point, heal your health to full, repair a specific item to full, say “word” to say something (outputs the word), check stats, and quit the game.

     To start, I’ll explain the biggest parts of the system. Game handles the user input. A bool finished is always false unless the player types ‘quit’, and it becomes true, exiting the game. The loop shows the player name and then reads in a parsed command from Console.ReadLine(). The parser splits the input by ‘ ‘ and command of type Command takes in these words as the base word (first), secondWord, or thirdWord. Each command in the game uses this command system, and has a base word, like go command’s base word is go, and “west” would be a secondWord. The parser also pushes and pops commands from the CommandWords, a stack that contains all of the available commands. The parser also has different states, using IParserStates, that changes what is pushed onto the commandArray with an Enter(Parser parser) method, Exit(Parser parser) that pops current commands off, and State, which exits current state, changes the state to another value, then enters that state. This is used so that for example, the player says start battle, the system goes into Battle State, and the player can only attack, exit, or see their stats. Also in parser, there is a PlayerWillEnterState notification that happens when the player says exit in either the lobby or they lose or win a battle, and it changes the state to a normal state in case of a battle, but moves the character from the lobby to the centralRoom in case of the player saying exit in the lobby. In the player exit command, 
     Dictionary<string, object> userInfo = new Dictionary<string, object>();
                userInfo["state"] = new ParserNormalState();
     NotificationCenter.Instance.postNotification(new Notification("PlayerWillEnterState", this, userInfo));
                this.outputMessage("\n" + this._currentRoom.description());
                
the code above allows this, by first setting the state to a normal state and then sending that info with the Notification to the PlayerWillEnterState in the Parser, as well as showing the output for the player about the room description.

     In the GameWorld, that is where every room and almost every item are set up. A room is set up to hold several different things, such as items and a monster. It shows exits and contains a backroom stack in order to travel back a room as well. Several different items are defined in the GameWorld. An item uses the interface IItem, which gives an item many different stats, from attack to speed, and a name, type, weight, sell value, buy value, and durability, with a ToString() available to print out details on the item. Aside items, each door is defined in the GameWorld. A door uses the MakeDoor function in the Door class, which takes two rooms and sets their exits to each other, but in opposite directions, so one.setExit(“west”, door) and two.setExit(“east”, door), with one and two being 2 different rooms that were already created previously in GameWorld. Door also has an enumerator OCState with Open, Closed, and Locked. An interface is also used called IOCState that contains State, open(), close(), locked(), and unlock(). In my system, if a door is created with an item parameter that has a name = “”, the state of the door is open, so it can only be closed and opened back up, according to the OpenState. Unlock would also return the open state since it’s already unlocked. If the name is not “”, the door would be in Locked State. In that State, the door only opens if the player has the key (same item the door has), which the player gets from defeating the enemy in the locked door room, and when the player says unlock “locked door direction”. Some rooms are different from others, and are assigned in the GameWorld using a delegate system, which was created using an IRoomDelegate interface in Room that uses a Container, getExit, and getExits. Each extra type of Room has a Container that gets the type of room it is, and these rooms function differently, such as the trap room, which prevents the user from leaving (trying will say you are trapped) until you say a magic word, setting the container delegate back to null instead of a trap room.
     
     After combat items are created, they are dropped into rooms using the (RoomName).drop(item), which uses the put command in itemContainer, which works by adding the item to a container, in this case, the room. The dropMonster command in the Room works differently, as I set a Monster monster in each room, and I pass the information from GameWorld into that monster in the room, such as attack, defense, item, etc. Getting all the boss keys by defeating them, and going back to the centralRoom is how you trigger the path to the final boss. I put an addObserver notification in GameWorld tied to PlayerDidUnlockFinal and PlayerCanFinalBoss. When the player uses go or back, I put a notification to PlayerDidUnlockFinal, which goes to PlayerCanFinalBoss in the GameWorld. PlayerCanFinalBoss checks the current room of the player, and if the player is in the _trigger room (centralRoom) and player.hasKeys() == true (has all 12 boss keys), a message will pop up saying the final boss room is available and to “go up” to fight it, and a new Door is made to the final Room from centralRoom. After beating the boss you get a congratulatory message and are told you can continue playing or type ‘quit’ to leave the game.
     
Commands:

Attack – can “attack Physical” or “attack Magic”, it attacks based on your characters attack/magic attack stats and damage is reduced by enemy defense/magic defense. Speed determines who attacks first. For enemies, they randomly roll between 0-1 whether they attack with physical or magic against you. Whether you win or lose, your equipped items wear by 0.5 durability. If you win, you pickup the monsters key and gain exp, and if you level up you go back to max health and gain stats and the monster gains full health so you can fight it again at full health. If you beat a monster with the final boss name, it gives you a very powerful item only once, and you are given the win text. If you lose, your health goes back to 1 and you get nothing. The exit() command is used to exit combat if you win or lose, or you can type ‘exit’ to leave early.

Back – explained earlier but as you go into a room, it saves the room in a stack, so to get back to the starting point, back keeps popping, which reverses the way you came back to the start. addBackRoom in player is what pushes the rooms onto the stack, which happens each time you move to another room.

CharacterSelectCommand – this command lets you choose your character with select “class name”. The stats of characters are defined in player and the players stats are modified accordingly when you choose what you want. If your stats are all 0, you’ll get a warning message until you choose one.

Close Command – closes a door by changing its state from OpenState to ClosedState
Drop Command – drops an item into a room by first taking it from the player inventory (remove function in itemContainer) and putting it in the room (put function in itemContainer).

EquipCommand – Moves the equipment from the inventory itemContainer to the equipment itemContainer in player, and adds the stats to the character. Also changes the weight lost from the inventory to be applied to the weight of the equipment. It works by first removing the item from the inventory (remove in itemContainer)  and putting it in the equipment (put in itemContainer)

ExitCommand – exits different states and puts the state back to NormalParserState

Go Command – moves the player to the direction after go, if there is a path that way and it is not closed/locked/other issues

Heal Command – restores the players health back to maximum

Help Command – tells the user what commands are currently available and what each command does

Inventory Command – shows the player’s current inventory, inventory weight, equipment, and equipment weight

Name Command – gives the player a name. It shows up before the input line that the player types in

Open Command – opens doors if they are closed. Does not work on a locked door

Pickup Command -  picks up an item from a room using pickup in player, which uses pickup in room, a function that first takes the item from the room (remove in itemContainer) and puts it into the player inventory (put in itemContainer) if the weight of the added item is not over 100, or does not make the weight of the inventory plus the weight of equipment over 100.

Quit Command – exits the game completely

Repair Command – repairs equipment named to max durability, if you have that item in your inventory

Say Command – Outputs the word the user enters after say. In an echo room, the word is repeated several times

Start Command – used to start different things, such as battle, trading, etc. For start battle, it changes the player’s commands to only be attack, exit, and stats (check stats) by chaning from NormalParserState to BattleParserState, a different set of commands on the Stack.

Stats Command – Shows the player’s current stats, including experience

Unequip Command – it works exactly like the equip except it removes from the equipment and then puts into inventory, shifting weight of the containers accordingly. Stats are also decremented properly. If durability of an item is 0 or lower, it is automatically unequipped and cannot be equipped until repaired over 0.

Unlock Command – it works similarly to open, except it works on doors with a LockedState, changing them to an OpenState
Player Specifics not mentioned:

getMonster() – it gets the monster from the current room and puts it into the Monster monster created in player. It displays the monster you are fighting, their health, and the key (reward) they hold. Sets fullMonsterHealth to the monsters current health (max before fight) so it can be called after the fight to reset the monster’s health back to full.

outputMessage – normal messages displayed with Console.WriteLine(message)

coloredMessage – used to change the color of the message to a chosen color, output it, then return the message output back to the original grey. debugMessage, warningMesage, and informationMessage all use this, but with different colors (warningMessage has a Red color).
