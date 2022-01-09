---
layout: post
title: "Render an OpenGL scene in ImGui window"
subtitle: "This post trying to explain how to render OpenGL window in ImGui viewport"
date: 2022-01-08 00:00:00 -0400
background: '/img/posts/01.jpg'
---

The easiest way to add a nice look or functionality to your application developed with OpenGL is to use a GUI library. In this way, you will have tools that can be used while developing applications such as buttons, text fields or color picker. In this article, I talked about how the window created with OpenGL is rendered within the ImGUI window and how the window is scaled after rendering.

## Creating a framebuffer
We need to create a copy of the scene created with OpenGL and keep this copy in a buffer. At this stage, the framebuffer comes to our aid. You can find a more detailed article about Framebuffer at https://learnopengl.com/Advanced-OpenGL/Framebuffers. After creating framebuffer we have to attach to a texture for using it. 

Lets create our framebuffer and bind it. 

```c++
unsigned int fbo;

glGenFramebuffers(1, &fbo);
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
```

After creating and binding framebuffer, we can create texture and attach to the framebuffer now. 

```c++
unsigned int texture;

glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```

If we want to keep rendering data with performance, we need renderbuffer. Renderbuffer stores all the data without any conversions. Creating renderbuffer is same as other creations.


```c++
unsigned int rbo;

glGenRenderbuffers(1, &rbo);
glBindRenderbuffer(GL_RENDERBUFFER, rbo);
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

After all this buffer and texture creation lets check framebuffer. We dont want to render the wrong buffers so make sure to unbind the buffers.

```c++
if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
	std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
glBindFramebuffer(GL_FRAMEBUFFER, 0);
glBindTexture(GL_TEXTURE_2D, 0);
glBindRenderbuffer(GL_RENDERBUFFER, 0);
```
