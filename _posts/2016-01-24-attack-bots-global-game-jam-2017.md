---
layout: post
title: "Creating Attack Bots at the Global Game Jam 2016"
category: 
- Unity3d
- Hackathon
- Global Game Jam 2016
- Games
meta_description: 'Creating Attack Bots at the Global Game Jam 2016'
browser_title: 'Creating Attack Bots at the Global Game Jam 2016'
banner_image: attack-bots.png
---
I took my 10 year old son to the 2016 Global Game Jam and this is the game we made together.  Prior to the game jam I spent some time putting together a library of modular pieces: such as halls, intersections, tees, etc.  I let my 10 year old
practice making levels with these pieces prior to the game jam.  I also created all the scripting and prefabs for the enemies and the player.  This allowed Ryan to use the designer during the game jam to create a level using these modular pieces and prefabs.  

## WebGL Demo

<http://worthingtonjg.github.io/AttackBots/>{:target="_blank"}

## Disclaimer

This code was written in 48 hours for a Game Jam / Hackathon. As such there are some "hacks" and quick fixes that were implemented due to time constraints. Please keep your expectations for code quality low and keep in mind 48 hours is not a lot of time to finish a project. And much of this project was done on very little sleep.

## Prefabs

- Doors: Blue Door, Green Door, RedDoor, YellowDoor
- Enemies: AttackBot
- Keys: Blue, Green, Red, Yellow
- ModularWalls: Corners, DeadEnds, Halls, Intersections, Tees, Rooms
- Pickups: Health
- Projectiles: enemyLaser, playerLaser

## Scripting

Below is the list of scripts I created and attached to the prefabs.

### ActivateAI

Purpose: 

- Sends a "PlayerInRange" or "PlayerOutOfRange" message when the "Player" enters and exits the collider

### Door

Purpose:

- Allows a single door to be opened

Public Properties:

- OpenPosition
- OpenSpeed
- DoorLocked

Messages:

- TakeDamage
- Unlock
- OpenDoor

### Exit

Purpose:

- OnTriggerEnter: Loads the next level

Public Properties:

- NextLevel

### Health

Purpose:

- Updates the health bar of the Player or enemy
- Increments Health when health is picked up
- Decrements Health when damage is taken
- Kills the player or enemy when health is below zero

Messages:

- TakeDamage
- PickupHealth

### Keys

Purpose:

- Keeps track of the keys that the player has
- Sends the Unlock message if has the correct key 

### LocatePlayerAI

Purpose:

- If the player is out of range then we move the enemy to the last known location of the player and stop
- IF the player is in range, then do a raycast to see if we can see the player, if we can see them then look at the, start shooting and move toward them
- Navmesh is used for navigation
- Receives the PlayerInRange and PlayerOutOfRange messages

### Pickup

Purpose:

- Makes an object a "Pickup"
- Sends the "Pickup" message to the player with the tag of the pickup 

### Projectile

Purpose:

- Detects if the projectile hits another object
- Sends the "TakeDamage" message to the other object that was hit

### Shoot

Purpose:

- Instantiates new projectiles at the firing speed
- Moves (shoots) the projectile at the bullet speed
- Detects "Fire1" input from the player
- Or automatically fires the weapon when the enemy is in range of the player

## Gam Jam Participants

- Ryan Wortington ~ Level Creation 
- Jon Worthington ~ Programming   
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
- Dev Post: <http://devpost.com/software/attack-bots>{:target="_blank"}
- GitHub: <https://github.com/worthingtonjg/AttackBots>{:target="_blank"}
- Play At: <http://worthingtonjg.github.io/AttackBots/>{:target="_blank"}
