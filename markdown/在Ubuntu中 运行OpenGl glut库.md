###在Ubuntu中 运行OpenGl glut库

- This time we focus on how to set up OpenGL for your Linux Mint 12 (It also should work fine with Ubuntu 12.04). For this task freeglut is going to be used. This is a open source and up to date alternative to the OpenGL Utility Toolkit. The official description of the freeglut project  says that

- freeglut is a completely OpenSourced alternative to the OpenGL Utility Toolkit (GLUT) library. GLUT was originally written by Mark Kilgard to support the sample programs in the second edition OpenGL 'RedBook'. Since then, GLUT has been used in a wide variety of practical applications because it is simple, widely available and highly portable.

- GLUT (and hence freeglut) allows the user to create and manage windows containing OpenGL contexts on a wide range of platforms and also read the mouse, keyboard and joystick functions.
**Step 1 - Install Compiler and necessary tools.**
 Since we are going to be programming with C++ , it is important to have the tool in your computer.  This following command will install a c++ compiler and **cmake** which is used to manage projects.
```
sudo apt-get install g++ cmake
```
**Step 2 - Lets install freeglut.**

Done ! But we need to test it.
Now you can create a simple test program.

**Step 3 - Test program - Create a 'main.cpp'**

```
#include <GL/glut.h>


void draw(void) {

    // Black background
    glClearColor(0.0f,0.0f,0.0f,1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    //Draw i
    glFlush();

}

//Main program

int main(int argc, char **argv) {

    glutInit(&argc, argv);

    /*Setting up  The Display
    /    -RGB color model + Alpha Channel = GLUT_RGBA
    */
    glutInitDisplayMode(GLUT_RGBA|GLUT_SINGLE);

    //Configure Window Postion
    glutInitWindowPosition(50, 25);

    //Configure Window Size
    glutInitWindowSize(480,480);

    //Create Window
    glutCreateWindow("Hello OpenGL");


    //Call to the drawing function
    glutDisplayFunc(draw);

    // Loop require by OpenGL
    glutMainLoop();
    return 0;
}
```

**Step 4 - Let's Compile it**

The first and the easiest option to compile is by using the command :

```
g++ -lGL -lglut main.cpp  -o test
```
Step 5 - Run it

```
./test
```
Using Cmake
That is it. If you would like to use the cmake to manage and compile you project you will need to ignore step 4 and 5 and follow this post: How to use [OpenGL/ Freeglut and CMake](http://igorbarbosa.com/articles/how-to-use-opengl-freeglut-and-cmake/)