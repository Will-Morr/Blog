---
title: "Marble Fountain"
date: 2025-11-01
LastMod: 2025-11-01
draft: false
tags:
- ProceduralGeneration
- Art
- 3D Printed
- Python
---

I really enjoy procedural generation, especially systems designed to work with hardware outputs. After starting work at Formlabs in September of 2023 and gaining access to much nicer printers than I was used to, I started wanting to tackle some large algorithmic structure projects. Complexity is free in 3d printing, the limit of design geometry is mostly how much time you're willing to spend in CAD. I wanted to print the most complicated art piece I could think of. Marble Fountain is what I came up with. 

{{<youtube ULpNiZU2tpg>}}

## Tracks

My initial system came together quickly. Randomly placing spaced out points, drawing a spline through them, and setting a constant slope just works. My first draft was just subtracting a tube from the solid support structure which worked but was super limited. I wanted to add more parts and so I started working on a path solver. I wanted to fit as much motion into the volume of the printer as possible. This turned out to be extremely challenging. 

The solver starts by making a random series of line segments connecting the top and bottom of the lift. There are several different algorithms to generate this guess, as the initial conditional has a noticeable impact on the shape of the structure for lower path counts. It's interesting to play with different variants of the starting conditions and see how they change during generation. 

A series of functions update the positions over time to "pull" the points into a followable path. 
The points making up each path:
 - Stay in the bounding box
 - Evenly space themselves out
 - Pull towards a fixed height to enforce a constant slope
 - Enforce min and max turning radius of the path
 - Repel away from other tracks
 - Repel away from distant sections of our own track
 - Smooth out changes in slope to prevent jumps
 - Prevent slope from ever increasing

<!-- {{< figure
    src="/img/MarbleFountain/tracks.jpg"
    class="center" 
    alt="Matplotlib plot of track generation"
    style="border-radius: 8px; width:80%"
>}} -->

Velocity is a much harder problem than I anticipated. The tracks break a lot of the obvious assumptions if you act like the marble is a point mass, as changing the bank of the track moves the axis of rotation and can burn off rotational inertia to friction. Long straight sections would build up too much speed and bearings fly off on the turns, but balls taking sharp turns at slow speed will lose too much momentum and stop. I settled on setting a minimum turn radius for the track and banking much more aggressively than is technically necessary for any given speed, so it constantly snakes back and forth to burn off speed. 

One of the most elegant designs of the whole structure is how the lift acts like a ball screw. The the screw is constrained by the balls on all sides which allows it to run with no bearing at the top. This also leads to a failure mode where if the screw ever only has balls on one side it will immediately start wobbling badly enough that all the balls currently rolling will fall off the tracks. 

## Supports

The support generation was surprisingly simple. Iterating from the top down and treating the support pillars as a particle system is quite robust. I spent more time tweaking the geometry for aesthetics than I did for actual structure and collision issues, although I did heavily lean on the overhang tolerance of the printer.

Each support:
 - Pulls towards other supports, weighted by distance and similarity in size
 - Repels away from other supports
 - Pulls to stay in the bounding box
 - Pulls towards a fixed radius from the center of the structure  

Supports also have inertia, which is where the arcs in the structure of the support columns come from. 

## Looking forward

The final models take around 5-20 minutes to export. There's a ton I could do to optimize the models, but at this point the geometry is simply beyond the scope of OpenSCAD. If I was rewriting this I would probably use a different tool more optimized for this type of organic geometry, likely an SDF library. I have vague ambitions to do a big rewrite eventually but figured sharing [janky code](https://github.com/Will-Morr/MarbleFountain) is better than none. I started this just planning for the janky splines as a weekend project but it has gotten thoroughly out of hand. 

I have a ton of other ideas to play with if I do that big rewrite. There is no realistic velocity estimation at any point in the whole system right now, just a pile of heuristics. I was originally trying to not overcomplicate but building a proper acceleration model by measuring velocity with a camera would have almost definitely saved time overall. Trying to maintain a fixed slope makes collision prevention much harder but is required to keep speed within bounds. At this point I'm also just curious about the response curve, there's a knee somewhere where the surfaces start to slip that I want to track down.

## Looking back 

This was the most work I have ever put into a hobby project. I started in February 2024 and worked on it on and off until September. I applied to show it in a gallery (shoutout to [New Alliance Gallery](https://www.newalliancegallery.com/) in Somerville) with two months of warning, which wound up leading to a large crunch trying to make the system reliable enough to show in person in the weeks before the show. I was able to get it working consistently, although it did lose 2-3 balls an hour and could only run for a few hours without the motor overheating. I got pretty burned out and dropped the project, which is why I shelved it for a full year before sharing anything. 

Finally, a huge thanks to my friend Alex who listened to me ramble about marbles for several months every day while walking home from work, gave a ton of helpful input, and lived with the dozens of ball bearings scattered across our apartment. 

{{< figure
    src="/img/MarbleFountain/gallery.jpg"
    class="center" 
    alt="Marble Fountain installed in New Alliance Gallery"
    style="border-radius: 8px; width:75%"
>}}

[GitHub Repo](https://github.com/Will-Morr/MarbleFountain)
