---
title: "The Cross Product"
date: 2023-07-15
categories: [Math, Linear Algebra]
tags: [3D Math, C++]
pin: false
math: true
image:
  path: /assets/img/posts/2023-06-03/code.jpg
---

Another way to multiply two vectors is called the cross product. Other names for it are the outer product and the vector product. Given that the dot product produces a scalar value and is also known as the scalar product, you'd be right if you guessed the cross product results in a  whole new vector.

The cross product also differs from the dot product in that it's only defined for three dimensional vectors. Here's how I've implemented it in code:

```cpp
Vec3 cross(Vec3 const &a, Vec3 const &b) {
    return Vec3 {
        (a.y * b.z) - (a.z * b.y),
        (a.z * b.x) - (a.x * b.z),
        (a.x * b.y) - (a.y * b.x),
    };
}
```

As you can see, it "crosses" all over the place, multiplying different components. The result of this arithmetic is a new vector that's perpendicular to the original two. If a dot product can be seen as a measure of how similar two vectors are, then a cross product can similarly be said to measure how different two vectors are.

Remember how the dot product of two vectors relates to the cosine of the angle between those vectors? There is a relationship between the cross product of two vectors and the sine of the angle between them. Specifically, the length of the vector you get from crossing $\vec a$ and $\vec b$ is equal to the length of $\vec a$ and $\vec b$ times the sine of the angle between them. Written as an equation:

$$||\vec a \times \vec b|| = ||\vec a||\ ||\vec b||\ sin(\theta)$$

The fact that dot relates to cosine and cross relates to sine helps emphasize how their uses are opposed. Calling them opposites would be silly, but they both explore how a pair of vectors relate to one another and whether you're looking for a metric of similarity or difference influences which product is helpful to you.

## Basic Unit Tests
I'll test with some fixed values first:

```cpp
TEST_CASE("Basic functionality", "[vectors][cross product]") {
    Vec3 a(-2, 10, -6);
    Vec3 b(8, -1, 2);
    Vec3 a_cross_b(14, -44, -78);

    REQUIRE(cross(a, b) == a_cross_b);

    a = { 7, 1, 0 };
    b = { 4, 8, -10 };
    a_cross_b = { -10, 70, 52 };

    REQUIRE(cross(a, b) == a_cross_b);

    a = { 1, 8, 5 };
    b = { 3, -7, 0 };
    a_cross_b = { 35, 15, -31 };

    REQUIRE(cross(a, b) == a_cross_b);
}
```

Even though the dot and cross products can both be used to examine how vectors relate, some of their properties are quite different. For starters, the cross product is anti-commutative, which means:

$$(\vec a \times \vec b) = -(\vec b \times \vec a)$$

I'll test that with a few random vectors:

```cpp
TEST_CASE("Cross product is anti-commutative", "[vectors][cross product]") {
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();
        Vec3 const a_cross_b = cross(a, b);

        REQUIRE(a_cross_b == -cross(b, a));
    }
}
```

Crossing any two negated vectors will give you the same result as crossing the original two vectors. More formally:

$$\vec a \times \vec b = (-\vec a) \times (-\vec b)$$

I'll write another loop to test this:

```cpp
TEST_CASE("Crossing negative vectors yields the same result",
          "[vectors][cross product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();
        Vec3 const a_cross_b = cross(a, b);

        REQUIRE(a_cross_b == cross(-a, -b));
    }
}
```

If you cross any vector with itself, the result will always be the zero vector, so:

$$\vec a \times \vec a = 0$$

Following the form of the last few tests:

```cpp
TEST_CASE("Crossing a vector with itself yields the zero vector",
          "[vectors][cross product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();

        REQUIRE(cross(a, a) == Vec3::zero);
    }
}
```

The last two properties of a cross product we'll test are shared between the two types of vector multiplication. Like the dot product, the cross product distributes over addition and is commutative with scaling. That is:

$$\vec a \times (\vec b + \vec c) = (\vec a \times \vec b) + (\vec a \times \vec c)$$

Which I'll test like this:

```cpp
TEST_CASE("Cross product distributes over vector addition and subtraction",
          "[vectors][cross product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();
        Vec3 const c = random_vec3();

        REQUIRE(cross(a + b, c) == cross(a, c) + cross(b, c));
        REQUIRE(cross(a, b + c) == cross(a, b) + cross(a, c));
        REQUIRE(cross(a - b, c) == cross(a, c) - cross(b, c));
        REQUIRE(cross(a, b - c) == cross(a, b) - cross(a, c));
    }
}
```

And, for some scalar $x$:

$$x (\vec a \times \vec b) = (x \vec a) \times \vec b = \vec a \times (x \vec b) $$


Which can be tested this way:

```cpp
TEST_CASE("Cross product is associative with scalar multiplication",
          "[vectors][cross product]")
{
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 const a = random_vec3();
        Vec3 const b = random_vec3();

        float const x = Catch::Generators::random(-1.0f, 1.0f).get();
        REQUIRE(x * cross(a, b) == cross(x * a, b));
        REQUIRE(x * cross(a, b) == cross(a, x * b));
    }
}
```

What happens when there _is no_ perpendicular vector relative to the two vectors being fed into the cross product? For example, what happens if you cross two parallel vectors? Or what if you cross some vector with the zero vector? In both of these cases, the cross product will give you back the zero vector. Let's test that:

```cpp
TEST_CASE("Crossing parallel vectors yields zero", "[vectors][cross product]") {
    for(uint32_t i = 0u; i < TEST_REPEATS; ++i) {
        Vec3 a = random_vec3();

        // Crossing a vector with itself or its inverse yields the zero vector
        REQUIRE(cross(a, a) == Vec3::zero);
        REQUIRE(cross(a, -a) == Vec3::zero);
        REQUIRE(cross(-a, a) == Vec3::zero);

        // Crossing anything with the zero vector also yields the zero vector
        REQUIRE(cross(a, Vec3::zero) == Vec3::zero);
        REQUIRE(cross(Vec3::zero, a) == Vec3::zero);
    }
}
```

## Handedness
Finally, I'll talk briefly about something you might've noticed previously. I said a cross product produces a vector that's perpendicular to the original two, but doesn't that mean there are two possible answers?  Yep. Let's visualize this.

Hold up a fist, then extend your thumb, index, and middle fingers at right angles to one another. Provided you can, your hand will now be  making this neat, caltrop-like shape. Now imagine your thumb is the $x$ axis on a coordinate plane, while your index finger is the $y$ axis. If you orient your hand such that your thumb and index finger point in the positive direction of their respective axes, you'll notice something interesting.

You might've guessed that your middle finger will represent the $z$ axis, but if you chose to do this exercise with your right hand, your middle finger will be pointing toward you. If you used your left hand however, your middle finger will be pointing away from you. While the $x$ and $y$ axes are identical between both hands, the $z$ axes are opposite one another. Yet, both $z$ axes are perfectly perpendicular to the first two.

This is somewhat amusingly called handedness, and it can have a big effect on how your math turns out on screen. In graphics particularly, there is no standard between left- and right-handedness; you just have to be careful and keep track of which one you're working with. Much like normalizing your vectors before doing certain work with them, it's hard to overstate the importance of keeping handedness in mind while designing and debugging.

All that being said, we did just show that the cross product is anti-commutative. This means that even though any cross product can have two valid answers in terms of perpendicularity, we can control which one we get by controlling the order of the parameters. For example, since I have these as unit vectors in my code:

```cpp
Vec3 const Vec3::unit_x { 1.0f, 0.0f, 0.0f };
Vec3 const Vec3::unit_y { 0.0f, 1.0f, 0.0f };
Vec3 const Vec3::unit_z { 0.0f, 0.0f, 1.0f };
```

This test will pass:

```cpp
REQUIRE(cross(Vec3::unit_x, Vec3::unit_y) == Vec3::unit_z);
```

But this one will fail:

```cpp
REQUIRE(cross(Vec3::unit_y, Vec3::unit_x) == Vec3::unit_y);
```

As the second test's cross product results in a negative unit vector along the $z$ axis, or $[0,0,-1]$. So, here's is the test block I wrote for this behavior:

```cpp
TEST_CASE("Crossing unit vectors produces the a third unit vector",
          "[vectors][cross product]")
{
    REQUIRE(cross(Vec3::unit_x, Vec3::unit_y) == Vec3::unit_z);
    REQUIRE(cross(Vec3::unit_z, Vec3::unit_x) == Vec3::unit_y);
    REQUIRE(cross(Vec3::unit_y, Vec3::unit_z) == Vec3::unit_x);
}
```
