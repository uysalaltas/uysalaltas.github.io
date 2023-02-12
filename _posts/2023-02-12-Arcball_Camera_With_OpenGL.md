---
layout: post
title: "Arcball Camera With OpenGL"
subtitle: "Calculation of Rotation, Panning and Zoomin of Arcball Camera"
date: 2023-02-12 00:00:00 -0400
background: '/img/posts/Arcball_Camera_With_OpenGL/Camera Pic.jpg'
---
<a href="https://www.pexels.com/photo/close-up-photo-of-camera-shutter-414781/">Photo by Pixabay</a>

An arcball camera, also known as a spherical camera, is a type of 3D camera control that allows the user to interactively manipulate the position and orientation of a virtual camera in 3D space. The arcball camera is often used in 3D graphics and game development, as it provides an intuitive and user-friendly way for the user to navigate and view a 3D scene. The user can rotate the camera around the scene, zoom in and out, and change the camera's perspective by rotating it in any direction. We frequently come across Arcball camera in slicing programs used for 3D modeling and animation, Virtual reality and augmented reality applications, Industrial design and product visualization and 3D printers. In general, we use it when the user wants to view the object from more than one angle.

There are various methods for calculating an arcball camera. The two methods that are most frequently used to calculate arcball cameras are quaternions and euler angles. Of course, each strategy has advantages and disadvantages. I am not going to deep into mathematics behind those approaches but simply Euler angles are a set of three angles, representing the rotation about the X, Y, and Z axes, respectively. In the context of an arcball camera, Euler angles can be used to describe the orientation of the camera by specifying the rotation about the X, Y, and Z axes of the camera's reference frame. Euler angles are easy to understand and simple to implement, but they have the drawback of being prone to gimbal lock, which is when two or more of the angles become degenerate and the orientation becomes ambiguous. Quaternions, on the other hand, are a mathematical representation of rotations that can be used to describe the orientation of the camera. Unlike Euler angles, quaternions do not suffer from gimbal lock and can represent any orientation in three-dimensional space. In the context of an arcball camera, quaternions can be used to describe the orientation of the camera by specifying the axis of rotation and the angle of rotation. Quaternions are more complex to understand and implement than Euler angles, but they provide a more robust representation of rotations.  

We are gonna use euler angles in this article.
 
## Calculation of Arcball camera
Fist we need to get differences between mouse positions when we press a mouse click. Doing something like this is easy with the code that down below.

```c++
if (Input::IsMouseButtonPressed(TEA_MOUSE_LEFT))
{
    if (firstMouseClick)
    {
        lastMousePosRightClick.x = Input::GetMouseX();
        lastMousePosRightClick.y = Input::GetMouseY();
        firstMouseClick = false;
    }
    ArcBallCamera((lastMousePosRightClick.x - Input::GetMouseX()), (lastMousePosRightClick.y - Input::GetMouseY()));
    lastMousePosRightClick.x = Input::GetMouseX();
    lastMousePosRightClick.y = Input::GetMouseY();
}
else
{
    firstMouseClick = true;
}
```
At this stage, I use my own functions to obtain mouse clicks and positions. But you can simply use GLFW or different library events and functions. After calculating deltaX and deltaY mouse positions we can calculate arcball rotation now.

```c++
void ArcBallCamera(float deltaX, float deltaY)
{
    glm::vec4 position(GetEye().x, GetEye().y, GetEye().z, 1);
    glm::vec4 pivot(GetLookAt().x, GetLookAt().y, GetLookAt().z, 1);
    float deltaAngleX = (2 * M_PI / *m_width);
    float deltaAngleY = (M_PI / *m_height);
    float xAngle = deltaX * deltaAngleX;
    float yAngle = deltaY * deltaAngleY;
    float cosAngle = glm::dot(GetViewDir(), m_upVector);
    if (cosAngle * glm::sign(yAngle) > 0.99f)
        yAngle = 0;
    glm::mat4x4 rotationMatrixX(1.0f);
    rotationMatrixX = glm::rotate(rotationMatrixX, xAngle, m_upVector);
    position = (rotationMatrixX * (position - pivot)) + pivot;
    glm::mat4x4 rotationMatrixY(1.0f);
    rotationMatrixY = glm::rotate(rotationMatrixY, yAngle, GetRightVector());
    glm::vec3 finalPosition = (rotationMatrixY * (position - pivot)) + pivot;
    SetCameraView(finalPosition, GetLookAt(), m_upVector);
}
```
The deltaX and deltaY values represent the change in mouse position. deltaAngleX and deltaAngleY are the angles of rotation corresponding to deltaX and deltaY, respectively, and they are calculated based on the width and height of the window. The "cosAngle" variable is the dot product of the view direction vector and the up vector, used to check if the camera is facing straight up or down, in which case the camera should not rotate vertically. This is a precaution for gimbal lock. The "rotationMatrixX" and "rotationMatrixY" matrices are created to store the rotation transformations. The "rotationMatrixX" is calculated as a rotation around the up vector with angle xAngle, while the "rotationMatrixY" is calculated as a rotation around the right vector with angle yAngle. The position of the camera is then transformed by the "rotationMatrixX" and "rotationMatrixY" matrices and the result is stored in the "finalPosition" variable. Finally, the SetCameraView function is called with the "finalPosition", the look-at position, and the up vector to update the camera view. As you see GLM functions used for all the calculations.

## Calculation of Camera Panning

<img class="img-fluid" src="/img/posts/Arcball_Camera_With_OpenGL/Camera Panning.gif" alt="Plane-Triangle Intersection">

In many sceniario, we may want to change our look up positions. In this point we can use camera panning. Camera panning must proportional with direction of the camera for more intuitive movement. Panning a camera involves changing both the position of the camera and the position that the camera is looking at. When you pan the camera, you're essentially shifting its position in the right and up directions. This means that the camera position changes, but the direction the camera is pointing (the "look-at" position) remains unchanged. As a result, you need to update both the camera position and the view matrix, which defines the camera's orientation and perspective in the 3D scene. Let's code now.

```c++
void CalculatePanCamera()
{
    if (Input::IsMouseButtonPressed(TEA_MOUSE_RIGHT))
    {
        if (firstRightMouseClick)
        {
            lastMousePosRightClick.x = Input::GetMouseX();
            lastMousePosRightClick.y = Input::GetMouseY();
            firstRightMouseClick = false;
        }
        PanCamera((Input::GetMouseX() - lastMousePosRightClick.x), (lastMousePosRightClick.y - Input::GetMouseY()));
        lastMousePosRightClick.x = Input::GetMouseX();
        lastMousePosRightClick.y = Input::GetMouseY();
    }
    else
    {
        firstRightMouseClick = true;
    }
}

void PanCamera(float deltaX, float deltaY)
{
    deltaX *= 0.05f;
    deltaY *= 0.05f;
    glm::vec3 cameraFront = glm::normalize(m_lookAt - m_eye);
    glm::vec3 right = glm::normalize(glm::cross(cameraFront, m_upVector));
    glm::vec3 up = glm::normalize(m_upVector);
    m_eye -= deltaX * right + deltaY * up;
    m_lookAt -= deltaX * right + deltaY * up
    UpdateViewMatrix();
}
```
Again we need to get differences between mouse positions like calculation of arcball camera. We multiply the delta values for reducing the sensitivity. cameraFront is the front direction of the camera. It is the direction that the camera is facing, and is typically calculated as the normalized difference between the camera position and the look-at position. After that, we have to calculate right and up vectors. The right vector is calculated as the cross product of the front and up vectors. The up vector is normalized for consistency. The camera position and look-at position are then updated by subtracting the xoffset times the right vector and the yoffset times the up vector. This maintains the same relative position between the camera and the look-at position, while shifting both positions in the right and up directions. Finally we can update the view matrix. 

Here is whole camera.h file. 

```c++
# define M_PI           3.14159265358979323846  /* pi */

class Camera
{
public:
    Camera(glm::vec3& eye, glm::vec3& lookat, glm::vec3& upVector, int* width, int* height)
        : m_eye(std::move(eye))
        , m_lookAt(std::move(lookat))
        , m_upVector(std::move(upVector))
        , m_width(width)
        , m_height(height)
    {
        UpdateViewMatrix();
    }
    glm::mat4x4 GetViewMatrix() const { return m_viewMatrix; }
    glm::mat4x4 GetProjMatrix() const { return m_projMatrix; }
    glm::vec3 GetEye() const { return m_eye; }
    glm::vec3 GetUpVector() const { return m_upVector; }
    glm::vec3 GetLookAt() const { return m_lookAt; }
    glm::vec3 GetViewDir() const { return -glm::transpose(m_viewMatrix)[2]; }
    glm::vec3 GetRightVector() const { return glm::transpose(m_viewMatrix)[0]; }
    float GetFOV() const { return m_fov; }
    void CalculateArcballCamera()
    {
        if (Input::IsMouseButtonPressed(TEA_MOUSE_LEFT))
        {
            if (firstMouseClick)
            {
                lastMousePosRightClick.x = Input::GetMouseX();
                lastMousePosRightClick.y = Input::GetMouseY();
                firstMouseClick = false;
            }
            ArcBallCamera((lastMousePosRightClick.x - Input::GetMouseX()), (lastMousePosRightClick.y - Input::GetMouseY()));
            lastMousePosRightClick.x = Input::GetMouseX();
            lastMousePosRightClick.y = Input::GetMouseY();
        }
        else
        {
            firstMouseClick = true;
        }
    }
    void CalculatePanCamera()
    {
        if (Input::IsMouseButtonPressed(TEA_MOUSE_RIGHT))
        {
            if (firstRightMouseClick)
            {
                lastMousePosRightClick.x = Input::GetMouseX();
                lastMousePosRightClick.y = Input::GetMouseY();
                firstRightMouseClick = false;
            }
            PanCamera((Input::GetMouseX() - lastMousePosRightClick.x), (lastMousePosRightClick.y - Input::GetMouseY()));
            lastMousePosRightClick.x = Input::GetMouseX();
            lastMousePosRightClick.y = Input::GetMouseY();
        }
        else
        {
            firstRightMouseClick = true;
        }
    }
    void ZoomCamera()
    {
        if (Input::IsKeyPressed(TEA_KEY_W))
        {
            ProcessMouseScroll(1.0f);
        }
        else if (Input::IsKeyPressed(TEA_KEY_S))
        {
            ProcessMouseScroll(-1.0f);
        }
    }
    void SetFOV(float fov)
    {
        m_fov = fov;
    }
    void UpdateViewMatrix()
    {
        m_viewMatrix = glm::lookAt(m_eye, m_lookAt, m_upVector);
    }
    void UpdateProjMatrix()
    {
        m_projMatrix = glm::perspective(glm::radians(m_fov), (float) *m_width / *m_height, 0.1f, 200.0f * 20);
    }
    void SetCameraView(glm::vec3 eye, glm::vec3 lookat, glm::vec3 up)
    {
        m_eye = std::move(eye);
        m_lookAt = std::move(lookat);
        m_upVector = std::move(up);
        UpdateViewMatrix();
    }
    void ProcessMouseScroll(float yoffset)
    {
        m_fov -= (float)yoffset;
        if (m_fov < 1.0f)
            m_fov = 1.0f;
        if (m_fov > 45.0f)
            m_fov = 45.0f;
    }
private:
    glm::mat4x4 m_viewMatrix;
    glm::mat4x4 m_projMatrix;
    glm::vec3& m_eye;                // Camera position in 3D
    glm::vec3& m_lookAt;             // Point that the camera is looking at
    glm::vec3& m_upVector;           // Orientation of the camera
    int* m_width;
    int* m_height;
    float m_fov = 45;
    float pan_speed = .5f;
    float m_yaw = 0.0f;
    float m_pitch = 0.0f;
    bool firstMouseClick;
    bool firstRightMouseClick;
    glm::vec2 lastMousePosRightClick = glm::vec2(0.0f, 0.0f);
private:
    void ArcBallCamera(float deltaX, float deltaY)
    {
        glm::vec4 position(GetEye().x, GetEye().y, GetEye().z, 1);
        glm::vec4 pivot(GetLookAt().x, GetLookAt().y, GetLookAt().z, 1);
        float deltaAngleX = (2 * M_PI / *m_width);
        float deltaAngleY = (M_PI / *m_height);
        float xAngle = deltaX * deltaAngleX;
        float yAngle = deltaY * deltaAngleY;
        float cosAngle = glm::dot(GetViewDir(), m_upVector);
        if (cosAngle * glm::sign(yAngle) > 0.99f)
            yAngle = 0;
        glm::mat4x4 rotationMatrixX(1.0f);
        rotationMatrixX = glm::rotate(rotationMatrixX, xAngle, m_upVector);
        position = (rotationMatrixX * (position - pivot)) + pivot;
        glm::mat4x4 rotationMatrixY(1.0f);
        rotationMatrixY = glm::rotate(rotationMatrixY, yAngle, GetRightVector());
        glm::vec3 finalPosition = (rotationMatrixY * (position - pivot)) + pivot;
        SetCameraView(finalPosition, GetLookAt(), m_upVector);
    }
    void PanCamera(float deltaX, float deltaY)
    {
        deltaX *= 0.05f;
        deltaY *= 0.05f;
        glm::vec3 cameraFront = glm::normalize(m_lookAt - m_eye);
        glm::vec3 right = glm::normalize(glm::cross(cameraFront, m_upVector));
        glm::vec3 up = glm::normalize(m_upVector);
        m_eye -= deltaX * right + deltaY * up;
        m_lookAt -= deltaX * right + deltaY * up;
        UpdateViewMatrix();
    }
};
```

If you want to try how arcball camera works, you can check out my ShapeGenerator repository from <a href="https://github.com/uysalaltas/ShapeGenerator">here</a>. You can also contribute the repository. Also you should definetely check out the resuorces below for understanding Arcball camera more. 

* Arcball Camera Article. <a href="https://www.talisman.org/~erlkonig/misc/shoemake92-arcball.pdf">Link</a>
* Quaternions ans Gimbal Lock. <a href="https://www.youtube.com/watch?v=zjMuIxRvygQ&ab_channel=3Blue1Brown">Link</a>