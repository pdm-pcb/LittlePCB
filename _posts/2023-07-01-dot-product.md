---
title: "The Dot Product"
date: 2023-07-01
categories: [Math, Linear Algebra]
tags: [3D Math, C++]
pin: false
math: true
image:
  path: /assets/img/posts/2023-06-03/code.jpg
---

In the [last post](../vectors), we covered how vectors might be added or subtracted from one another, as well as how scalar multiplication (and division) works. However, we didn't touch on what happens when you want to multiply one vector by another. Since multiplication is just repeated addition, I would've expected being able to be able to multiply and divide vectors by other vectors, but it turns out there's more to it. There are actually two ways to multiply vectors: the dot product and the cross product. We'll cover the dot product today.

## Defining the Dot Product
One method of multiplying vectors, the dot product, is also called the inner product and the scalar product.  Calling it a scalar product makes some sense, because the dot product itself results in a scalar value. Generically, the dot product is the sum of the vector components of $\vec a$ multiplied by their corresponding components in $\vec b$, or:

$$\vec a \cdot \vec b = \sum_{i=1}^n a_i b_i$$

I'll implement `dot()` for a 3D vector as follows:

```cpp
float dot(Vec3 const &a, Vec3 const &b) {
    return (a.x * b.x) + (a.y * b.y) + (a.z * b.z);
}
```

You might notice that this is very similar to a function from last time, `length2()`, and you'd be right. If you dot a vector with itself, the resulting scalar is the vector's length, squared.

Another way to define the dot product is as the length of both vectors multiplied by the cosine of the angle between the vectors. If we define the angle between the vectors as *theta*, or $\theta$, then the dot product would be:

$$\vec a \cdot \vec b = ||\vec a|| \ ||\vec b|| \ cos(\theta)$$

Getting the angle between two vectors (or the cosine of it, anyway) for three multiplications and two additions is a pretty efficient way to start measuring stuff. And if your vectors are normalized before you dot them together, $\vec a$ and $\vec b$ both have lengths of $1$, so you get access to $cos(\theta)$ directly. It's easy to forget to normalize a vector during calculation, and sometimes the error won't show up till much later. These kinds of mistakes can be a pain to track down, so be careful and keep normalizing top of mind while debugging weird behavior.

Even without normalized vectors, there are three results you can keep an eye out for when dealing with the dot product:

- $\vec a \cdot \vec b \gt 0$ when the two vectors point in mostly the same direction
- $\vec a \cdot \vec b \lt 0$ when the two vectors point in mostly different directions
- $\vec a \cdot \vec b = 0$ when the two vectors are perpendicular to one another

Speaking of angles, there's one more relationship between vectors a dot product can give you. The scalar value you get from $\vec a \cdot \vec b$ is the signed length of the projection of $\vec b$ onto $\vec a$, times the length of $\vec a$. What's a projection? Think of it as a shadow. Vectors don't have position, meaning we can line them up so one could cast a shadow on another.

This analogy breaks down as soon as the light source moves, but let's suppose the light is perfectly fixed above our vectors. If the vectors are perpendicular to one another, they'll form a right angle. Suppose $\vec a$ is pointing to the right, $[1,0]$, while $\vec b$ is pointing straight up, $[0,1]$. The "shadow" cast by $\vec b$ onto $\vec a$ in our imaginary universe would have a length of zero, because $\vec b$ is pointing straight up at the noon-day sun. And as noted in the third bullet point just above, dotting two perpendicular vectors does indeed give zero as the product.

Likewise, if $\vec b$ is pointing in the exact same direction as $\vec a$, they're going to be parallel, and you'd expect $\vec b$'s shadow to completely cover $\vec a$. Since vectors of the same length and direction are equivalent, this situation adds a bit of visual "evidence" to the observation above that dotting a vector with itself will yield its length squared.

Now, the idea of a negative shadow is kind of ridiculous, but that's what I was referring to when I said the dot is the "signed" length of a particular projection. This is all getting a bit strained though, so let's just start writing some tests to ensure our new function does what it should.

## Basic Unit Tests
The first unit test will use some fixed, easily verified values:

```cpp
TEST_CASE("Basic functionality", "[vectors][dot product]") {
    Vec3 a(4, -2, 3);
    Vec3 b(-5, 9, -1);
    float a_dot_b = -41.0f;
    REQUIRE_THAT(dot(a, b), WithinAbs(a_dot_b, epsilon));

    a = { 1, -5, 0 };
    b = { -2, 1, -1 };
    a_dot_b = -7.0f;
    REQUIRE_THAT(dot(a, b), WithinAbs(a_dot_b, epsilon));

    a = { 4, 0, -3 };
    b = { -1, 6, -2 };
    a_dot_b = 2.0f;
    REQUIRE_THAT(dot(a, b), WithinAbs(a_dot_b, epsilon));
}
```

Now let's test some attributes of the dot product. The operation is commutative, meaning:

$$\vec a  \cdot \vec b = \vec b \cdot \vec a$$

My tests for that look like this:

```cpp
TEST_CASE("Dot product is commutative", "[vectors][dot product]") {
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();

        REQUIRE_THAT(dot(a, b), WithinAbs(dot(b, a), epsilon));
    }
}
```

Next, the dot product will "distribute" over vector addition and subtraction. That means:

$$\vec a \cdot (\vec b + \vec c) = (\vec a \cdot \vec b) + (\vec a \cdot \vec c)$$

And to test it:

```cpp
TEST_CASE("Dot product distributes over vector addition and subtraction",
          "[vectors][dot product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();
        Vec3 const c = random_vec3();

        REQUIRE_THAT(dot(a + b, c), WithinAbs(dot(c, a) + dot(c, b), epsilon));
        REQUIRE_THAT(dot(a, b + c), WithinAbs(dot(a, b) + dot(a, c), epsilon));
        REQUIRE_THAT(dot(a - b, c), WithinAbs(dot(c, a) - dot(c, b), epsilon));
        REQUIRE_THAT(dot(a, b - c), WithinAbs(dot(a, b) - dot(a, c), epsilon));
    }
}
```

And since the result of a dot product is just a scalar, it will be "associative" for scalar multiplication of vectors as well. Strictly speaking, for some scalar $x$:

$$x  (\vec a \cdot \vec b) = (x \vec a) \cdot \vec b = \vec a \cdot (x \vec b)$$

Which I'm testing with the following:

```cpp
TEST_CASE("Dot product is associative with scalar multiplication",
          "[vectors][dot product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();

        float const x = Catch::Generators::random(-1.0f, 1.0f).get();
        REQUIRE_THAT(x * dot(a, b), WithinAbs(dot(x * a, b), epsilon));
        REQUIRE_THAT(x * dot(a, b), WithinAbs(dot(a, x * b), epsilon));
    }
}
```

## Comparing Vectors
Let's restate some uses for the dot product from above, but write tests for those uses this time. Dotting any vector with itself yields its own length squared. Similarly, the dot product of a normalized vector with itself is always $1$. And, if you dot the zero vector with itself, the result will always be zero.

```cpp
TEST_CASE("Dotting a vector with itself", "[vectors][dot product]") {
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 a = random_vec3();

        // The dot product of a nonzero vector with itself must be positive
        REQUIRE(dot(a, a) > 0.0f);

        // It should also be the vector's length squared
        REQUIRE_THAT(dot(a, a), WithinAbs(length2(a), epsilon));

        // Normalized vectors dotted with themselves always yield 1
        a = normalize(a);
        REQUIRE_THAT(dot(a, a), WithinAbs(1.0f, epsilon));
    }

    // But dotting the zero vector with itself will yield zero
    REQUIRE_THAT(dot(Vec3::zero, Vec3::zero), WithinAbs(0.0f, epsilon));
}
```

That test covers one of three result types when comparing vectors. How about revisiting the nonsensical "negative shadow" from earlier? For example, when $\vec a = [1,1]$ and $\vec b = [-1,-1]$, the two vectors are pointing perfectly away from each other. In that case, the dot product will still be be the vectors' lengths squared, but negative.

```cpp
TEST_CASE("Dotting a vector with its inverse", "[vectors][dot product]") {
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 a = random_vec3();

        // Dotting a vector with its inverse will yield the length squared, but
        // negative
        REQUIRE_THAT(dot(a, -a), WithinAbs(-length2(a), epsilon));

        // Likewise, a normalized vector dotted with its inverse yields -1
        a = normalize(a);
        REQUIRE_THAT(dot(a, -a), WithinAbs(-1.0f, epsilon));
    }
}
```

The last test case for this section will check that perpendicular vectors dot together for a product of zero. Fortunately, there are three vectors that must be perpendicular already defined.

```cpp
TEST_CASE("Unit vectors are perpendicular", "[vectors][dot product]") {
    REQUIRE_THAT(dot(Vec3::unit_x, Vec3::unit_y), WithinAbs(0.0f, epsilon));
    REQUIRE_THAT(dot(Vec3::unit_x, Vec3::unit_z), WithinAbs(0.0f, epsilon));
    REQUIRE_THAT(dot(Vec3::unit_y, Vec3::unit_z), WithinAbs(0.0f, epsilon));
}
```

Not too shabby! Let's put these traits to some use.

## Exercise: Is the NPC Facing the Player?
Using the angle relationship the dot product gives us, let's try something fun. Say you're controlling a character in 2D space, defined by its position, $playerPos$. Then we'll insert a non-player character into the same game world, defined by its position and the direction it's facing. Let's call those $npcPos$ and $npcFwd$ respectively. Can we check whether the NPC can see the player?

One quick method would be to look at the dot product of $npcFwd$ and the vector we'd get from $(playerPos - npcPos)$. If the dot of these two vectors is positive, then the player is in front of the NPC and the NPC can see the player. The first trick here is the subtraction - remember that a vector can be thought of as the offset between two points. As a result, $(playerPos - npcPos)$ gives you the vector going from the NPC to the player. Let's call that new vector $npcToPlayer$.

The second trick is ignoring the precise result of the dot product. It might be reasonable to assume $npcFwd$ is a unit vector, but $npcToPlayer$ is almost certainly not. You could normalize $npcToPlayer$, but you don't need to in this case. If the dot product $npcToPlayer$ and $npcFwd$ is positive at all, then the vectors have an acute angle between them. If that same dot product is negative, the vectors will have an obtuse angle between them. This means if $npcFwd \cdot npcToPlayer$ is positive, the player is in front of the NPC. If it's negative, the player is behind the NPC.

The following unit tests are far from robust, but they're easy to draw if you're so inclined. I opted for easy-to-visualize examples in this text-only environment:

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

## Using the Dot Product: NPC Vision Cone
Let's improve our "game". Human vision doesn't afford a full 180° field of view without moving the eyes, right? Humans aren't two-dimensional creatures either, but let's run with what we've got. If we restrict the NPC's field of view, we now have two angles to compare. Let's call the angle between $npcFwd$ and $npcToPlayer$ *theta*, or $\theta$. We'll call the angle of the NPC's FOV *phi*, or $\phi$.

By comparing $\theta$ to _half_ of the NPC's FOV, $\phi / 2$, we will know if the NPC can see the player. Specifically, the player is within the NPC's vision cone if:

$$\theta \ge \phi / 2$$

Why do we use half the NPC's field of view in this comparison? The answer is related to something you might've noticed in the unit tests above: when the the player is directly left or right of the NPC, the dot product of $npcFwd$  and $npcToPlayer$ is zero in both cases. This checks out given the earlier assertion that perpendicular vectors will always dot to zero, but it still means the distinction between "left" or "right" is lost when doing this comparison.  Let's look at some concrete examples to make sense of this behavior. I've chosen to give our NPC a [generous field of view](https://en.wikipedia.org/wiki/Vision_span) of 150° in the following tests. I also added a tiny FOV of 20° for checking obvious extremes.

You may notice that I'm not testing the the exact same expression I outlined above. Instead of comparing the angles directly, I've chosen to compare their cosines. As a result, the player is within the NPC's vision cone if:

$$cos(\theta) \ge cos(\phi / 2)$$

The dot product of two _normalized_ vectors is the cosine of the angle between them. Since I'm already working with one cosine, grabbing the cosine of half the NPC's FOV once and comparing that with the dot of our two vectors results in fewer trig operations. While trig functions can be slow, it [definitely depends](https://www.reddit.com/r/explainlikeimfive/comments/mm7qjx/eli5_how_does_a_computer_calculate_sincostan/) on your target platform whether or not this is a meaningful optimization. In this case, I'm just doing it for simplicity.

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

## Conclusion
Whew! The dot product is useful all over the place in graphics, so look forward to seeing it again. Next time, we'll explore the other vector multiplication: the cross product.