---
layout: post
banner_image: banner-laugh-kingdom.png
---
I wrote this game for the 2017 Global Game Jam in 48 hours with a small team, including my two boys and nephew.  Laugh Kingdom is a LAN network multiplayer game written in Unity using Unity networking.

Players are racing against each other to collect the most jokes and fill their joke meter. The king has a terrible temper, and the only thing that will cheer him up is the most notable type of WAVE ~ the sound wave. All his peasants must collect jokes and funny insults to compete to become his court jester. The losers will be fed to the lions.

## Youtube Video

[![Youtube Play list](/assets/images/youtube-laugh-kingdom.png)](https://www.youtube.com/watch?v=mLil2ExB2pU){:target="_blank"}

Multiplayer data tracked on the server: 

Players: Name, Transform, Animation state 

Global state: winner Messages from the client to the server: 

NetworkTransform: automatically sends player transform to the server 

NetworkAnimator: sends the players animation state to the server (mechanim state) 

FoundLaugh: when a player finds a laugh that player's laugh count is updated on the server 

CheckWin: when a player arrives at castle with all his laughs collected Messages from server to client: 

Every client gets transform and animation state for every player sent to it from the server 

OnLaughCountChanged - sent to every player on every client when their laugh count changes 

OnWinnerChanged - sent to every player on every client when a winner is set Things we aren't syncing: 

Monster transform and animations - Each client has it's own copy

Team Members

- Ryan Worthington - Jokes, Music, Ideas
- Adin Gates - Modeling (Remote)
- Nathan Worthington - Level Creation, Design, Modeling
- Jon Worthington - Programming
- Jim Byer - Programming
- James Batchelor - Dialog and Storyline (Remote)
- Free Unity Store Assets Used:

Mountain Skybox: <https://www.assetstore.unity3d.com/en/#!/content/52251>

Stone and Brick Textures: <https://www.assetstore.unity3d.com/en/#!/content/25764>

Low Poly Rocks: <https://www.assetstore.unity3d.com/en/#!/content/43486>

Monsters: <https://www.assetstore.unity3d.com/en/#!/content/77703>

Animated Knight: <https://www.assetstore.unity3d.com/en/#!/content/24471>

King and Guards: <https://www.assetstore.unity3d.com/en/#!/content/18098>

Campfire Pack: <https://www.assetstore.unity3d.com/en/#!/content/11256>

Barrows and Wagons: <https://www.assetstore.unity3d.com/en/#!/content/33411>

Unity Network Lobby: <https://www.assetstore.unity3d.com/en/#!/content/41836>

<https://globalgamejam.org/2017/games/laugh-kingdom>