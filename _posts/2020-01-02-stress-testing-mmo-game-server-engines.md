---
layout: post
title:  "Stress Testing MMO Game Server Engines"
author: devfans
categories: [ MMO, stress, test, game, networking, multiplayer]
image: /images.pexels.com/photos/596750/pexels-photo-596750.jpeg?cs=srgb&dl=buttons-close-up-controller-596750.jpg&fm=jpg
tags: [sticky]
---
While selecting the fittest choice from several existing game server engine candidates is hard, especially for MMO games, doing valid comparison and some stress tests are the basic approaches to become familiar with them, though it's still hard some times to give a clear result if the game is still at early stage like when the gameplay direction and battle requirements are not clear yet, or the team lacks valid experience and skills. 

A typical stress test of MMO game server engines means the game maker would create a large scale test event to attract the real players from the game communities to join the battle when the event starts in the meantime the game already has basic gameplay features, then after the event, the maker could collect the data and analyze the system performance capacity and potential bottlenecks. Some times, the stress tests would last for several phases with iterations for bug fixes and improvement updates. 

For example, The Hadean which makes the Aether Engine for MMOs was cooperating with CCP for Aether War tech demos, here's one case study after the phase 2 test: https://www.hadean.com/blog/the-growth-of-aether-engine-a-next-generation-cloud-native-game-engine.  Another example is the Fractured studio Dynamight which made Linkrealms was making stress test after they start trying the SpatialOS game engine and here's the result: https://fracturedmmo.com/december-2019-stress-test-review/. 

Normally before a public stress test, the dev team could do internal stress tests with simulated clients or find other ways to simulate the game server load, and at this stage, the gameplay is kind already roughly formed. However, in this post, I am gonna talk about doing stress testing without valid gameplay implemented, with the purpose of trying to figure out the performance capacity of given server engine solutions with very basic(no-playable) gameplay features. This is necessary some times when you try to make the right decision even without knowing what the gameplay would be like, but of course, at least we should already know the game genre which could matter a lot when considering the choices. For us, surely it would be MMO with some RTS elements.

Go back to earlier days, when we were trying to select a valid game server engine for a basic gameplay prototype implementation, we compared a bit for several solutions which include Photon PUN, Photon Bolt, SpatialOS and Hadean's Aether Engine. The Aether Engine was not in the options for its product and SDK was not out yet, so we compared the rest. The factors we were taking into consideration include:
Product Maturity/Popularity
Performance capacity
Integration with Unity
Features(ECS, etc) Support
Technology Adopted
Deployment plan and cost estimation
Product and Service Support
Troubleshooting utilities

Maturer means more stable, fewer bugs and fewer key features/logic refactors, and this matters when you don't want to spend too much time to reimplement game server logics when the server engine releases new updates, and don't want to spend too much time to cooperate with engine providers to troubleshoot engine side bugs, which could easily let the dev down if there are not much useful troubleshooting tools available the progress has to depend on the provider's support. We also care about the technologies adopted by the engine. Like supporting ECS, it means it at least towards the right direction which improves a lot of the performance and provides more potential for the gameplay design. 

But about how we test out the performance capacity of each solution, we didn't get a clear idea at that time, so we were mainly trying out each product to get familiar with them and learn about their main concepts. Also, we were taking the official statement into consideration. Even though the game designers were saying the concurrent players would be less than 25 within one battle instance, we were still not able to trust that since the genre is MMO. According to the Photon's statements, their products are targeted towards 2-50 players and up to 100+ players with special cares, while SpatialOS states it's designed for MMO and supports tens of thousands of players. Right, so our team voted and it turned out we chose the SpatialOS for the gameplay prototype implementation. Besides that, I also tried out Unreal engine's built-in networking module for state replication and also implemented a custom solution with lockstep concepts. For the game server networking solution topic, please check my previous post.

Though someone might still be wondering something with lockstep concepts, which is evolving to C/S mode with deterministic client logic, from my understanding, it is mainly targeted for small scope arenas like MOBA or some RTS games, while it probably won't serve well for MMOs since MMOs prefers persistent state with large scale scope of players and each client might only care about a piece of it and does not need to compute the entity world logic. Looks this post (http://ithare.com/mmog-world-states-and-reducing-traffic/) stated the same opinion when talking about "deterministic lockstep".

With some feedback from the team after using the SpatialOS for a period and we are starting a new stage for the game, it looks like it's the time to rethink about the decision we made. The pains from dev teams mainly are it's sometimes buggy and not easy to troubleshoot. Also, the Aether Engine's starting to provide their SDKs for trying out. So we are focusing back on the comparison of the two engines.

#### Key Values
There're some values we care about while testing things out, which include: 
Server Tick Rate
Server State Replication Frequency
Client Tick Rate
Client Input Rate
Server-side Entity Number
Clients Number
Resource Usage(CPU, RAM, Traffic)

#### Test Cases
Also listing designed test cases from thoughts of the team:
A. MAX server-side moving entities with limited clients.
B. MAX clients with one moving entity being held by each client.

#### Concepts, Conditions, and the Variable
SpatialOS and Aether Engine have some similar concepts. For example, they are both implemented with ECS concepts and can have multiple server workers for server load sharding, while the SpatialOS can delegate some sub-systems to different workers or split the game world to several regions, Aether engine is using octree tech to dynamically allocating cells according to load estimation of the area. Also they both split networking handling from server logic, which makes sense for decoupling things and offloading. SpatialOS client workers connect to the runtime while Aether engine clients connect to the muxer(multiplexer) which shines for Aether supports to deploy multiple muxers to replicate world state and serve as proxy delegates to improve the networking performance especially when the players are distributed in a bigger physic scope. The muxer is also responsible for interest scope based offloading which means a client can replicate the world state but with selecting just a meaningful part of it. SpatialOS have similar concepts named QBI(Quey based interests). Aside from this, the SpatialOS looks much maturer than Aether engine when talking about community, utilities, Unity-Integrations, and documentations. 

Though some of the above concepts(like interest scope) may improve the performance a lot for real stress test environment, however, here we are making internal stress test and aiming to see much load can the engines bear with no networking offloading, and with limited sharding when possible. Here we also keep targeted performance counters as:
Server Tick Rate: 15hz
Server State Replication Frequency: 15hz
Client Tick Rate: 60hz
Client Input Rate: 60hz
Limited Resource: two computers with powerful i5/i7 CPU(6 or 8 cores) and 16GB RAM.

#### Test Case A
For testing max server-side moving entities, it's mainly about state management and replication capacity. When we have one or two clients to receive state updates from server at 15hz, the bottleneck might appear at:
Server Tick Rate(15hz): At each game loop, the server would compute new positions for maintained entities and update them, also send state updates to the connected clients. So when the number of entities goes up, the server load would increase and tick rate decreases.
Client Tick Rate(60hz): At each game loop, a client needs to try to receive new updates from the server and apply them to the local state, then update the rendering. More entities would mean more load on state updating and rendering, but since we are stress testing the server engine, we can remove the client rendering part when possible.
Network Traffic: Increased traffic will appear too when adding more and more entities. The bandwidth between the two computers is 100mbps. 

When we are doing the SpatialOS test, we get rid of the SpatialOS builtin Transform-Synchronization, but with a simple custom transform component instead to avoid too complex logic bound with extra performance overheads. Also, the SpatialOS has the op concept, which means each state update or remote command is composed of one or more op, and the op quantity is limited when running the server on their cloud depending on the size of the deployment. However, if we issue a transform update for each entity when its update is called, the performance is not good enough without knowing if SpatialOS will batch the op before sending them out to the clients, but after our team batch the transform updates manually, the performance truly increased a lot, and here's the result: When goes to 1.3K entities the runtime starts freezing with update rate dropping, and CPU(i7 8 cores) usage at about 53%. Knowing the resource usage is still ok, there's still some room for improvement if we can find the root cause of the freezing runtime.

For the Aether engine, after we improved the client a bit with a lightweight OpenGL client, the performance looks pretty good: When goes to more than 40K entities, the update rate starts dropping, with resource(ubuntu VM inside one computer's hyper-V) usage at CPU(4cores) 20%, traffic 46MBytes/s.

#### Test Case B
For simulating clients, the performance could be impacted by similar aspects as test case A, but with a bit difference:
Server Tick Rate(15hz): At each server loop, the server needs to apply the updates from clients and then issue a state replication to the clients. But it's not indeed directly from or to the clients since the networking handling part is split from server logic to the Aether muxer or the SpatialOS runtime.
Network Traffic: Each client needs to receive state updates for all the entities, so the traffic would increase a lot than test case A.
Client Tick Rate(60hz): At each client loop, the clients need to calculate a new position and send it to the server (Each client is owning one entity on the server), also needs to try to receive state updates for all the game world entities, and apply them to the local state when received. The load could increase a lot when simulating more and more clients.
Our team is simulating the clients using the SpatialOS client workers, without knowing where are the bottlenecks, when the number goes to about 40, the performance starts to drop.

For testing with Aether engine, after removing the rendering part, and with spawning the simulated clients to several threads, the performance is improved a lot and even more after some extra improvement. The result shows when the number of the clients goes to about 570, the state update rate starts to drop and the server tick elapse is about 57ms. But the real the situation is the simulated clients are running on the same computer as the server for the traffic goes to 232MB/s(1.8Gbps), so there's still room when we get better network cards and if we run the simulated clients on several other computers.

So far, it looks like we already get some rough test results, but the question in my mind is how much these tests can impact the comparison of two products. The tests are too simple, and without valid gameplay features nor physics simulation. So I am wondering we need a  test case C with a bit complex server logic at least with some additions of the physics simulation. 


