# Report for assignment 4
## Project

Name: Evennia

URL: https://github.com/evennia/evennia

Evennia is a Multi User Domain (MUD)-building system. Evennia provides a foundation for networking, behind-the-scenes administration and database usage.

## Complexity
### Lizard output for top most complex functions
![Components](Top_Lizard_output.pdf)
1. What are your results for the ten most complex functions?
   * Did all tools/methods get the same result?
   We did not receive the exact same results when calculating CCN by hand and using Lizard. We always got a lower CCN by hand.
   * Are the results clear?
   The results from Lizard are clear and easy to read.
2. Are the functions just complex, or also long?
   There seems to be a combination of the two. The methods we picked were very long, but also contained an abundance of conditionals.
3. What is the purpose of the functions?
See ***Documentation of High Complexity Methods*** below.
4. Are exceptions taken into account in the given measurements?
We did not take exceptions into account when doing our manual calculation and in our coverage tool.
5. Is the documentation clear w.r.t. all the possible outcomes?
See ***Documentation of High Complexity Methods*** below.

## Coverage

### Tools

We focused on building our own coverage tool, but utilized https://coveralls.io/github/evennia/evennia when learning about the coverage of Evennia.

### DYI

Show a patch that show the instrumented code in main (or the unit
test setup), and the ten methods where branch coverage is measured.

The patch is probably too long to be copied here, so please add
the git command that is used to obtain the patch instead:

git diff ...

What kinds of constructs does your tool support, and how accurate is
its output?

### Evaluation

Report of old coverage: [link]

Report of new coverage: [link]

Test cases added:

git diff ...

## Effort spent

For each team member, how much time was spent in

1. plenary discussions/meetings;

2. discussions within parts of the group;

3. reading documentation;

4. configuration;

5. analyzing code/output;

6. writing documentation;

7. writing code;

8. running code?

## Overall experience

The main take away was the importance of tests and good documentation for an open source project. This time all of us struggled a lot to even be able to run the tests and the program itself on our computers. This meant that everything ended up taking at least twice the amount of time we expected. In summary, we learned that if you want to create a useful open source project it has to be easy to run.

## Documentation of High Complexity Methods

#### 1. func@278-330@./evennia/commands/default/comms.py
##### Description of method
Evennia supports a “MUX-like” communication system for communication among different Accounts, Objects (predominantly characters) or Channels. Channels are used for distributing messages to all channel listeners. Channels, also, regulate who can listen/send to the channel by using locks. In summary, the function “func” in the Class CmdChannels, defines a command for listing all channels available to the user, whether they listen to any or not. 

##### Branches
* Get all available channels; if no channels are available the method returns (exits) and prints a “no available channels” message.
* Get all subscribed-to channels, if there are any, and display them with no extra info.
* Else, display a full listing of channels that the user is able to listen to.

##### Why the method has high complexity
Displaying the subscribed to or all available channels requires nested iterations for the fetching of data from the database. This, in turn, also adds additional checks.

##### Suggestions for refactoring
The nested iteration might be hard to abstract into a separate methods; however, the method also contains formatting for a ASCII table output. It adds rows to the output table and fixes the columns, which is hard coded. This section could be moved out into a method, which could take the table to be created and the data to be added to it.

##### Documentation
This method has fairly good documentation and partly the branch outcomes. More detailing would probably just clutter the description, so we would consider the documentation sufficient.


#### 2. 70 func@2443-2512@./evennia/commands/default/building.py
##### Description of method
Evennia enables adding Scripts to Objects (such as Characters) through the in-game command interface. This enables the users of Evennia to define their own desired behavior for an Object. For clarity purposes, the documentation defines the correct usage of the command as: “@script[/switch] <obj> [= script_path or <scriptkey>]”. In summary, the function “func” in the Class CmdScript, attaches a script to an object. 

##### Branches
* Check if the input command follows the defined usage for adding a script, if it does not adhere to defined usage, print how to use the add script command and exit
* Check if the input command has anything to the left hand side of ‘=’ as defined by the usage, if not, print message for the correct format of creating a global script and exit
* Check that the input command has an Object on the left hand side of ‘=’, if not, return and exit.
* If there is no input to right hand side of ‘=’, operate on all scripts.
* If there there is input to the right hand side of ’=’, add the new script and start it

##### Why the method has high complexity
The check for the right hand side of ‘=’ contains control flow, which also has several checks and iterations. These do not only handle the correct behavior, but also error behavior and finalizing the correct output and actually adding the script to the object.

##### Suggestions for refactoring
One suggestion for refactoring is to abstract away the control flow for handling the contents of the right hand side. As it stands, one can say the command does primarily three things; check for correct formatting of the right and left hand side. Try to add the script depending on what it is and start or even stop it. It’s reasonable to keep the checks for formatting in the method, but the defined control flow for adding the script and then starting/stopping can be moved out to separate methods. To minimize the passing of data, the new methods could be defined in the same class and made static.

##### Documentation
This method has excellent documentation, especially coupled with its superclass “muxcommand”. Despite being very long and having multiple possible branches, it’s fairly evident what each branch’s purpose is.


#### 3. parse@223-318@./evennia/utils/eveditor.py
##### Description of method
This method is in charge of pre-parsing for EvEditor (Evennia Line Editor). EvEditor, in turn, is Evennia’s own line editor, which is used for editing texts in-game. The method mimics the commands li, w and txt. From the now parsed input, the method then stores the information gathered, such as number of lines (nlines), the arguments given (arglist) and line range (lstart and lend).

##### Branches
* The editor is set (ndb._eveditor) and validated (if-statement).
* Iterate through arguments (for-loop) and check format of each (if-statement). 
* Check for optional argument line range (if-statement); that is, the first argument should have the look <lstart>:<lend>. 
* Check if lstart is digit and save/format values (nested if- and elsif-statements). Do the same for lend (if no lend available, simply set lend = lstart + 1).
* Check if arglist is empty or not (if-statement) and set argument variables to different values in arglist. 

##### Why the method has high complexity
The reason is that parsing of a text requires a lot of checks on the text to be done, which in turn means a lot of “ifs”. In addition to this, iterating through all the lines of the parse input means that we have for loops. Also, checks are done on the arguments, which means even further conditionals. 

##### Suggestions for refactoring
The complexity could be reduced if the checks were done in helper methods, which might also increase the readability. For example, you could set the editor in one method, iterate through the arguments in one method and, finally, check for and set lstart and lend in one method.

##### Documentation
All in all, the documentation is sufficient with comments and relatively clear code. You have to follow the branches yourself and deduce what happens when a branch is taken.


#### 4. at_first_save@924-983@./evennia/objects/objects.py
#### Description of method
The first time a new instance of the class DefaultObject, which is “a typeclass object representing all entities that actually have a presence in the game”, is saved this method is called. The method starts the startup hooks required for the different parts of the game. 

##### Branches
* The method, first, checks if the object has the attribute _createdict (which it will if it was created by the utils.create function) (if-statement).
* Then the method iterates through all kwargs (dictionary for arguments) of interest to see whether they are set, what their values are and if they should be given another value (which make up a long section of nested if-, elif- and for-statements). These checks are so similar that they can be seen as a group of conditionals for validating kwargs. In general, null-pointers (no value set) and/or values that differ from cdict can cause branching. 

##### Why the method has high complexity
The reason why the method has high complexity is the long list of checks for the kwargs. In a way this is hard to not have this complexity since you have to go through the long list of kwargs to check that they are correct or set them to the correct value. 

##### Suggestions for refactoring
One solution could of course be to simply put the checks in its own methods, but then you would have a lot of methods instead.

##### Documentation
All in all, the documentation is sufficient with comments and relatively clear code. You have to follow the branches yourself and deduce what happens when a branch is taken.


#### 5. to_pickle.process_item@493-519@./evennia/utils/dbserialize.py
##### Definition of method
The process_item method is a helper function for the method to_pickle. The purpose of the function is, essentially, to take in a data parameter containing objects with nested structures and return data from previous data processing and to "pickle" it, meaning preparing them for database entry. 

##### Branches
* We need to identify the type of data that should be handled and this is represented in the code by a bunch of elif's checking the type of the object. If the structure matches the condition (i.e a tuple, list, dict...) the if statement evaluates to true and data is return on its correct form. 

##### Why the method has high complexity
The complexity in this function is fairly high due to the number of type checks that have to be made. Python does not support something like switch cases which could be suitable in this situation, but in all honesty, the if/elif's do not add much visual complexity and it makes the code more readable. 

##### Suggestions for refactoring
Adding help methods could reduce the complexity, but it would just make it harder to read. Due to the nature of the function, it is hard to refactor it without doing a larger refactoring outside of the current scope. 

##### Documentation
The function is poorly documented and the branches are not explained thoroughly. 


#### 6. from_pickle.process_tree@574-615@./evennia/utils/dbserialize.py
##### Description of method
The process_tree function is a helper method for from_pickle. The purpose of the function is, essentially, to take in a data parameter containing so called "pickled" data, which is data that already is contained in the database, and to return it as objects. 

##### Branches
* This is done by comparing the attribute dtype using if statements, and then returning the object as that type. In this case, we are checking for tuples, dicts, sets, OrderedDicts and deques.

##### Why the method has high complexity
The complexity in this function is fairly high due to the number of type checks that have to be made. Python does not support something like switch cases which could be suitable in this situation, but in all honesty, the if/elif's do not add much visual complexity and it makes the code more readable. 

##### Suggestions for refactoring
Adding help methods could reduce the complexity, but it would just make it harder to read. Due to the nature of the function, it is hard to refactor it without doing a larger refactoring outside of the current scope. 

##### Documentation
The function is poorly documented and the branches are not explained thoroughly. 


#### 7. func@158-251@./evennia/contrib/dice.py
##### Description of method
Evennia supports RPG, “d-style”, dice rolling, for roleplaying, in-game gambling or GM:ing. It defines a command for rolling a dice, the documentation defines the usage as follows: “dice[/switch] <nr>d<sides> [modifier] [success condition]”. The method which implements the behaviour is in the class “CmdDice” in the function “func”.

##### Branches
The method primarily handles parsing and as such the branches will be summarized.
* Check if the input command follows the defined usage, else print the defined usage and exit
* Multiple branches for handling length intervals for various dices, i.e d6 dice, d13 dice etc.
* Exception for not supplying valid integers modifiers or operators
* Branches for output formatting depending on the amount of rolls, modifiers etc

##### Why the method has high complexity
As it is a parsing method, it inherently will have several branches. In addition to this it has to handle maximum number of dice and their sides and then different outcomes for incorrect and correct usage. The logic for the dice isn’t the primary cause of the complexity, rather its the output formatting which has multiple branches. 

##### Suggestions for refactoring
As with other commands, it seems that a strategy could be to create methods for formatting the output in the same class. These methods would be passed data for the dice (including modifiers) and the outcomes. 

##### Documentation
The method had clear and concise documentation, which also contained the outcomes of the many branches. 


#### 8. list_callbacks@177-272@./evennia/contrib/ingame_python/commands.py
##### Description of method
Evennia offers a prepared system for in-game Python. This system enables for dynamic addition of features to individual objects. The system is centered around events which trigger callbacks. Callbacks contain arbitrary, behavior-specifying, code that is executed for an individual object. The method uses an object and asked-for callbacks (and related parameters) and collects and displays connected callbacks.

##### Branches
* Check if the name of the callback can be found in the object, if no callback name is found, display a table containing all callback’s name number and description.
* If no callbacks match the one specified by the user, print message and exit
* If there are callbacks, check that the parameter points to an existing callback, if it does assert the number, if the number cannot be asserted, throw an exception
* If no parameter has been specified, display the table of callbacks.

##### Why the method has high complexity
As with other commands, the method parses a lot of input and deals with many possible outcomes depending on the supplied input to the called command. This entails multiple checks for each outcome as well as some iteration and exception handling.

##### Suggestions for refactoring
A general strategy would be to perhaps create methods for the control flows which deal with the object having the specified callback name versus no name being supplied. Additionally, code for formatting and creating an output table is contained within the method. This would be the primary candidate for being moved to a separate method. The passing of data from within the control flow could then pose a challenge. It would not only contribute with reducing the amount of branches but, also shortening down the code

##### Documentation
To understand this method one must first read a separate README for the in-game python system. The README specifies how calls are handled and what parameters are available, among many other things. The branch coverage is perhaps a bit lacking since the each outcome contains many other checks which are not explicitly documented and can be a bit hard to understand from the context.


#### 9. 78 shutdown@350-427@./evennia/server/server.py
#### Description of method
An Evennia session is an established connection to the server. Evennia contains accounts, which are connected to the portal while the Multi User Domain is run on the server. The server and portal are, in turn, connected via an Asynchronous Messaging Protocol. The purpose of the method is, simply, to shut down the server.

##### Branches
The shutdown has several distinct modes which appear as branches in the code:
* Reload: The server restarts, the game is briefly paused, and no persistent scripts are stopped. No connections are lost since the connection to Portal is still active. “at_reload” hooks called on all objects so temporary properties can be saved. 
* Reset: The server restarts and behaves as it was fully shut down. No sessions are disconnected but all non-persistent scripts are stopped. “at_shutdown” hooks called. Resetting can be useful for cleaning unsafe scripts.
* Shutdown: The server shuts down (and is not restarted), sessions are disconnected. Systems are saved and turned off.

##### Why the method has high complexity
Each optional mode of shutdown needs to save the state of the server (every object and account) and pause all scripts. This amounts to a high complexity.

##### Documentation
The documentation in the code itself along with the “Start Stop Reload” section of the Evennia documentation was sufficient to understand the method and the difference of branches.


#### 10. 84 decode_msdp@273-356@./evennia/server/portal/telnet_oob.py
##### Description of method
Multicast Source Discovery Protocol (MSDP) is a (multicast) routing protocol. MSDP uses TCP as its transport protocol. The purpose of the method is to decode MSDP data. The MSDP data should be sent in one of the following forms:
        	cmdname ''      	-> [cmdname, [], {}]
        	cmdname val     	-> [cmdname, [val], {}]
        	cmdname array   	-> [cmdname, [array], {}]
        	cmdname table   	-> [cmdname, [], {table}]
        	cmdname array cmdname table -> [cmdname, [array], {table}]


##### Branches
The function has branches to deal with decoding the possible data structures:
* Decode tables.
* Decode arrays.
* Decode remainders.
* Merge matching table/array/variables.

##### Why the method has high complexity
The high complexity comes from the need to regex the MSDP string, find all data and place it subsequently into appropriate data-structures.

##### Documentation
All in all Portal needs better documentation in order for it to be easy to follow the branches and understand the code.
