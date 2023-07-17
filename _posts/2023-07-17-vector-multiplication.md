---
title: "Using Vector Multiplication"
date: 2023-07-17
categories: [Math, Linear Algebra]
tags: [3D Math, C++]
pin: false
math: true
image:
  path: /assets/img/posts/2023-06-03/code.jpg
---

The dot and cross products are used all over the place in 3D graphics and games more broadly. Even without visualization, we can "test" some of these applications with the bare-bones `Vec3` assembled over the last few posts.

## Is the NPC Facing the Player?
Say you're controlling a character in 2D space, defined by its position, $playerPos$. Then we'll insert a non-player character into the same game world, defined by its position and the direction it's facing. Let's call those $npcPos$ and $npcFwd$ respectively. Using the angle relationship of two vectors the dot product offers, can we check whether the NPC can see the player?

One quick method would be to look at the dot product of $npcFwd$ and the vector we'd get from $(playerPos - npcPos)$. If the dot of these two vectors is positive, then the player is in front of the NPC and the NPC can see the player. The first trick here is the subtraction - remember that a vector can be thought of as the offset between two points. As a result, $(playerPos - npcPos)$ gives you the vector going from the NPC to the player. Let's call that new vector $npcToPlayer$.

The second trick is ignoring the precise result of the dot product. It might be reasonable to assume $npcFwd$ is a unit vector, but $npcToPlayer$ is almost certainly not. You could normalize $npcToPlayer$, but you don't need to in this case. If the dot product $npcToPlayer$ and $npcFwd$ is positive at all, then the vectors have an acute angle between them. If that same dot product is negative, the vectors will have an obtuse angle between them. This means if $npcFwd \cdot npcToPlayer$ is positive, the player is in front of the NPC. If it's negative, the player is behind the NPC.

The following unit tests are far from robust, but they're easy to draw by hand if you're so inclined. I opted for easy-to-visualize examples in this text-only environment:

```cpp
TEST_CASE("Am I in front or behind the NPC?", "[vectors][exercises]") {
    // The NPC is at the origin, looking "up" along the Y axis
    Vec3 const npc_pos = Vec3::zero;
    Vec3 const npc_fwd = Vec3::unit_y;

    // The player is up and to the right from the NPC, which means they're in
    // front of the NPC
    Vec3 player_pos(1, 1, 0);
    Vec3 npc_to_player = player_pos - npc_pos;
    REQUIRE(dot(npc_fwd, npc_to_player) > 0.0f);

    // Now move the player to be directly behind the NPC
    player_pos = -Vec3::unit_y;
    npc_to_player = player_pos - npc_pos;
    REQUIRE(dot(npc_fwd, npc_to_player) < 0.0f);

    // Finally try putting the player directly right or left of the NPC, which
    // will result in a dot of zero for both positions
    player_pos = Vec3::unit_x;
    npc_to_player = player_pos - npc_pos;
    REQUIRE_THAT(dot(npc_fwd, npc_to_player), WithinAbs(0.0f, epsilon));

    player_pos = -Vec3::unit_x;
    npc_to_player = player_pos - npc_pos;
    REQUIRE_THAT(dot(npc_fwd, npc_to_player), WithinAbs(0.0f, epsilon));
}
```

## NPC Vision Cone
Let's improve on our "game" from above. Human vision doesn't afford a full 180° field of view without moving the eyes, right? Humans aren't two-dimensional creatures either, but let's run with what we've got. If we restrict the NPC's field of view, we now have two angles to compare. Let's call the angle between $npcFwd$ and $npcToPlayer$ *theta*, or $\theta$. We'll call the angle of the NPC's FOV *phi*, or $\phi$.

By comparing $\theta$ to _half_ of the NPC's FOV, $\phi / 2$, we will know if the NPC can see the player. Specifically, the player is within the NPC's vision cone if:

$$\theta \ge \phi / 2$$

Why do we use half the NPC's field of view in this comparison? The answer is related to something you might've noticed in the unit tests above: when the the player is directly left or right of the NPC, the dot product of $npcFwd$  and $npcToPlayer$ is zero in both cases. This checks out given the earlier assertion that perpendicular vectors will always dot to zero, but it still means the distinction between "left" or "right" is lost when doing this comparison.  Let's look at some concrete examples to make sense of this behavior. I've chosen to give our NPC a [generous field of view](https://en.wikipedia.org/wiki/Vision_span) of 150° in the following tests. I also added a tiny FOV of 20° for checking obvious extremes.

You may notice that I'm not testing the the exact same expression I outlined above. Instead of comparing the angles directly, I've chosen to compare their cosines. As a result, the player is within the NPC's vision cone if:

$$cos(\theta) \ge cos(\phi / 2)$$

The dot product of two **normalized** vectors is the cosine of the angle between them. Since I'm already working with one cosine, grabbing the cosine of half the NPC's FOV once and comparing that with the dot of our two vectors results in fewer trig operations. While trig functions can be slow, it [definitely depends](https://www.reddit.com/r/explainlikeimfive/comments/mm7qjx/eli5_how_does_a_computer_calculate_sincostan/) on your target platform whether or not this is a meaningful optimization. In this case, I'm just doing it for simplicity.

Now, those tests:

```cpp
TEST_CASE("NPC vision cone", "[vectors][exercises]") {
    // The NPC is at the origin, looking "up" along the Y axis, and has a
    // 150-degree horizontal field of view
    Vec3 const npc_pos = Vec3::zero;
    Vec3 npc_fwd = Vec3::unit_y;
    float const cos_half_fov = std::cos(radians(150.0f / 2.0f));

    // If the FOV is super narrow, then it's easier to hide from the NPC
    float const cos_tiny_fov = std::cos(radians(10.0f / 2.0f));

    // The player is up and to the right from the NPC, within the normal FOV
    Vec3 player_pos(1, 1, 0);
    Vec3 npc_to_player = normalize(player_pos - npc_pos);
    REQUIRE(dot(npc_fwd, npc_to_player) >= cos_half_fov);
    REQUIRE(dot(npc_fwd, npc_to_player) < cos_tiny_fov);

    // Move the player to the left of the NPC, but keep them within the NPC's
    // normal vision cone
    player_pos = { -1, 1, 0 };
    npc_to_player = normalize(player_pos - npc_pos);
    REQUIRE(dot(npc_fwd, npc_to_player) >= cos_half_fov);
    REQUIRE(dot(npc_fwd, npc_to_player) < cos_tiny_fov);

    // If the player is directly to the left or right of the NPC, the dot of
    // our two vectors will still yield zero, but falling outside of the NPC's
    // vision cone means this check will fail in the same way as the others
    player_pos = Vec3::unit_x;
    npc_to_player = normalize(player_pos - npc_pos);
    REQUIRE(dot(npc_fwd, npc_to_player) < cos_half_fov);

    player_pos = -Vec3::unit_x;
    npc_to_player = normalize(player_pos - npc_pos);
    REQUIRE(dot(npc_fwd, npc_to_player) < cos_half_fov);

    // The player somehow sneaks directly behind the NPC
    player_pos = -Vec3::unit_y;
    npc_to_player = normalize(player_pos - npc_pos);
    REQUIRE(dot(npc_fwd, npc_to_player) < cos_half_fov);

    // Flip the NPC's view direction around, and suddenly the player is smack
    // in the center of the NPC's view. The player is also now within the tiny
    // FOV.
    npc_fwd = -Vec3::unit_y;
    REQUIRE(dot(npc_fwd, npc_to_player) >= cos_half_fov);
    REQUIRE(dot(npc_fwd, npc_to_player) >= cos_tiny_fov);
}
```

In order to better explain why half of the NPC's FOV is used in this comparison, let's look at the numbers the first two unit tests are working with. In the both tests, the NPC is oriented in the same way:

$$\begin{equation}
\begin{aligned}
	npcPos &= (0,0) \\
	npcFwd &= [0,1] \\
	cosHalfPhi &\approx 0.259
\end{aligned}
\end{equation}$$

And in the first comparison:

$$playerPos = (1,1)$$

Our first calculation is the vector from the NPC to the player:

$$\begin{equation}
\begin{aligned}
	npcToPlayer &= playerPos - npcPos \\
	&= (1,1) - (0,0) \\
	&= [1,1]
\end{aligned}
\end{equation}$$

Don't forget to normalize the vectors before comparing angles!

$$normalize(npcToPlayer) \approx [0.707, 0.707]$$

And if we work our way through the dot product:

$$\begin{equation}
\begin{aligned}
	dot(npcFwd, npcToPlayer) &= (npcFwd_x \cdot npcToPlayer_x) + (npcFwd_y \cdot npcToPlayer_y) \\
	&\approx (0 \cdot 0.707) + (1 \cdot 0.707) \\
	&\approx 0.707
\end{aligned}
\end{equation}$$

The player is in the NPC's vision cone as expected, since $0.707 \ge 0.259$. The next test sees the player move left across the NPC's field of view, from $(1,1)$ to $(-1,1)$. Since the NPC is still at $(0,0)$,  $npcToPlayer$ is still equivalent to $playerPos$:

$$\begin{equation}
\begin{aligned}
	npcToPlayer &= playerPos - npcPos \\
	&= (-1,1) - (0,0) \\
	&= [-1,1]
\end{aligned}
\end{equation}$$

And normalizing it yields roughly $[-0.707, 0.707]$. If we revisit the dot product, we find:

$$\begin{equation}
\begin{aligned}
	dot(npcFwd, npcToPlayer) &= (npcFwd_x \cdot npcToPlayer_x) + (npcFwd_y \cdot npcToPlayer_y) \\
	&\approx (0 \cdot -0.707) + (1 \cdot 0.707) \\
	&\approx 0.707
\end{aligned}
\end{equation}$$

As you can see, because the $x$ component of $npcFwd$ remains zero, it doesn't matter where $playerPos$ is on the $x$ axis. This is where the loss of "left" and "right" comes from, and why we're only testing against half of the NPC's FOV. It's a bit of a strange artifact, but it makes more sense when I see the arithmetic in front of me.

## Calculating a Surface Normal
When you're doing things like collision detection or light calculations, you regularly work with a vector that's perpendicular to the surface you're concerned with. That vector is called a *normal*. Finding a normal for a given surface is easy if you can define the surface as a collection of three points. Almost everything in computer graphics is built with triangles, so there will be many such surfaces defined as a matter of course.

From these three points, you can get two vectors that themselves must be on the surface we're looking at. Taking the cross product of these two vectors yields the surface normal. Let's look at it in code.

```cpp
TEST_CASE("Finding the normal to a surface", "[vectors][exercises]") {
    // Three points which can define the XY plane
    Vec3 a(1.0f, 1.0f, 0.0f);
    Vec3 b(-3.0f, 6.0f, 0.0f);
    Vec3 c(0.0f, -2.0f, 0.0f);

    // And the normal to that plane is the Z unit vector
    Vec3 normal = normalize(cross(b - a, c - a));
    REQUIRE(normal == Vec3::unit_z);

    // Define the XZ plane with similar points
    a = { 1.0f, 0.0f, 1.0f };
    b = { 0.0f, 0.0f, -1.0f };
    c = { -3.0f, 0.0f, 6.0f };

    // And the normal to that plane is the Y unit vector
    normal = normalize(cross(b - a, c - a));
    REQUIRE(normal == Vec3::unit_y);

    // Finally, the YZ plane
    a = { 0.0f, 1.0f, 1.0f };
    b = { 0.0f, -3.0f, 6.0f };
    c = { 0.0f, 0.0f, -1.0f };

    // And the normal to that plane is the X unit vector
    normal = normalize(cross(b - a, c - a));
    REQUIRE(normal == Vec3::unit_x);
}
```

The points I've chosen are somewhat random, but their order is not. Like I mentioned in my [first post on it](../cross-product), the cross product isn't commutative, so the order of the operands matters.

My first trio of points was $(1, 1, 0)$, $(-3, 6, 0)$, and $(0, -2, 0)$ and the resulting normal vector was $[0,0,1]$, which is the same as the unit vector along the $z$ axis. If you draw those points on a sheet of paper, the normal vector points straight out of the paper toward you, just like the $z$ axis would. If I'd swapped the values for $b$ and $c$ however, the normal would've been $[0,0,-1]$, pointing in the opposite direction.

This is an example of something called "triangle winding" in computer graphics. Because the cross product cares about the order of its operands, and we're deriving those operands from the points of a triangle, the order we specify the points in matters too. Triangle winding is said to be done in a clockwise or counter-clockwise fashion. If you visualize the points from each example above, you can see that I'm winding my triangles counter-clockwise.

Much like whether your coordinate space is left or right handed, there is no standard direction to wind a triangle in graphics. Different tools and resources will follow different conventions and accounting for that during debugging and design is just part of process of drawing pictures on a screen.

## Conclusion
While the examples here are arguably not suited to be used as unit tests, I hope they offer some glimpse of what's possible with something as deceptively simple as a vector. It's strange to say that a handful of numbers captures the whole concepts of direction and magnitude simultaneously, but human beings are endlessly clever creatures. Seeing things like this put to use awes me time and again.