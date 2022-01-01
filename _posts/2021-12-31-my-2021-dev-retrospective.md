---
title: My 2021 dev retrospective
date: 2021-12-31 00:00:00 +0100
categories: [Meta, career]
tags: [meta, personal development]
---

I'm working on improving my habits so I'll be writing annual yearly retrospectives... starting today! This is what 2021 brought about, professionally-wise:

![2021 dev retrospective](/assets/img/posts/2021_retrospective.png){: width="700"}

## Soft skills

I'm slowly **navigating corporate world**, organizationally-wise as well as human-wise.
* I strive to ask the right questions to the right people.
* I try to engage people on subjects of general interest (being up-to-date with business and technical decisions, learning opportunities etc).
* I try to engage management on subjects of *personal* interest (learning opportunities). Currently failing at it, but I'll get better :grinning:

I've also started **networking** a bit in order to keep in touch with ex-colleagues and acquaintances, as well as to keep up-to-date with tech trends that interest me.

## Personal brand

I've managed to **blog (somewhat) consistently** throughout 2021, averaging one blog post every two weeks. There have been entire months where I haven't published anything, and months where I published 9 posts.

I finally created my **LinkedIn profile** and started getting in touch with people in my past and present professional networks.

## Career

I have a **career plan**, yay! It's not that I didn't have one before; it's just that I realized I prefer *back-end engineering* to *embedded engineering*. More on the actual plan in the New Year's Dev Resolutions post.

## Work

Throughout 2021, my work has gotten more **back-end oriented**, which is right on track with my career plan. I also got to design an awesome hardware testing framework for our products, written in Python and loosely modeled upon [OpenHTF][]. It makes writing tests a breeze with an intuitive API, and it strives to minimize time spent by test operators at the test bench. I hope to get the management's agreement to publish it under an open-source license sometime during 2022.

## Tech skills

*So much to learn!... So little time!...* --> My motto and the story of my life. :joy:

I didn't have a proper learning plan for 2021 (which is something I intend to correct for 2022) but I managed to learn a thing or two during this past year:
* Early 2021 I played around with **Unreal Engine 4**. Just because it's cool. I haven't finished the course yet but I was able to make a [playable level][escape] for a puzzle game where the player has to escape from a creepy dungeon.
* At work, **Python** kept me good company. I got the chance to intimately discover `asyncio` and `flask`. I've also had a lot of fun with `selectors`, which allowed me to implement some groovy finite state machines for a TCP-based protocol using raw `socket`s.
* During autumn 2021, I got up to speed with **modern C++** (C++11, C++14 and C++17 standards). I started porting my C-based [Gal4xy][] turn-based game to C++.
* During winter 2021, I familiarized myself with [Protocol Buffers][protobuf] and [gRPC][grpc]. The plan is to use both in the Gal4xy C++ port in order to render it multiplayer.
* Throughout the year I've read bits and pieces on **architecture and system design**. Scalability, reliability, microservices, load balancing, that sort of stuff. I still lack context. It will be one of my main focus points for 2022.

## Conclusion

It's been a good year. I grew a bit. I'll strive to do better in 2022. I still don't understand why I've waited so long to start writing yearly wrap-ups and to plan for the year ahead. Humans are weird, and I'm one of them. :smirk:

<!-- links -->
[OpenHTF]: https://github.com/google/openhtf
[escape]: {% post_url 2021-01-28-learning-ue4-dungeon-prison-breakout %}
[protobuf]: https://developers.google.com/protocol-buffers/docs/overview
[grpc]: https://www.grpc.io/docs/
[Gal4xy]: https://github.com/alexandra-zaharia/Gal4Xy