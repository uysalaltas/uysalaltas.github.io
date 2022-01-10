---
layout: post
title: "Render an OpenGL scene in ImGui window"
subtitle: "This post trying to explain how to render OpenGL window in ImGui viewport"
date: 2022-01-08 00:00:00 -0400
background: '/img/posts/01.jpg'
---

The easiest way to add a nice look or functionality to your application developed with OpenGL is to use a GUI library. In this way, you will have tools that can be used while developing applications such as buttons, text fields or color picker. In this article, I talked about how the window created with OpenGL is rendered within the ImGUI window and how the window is scaled after rendering.

## Creating a framebuffer
We need to create a copy of the scene created with OpenGL and keep this copy in a buffer. At this stage, the framebuffer comes to our aid. You can find a more detailed article about Framebuffer at [learnopengl.com]({% link https://learnopengl.com/Advanced-OpenGL/Framebuffers %}). After creating framebuffer we have to attach to a texture for using it. 

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

I like to keep things organized. So let's move all this in the FrameBuffer class and call it in the main loop. 

```c++
// FrameBuffer.h

#pragma once
#include <iostream>
#include <glad/glad.h>
#include <glm/glm.hpp>

class FrameBuffer
{
public:
	FrameBuffer(float width, float height);
	~FrameBuffer();
	unsigned int getFrameTexture();
	void RescaleFrameBuffer(float width, float height);
	void Bind() const;
	void Unbind() const;
private:
	unsigned int fbo;
	unsigned int texture;
	unsigned int rbo;
};
```
As you can see i have few extra function like ResaleFrameBuffer. We are gonna need that function later.

```c++
// FrameBuffer.cpp

#include "FrameBuffer.h"

FrameBuffer::FrameBuffer(float width, float height)
{
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	glGenTextures(1, &texture);
	glBindTexture(GL_TEXTURE_2D, texture);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);

	glGenRenderbuffers(1, &rbo);
	glBindRenderbuffer(GL_RENDERBUFFER, rbo);
	glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height);
	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);

	if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
		std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;

	glBindFramebuffer(GL_FRAMEBUFFER, 0);
	glBindTexture(GL_TEXTURE_2D, 0);
	glBindRenderbuffer(GL_RENDERBUFFER, 0);
}

FrameBuffer::~FrameBuffer()
{
	glDeleteFramebuffers(1, &fbo);
	glDeleteTextures(1, &texture);
	glDeleteRenderbuffers(1, &rbo);
}

unsigned int FrameBuffer::getFrameTexture()
{
	return texture;
}

void FrameBuffer::RescaleFrameBuffer(float width, float height)
{
	glBindTexture(GL_TEXTURE_2D, texture);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);

	glBindRenderbuffer(GL_RENDERBUFFER, rbo);
	glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height);
	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
}

void FrameBuffer::Bind() const
{
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);
}

void FrameBuffer::Unbind() const
{
	glBindFramebuffer(GL_FRAMEBUFFER, 0);
}
```

After that we can create object from FrameBuffer in our main cpp file. Framebuffer takes width and height of the screen as parameter.

```c++
// main.cpp

int main()
{
	...
	FrameBuffer sceneBuffer(gl_window.WIDTH, gl_window.HEIGHT);

	while (!glfwWindowShouldClose(gl_window.window))
	{
		sceneBuffer.Bind();

		...

		sceneBuffer.Unbind();
	}
}

return 0;
```

## Rendering On ImGUI Window
After handling all framebuffer stuff now we can transfer our image to imgui window. Assuming that the ImGui is currently working on the application, So i am only sharing the codes of the part where the image is transferred to the screen. If you dont know how to use ImGui with OpenGL, The Cherno has great tutorial about that. Definetely go [check that out]({% link https://www.youtube.com/watch?v=nVaQuNXueFw %}).

```c++

while (!glfwWindowShouldClose(gl_window.window))
{
	...

	ImGui::Begin("Scene");
	{
		ImGui::BeginChild("GameRender");
		
		float width = ImGui::GetContentRegionAvail().x;
		float height = ImGui::GetContentRegionAvail().y;
		
		*m_width = width;
		*m_height = height;
		ImGui::Image(
			(ImTextureID)sceneBuffer->getFrameTexture(), 
			ImGui::GetContentRegionAvail(), 
			ImVec2(0, 1), 
			ImVec2(1, 0)
		);
	}
	ImGui::EndChild();
	ImGui::End();

	...
}
```
## Window Scaling
Now our scene window is ready but there is small problem here. As you can notice when you scale your scene window, image might be little glitchy. The reason is, our image size is static but we can scale our window as long as the computer screen allows. To solve this problem, we can use the RescaleFrameBuffer function in our FrameBuffer object. We can use the Callback functions in the GLFW library to understand the change in screen size.

```c++
glfwSetWindowSizeCallback(window, window_size_callback_static);

int main
{
	while (!glfwWindowShouldClose(gl_window.window))
	{
		...
	}
}

void GLWindow::window_size_callback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
	sceneBuffer.RescaleFrameBuffer(width, height);
}

```

## Result

Here is the result! We have fully scalable window.

<div style="width:100%;height:0;padding-bottom:54%;position:relative;"><iframe src="https://giphy.com/embed/kigzokOzM3PDnFl6Y2" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/resizing-window-kigzokOzM3PDnFl6Y2">via GIPHY</a></p>

I hope it helps. If you have any issues or question dont hesitate to contact with me.
