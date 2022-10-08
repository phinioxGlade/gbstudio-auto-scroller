# GB Studio Tutorial - Auto Scroller
![](/assets/backgrounds/drill-chase.png "The background")

A simple auto scroller example using GB Studio 3.1 which features a screen size moving boss using the window/overlay layer.

### GB Studio Tutorials:

- [1.0 - Animated Tiles](https://github.com/phinioxGlade/gbstudio-3-animated-tile-tutorial)
- [2.0 - Background Sprites](https://github.com/phinioxGlade/gbstudio-background-sprites-tutorial)
- [3.0 - Auto Scroller](https://github.com/phinioxGlade/gbstudio-auto-scroller) &lt; This Tutorial

### Key Concepts:

- Moving the camera independently of the player
- Detecting player position relative to camera to create dynamic 1D collision boundary 
- Utilizing the window/overlay layer and invisible sprites to create a screen sized moving boss

### How-to auto scroll the screen
This is very simply, you need to decouple the camera from the player and move the screen using the inbuilt events

1. Scene "On Init" add the "Lock Camera to Player" event with both axis' unchecked. The camera will no longer follow the player.
2. Scene "On Init" set the initial camera position.
3. Add a sprite, pin it to the screen and hide it. This sprite will be used as the updater of the camera position.
4. Sprite "On Update" add "Move Camera To" event and set it to the desired end position.

When you run the game, the camera is initlized to a specific position and then scroll to the target position. GBS does not prevent the player from leaving the screen, its up to you to handle this yourself.

### Dynamic collision boundaries
Currently I'm not aware of any built in features which will prevent a player from leaving the screen and unforunately this is not a particularly easy task to get working without issues. So to acheive this we must compare the camera's position to that of the player.

The "Store Actor Actor Position in Varaibles" event can be used to get the player's position in pixels but how do we get the current camera position? They're assessible via the GBVM variables "_camera_x" and "_camera_y".

#### GBVM Script
<pre>
; get the camera position and store them in global variable
; CX and CY are global variables, to reference them in GBVM  
; you need to prepend VAR_ to there name in uppercase and 
; replace spaces with the underscore '_' character
VM_GET_INT16 VAR_CX, _camera_x
VM_GET_INT16 VAR_CY, _camera_y
</pre>

#### Calculating the screen boundary
Now you have the camera's position calculate the screen boundary:

* $CL = $CX - 80 = left edge in pixels
* $CR = $CX + 80 = right edge in pixels
* $CT = $CY - 72 = top edge in pixel
* $CB = $CY + 72 = bottom edge in pixels

Adjusting the amount you add or subtract from $CX or $CY to vary the play zone.

#### Calculating the player collision boundary box
Since your manually handling  collision detection, you'll need to calculate the player's collision boundary box at it current location and this will need to be done very frame. Luckily this is done all using build in events, no GBVM code required but it might be more performant.

For this example we will presume the sprite has a width and height of 32 pixels but adjust as needed for your boundary box.

1. Add "Store Actor Actor Position in Varaibles" event to store the player {x, y} in $px and $py
2. Add "Math Expression" events to calculate the collision boundary and store them:
  * $pl = $px
  * $pr = $px + 32
  * $pt = $py
  * $pb = $py + 32

If the player's collision boundary box changes for different frames, you'll need to cater that as well.  

#### Checking player screen-edge collisions
This is done using the "If Math Expression" event.

* Player collided with left screen edge: $pl <= $cl
* Player collided with right screen edge: $pr >= $cr
* Player collided with top screen edge: $pt <= $ct
* Player collided with bottom screen edge: $pb >= $cb

#### Preventing the player from leaving screen
Next we need to apply those calculations using a second sprite's On Update. The camera movenment and colision check can't use the same sprite, you might be able to but i've found that you need to seperate them.

1. Add a second pinned sprite to the screen
2. Sprite "On Update" add the following:
  *   GBVM to read the camera position
  *   Calculate the screen edge(s)
  *   Store the player's position in variables
  *   Calculate the player's collision boundary box
  *   Use "If Math Expression" event(s) to check if the player's collision boundary is outside the screen boundary
  *   If outside then set the player's position in the opposite direct by say 2 to 4 pixels 

You should only apply this to dynamic edges you want to stop the player moving beyond. More calculation you do, the slower your game runs. In the example game code I'm only checking against left edge and the right edge is handled by a moving sprite which kills the player "On Hit".

### Currently the rest of the tutorial is only available within the source code
