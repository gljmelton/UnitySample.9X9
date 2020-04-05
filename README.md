# UnitySample.9X9
Code examples from the mobile game 9X9

## Camera View Resizing
9X9 utilizes a system that ensures all levels stay contained on the screen. As opposed to resizing and spacing all of the blocks out to fit within the screen, the orthographic size of the games viewport is scaled to fit. The block within the game are always the same size and spaced out evenly, allowing the camera size ratio to be scaled at a nearly 1:1 ratio. See below for an example gif and example code.
Additionally, the code utilizes DOTween to ease from one view size the next, providing a smooth transition between levels.

![camera resizing](https://github.com/gljmelton/UnitySample.9X9/blob/master/Images/CameraResizing.gif?raw=true)


```C#
//Resizes the cameras orthographic view to contain the current level.
public void SetViewSize()
{
    int largestSide = currentLevel.gridWidth >= currentLevel.gridHeight ? currentLevel.gridWidth : currentLevel.gridHeight; //determine whether the grid wide or tall
    float camSize;

    camSize = largestSide + (largestSide * 0.2f);

    if (camSize < minViewSize)
    {
        camSize = minViewSize; //Clamp the view size so that smaller levels don't fill the entire screen.
    }

    Camera.main.DOOrthoSize(camSize, 0.5f); //Use DOTween to tween to the new camera size.
    }
```
