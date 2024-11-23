---
title: "The one where WebSocket always time out on a UE initiated connection"
date: 2024-11-22
author: jli
tags: engineering unreal aws
---
**When ashes are two, a flame alighteth**

> TL;DR - We ran into a bug where UE 5.4.4 failed to connect to AWS Websocket Gateway while any other test client could. And UE could connect to any local websocket service we threw at it just not the one we put behind AWS Gateway. It takes 2 to make the bug happen.
>

### The project

I started to teach my older son Unreal, and we set out for an adventure to develop a multiplayer brawler game together. He takes on the design and blueprinting. I became his backend and infrastructure engineer on his team. We recently ran into a bug where our test client connects to an AWS api gateway just fine, but the Unreal engine always timeout on connection. Since the whole debug process is too tedious for him right now, I decided to write it down so hopefully when he’s come of age, he’d be entertained by the bug or the fixing of it.

### Mysterious timeout

"That's it, the matchmaking service is now working!" I said to Luke. Over the past few days, we had been experimenting with AWS CloudFormation and GameLift to host a UE dedicated server for our online multiplayer game. After several trials and errors, we finally got the matchmaking service working. Using the Postman testing client, we could connect to the game's WebSocket API endpoint, start matchmaking, and receive game info once a match was placed. Our next step was to integrate this service back into the game. Since UE already comes with a WebSocket plugin, this should be straightforward.

However, UE fails to connect to the endpoint. The connection silently times out without invoking any of the registered WebSocket handlers. After enabling verbose debug logging for the WebSocket plugin, we confirmed that while the endpoint URL is correct, the WSS connection never fully establishes. The backend logs further verify this—there's no record of the service being accessed at all. Strangely, our test client successfully connects to the backend using identical parameters.

Could this happen to all websocket initiated from UE with its websocket plugin? We wrote a mock service locally to let Unreal to validate the hypothesis, but UE was able to connect without any problem, with or without TLS.

So in short, the AWS API endpoint works with Postman but not UE. UE works with locally mocked test service but not with the one on AWS gateway.  The problem only happens when UE connects to the AWS api gateway endpoint. The plot thickens. We have to see what UE and the API gateway are sending to each other to get a better idea what’s happening.

### Tapping the wire

Wireshark to the rescue! UE’s capture shows the timeout happening, and we can see that the client and server actually finished the first part of handshaking and the client already sent a few application data, which should be the http requests to upgrade the connection to Websocket. However, due to SSL/TSL encryption the exact packet content cannot be seen.

As a comparison, a test client simply doesn’t hit the timeout. Everything before the timeout looks similar, but again we don’t know for sure, as the exact content are all hidden behind TLS encryption.

![image.png](/assets/images/ue-wireshark-1.webp)

We’ll need to decrypt the content protected by the TLS!

### “Cracking” TLS

Now we want to decrypt the network traffic for both UE initiated connection and test client so we can have more meaningful comparison.

Despite WireShark has the [capability to decrypt tls traffic](https://wiki.wireshark.org/TLS), it’s needs the application to support dumping the keys in the first place. Postman doesn’t support dumping the keys, but most browser has support. We worked around the test client key dumping problem by switching to a new test client on Chrome. However, these clients implemented the websocket client based on the browser’s websocket client, which does NOT support custom Authorization header by our websocket api. We had to detour to update the service to support extracting the token from a query parameter so it can work with those test client.

Next, we’ll have UE dump the ssl keys. UE uses [libwebsocket](https://libwebsockets.org/) underneath, which supports dumping the keys in its master https://github.com/warmcat/libwebsockets/commit/b8c4820be477b5237378b04a4216fb274c6afbbd, however, unreal has a very old patched version of libwebsocket. So now, we have to compile and link the latest version libwebsocket. Luckily, this mostly worked out of the box other than some minor updates needed for the UE websocket plugin.

### The Culprit

After these work done, we finally can see what’s in the payloads!

Chrome websocket test client’s traffic during the websocket connection handshakes.

![image.png](/assets/images/ue-wireshark-2.webp)

This whole communication sequence is very normal. Let’s take a look at the UE one.

![image.png](/assets/images/ue-wireshark-3.webp)

The connection is just hanging there after the Get request until the tcp connection gets a timeout! But WHY??? We also noticed, the path being requested is `//` rather than `/` could this be the culprit? With little hope, we added another slash in the url in our test client, hoping to reproduce the problem.

Unexpectedly, the test client got a timeout! AWS WebSocket will hold the tcp connection util timeout if the path is `//` , on any other non-existed path, it will return 403 unauthorized immediately. Ok, it’s up to them to fix the inconsistency there. But why there’s an additional slash? We didn’t put in that in our code! After a bit of tracing, we found that UE’s WebSocket wrapper adds an additional slash even if the path is empty. Well, at least it can be an easy fix in the code we can control. Praise open source.

![image.png](/assets/images/ue-source-4.webp)

(At the time of writing, AWS is fixing the inconsistent API gateway behavior and is likely going to roll out the fix soon.)