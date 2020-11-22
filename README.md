# ROBOT FIGHT SIMULATOR
Robot Fight Simulator is a Python program that proposes a simulation of fights between robots of 2 teams (Red and Blue).
In this README, you will find information on how to use the Simulator, the simulation unfolding and the program structure.

## USE THE SIMULATOR
### GAME MENU
The game Menu has 2 command:
* **New Game**: The game will be reinitialised with an empty board of the size of the option selected (see **Options Menu**)

* **Quit**: Will exit the application

### OPTIONS MENU
Several options are available to configure the simulation as shown in the below screenshot:
<p align="center">
    <img src="./Images/Options.png" alt="Options Menu" height=250 >
</p>

* **Select board size**: Allows the user to select the size of the board for the simulation. 3 sizes are available and the option is taken into account only when `Game -> New Game` is selected.
* **No Friendly Fire**: When selected, Robots from the same team don't inflict damages to their teammates.

* **Smart Fire**: When selected, robots that are on the edge of the board won't face the edge. This means they will use their weapons towards the inside of the board. This avoids robots attacks to be wasted by them shooting outside of the board.
* **Smart Movement**: When selected, robots arriving to the edge of the board will change their movement to avoid being stuck on the same cell the whole simulation. For example, a robot arriving on the upper right corner will change its movement and get a random value between `Down` and `Left` to allow it to move at the next turn.
* **Deact. Robots fire**: When selected, Deactivated robots use their weapons. All of them only have a `Basic Shoot` and at each turn, one randomly chosen deactivated robot will fire in a random direction after the turn of the Red or Blue robot. Deactivated Robot will also beneficiate from the **Smart Fire** if selected.
* **Add Deact Robots every 10 turns**: When selected, additional Deactivated Robots will be added every 10 turns. The quantity of added Robots is randomly chosen between 1 and 5. They will be added only if at least one free cell remains after the addition.


### TEAM SELECTION
When the simulator is launched or `Game -> New Game` is selected, the below screen allows the user to select the quantity and types of robots in each team.
<p align="center">
    <img src="Images/Team.png" alt="Team Selection Screen" height=350 >
</p>

There are **4** Types of Robots are available with different behaviours. However they all beneficiate from the **Smart fire** and **Smart Movement** options when they are activated. The number of robots available for each team is between 1 and the number equivalent to the board size. Any combination of behaviour is possible. Once the composition of each team has been selected, the user needs to click on **Fill Grid** to launch the simulation.
* **Random**: 
    * ***Body***: Randomly equipped
    * ***Weapons***: Randomly equipped
    * ***Pick Up***: Robot will pick up only Bodies and Weapons it doesn't have yet      
* **Aggressive**: 
    * ***Body***: `Battle body` will be equipped if available to beneficiate from the additional weapon. If not available, body will be selected randomly.
    * ***Weapons***: All available `Twin Weapon` will be equipped first to beneficiate from the maximum of weapon slots. Then weapons are equipped in the order they are ranked. `Dual Laser` being the highest and `Basic shot` the lowest.
    * ***Pick Up***: Robot will pick up only Bodies it doesn't have yet. It will pick up any Weapons except Basic shoot to maximise chance of equipping a better weapon.      
* **Defensive**:
    * ***Body***: `Hard body` will be equipped if available to beneficiate from highest HP. If not available, body will be selected randomly.
    * ***Weapons***: `Explosion` will be equipped in priority to attack enemies all around the robot. Then weapons are equipped in the order they are ranked. `Dual Laser` being the highest and `Basic shot` the lowest.
    * ***Pick Up***: Robot will pick up only Bodies and Weapons it doesn't have yet  
* **Survivor**:
    * ***Body***: Will equip the available body with the highest number of HP to maximise chance of survival.
    * ***Weapons***: All available `Twin Weapon` will be equipped first to beneficiate from the maximum of weapon slots. Then weapons are equipped in the order they are ranked. `Dual Laser` being the highest and `Basic shot` the lowest.
    * ***Pick Up***: Robot will pick all the Bodies. It will pick up any Weapons except Basic shoot to maximise chance of equipping a better weapon. 


## SIMULATION UNFOLDING



<p align="center">
    <img src="Images/Turn.png" alt="Simulation Screen" height=350 >
</p>

On this screen, the board is now filled with the robots and separated in 2 parts. The board in the top part will display the behaviour of the robots. A counter for each team is showing how many robots are still active for each team. Two buttons are available to launch the simulation:
* **Next Turn**: This button will launch one turn. The Red Team starts and then it is one after the other. In one turn, only one robot plays, it is randomly selected from the team playing.

* **Simulate the following number of turns**: This button simply automate the management of the simulation. The user can select the number of turns he wants to simulate by using a pre-selected value or by typing one. A pop up asking to confirm will appear to avoid launching a long simulation by mistake. If any team wins in the middle of the selected number of turns, the simulation will stop and display the winning team.


## PROGRAM STRUCTURE
This Program is in Python and uses Tkinter for the interface.
More generally it uses the below libraries:
```
import random
import operator
import tkinter as tk
from tkinter import ttk
from PIL import Image, ImageTk
import time
import numpy as np
```
### CLASSES
#### ROBOTS
The class **ROBOT** takes the below arguments:

```
class Robot:
    def __init__(self,position,direction,wslot,mvt,bodies,weapons,equipped_W,team_nb,team_name,behavior='Random',Destroyed=False):
```
It has several class functions: 
* `move()`: Manages the movement of the robots and its direction. It is called at each turn and the **Smart Movement** and **Smart Fire** options are managed in this function.
* `pick_up()`: Manages whether the robot picks up an object or not when moving on a cell. It depends on the robot behavious type.
* `attack()`: Runs through the equipped weapons to identidy affected cells and damage. It returns a list of tuple `((row,col),damage)` that will be used in the simulation.
* `rob_damage(damage)`: Is called when a robot is on cell affected by an attack. It will deduce the damage from the HP of the robot's equipped body and put the robot status as `Destroyed` if HP<0
* `Choose_W_B()`: It is called at the end of each turn and select the body and weapons that are equipped on the Robot until next turn. This is where the different ***Behaviours*** are managed.

There is also a subclass for the Deactivated Robots:
```
class Deact_robot(Robot):
    def __init__(self,position,direction):
        super().__init__(position,direction,1,[],[deact_body()],[Basic_shoot()],[Basic_shoot()],3,"Grey")
```
Only the `move()` and `Choose_W_B()` methods are different from the parent class.
As the deactivated robots don't move, the `move()` function is only managing the **Smart Fire** option and the `Choose_W_B()` is empty as they are not changing any equipment.


#### BODIES
The class **BODIES** takes the below arguments and no class function:
```
class Bodies:
    def __init__(self,HP,mvt,wslot,name,index):
```
<details><summary><b>Details of BODIES Subclasses</b></summary>

```
class Simple_body(Bodies):
    def __init__(self):
        super().__init__(2,0,0,"Simple Body",10)

class Hard_body(Bodies):
    def __init__(self):
        super().__init__(5,0,0,"Hard Body",11)

class Light_body(Bodies):
    def __init__(self):
        super().__init__(3,1,0,"Light Body",12)

class Battle_body(Bodies):
    def __init__(self):
        super().__init__(2,0,1,"Battle Body",13)

class deact_body(Bodies):
    def __init__(self):
        super().__init__(1,0,0,"Deactivated Robot Body",20)
```
</details>

#### WEAPONS
The class **WEAPONS** takes the below arguments and no class function:

```
class Weapons:
    def __init__(self,index,name,damage):
```
However, the subclasses have an `effect(direction,position)` class function that returns a list of affected cells depending for each attack. It is used in the `attack()` function of the Robots class.

<details><summary><b>Details of WEAPONS Subclasses</b></summary>

```
class Basic_shoot(Weapons):
    def __init__(self):        
        super().__init__(1,"Basic Shot",1)
        
class Laser(Weapons):
    def __init__(self):
        super().__init__(2,"Laser",1)

class Sword(Weapons):
    def __init__(self):
        super().__init__(3,"Sword",2)

class Explosion(Weapons):
    def __init__(self):
        super().__init__(4,"Explosion",1)

class Dual_Laser(Weapons):
    def __init__(self):
        super().__init__(5,"Dual Laser",1)

class Twin_Weapon(Weapons):
    def __init__(self):
        super().__init__(6,"Twin Weapon",0)
```
</details>

### LOGIC
The Logic of the the simulator is managed 5 by functions:
* `initialise_game()`: It creates 2 ndarrays *NxN* where N is the size of the board (board_pos and item_pos). Then it create a list of object robot for each team - 1/Red, 2/Blue, 3/Grey(ie:Deactivated Robot). The quantity of **Robots** for Red and Blue teams comes from the user while the number of Deactivated Robots is fixed at 8. Finally each robot is placed in the ndarray (board_pos), represented by its team number. item_pos is empty at this stage but will contains items when they are dropped by destroyed **Robots**.
* `turn(robot)`: This functions takes a **Robot** as argument and manages its behaviour. It calls `move()` then `attack()` from each robot. Then it checks if **Robots** are affected by the attack(s), if yes it calls `damage()` of the affected **Robots**. The option **Friendly Fire** is also managed by this function.
* `Robot_at(row,col,team)`: Function that checks if a **Robot** of a specific team is on a cell [row,col] and return the matching **Robot** or `None` if there is no **Robot** on this cell.
* `next_T()`: This function is calle by **Next Turn** button or `complete_sim()` and manages the turn count to know which teams plays next. It also manages the options  **Deact. Robots fire** and **Add Deact Robots every 10 turns**. It aslo disable the **Next Turn** and **Simulate the following number of turns** buttons while the simulation is running to avoid issues if user clicks on these buttons. Finally, it updates the number of active robots per team and call `Victory_popup()` if one team is out of **Robots**.
* `complete_sim()`: This function is called by **Simulate the following number of turns** and iterates the number of turns selected by the user. It calls `next_T()` at each iterations until all the turns are performed or one team has won the match.

### INTERFACE

The interface is created in a class with the following class functions:

* `__init__()`: Create the instance of the interface
* `configure_gui()`: Configure the window size
* `create_widgets()`: Create the 3 frames used (Frame_grid,Frame_team,Frame_turn) and all the widgets used in these frame. The empty board is created in this function.
* `Fill_Robot()`: This function iterates on the 2 ndarrays (board_pos, item_pos) to check what is on each cell and create the matching illustration whether it is a robot, a weapon or a body.
* `Fill_board()`: Is triggerd by a click on the **Fill Grid** button. It checks that the number of robots selected for each team is within the acceptable range, hide the Frame_team and show Frame_turn and then launch the  `initialise_game()` function.
* `new_game()`: Check the current selection for the board size and then destroy all frames. It adapts the window size to the selected board size and call `create_widgets()`.
* `quit_game()`: Close the application

At each turn, the cell of the robot which is firing is displayed in *black* while the cells affected by the attack will be in *red*. When a body or a weapon is dropped, a specific icon is displayed with the index of the item.

Information on the status of each robot can be obtained by hovering over the cells. It will display:
* **Red/Blue Robots**: Robot team and index, its HP, its behaviour, the equipped body and weapons as well as the movements.
* **Deactivated Robots**: Just a message *Deactivated Robot*
* **Weapon/Body**: The name of the item

The inteface also displays information in pop ups:
* `wrong_quantity(type,team)`: Called when the user selected a number of robots not within the correct range. It is called with type and team. Type is a boolean False if quantity is 0, True is quantity>size of the board and team is the name of the affected team
* `Complete_popup()`: Pop up triggerd by **Simulate the following number of turns** button. It asks the user to confirm they want to launch the simulation.
* `stalemate()`: Pop up displayed at the end of the simlation of X turns if no team has won.
* `Victory_popup()`: Pop up displayed when all the robots of a team are destroyed and shows the winning team.


### EXTERNAL CODE
To code display of robots attributes by hovering over the cells, I used a code proposed by [squareRoot17 on StackOverflow](https://stackoverflow.com/questions/20399243/display-message-when-hovering-over-something-with-mouse-cursor-in-python
). It is clearly marked in the code via comments.

