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

One method of multiplying vectors together, called the dot product, is also called the inner product or the scalar product.  Scalar product is a sensible name for the dot product, because the operation results in a scalar value. Generically, the dot product is the sum of the vector components of $\vec a$ multiplied by their corresponding components in $\vec b$, or:

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
