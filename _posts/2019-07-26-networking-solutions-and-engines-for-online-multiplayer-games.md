---
layout: post
title:  "Networking Solutions and Engines for Online Multiplayer Games"
author: devfans
categories: [ MMO, MOBA, game, networking, multiplayer, lockstep ]
image: https://static.livefeed.cn/static/blog/buttons-close-up-controller-596750.jpg
tags: [featured]
---

In modern days, the Internet is showing its power greatly and peoples are getting faster and faster Internet access around the world, and even further with the 5G coming soon. Gaming online has been popular for a long time and MOBAs like LOL and MMOs like Fortnite are showing the benefits from the faster Internet. This blog is ganna talk about networking solutions and available networking engines in current days, but mainly based on my research and might have some misunderstanding at some points.

## Online Multiplayer Games
A typical online multiplayer means each player connected as a single client can see other player's realtime state like positions and behaviors and react upon that. In the sense of  `realtime`, different games may have different requirement levels. For example, MOBA games like Dota2, the realtime state means the player can do the right actions according to what he can see or he'll miss the target if he's seeing the position of the target which actually already changed several frames ago. For MMO games, the requirements might be a bit lower, if not many interactions happening between two targets. 

## Networking Solutions

#### Demerministic Architecture

A traditional way to handle this is called `lockstep` way,  which means in each game logic loop, a client should be able to successfully send its input to every other client and get the inputs from other players before the game logic is simulated. So the architecture would be like peer to peer networking. With this kind of architecture, the game client needs to generate deterministic behaviors with given inputs, that means when every client simulates the game logic, they should give the same output otherwise the players would see different game state on the screen and the game can not continue anymore. 

With peer to peer networking topology, there are some drawbacks, because it requires every client to have a good connection with every other client. It could work well in a LAN network but could be an issue on the slower Internet. So, a new solution is to host a dedicated server to receive the inputs from each client and broadcast the data. It still requires the client to do the simulations and give deterministic results on each client. With this improvement, higher throughput can be served and players can have better game experience. Modern MOBA games normally use this solution nowadays.

Deterministic behavior is doable if all the client hardware and middle stack are covered for technical differences that may give a bit different computation output, which might also give different precision of the result. However, given a different behavior, the game would not be able to continue, or can not tell which client the global game state should trust. So hacking or some cheat might happen at the client-side with this solution. 

Even though, the strength of deterministic architecture is quite obvious. The clients won't need to replicate the state of each game object and only transfer the latest inputs. In addition, the client won't need to care about the authoritative server things, because all the simulations happen at the client-side, so the developer only needs to focus on client-side game logic.

#### Authoritative Server

So, another popular networking solution is to host the dedicated authoritative server to simulate all the game logics and replicate the game state to each client.  With this solution, every client has to trust the server and fetch the latest game state from it from time to time. And if a player wants to take any action in the game, he should send the input to the server to make the action really happen. There are mainly two drawbacks coming with this solution. One is the replication of every game unit in the game would require plenty of traffic when the quantity goes up. Another thing is the server would consume CPU resources to simulate the game logic, especially when the game logic is complex and there are lots of game units doing realtime interactions with each other.

When people are making networking engines with authoritative servers, they design ways to overcome these issues. For the first one, with plenty of game targets, the client only has to fetch the realtime state of the target which is in its interest scope. If one target is out of the scope which could mean it's currently far away from you and you won't do any interaction with each other and you even can not see it on your screen, then you don't need to replicate its state from time to time. So with the concept of interest scope or some engine designed a replication graph to ease the load of state replications. For the second issue, customized engines are designed to dynamically consume hardware resource by auto-scaling on the cloud side, and the server only simulates the physical that matter in the game logic.

## Networking Engines

#### Unreal Engine
The Unreal Engine of Epic Games has built a networking module to support hosting the authoritative server and the client controller can use RPC calls to take game actions.

#### Unity
The networking module in Unity was called UNet, which was already deprecated and Unity is working to make a new much powerful one, on which google might be keeping a partnership with Unity.

#### Photon
The Photo of Exit Games has built a cloud platform to have online multiplayer game developers, and they provide several solutions for game makers. The Photon Realtime is mainly targeted as a cloud-based full-stack networking solution with state replication architecture which can provide customized service to use their cloud resources. The Photon Unity Networking also called PUN is a solution based on the realtime solution but is focusing on integration with the Unity game engine which reimplemented Unity networking modules and enhanced it. The Photon Bolt is a customized solution with inspirations in design which need to run a Unity instance to serve as the authoritative server, also it provides flexible high-level APIs to optimize the performance. Another product of them is called Photon Quantum which is based on deterministic architecture and focuses on MOBA games and action games.

#### SpatialOS
The SpatialOS provided by the Improbable.io starts to show it's power with MMOs as the main target. With customized server worker running with the SpatialOS, it enables one game instance to has a large map and serve at least hundreds of players while the authoritative server is cross-host and dynamically scaling. With the investment from the Netease, SpatialOS has started the exploration of the Chinese market.

#### Aether Engine
The company called Hadean is a group of minds designing deep-level customization of how to let the application which espectially requires the realtime distributed computations to consume the hardware resource super efficiently and with automatic scaling according to the resource requirement. Their product is called HadeanOS which empowered some science computations work and shows its potential on MMO games. They established a partnership with EVE's maker the CCP and started to work on the amazing EVE: Aether Wars project. With this tech demo, it showed its potential in MMO realtime games. According to their statistics, the game instance had 3852 participants from all around the world and 14264 ships joined the war. Behind the project, the product called the Aether Engine started the attempts in the game networking engine market, especially for MMO realtime games.

#### Customized Engine
For big game makers, they would normally make their own engine for their speicific game, which is deeply designed for the features required by the game. This would require rich game development experience and cost dedicated staff resources to focus on the networking part, while this way could provide better performance than a general networking engine product. 


In addition, I am not a native English speaker, just trying to get used with writing posts in it.
