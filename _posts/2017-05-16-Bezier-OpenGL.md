---
title: "【OpenGL】绘制可编辑的Bezier曲线"
layout: post
excerpt: "实验：OpenGL固定管线，交互式绘制可编辑、精度可控、高阶的Bezier曲线。"
last_modified_at: 2012-02-05T10:27:01-05:00
tags:
  - OpenGL
  - Bezier
  - Algorithm
categories: 解问题
---



实验：OpenGL固定管线，交互式绘制可编辑、精度可控、高阶的Bezier曲线。

------

Bezier曲线的定义式：

![img](http://ohn6qfqhe.bkt.clouddn.com/bezier-1.png)

式中Vi是用户通过鼠标点击产生控制点，u从0递增至1，递增的速度即为精度。括号中竖写的n和i即排列组合概念中的组合数。

下面给出一个用固定管线的OpenGL版本来绘制和编辑Bezier曲线的程序源码。

先给出宏定义和全局变量：

```c++
#include <GL/GLUT.h>
#include <math.h>
#include <iostream>

//////////////////////////////宏定义和全局变量///////////////////////////
// 定义控制点(Control Points)数目的最大值                               //
//
#define MAX_CP 8

// 实际控制点个数
//
int num_cp=0;

// 窗口大小
//
static int width=600,height=600;

// 点的结构体
//
typedef struct
{
    GLfloat x,y;
} Point;

// 存储控制点的转换坐标
//
Point cpts[MAX_CP];
// 存储控制点的实际坐标
// 加入实际坐标是为了便于判断编辑点的选择
// 因为规范化后的坐标在[-1,1]范围内，不便于直接观察
//
Point Rcpts[MAX_CP];

// 编辑状态和待编辑顶点的索引
//
bool editFlag = false;
int editIndex = -1;

// 精度(Accuracy)
//
int acc = 20;
//                                                                     //
/////////////////////////////////////////////////////////////////////////
```

首先，我们需要定义几个便于我们做代数计算的函数。

1.计算阶乘

不要用递归！不要用递归！不要用递归！既占内存又耗时。

```c++
// 求n的阶乘(factorial)                                                  
//
int fac(int n)
{
    
    if(n==1||n==0)
        return 1;
    for(int i = n-1;i > 1;i--)
        n = n*i;
    return n;
}
```

2.计算组合数

根据定义式计算即可。

```c++
// 求组合数(combination)
//
double comb(int n,int i)
{
    return fac(n)/(fac(i)*fac(n-i));
}
```

接下来定义绘制基本的图元必要的函数，包括点和直线段。为了更好的表达出控制点的位置，我们将点放大表示。

1.画点

```c++
void setPoint(Point p)
{

    glBegin(GL_POINTS);
    glVertex2f(p.x, p.y);
    glEnd();
    glFlush();
}
```

2.画线

```c++
void setLine(Point p1, Point p2)
{
    glColor3f(0.0,0.0,0.0);
    glBegin(GL_LINES);
    glVertex2f(p1.x,p1.y);
    glVertex2f(p2.x, p2.y);
    glEnd();
    glFlush();
}
```

处理鼠标和键盘事件：交互的核心部分

1.鼠标响应函数(左键控制输入和右键编辑)

```c++
static void mouse(int button, int state,int x,int y)
{
    float wx,wy;
    if(state == GLUT_DOWN)
    {
        //鼠标左键为绘制
        if(button == GLUT_LEFT_BUTTON)
        {
            //判断控制点数目是否超过最大值
            if(num_cp == MAX_CP)
                return;
            Rcpts[num_cp].x = x;
            Rcpts[num_cp].y = y;
            //转换坐标
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            //存储控制点
            cpts[num_cp].x=wx;
            cpts[num_cp].y=wy;
            num_cp++;
            display();
        }
        //非编辑状态下，右键点选后，将待修改的点进行标记
        if(button == GLUT_RIGHT_BUTTON && editFlag == false)
        {
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            for(int i = 0;i < num_cp;i++)
            {
                if(pow(x-Rcpts[i].x,2)+pow(y-Rcpts[i].y,2) < 100.0)
                {
                    editIndex = i;
                    editFlag = true;
                    display();
                    drawBezier(cpts);
                    return;
                }
            }
        }
        //已经有标记点的情况下，再按右键，记录这个点的最终位置
        if(button == GLUT_RIGHT_BUTTON && editFlag == true)
        {
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            Rcpts[editIndex].x = x;
            Rcpts[editIndex].y = y;
            cpts[editIndex].x = wx;
            cpts[editIndex].y = wy;
            editFlag = false;
            display();
        }
    }
}
```

2.键盘操作

```c++
// 键盘回调函数
//
void keyboard(unsigned char key,int x,int y)
{
    switch (key)
    {
        //Q键退出程序
        case 'q': case 'Q':
            exit(0);
            break;
        //C键重绘
        case 'c': case 'C':
            num_cp = 0;
            glutPostRedisplay();
            break;
        //R键撤销一个点
        case 'r': case 'R':
            num_cp--;
            glutPostRedisplay();
            break;
    }
}
// 基于方向键的键盘回调函数
//
void keyboard1(int key,int x,int y)
{
    switch (key)
    {
        case GLUT_KEY_UP:
            if(acc < 100)
                acc += 1;
            glutPostRedisplay();
            break;
        case GLUT_KEY_DOWN:
            if(acc >= 5)
                acc -= 1;
             glutPostRedisplay();
            break;
    }
}
```

鼠标的输入坐标会存储在全局变量数组cpts[]中，便于绘制bezier曲线的函数进行调用。

下面根据定义绘制Bezier曲线：

```c++
void drawBezier(Point *p)
{
    if(num_cp<=0) return;
    
    Point *p1;
    p1=new Point[100];
    GLfloat u=0,x,y;
    int i,num=0;
    
    for(u=0;u<=1;u=u+(1.0/acc))
    {
        x=0;
        y=0;
        //曲线次数=控制点个数-1
        for(i = 0;i <= num_cp-1;i++)
        {
            x+=comb(num_cp-1,i)*pow(u,i)*pow((1-u),(num_cp-1-i))*p[i].x;
            y+=comb(num_cp-1,i)*pow(u,i)*pow((1-u),(num_cp-1-i))*p[i].y;
        }
        p1[num].x=x;
        p1[num].y=y;
        num++;
    }
    
    
    for(i = 0;i < num_cp;i++)
    {
        glPointSize(5.0);
        glColor3f(0.0,0.0,0.0);
        if(i == editIndex)
            glColor3f(1.0, 0.0, 0.0);
        setPoint(cpts[i]);
        if(i != 0)
            setLine(cpts[i-1],cpts[i]);
    }
    
    glColor3f(1.0,0.0,0.0);
    glBegin(GL_LINE_STRIP);
    for(int k=0;k < num;k++)
        glVertex2f(p1[k].x,p1[k].y);
    glEnd();
    delete [] p1;
    glFlush();
}
```

再在适当的位置加上ReShape/Display/Main函数，最终代码如下：

```c++
#include <GL/GLUT.h>
#include <math.h>
#include <iostream>

//////////////////////////////宏定义和全局变量///////////////////////////
// 定义控制点(Control Points)数目的最大值                               //
//
#define MAX_CP 8

// 实际控制点个数
//
int num_cp=0;

// 窗口大小
//
static int width=600,height=600;

// 点的结构体
//
typedef struct
{
    GLfloat x,y;
} Point;

// 存储控制点的转换坐标
//
Point cpts[MAX_CP];
// 存储控制点的实际坐标
// 加入实际坐标是为了便于判断编辑点的选择
// 因为规范化后的坐标在[-1,1]范围内，不便于直接观察
//
Point Rcpts[MAX_CP];

// 编辑状态和待编辑顶点的索引
//
bool editFlag = false;
int editIndex = -1;

// 精度(Accuracy)
//
int acc = 20;
//                                                                     //
/////////////////////////////////////////////////////////////////////////

///////////////////////////////代数计算////////////////////////////////////
// 求n的阶乘(factorial)                                                  //
//
int fac(int n)
{
    
    if(n==1||n==0)
        return 1;
    for(int i = n-1;i > 1;i--)
        n = n*i;
    return n;
}

// 求组合数(combination)
//
double comb(int n,int i)
{
    return fac(n)/(fac(i)*fac(n-i));
}
//                                                                     //
/////////////////////////////////////////////////////////////////////////

//////////////////////////////基础图元绘制/////////////////////////////////
//绘制点                                                                //
//
void setPoint(Point p)
{

    glBegin(GL_POINTS);
    glVertex2f(p.x, p.y);
    glEnd();
    glFlush();
}

// 绘制直线段
//
void setLine(Point p1, Point p2)
{
    glColor3f(0.0,0.0,0.0);
    glBegin(GL_LINES);
    glVertex2f(p1.x,p1.y);
    glVertex2f(p2.x, p2.y);
    glEnd();
    glFlush();
}
//                                                                     //
/////////////////////////////////////////////////////////////////////////

//////////////////////////////Bezier曲线绘制//////////////////////////////
// 绘制bezier曲线                                                       //
//
void drawBezier(Point *p)
{
    if(num_cp<=0) return;
    
    Point *p1;
    p1=new Point[100];
    GLfloat u=0,x,y;
    int i,num=0;
    
    for(u=0;u<=1;u=u+(1.0/acc))
    {
        x=0;
        y=0;
        //曲线次数=控制点个数-1
        for(i = 0;i <= num_cp-1;i++)
        {
            x+=comb(num_cp-1,i)*pow(u,i)*pow((1-u),(num_cp-1-i))*p[i].x;
            y+=comb(num_cp-1,i)*pow(u,i)*pow((1-u),(num_cp-1-i))*p[i].y;
        }
        p1[num].x=x;
        p1[num].y=y;
        num++;
    }
    
    
    for(i = 0;i < num_cp;i++)
    {
        glPointSize(5.0);
        glColor3f(0.0,0.0,0.0);
        if(i == editIndex)
            glColor3f(1.0, 0.0, 0.0);
        setPoint(cpts[i]);
        if(i != 0)
            setLine(cpts[i-1],cpts[i]);
    }
    
    glColor3f(1.0,0.0,0.0);
    glBegin(GL_LINE_STRIP);
    for(int k=0;k < num;k++)
        glVertex2f(p1[k].x,p1[k].y);
    glEnd();
    delete [] p1;
    glFlush();
}
//                                                                     //
/////////////////////////////////////////////////////////////////////////

void display(void)
{
    glClear(GL_COLOR_BUFFER_BIT);
    glFlush();
    drawBezier(cpts);
}

//////////////////////////////鼠标和键盘响应///////////////////////////////
// 输入新的控制点                                                        //
static void mouse(int button, int state,int x,int y)
{
    float wx,wy;
    if(state == GLUT_DOWN)
    {
        //鼠标左键为绘制
        if(button == GLUT_LEFT_BUTTON)
        {
            //判断控制点数目是否超过最大值
            if(num_cp == MAX_CP)
                return;
            Rcpts[num_cp].x = x;
            Rcpts[num_cp].y = y;
            //转换坐标
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            //存储控制点
            cpts[num_cp].x=wx;
            cpts[num_cp].y=wy;
            num_cp++;
            display();
        }
        //非编辑状态下，右键点选后，将待修改的点进行标记
        if(button == GLUT_RIGHT_BUTTON && editFlag == false)
        {
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            for(int i = 0;i < num_cp;i++)
            {
                if(pow(x-Rcpts[i].x,2)+pow(y-Rcpts[i].y,2) < 100.0)
                {
                    editIndex = i;
                    editFlag = true;
                    display();
                    drawBezier(cpts);
                    return;
                }
            }
        }
        //已经有标记点的情况下，再按右键，记录这个点的最终位置
        if(button == GLUT_RIGHT_BUTTON && editFlag == true)
        {
            wx=(2.0*x)/(float)(width-1)-1.0;
            wy=(2.0*(height-1-y))/(float)(height-1)-1.0;
            Rcpts[editIndex].x = x;
            Rcpts[editIndex].y = y;
            cpts[editIndex].x = wx;
            cpts[editIndex].y = wy;
            editFlag = false;
            display();
        }
    }
}

// 键盘回调函数
//
void keyboard(unsigned char key,int x,int y)
{
    switch (key)
    {
        //Q键退出程序
        case 'q': case 'Q':
            exit(0);
            break;
        //C键重绘
        case 'c': case 'C':
            num_cp = 0;
            glutPostRedisplay();
            break;
        //R键撤销一个点
        case 'r': case 'R':
            num_cp--;
            glutPostRedisplay();
            break;
    }
}
// 基于方向键的键盘回调函数
//
void keyboard1(int key,int x,int y)
{
    switch (key)
    {
        case GLUT_KEY_UP:
            if(acc < 100)
                acc += 1;
            glutPostRedisplay();
            break;
        case GLUT_KEY_DOWN:
            if(acc >= 5)
                acc -= 1;
             glutPostRedisplay();
            break;
    }
}
//                                                                     //
/////////////////////////////////////////////////////////////////////////

// 重绘函数
//
void reshape(int w,int h)
{
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    glOrtho(-1.0,1.0,-1.0,1.0,-1.0,1.0);
    glMatrixMode(GL_MODELVIEW);
    glViewport(0,0,w,h);//调整视口
    width=w;
    height=h;
}

int main(int argc, char **argv)
{
    //初始化
    glutInit(&argc,argv);
    glutInitDisplayMode(GLUT_RGB);
    glutInitWindowSize(width,height);
    glutCreateWindow("DrawBezier");
    //注册回调函数  
    glutDisplayFunc(display);
    glutMouseFunc(mouse);
    glutKeyboardFunc(keyboard);
    glutSpecialFunc(keyboard1);
    glutReshapeFunc(reshape);  
    glClearColor(1.0,1.0,1.0,1.0);
    glColor3f(0.0,0.0,0.0);
    glutMainLoop();  
}
```

如果是macOS系统，只需要把头文件改成：

```c++
#include <GLUT/GLUT.h>
#include <OpenGL/OpenGL.h>
#include <math.h>
#include <iostream>
```

并在项目设置里导入自带的OpenGL和GLUT的Framework。

运行结果：

![img](http://ohn6qfqhe.bkt.clouddn.com/bezier-2.png)

右键编辑操作：

![img](http://ohn6qfqhe.bkt.clouddn.com/bezier-3.png)

![img](http://ohn6qfqhe.bkt.clouddn.com/bezier-4.png)

其它：键盘上下方向键调节绘制精度，「R」删除一个点，「C」清屏，「Q」退出。

源码框架参考资料：

[CSDN – OpenGL绘制Bezier曲线](http://blog.csdn.net/zjccoder/article/details/42807645)

------

完毕。

这周刷另一个上机作业——绘制B样条。



于开空调的寝室