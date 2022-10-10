# GB Studio Tutorial - Auto Scroller with Screen Sized Boss
![](/assets/backgrounds/drill-chase.png "The background")

A simple auto scroller example using GB Studio 3.1 which features a screen size moving boss using the window/overlay layer.

### GB Studio Tutorials:

- [1.0 - Animated Tiles](https://github.com/phinioxGlade/gbstudio-3-animated-tile-tutorial)
- [2.0 - Background Sprites](https://github.com/phinioxGlade/gbstudio-background-sprites-tutorial)
- [3.0 - Auto Scroller](https://github.com/phinioxGlade/gbstudio-auto-scroller) &lt; This Tutorial

### Play in your Browser or on real hardware
You can play the interactive version or download the rom at https://phinioxglade.itch.io/gb-studio-3-auto-scroller

### Key Concepts:

- Moving the camera independently of the player
- Detecting player position relative to camera to create dynamic 1D collision boundary 
- Utilizing the window/overlay layer and invisible sprites to create a screen sized moving boss

The tutorial is broken to two parts

1. [Part 1 - Auto Scroller](#part-1---auto-scroller)
    1.  [How-to auto scroll the screen](#how-to-auto-scroll-the-screen)
    2.  [Dynamic Collision Boundaries](#dynamic-collision-boundaries)
3. [Part 2 - Screen Sized Boss](#part-2---screen-sized-boss)
    1.  [How-to create a moveable screen sized boss](#how-to-create-a-moveable-screen-sized-boss)
    2.  [Filling the overlay window layer](#filling-the-overlay-window-layer)
    3.  [Rendering sprites on top of the overlay](#rendering-sprites-on-top-of-the-overlay)

## Part 1 - Auto Scroller

### How-to auto scroll the screen
This is very simply, you need to decouple the camera from the player and move the screen using the inbuilt events.

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
Now you have the camera's position calculate the screen boundary, the _camera_x/y are relative to the centre of the screen.

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

## Part 2 - Screen Sized Boss

### How-to create a moveable screen sized boss
The primary threat of the level is a screen sized drill that instantly kills the player if they collide. The player must avoid random thrust forward attacks as the screen automatically scrolls left. The boss is always visible during play with a dynamic position due thrust attacks. The boss is created using the [Overlay Window Layer](https://www.gbstudio.dev/docs/scripting/script-glossary/screen) and a sprite with largest possible collision boundary box (128 x 128px). The sprite's location is synchronized with the overlay windows movenment, the difficultly being that the overlay window and the sprite coordinates are relative to the screen and camera respectively. Pinned sprite's cannot be used to overcome disparity as they lack the "On Hit" event, instead you'll need calculate the overlay window position as if is camera relative. Once the calculated overlay position is known, set the sprite to that location. This needs to be done every frame to insure that the boss' collision boundary box is in the correct position. 

### Limitations of the overlay
The drill is positioned on the right and attacks from the right, this is intentional. The top left corner of the overlay window can be positioned between (x=0, y=0) to (x=160, y=144), it can not be position at (x=-80, y=40) or any negative coordinate. I have designed this boss to work within the layer's limitations. So it is easy to use this to create a boss attack from right to left, bottom to top or bottom right to top left corner. 

### Filling the overlay window layer
You can copy a subsection of the background layer into the window layer using [VM_OVERLAY_SET_SUBMAP](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_overlay_set_submap) or [VM_OVERLAY_SET_SUBMAP_EX](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_overlay_set_submap) GBVM commands. For our needs the VM_OVERLAY_SET_SUBMAP is simpler as we can use hard coded values.

#### [Taken from the offical documentation:](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_overlay_set_submap)

<pre> VM_OVERLAY_SET_SUBMAP X, Y, W, H, SX, SY</pre>

| Parameter | Purpose | 
|---|---|
| X | X-coordinate within the overlay window of the upper left corner in tiles |
| Y | Y-coordinate within the overlay window of the upper left corner in tiles |
| W | Width of the area in tiles |
| H | Height of the area in tiles |
| SX | X-coordinate within the level background map |
| SY | Y-coordinate within the level background map |


#### Example
![](/tutorial-resources/Background-To-Overlay.png)

At its largest the overlay window is the same size as the screen 160 x 144 pixels or 20 tiles horizontal and 18 tiles vertical. Ths background artwork contains a drill with the same dimimsions (20 x 18 tiles) located top left tile [x=60, y=0] to bottom right tile [x=79, y=17].

<pre> 
; copies a rectangle of background tiles from top left tile [x=60, y=0] 
; to bottom right [x=79, y=17] over top of the entire overlay window layer
VM_OVERLAY_SET_SUBMAP 0, 0, 20, 18, 60, 0
</pre>

### Moving the overlay window layer
Once the overlay is populated [Overlay Move To](https://www.gbstudio.dev/docs/scripting/script-glossary/screen#overlay-move-to) event or [VM_OVERLAY_MOVE_TO](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_overlay_move_to) can set position.

<pre>
; the overlay will scroll/move to the current position to the new position
; the position is tile based, tile (x=14, y=8) equals pixel (x=112, y=64)
VM_OVERLAY_MOVE_TO 14, 3, .OVERLAY_IN_SPEED
</pre>

You will also need to set the [VM_OVERLAY_WAIT](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_overlay_wait) so the overlay can move without stopping player movenment:

<pre>VM_OVERLAY_WAIT .UI_MODAL .UI_WAIT_NONE</pre>

You only need to set the VM_OVERLAY_WAIT before you move the layer for the first time and after displaying dialogue since that event reset the wait.

### Rendering sprites on top of the overlay
Be default sprites are rendered behind the overlay window layer but can rendered in front of if desired. As of GB Studio 3.1 there in no inbuilt event but there is a GBVM variable, _show_actors_on_overlay and it can be set to 0 or 1, 0 being behind and 1 being in front. [VM_SET_UINT8](https://www.gbstudio.dev/docs/scripting/gbvm/gbvm-operations#vm_set_int8) is used to set the value. This applies to all sprites on screen, if you want to selectivaly hide or show sprites you'll need to do that manually.

<pre>
; Sprites behind of the overlay window layer
VM_SET_UINT8 _show_actors_on_overlay, 0

; Sprites in front of the overlay window layer
VM_SET_UINT8 _show_actors_on_overlay, 1
</pre>

### Collision detection
Since the overlay itself lacks native collision detection we need to find a solution. The 1D method described above could work but is less performant and harder to work with standard collision group events. To leverage the inbuilt collision functionality we can use a sprite, which we will refer to as the "boss" sprite. 

#### Setup collision boundary box
Within the sprite editor you can set the collision boundary box, which has a maximum 128 pixels for each axis. This unforunately is less the maximum size of the overlay (160 x 144 pixels), if you want to cover the whole overlay there are a few solutions:

1. Use multiple sprites which will impact performance
2. Adjust the "boss" sprite's position so that aligns where the player is likely to collidate, i.e if the player is near the top of the overlay set the "boss" sprite to top left and if the near the bottom set the "boss" sprite to bottom left

For the purposes of the tutorial the "boss" sprite only need be 16 tiles high (128px) as the player's collision box is 24 pixels, so even if the player is on the bottom the of the screen it will still collidate, as well positioning the sprite at y=0 as to prevent the player from jumping over.

To illustrate "boss" sprite's collision boundary the sprite frame contains art at the top and bottom. This exists to help your understanding and is unnecessary outside of debugging unless you provide the player reason, i.e. animated smoke.

#### Synchronized collision boundary box with overlay position
We need to add logic to the "boss" sprite "On Update" so that we synchronize the sprite's position with the overlay.
To do this we need to calculate the overlay's position relative to the camera, this is stored in two varaibles:

| Variable | Purpose | 
|---|---|
| _WX_REG | overlay's X-coordinate within the screen |
| _WY_REG | overlay's Y-coordinate within the screen |

<pre>
; get the overlay and camera position
VM_GET_UINT8 VAR_WX, _WX_REG
VM_GET_UINT8 VAR_WY, _WY_REG
VM_GET_INT16 VAR_CX, _camera_x
VM_GET_INT16 VAR_CY, _camera_y
</pre>

With the the overlay's position, we can calculate its offset relative to the background.

| Variable | Equation | |
|---|---|---|
| boss sprite X | camera x - (screen width / 2) + overlay x | $bx = $cx - 80 + $wx |
| boss sprite Y | camera y - (screen height / 2) + overlay y | $by = $cy - 80 + $wy |

And finally use the "Set Actor Position" event using the $bx and $by as the values. You'll need to change the value type from tiles to px.

### Currently the rest of the tutorial is only available within the source code
