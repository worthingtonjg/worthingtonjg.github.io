---
layout: post
title: "Creating Relic Hunter at the Global Game Jam 2016"
category: 
- Unity3d
- Hackathon
- Global Game Jam 2016
- Games
meta_description: 'Creating Relic Hunter at the Global Game Jam 2016'
browser_title: 'Creating Relic Hunter at the Global Game Jam 2016'
banner_image: relic_hunter-1.png
---
I participated in the 2016 Global Gam Jam with my two sons and a couple of my friends from work.  The theme of the game jam was "Ritual".  Going into the Game Jam we had decided before hand to make a Descent like clone (or get as far as we could on this in 48 hours).

The game we made is called Relic Hunter.  Below is the intro screen to our game:

![Intro screen](/assets/images/relic_hunter-2.png) 

## Level Design

The level was designed entirely in blender by Matt Morey and Nathan Worthington.  Prior to the game jam we did some training in Blender to prove that the level design could be done this way.  

- The walls for the level are essentially one big mesh.
- The doors are seperate models
- The fan in the first room is was modelled in blender and is made from several pieces so we could spin the inner fan    

## Ship Models and Pickups

- Enemy Ships: Nathan did the modelling for the enemy ships.  He designed several ships and detachable guns but we only had time to use two of his models. 
- Health Pickup: Nathan modeled the health pickup as well as the health console
- Ship Cockpit: Jon (me) modelled the ships cockpit.

## Scripting

When creating the scripts for the game we tried to keep each script small and re-usable with only one purpose.  We use SendMessage to send messages between components, allowing each script to have it's own responsbilities but still communicate with other components.

### Door

Purpose:

- Manages the state of a single door
- Detects when a door should be opened

Public Variables:

- OpenPosition: The position the door should slide to when it opened.
- OpenSpeed: How fast the door should open.
- DoorState: The current state of the door (closed, open, locked).
- Listeners: List of game objects that are listening for a door to be opened.

Messages Recieved:

- TakeDamage: When a door takes damage the door will open if it is unlocked
- Unlock: Will open a locked door

Messages Sent:

- DoorOpened: Sent to Listeners to tell them the door was opened (this wakes up enemies)
- UnlockDoor: Sent to the player when a locked door takes damage

### ActivateEnemy

Purpose: 

- Sends either a "PlayerInRange" or "PlayerOutOfRange" message to the enemy when the trigger is entered or exited by the Player

### EnemyHealth

Purpose:

- Allows the enemy to recieve damage from projectiles
- Explodes the enemy if it's life reaches zero

TakeDamage Message:

- Decrements the enemies health
- Updates the health bar
- Plays the "hitClip" audio
- Checks to see if the enemy is dead
- If dead instantiates explosion, plays the explosion clip, and destroys the prefab
- Sends the EnemyKilled or BossKilled message

### LocatePlayer

Purpose: This is the enemy AI

PlayerInRange and HitByPlayer Messages:

- These messages will cause the enemy to start looking for the player
- The enemy will look toward the player
- It will move toward the player's position until it reaches it's stopping distance
- If the enemy can see the player (with a raycast), then it will "Shoot" the player

PlayerOutOfRange Message:

- The enemy will move the the last place it saw the player and then stop 

### Pickup

Purpose:

- Can be attached to an item that can be picked up

Variables:

- PickupMessage
- PickupValue

OnTriggerEnter:

- When the collider is entered by the Player, the PickupMessage and PickupValue is sent to the player.

### Projectile

Public Variables:

- Damage: How much damage the projectile does
- FiredBy: Who fired the projectile

OnTriggerEnter:

- The code simply sends the "TakeDamage" message if the projectile hits another object
- Then destroys the projectile

### Shooter

Purpose:

- Used by both enemies and players to control shooting weapons
- Each weapon equipped will have a shoot script 
- Fires a projectile (playing the shoot sound) either automatically (for an enemy) or when the "Fire1" button is pressed (for a player)

Public Variables:

- FiringSpeed: How fast / frequently a bullet can be fired
- ProjectileSpeed: The velocity of the bullet
- ProjectileLife: How long a bullet lasts before it destroys itself
- AutoFire: Enemies using the shoot script will have autofire set to true 

### MovePlayer

Purpose:

- Takes input from the keyboard or xbox controller
- Uses a CharacterController component to move the player based on the input

### Inventory

Purpose:

- Keeps track of which relics (keys) a player has found
- Keeps track of how many enemies a player has killed
- Unlocks doors if has the correct relic

Message Recieved:

- PickUpRune: Logs which rune was picked up, and destroys the prefab
- UnlockDoor: Checks to see if the appropriate key (relic) is in inventory, and if so Sends "Unlock" message back to door
- EnemyKilled: Updates the enemy killed count
- BossKilled: Logs that the boss was killed 

### PlayerHealth

Purpose:

- Keeps track of the player's health and updates his health bar
- TakesDamage
- Checks if dead

## Gam Jam Participants

- Matt Morey ~ Level Design & Blender Models 
- Nathan Worthington ~ Level Design & Blender Models 
- Jim Byer ~ Programming   
- Jon Worthington ~ Programming   
- Cameron Byer, Brooklyn Byer and Ryan Worthington ~ Additional Level Design & Testing 
- Michael Fewkes ~ Game Music 

## External Art Work, Music and Toolkits Used from Unity Asset Store:

- Scene Transitions Fungus Games Toolkit
- Metal Mayhem Music Pack Unity Technologies
- Explosion Particles Merza Beig
- Generated Sound Effects www.bfxr.net
- Skyboxes  Hedgehog Team
- Player Controller Unity Standard Asset
- Industrial Textures Arkham Interactive
- SciFi Door  3D Mondra
- 3D Holographic GUI Skin MPixels
- PBS Materials  Integrity Software
- Yughues Free Metal Nobiax / Yughues
- Ultra Emissive ParticlesGalactic Studios
- Free ArtskillZ Textures Luca Eberhart
- Scifi Textures  ArtzkillZ Texture Pack

## Links:

- Jam Site: [IGDA Salt Lake City](http://globalgamejam.org/2016/jam-sites/igda-salt-lake-city){:target="_blank"}
- GitHub: <https://github.com/worthingtonjg/RelicHunter>
- Play At: <http://www.wetechgroup.com/UnityGames/RelicHunter/index.html>
- Xbox Controller is required to play this game
