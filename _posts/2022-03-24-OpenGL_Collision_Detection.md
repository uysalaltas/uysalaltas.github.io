---
layout: post
title: "Triangle Based Collision Detection With OpenGL"
subtitle: "This post trying to explain different approach of collision detection using OpenGL Renderin API"
date: 2022-03-24 00:00:00 -0400
background: '/img/posts/OpenGL_Imgui/pexels1.jpg'
---

In general, bounding box techniques such as AABB are used to provide collision detection in 3D environments. Although the implementation and calculation time of these algorithms are fast, their sensitivity is not sufficient in applications where each triangle is important. If you want to cover details about you should visit <a href="https://developer.mozilla.org/en-US/docs/Games/Techniques/3D_collision_detection">this article</a>. This article will show a simple implementation on OpenGL based on the intersection between triangle and point.

## Triangle Point Intersection
Triangles are basic shapes of 3D meshes and they are represented by three coplanar planes. For checking intersection we need to know position of 3 edges of triangles. We need some geometrical approach for checking intersection between triangle and point. If we create pyramid from point and triangle, we can check is pyramid faces are flat. Whether the surfaces between the pyramid and the point are parallel to each other tells us whether the point is inside the triangle. If all three surfaces of the pyramid are parallel to each other, the point is inside the triangle, otherwise the point is outside the triangle. 

Lets say we have triagle named ABC and point P. When we create pyramid we will have PAB, PBC and PCA faces. For checking this faces direction, we need cross and dot products. Cross product gives us the normal of a triangle, and dot product gives us normals pointing direction. If those directions pointing same direction, point P is inside ABC triangle.

<img class="img-fluid" src="/img/posts/OpenGL_Collision_Detection/Frame 1.png" alt="Triangle to Pyramid">



I hope it helps. If you have any issues or question dont hesitate to contact with me.
