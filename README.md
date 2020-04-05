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

## Level and grid generation
Each level in 9X9 has a level info component containing information including the dimensions and layout of a level as well as user data like scores. The level info is fed into the grid builder script to dynamically build a given level. This system is paired with a in-editor level building tool to expediting the level building process, and prevents level needing to be build by hand in the scene.

A levels layout is stored in a one dimensional array, and through the buildign process each block is mapped is to a grid location. With a simple function to get level coordinates carrying out the main mechanic of the game, toggling a block and all adjacent blocks, is a sinch.

Below is a code sample showing how the level builder accesses the info from the level to be played, creates the block, and sets up all necessary parameters. 9X9 also provides a random level mode, which uses a very similar function to this.

```C#
private void BuildAuthoredLevel()
    {
        ClearGrid(); //Clear out the last level so we only have one at a time.

        int blockNum = 0;

        blockScale = 1f;
        spacing = 1.2f;

        grid = new StandardBlock[currentLevel.gridWidth, currentLevel.gridHeight]; //Initialize our 2D array for the grid. Blocks will be mapped to this array as they are created.

        //Determine the space that the grid will take up.
        float trueWidth = (spacing * currentLevel.gridWidth) - spacing;
        float trueHeight = (spacing * currentLevel.gridHeight) - spacing;

        for (int i = 0; i < currentLevel.gridWidth; i++)
        {
            for (int j = currentLevel.gridHeight - 1; j >= 0; j--)
            {
                //Create and name block
                StandardBlock clone = Instantiate(piecePrefab, Vector3.zero, Quaternion.identity).GetComponent<StandardBlock>();
                clone.name = clone.name + i + j;
                clone.transform.position = new Vector3((i * spacing) - (trueWidth / 2), (j * spacing) - (trueHeight / 2), 0);
                clone.transform.localScale = new Vector3(0.01f, 0.01f, 0.01f);

                //Add the block we just created to list and set the block type of this block to basic if level data doesn't say what type to make it.
                allBlocks.Add(clone);
                StandardBlock cloneBlock = clone.GetComponent<StandardBlock>();
                if (blockNum > currentLevel.blockTypes.Count - 1)
                {
                    currentLevel.blockTypes.Add(StandardBlock.BlockType.Basic);
                }
                cloneBlock.blockType = currentLevel.blockTypes[blockNum];

                cloneBlock.Init();
                cloneBlock.coordinates = new int[,] { { i, j } };

                //Turn off block
                cloneBlock.parentGrid = this;
                cloneBlock.isOn = false;
                cloneBlock.SetColor();

                //If a basic block add 1 to valid blocks. Valid blocks count toward level completion percent.
                if (cloneBlock.blockType == StandardBlock.BlockType.Basic)
                {
                    validBlockCount++;
                }

                //Assign new block to its grid location
                grid[i, j] = cloneBlock;

                cloneBlock.gameObject.SetActive(false);

                blockNum++;
            }

            activeBlockCount = 0;
        }
    }
```

## In-Editor Level Building Tool
As mentioned above, a full in-editor tool was built to help speed up the level creation process. The editor lets you define the desired width and height of the level then use the grid to define the layout of the level. There are button to clear out the grid if you don't like the current layout, generate a random layout that you can then edit, and a button to save your layout to the level info script.
One of the big advantages of this tool is that it allows me to create levels based on my own ideas or generate a random level that might only need a little clean up. A handy slider allows me to adjust the chance that a block is removed from the grid, allowing further control over the random generation process. 

![inspector level editor](https://github.com/gljmelton/UnitySample.9X9/blob/master/Images/LevelEditorTool.gif?raw=true)

Once levels are commited a couple more editor tools are used to finalize. One tool automatically generates the level select UI, ordering and naming the levels based on their hierarchy in the Hierarchy panel, relieving me having to create the menus for over 100 levels by hand. Another tool collects all of the levels into a list to be used by the games save data system. This tool also automatically designates the level progression system by assigning each level to be the level to unlock for the previous level.
