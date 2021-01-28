---
title: Learning Unreal Engine 4&#58; Dungeon prison breakout
date: 2021-01-28 19:00:00 +0100
categories: [Unreal Engine 4, learning]
tags: [ue4, game design]
---

![Dungeon prison breakout demo level](/assets/img/posts/dungeon_prison_breakout.png){: width="800"}

The whole [Mass Effect][ME] trilogy was made using Unreal Engine 3! _'Nuff said!_ I guess I'll be
learning Unreal Engine 4, then :grin: No, wait. The real reason is that, unlike Unity, UE4 runs on 
Linux. But none of this would matter if I wasn't dead curious about game engines to start off, which I am.

I chose to embark on this journey with the Udemy [Unreal Engine C++ Developer][Udemy-UE4-Course] course 
by Ben Tristem and the GameDev.tv team. Of course this is just to get one's fingers dipped. 
In order to have a better understanding of game development, you'd need a lot more than 
just a single course:

* solid grasp on game design principles and game mechanics
* level design
* specific to 3D games: 3D modelling and sculpting (e.g. Blender), 3D environments
* specific to Unreal Engine 4: be able to use both C++ and blueprints
* character animation 
* visual and sound effects

And if you ever plan on going the solo route, on top of those technical skills you'd need to 
create and manage your indie game dev studio, as well as to effectively market your games. 

As far as I'm concerned, I have zero intention of going that way. I'm just curious about game 
development in general and fascinated by what can be achieved. After so much time of dissecting how 
video games are built, I just thought I'd take a crack at building something myself. 

## Enter UE4

Despite being so powerful and versatile, it is alas very poorly documented! But since I'm dead set on using Linux and I refuse to learn Godot as my first game engine, Unreal Engine 4 is my only option. I'll figure it out along the way, right? _...right?_

I went more than halfway through the Udemy course when I got my first real challenge: using what I 
had just learned to make an actually playable demo level. So what exactly had I learned?

* The course starts off by explaining how to set up Unreal Engine 4.
* Then a cursory introduction to C++ takes place in the form of speeding you through writing a simple console application.
* The first actual contact with the Unreal Editor is through a console-based game that is actually 
implemented in a virtual terminal inside a 3D environment.
* **"Building escape"**. This is the chapter for which I made the demo below. It's also the real 
introductory chapter to Unreal Engine 4, teaching you:
    * The basics of Unreal classes and components
    * What static meshes are
    * How to build basic geometry using BSP
    * How to set up basic lighting
    * What collision is 
    * How to create a grab/release component for the default pawn
    * How to create a basic actor component with a precise role (e.g. for opening and closing a door) 
    * How to add basic sound effects

## Building escape final challenge

The Udemy course then sends you off spiraling into the dark abyss, giving you the challenge to create an actual playable level, as polished as you want it to be. Challenge accepted!

### Game design

The level is supposed to have the player solve a series of puzzles by manipulating objects in the game environment in order to escape an enclosed space. My choice of asset pack went with the [Medieval Dungeon][] pack from the Unreal Marketplace. This resulted in creating a sadistic dungeon prison environment from which the player must free herself.

The player can exit the dungeon by placing four "magic cubes" in the appropriate locations, as well as a "magic crystal". When all of the five items are in place, the dungeon doors open and the game is over.

### My demo level

Here is the Youtube link to the playable level:

<figure class="video_container">
  <iframe width="700" height="395" src="https://www.youtube.com/embed/18r5_2hjHUo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</figure>

### Level design

Below is a simplified level design document for the dungeon prison breakout level (most of the props are not shown). The player starts in the prison cell located in the bottom right corner of the map:

![Minimalist level design](/assets/img/posts/dungeon_prison_breakout_level_design_labels.jpg){: width="800"}

With the help of this document, I will attempt to explain how the level plays. In order to get the cubes and the crystal, the player goes through a certain number of interactions:
* In the prison cell where the player first finds herself, two wooden planks must be moved to uncover a magic cube. This magic cube activates a pressure plate that opens the gate of the prison cell.
* A magic cube is out in the open in the "main hall", where four statues have empty statue stands in front of them; when a cube is placed on the correct empty statue stand, the corresponding statue is "activated" with a sound, a particle effect and a light source.
* Another cube can be found inside a wooden coffin in the torture hall, right in front of the prison cells.
* In the memorial chamber (accessible through the main hall), a ceramic pot may be moved in order to uncover a key. The key opens one of the remaining prison cells.
* Entering this prison cell, a big stone can be moved in order to gain access to the adjacent cell. From the adjacent cell the player climbs on a platform that allows a jump to the final prison cell. In this last prison cell, two small stones can be used to activate the pressure plate that opens the gate of the prison cell, and a magic cube may be dragged outside of the cell along the way.
* The big stone in the cell that can be opened with the key can be exchanged with the cube on the pressure plate in the player's cell. Now all the four cubes should be in place on the initially empty statue stands in front of the statues in the main hall.
* The last key to the puzzle is a crystal that must be placed on a fifth statue stand in the middle of the four stands with the magic cubes. The crystal can be accessed by removing the stone coffin lids in the mortuary chamber, accessible through the main hall.

## My learning challenges

While building this level I came across some interesting challenges. Most of them involve blueprints, an alien visual scripting language designed to torment the souls of UE4 newbies. At times it's frustrating as I feel like I have no idea what I'm doing :stuck_out_tongue_winking_eye:

### Outline for interactive items

I wanted to guide the player by giving her a visual cue for objects that she can interact with. The idea is to use a custom depth stencil value and a post-processing volume. Every interactive object has its own blueprint. There are actually two distinct custom stencil values that I use, because I wanted to make the key really stand out since it is very small. I used [this tutorial][YT stencil] to get started.

### Flexible C++ components for doors

I wanted to have doors open and close according to a variety of conditions and behave a certain number of ways, for example:

* Once open, some doors must remain that way (the gate to the prison cell in the middle).
* Some doors are activated by the player pawn (the doors to the mortuary and the memorial chambers).
* Some doors are activated by another actor (the key for the gate to the prison cell in the middle).
* Some doors are activated by the total mass on the pressure plate (the gate to the player's prison cell).
* Some doors are activated by the presence of several actors in a single trigger volume (the gate to the last prison cell).
* Some doors are activated by the presence of several actors in several trigger volumes (the gates for the dungeon's exit need the four cubes and the crystal in place).

I achieved this by exposing many variables to the Unreal Editor and by adopting a modular class design. The base class implements basic behavior (opens and closes a door at a given angle and speed). It is extended by two derived classes that describe how certain actors are expected to trigger the conditions for controlling the door.

### First contact with particle systems

I created a very simple particle system following [this tutorial][YT particle]. Its role is to emphasize that the cubes and the crystal have been placed in the proper locations. The particle system is handled by a C++ component that I attach to the static meshes that must be emphasized. In order to activate the particle system when the right conditions are met, I struggled with a few nasty crashes before I understood that setting the location of the particle system (`SetRelativeLocation()`) can only be done in `BeginPlay()`, *not* in the component's constructor, since at that point the owner's location is unknown.

### Playing a sound when an item is dropped

Initially I had naively added two audio components for items that the player can interact with: one was played on grab, the other on release. However, playing a sound when the object is released is not the same as when it hits the floor. From [this tutorial][YT sound] I learned how to make the sound play only when a hit event (collision) occurs. But this wasn't the full answer to my problem, since every time the player moved around with an item attached to the physics handle and the item collided with something in the environment, the sound would play. I ended up exposing a C++ variable in the grab/release component to the blueprint editor in order to check which actor is currently attached to the physics handle. In the blueprint I implemented the desired behavior: if a collision is registered for an actor **and** if that actor is no longer attached to the physics handle of the player pawn, **only then** play the "drop" sound. It's not perfect but it was an interesting exercise!

## Resources used

* Models/meshes:
    * The [Medieval Dungeon][] asset pack from the Epic Marketplace
    * The [mossy stone][] mesh and material from Quixel
* Surfaces (from Quixel):
    * [Castle wall][]
    * [Marble detail][]
* Sounds (from Freesound):
    * Ambient track: <https://freesound.org/people/DrMinky/sounds/166187/>
    * Pick up sound used for cubes, rocks etc.: <https://freesound.org/people/Mediaman57/sounds/347149/>
    * Drop sound used for cubes, rocks etc.: <https://freesound.org/people/Bird_man/sounds/275160/>
    * Drop sound used for stone coffin lids: <https://freesound.org/people/Robinhood76/sounds/503554/>
    * Drop sound for planks and wooden coffin lid: <https://freesound.org/people/kingsrow/sounds/194692/>
    * Shattered cups sound: <https://freesound.org/people/tezzza/sounds/21612/>
    * Ceramic click sound: <https://freesound.org/people/Mafon2/sounds/371278/>
    * Key pick up sound: <https://freesound.org/people/BeezleFM/sounds/512137/>
    * Key drop sound: <https://freesound.org/people/Kyanite_/sounds/432913/>
    * Crystal pick up sound: <https://freesound.org/people/DWOBoyle/sounds/474179/>
    * Magic activation sound: <https://freesound.org/people/kneekoo/sounds/548497/>
    * Wooden door open sound: <https://freesound.org/people/joedeshon/sounds/117416/>
    * Wooden door close sound: <https://freesound.org/people/joedeshon/sounds/117415/>
    * Metal door open sound: <https://freesound.org/people/LittleRobotSoundFactory/sounds/270461/>
    * Metal door close sound: <https://freesound.org/people/newagesoup/sounds/339369/>

## Conclusion

Completing my "dungeon prison breakout" level was a very interesting and challenging journey. I've come a long way since I enrolled in this course, but there are so many things I'm yet to discover. I leave this little project behind me with a few small regrets:

* The landscape outside the dungeon: that's not a landscape, it's an abomination. I need to learn how to do this properly, how to sculpt the environment, use brushes, add foliage and so on.
* Destructible meshes cannot be moved, so if I wanted the ceramic pot to break when thrown I'd need to hot-swap the movable mesh with the destructible mesh when the player releases it from the physics handle. I'm not sure yet how to do that.
* The main hall feels empty (it's too big for the props that I've sparsely placed in the level).
* The particle system is ugly.
* The lighting is fake. The shadows of the ceiling chandeliers should not be seen on the floor.
* Some volumetric fog could add to the general creepy feeling I wanted the dungeon to convey.

But it's time to move on with the course. "Toon Tanks", here I come!

<!--links-->
[ME]: https://en.wikipedia.org/wiki/Mass_Effect
[Udemy-UE4-Course]: https://www.udemy.com/course/unrealcourse/
[Medieval Dungeon]: https://www.unrealengine.com/marketplace/en-US/product/a5b6a73fea5340bda9b8ac33d877c9e2
[YT]: https://youtu.be/18r5_2hjHUo
[YT stencil]: https://www.youtube.com/watch?v=YVTcH6da32Y&list=PLtP1toxty8o27kpPPLyfqLIr27ZRDFRq7&index=6&ab_channel=wizvanmeter
[YT particle]: https://www.youtube.com/watch?v=5xUENgiGrRw&list=PLtP1toxty8o27kpPPLyfqLIr27ZRDFRq7&index=9&ab_channel=DeanAshford
[YT sound]: https://www.youtube.com/watch?v=_UVwKN8dtQA&list=PLtP1toxty8o27kpPPLyfqLIr27ZRDFRq7&index=13&ab_channel=JoeHudson
[mossy stone]: https://quixel.com/megascans/home?category=3D+asset&search=castle&assetId=sdAfk
[Castle wall]: https://quixel.com/megascans/home?category=surface&category=stone&category=castle&search=castle&search=wall&assetId=tbhoajkr
[Marble detail]: https://quixel.com/megascans/home?search=black&search=marble&assetId=sjvobj1c