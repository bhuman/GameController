# RoboCup SPL and Humanoid League GameController

This is the GameController developed by team B-Human for the RoboCup SPL and Humanoid League.

If there are any questions, please contact seba@informatik.uni-bremen.de .

Follow @BHumanOpenSrc on Twitter to get notifications about recent activity.


## 1. Building from Source

To build it from the source code you may use Apache Ant.
There are some ant targets:

- clean
	cleans up the project folder

- compile
	compiles the code and stores files in /build/classes
	
- jar
	creates a jar package and stores it in /build/jar

	
## 2. Executing the Jar

Double-click GameController.jar or run 

Usage: `java -jar GameController.jar {options}`

    (-h | --help)                   display help
    (-b | --broadcast) <address>    set broadcast ip (default is 255.255.255.255)
    (-l | --league) (spl | hl_kid | hl_teen | hl_adult)
                                    select league (default is spl)
    (-w | --window)                 select window mode (default is fullscreen)


## 3. Usage
### Start Dialog

Select your league. The default can be specified as a command line parameter (see above).

Pick the two teams that are playing. They have to be different teams. If you are practicing alone, use the "Invisibles" as second team.

SPL: You also have to select whether you play a game in the preliminaries or a play-off game. In the preliminaries the clock will continue to run during game stoppages and there will be no penalties shootout in case of a draw.

HL: You also have to select whether you play a normal game or a knock-out game. A knock-out game will continue after a draw with two halves of extra time (if goals were scored before) and then a penalty shoot-out if necessary. 

You can select whether the GameController should run in fullscreen mode or in windowed mode. Note that the fullscreen mode does not work correctly on some Linux desktops, because although they report to Java that they would support this feature, they do not.

You can also select whether teams exchange their colors in the halftime.


### Main Screen

The use of the main screen should be rather obvious in most cases. Therefore, we only focus on the specialties.

When ever you made a mistake, use the undo history at the bottom of the screen to correct it. You cannot correct individual decisions (except for the last one). Instead, you can only roll back to a certain state of the game. Click the oldest decision in the history you want to undo. After that all decisions that would be undone will be marked. Click the decision again to actually undo it together with all decisions that followed. 

To penalize a robot, first press the penalty button, then the robot button. For unpenalizing a robot, just press the robot button. A robot can only be unpenalized, when its penalty time is over or when the game state changes (SPL only). Five seconds before the penalty time is over, the robot's button starts flashing yellow. For regular penalties, it continues to flash until the button is pressed. Only buttons of robots that were requested for pickup stop flashing after five seconds and simply stay yellow until they are pressed, as a reminder that the robot can return as soon as it is ready.

Before unpenalizing a robot, please make sure that it was put back on the field by the assistant referees. For that reason, robots are never unpenalized automatically.

To substitute a robot, press "Substitute" and then the robot that should leave the field. Afterwards, any of the substitutes can be be activated.

When pressing "+" (goal), "Timeout", "Kickoff Goal", or "Global Game Stuck", the other team gets the next kick-off. "Kickoff Goal" and "Global Game Stuck" share the same button.


## 4. Shortcuts

While the GameController is running, you may use the following keys on the keyboard instead of pushing buttons:

    Esc		    - press it twice to close the GameController
    Delete		- toggle test-mode (everything is legal, every button is visible and enabled)
    Backspace	- undo last action

only SPL

    B	- out by blue
    R	- out by red

    P	- pushing
    L	- leaving the field
    F	- fallen robot
    I	- inactive robot
    D	- illegal defender
    O	- ball holding
    H	- playing with hands
    U	- request for pickup
    C   - coach motion
    T   - teammate pushing
    S	- substitute

only Humanoid-League

    C	- out by cyan
    M	- out by magenta

    B	- ball manipulation
    P	- physical contact
    A	- illegal attack
    D	- illegal defense
    R	- service / incapable
    S	- substitute


## 5. libgamectrl (SPL)

libgamectrl automatically provides the GameController packets in ALMemory. 
It also implements the return channel of the GameController. It handles the 
buttons and LEDs according to the rules (with a few additions).


### Installation

Put the file libgamectrl.so somewhere on your NAO and add the library to your 
file "autoload.ini" so that NAOqi can find it. 

It is also possible to build the library from source using Aldebaran's qibuild
framework. The qiproject.xml and CMakeList.txt have been placed in 
libgamectrl's source folder. Just follow the instructions of the README file there.


### Usage

In your NAOqi module, execute the following code at the beginning (only once):

    AL::ALMemoryProxy *memory = new AL::ALMemoryProxy(pBroker);
    memory->insertData("GameCtrl/teamNumber", <your team number>);
    memory->insertData("GameCtrl/teamColour", <your default team color>);
    memory->insertData("GameCtrl/playerNumber", <your robot's player number>);

The team number must be non-zero. Setting the team number will reset 
libgamectrl (i.e. go back to the initial state). libgamectrl will also set 
"GameCtrl/teamNumber" back to zero, so it will recognize the next time your 
application is started.

You can receive the current GameController packet with:

    RoboCupGameControlData gameCtrlData; // should probably zero it the first time it is used
    AL::ALValue value = memory->getData("GameCtrl/RoboCupGameControlData");
    if(value.isBinary() && value.getSize() == sizeof(RoboCupGameControlData))
        memcpy(&gameCtrlData, value, sizeof(RoboCupGameControlData));


### Deviations from the Rules

The first time the chest button is pressed it is ignored, because many teams 
will use it to let the robot get up.

In the Initial state, it is also possible to switch between "normal", 
"penalty taker" (green LED), and "penalty goalkeeper" (yellow LED) by pressing 
the right foot bumper. The state is shown by the right foot LED, and only in 
the Initial state. An active GameController will overwrite these settings.


## 6. Misc

The format of the packets the GameController broadcasts and receives is 
defined in the file RoboCupGameControlData.h. It differs from the version
used in 2013 in several ways:

- Each TeamInfo now contains information about the coach and its current message
  (coach, coachMessage).
  
- The field goalColour was removed, since all goals are yellow.

- Information for the GameStateVisualizer is part of the packet now (penaltyShot,
  singleShots, secondaryTime).
  
- A counter that is increased for each packet sent (packetNumber) was added. It 
  allows determining whether a new packet arrived even when using libgamectrl.
  
- The PENALTY_SUBSTITUTE is now also used in the SPL for a robot that is a 
  substitute. Player number 6 is penalized this way right from the beginning,
  waiting for being substituted for another player.

- Because of the substitute, playersPerTeam is 6 now in the SPL.

- The new PENALTY_SPL_COACH_MOTION was added.
  
- The custom types uint8, uint16, and uint32 were replaced by the official standard
  types uint8_t, uint16_t, and uint32_t defined in <stdint.h> (or <cstdint>). Please
  note that including RoboCupGameControlData.h inside a namespace now has strange
  effects. They can avoided by including <stdint.h> before opening the namespace.
  
- Many fields use smaller data types now.


## 7. Known Issues

There are still a number of issues left:

- Substitution zeroes the penalty timer when a penalized robot is substituted.
  Substituting an unpenalized robot does not result in any penalty. Please note
  that the substitution procedure will change: first the player that will be
  removed from play has to be selected, then the substitute that will replace it.
  
- The goal keeper can still be substituted.

- SPL: The game time cannot be increased when the referee decides that time was lost.

- The LogAnalyzer does not know the new penalties yet.

- All teams can be selected for the drop-in player competition.

- When running on the same PC, the GameStateVisualizer sometimes does not
  receive the GameController packets anymore. This error is hard to reproduce,
  but it happened quite often in Eindhoven.
  
- The qibuild file for libgamectrl is untested.

- The alignment of button labels is bad if the buttons are small.
