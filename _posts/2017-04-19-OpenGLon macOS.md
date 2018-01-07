---
title: "【OpenGL on macOS】红宝书第八版第一个程序"
layout: post
excerpt: "从昨天下午开始配置OpenGL环境，到今天上午成功完全跑通了红宝书的第一个例子。"
last_modified_at: 2012-02-05T10:27:01-05:00
tags:
  - OpenGL
  - MacOS
  - Environment
categories: 解问题
---



从昨天下午开始配置OpenGL环境，到今天上午成功完全跑通了红宝书的第一个例子。

------

Mac系统用起来爽，但是配置什么东西都需要折腾，关键能参考的教程还不多。

看了各种各样的问题解答，这篇文章做个归纳。

### 一、配环境

先从配环境开始讲。

配环境这个东西，不同的配置教程方法各异，有的特别简单，有的复杂度惊人。博主差不多尝试了各种神奇的配置方法，包括但不限于：

> <https://zrz0f.com/2016/02/21/glfw/>

一开始一直觉得这个配置过程看起来又专业又有效，各种膜。

原作者写的确实很好，但是红宝书似乎用不上那一套，因为后来我只引了两个Xcode自带的openGL库，就发现这代码能跑了。

过程：新建命令行(Command Line Tool)工程，在工程设置里引入两个库：OpenGL.frameworks和GLUT.frameworks

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-1.png)

之后，修改search path，把Always Search User Paths改为Yes:

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-6.png)

没错，似乎这样环境就ok了（不用折腾什么glfw之类，不过如果有需要的话，上面那个链接的文章值得参考），接着往下看，坑在后面。

------

### 二、敲代码

红宝书的官网在这方面有点不人性化，居然只能下第九版书本的源码（官方我求你更新慢一点好吗），并且还和第八版存在一些明显的差异（包括用的库都不一样）。后来在51CTO上面找到了资源<http://down.51cto.com/data/2193129>。不过为了下载方便一点，还是请用下面的度盘链接：

> 链接: <https://pan.baidu.com/s/1bppISqF> 密码: dqhm

拿到源码以后：震惊！居然没有第一个例子的代码(triangle.cpp)！

强烈推荐手敲一遍，一方面可以再仔细读一遍源码，另一方面还可以理解一下源码的流程和思想。岂不美哉。ps.千万别去复制第九版的代码。

好吧我知道有人不愿意敲（比如我自己一开始就不愿意，后来…），在这里贴上来：

```c++
//////////////////////////////////////////////////////////////////////////////
//
//  Triangles.cpp
//
//////////////////////////////////////////////////////////////////////////////

#include <iostream>
using namespace std;

#include "vgl.h"
#include "LoadShaders.h"

enum VAO_IDs { Triangles, NumVAOs };
enum Buffer_IDs { ArrayBuffer, NumBuffers };

enum Attrib_IDs { vPosition = 0 };

GLuint VAOs[NumVAOs];
GLuint Buffers[NumBuffers];

const GLuint NumVertices = 6;

//----------------------------------------------------------------------------
//
// init
//

void init(void)
{
    glGenVertexArrays(NumVAOs, VAOs);
    glBindVertexArray(VAOs[Triangles]);
    
    GLfloat vertices[NumVertices][2] =
    {
        { -0.90, -0.90 },
        {  0.85, -0.90 },
        { -0.90,  0.85 },
        {  0.90, -0.85 },
        {  0.90,  0.90 },
        { -0.85,  0.90 }
    };
    
    glGenBuffers(NumBuffers, Buffers);
    glBindBuffer(GL_ARRAY_BUFFER, Buffers[ArrayBuffer]);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
    ShaderInfo shaders[] =
     {
     { GL_VERTEX_SHADER, "triangles.vert" },
     { GL_FRAGMENT_SHADER, "triangles.frag" },
     { GL_NONE, NULL }
     };
    
    GLuint program = LoadShaders(shaders);
    glUseProgram(program);
    
    glVertexAttribPointer(vPosition, 2, GL_FLOAT, GL_FALSE, 0, BUFFER_OFFSET(0));
    glEnableVertexAttribArray(vPosition);
}

//----------------------------------------------------------------------------
//
// display
//

void display(void)
{
    glClear(GL_COLOR_BUFFER_BIT);
    
    glBindVertexArray(VAOs[Triangles]);
    glDrawArrays(GL_TRIANGLES, 0, NumVertices);
    
    glFlush();
}

//----------------------------------------------------------------------------
//
// main
//

int main(int argc, char ** argv)
{
    glutInit(&argc, argv);
    
    glutInitDisplayMode(GLUT_RGBA);
    
    glutInitWindowSize(512,512);
    
    glutInitContextVersion(4, 3);
    glutInitContextProfile(GLUT_CORE_PROFILE);
    
    glutCreateWindow(argv[0]);
    
    
     if(glewInit())
     {
     cerr << "Unable to initialize GLEW ... exiting" << endl;
     exit(EXIT_FAILURE);
     }
     
    init();

    glutDisplayFunc(display);
    
    glutMainLoop();
}
```

把main.cpp重命名成triangle.cpp后，码完上述代码。

之后，将oglpg-8th-edition/lib/LoadShaders.cpp，和oglpg-8th-edition/include/LoadShaders.h拖入工程。为了防止对源文件的改动，我们勾选「Copy items if needed」

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-3.png)

现在运行程序还是会报很多错的，下面开始修改代码。

------

### 三、修改代码

#### 1.头部

既然开头就是要引入vgl.h，那不妨先看看这个头文件里有些啥：

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-2.png)

一堆条件编译，以及有关freeglut/glew的声明。然而，mac用不上glew（至少这个例子用不上），我们也不用去使用freeglut..所以我们只需要把头文件里最有用的bufferoffset的定义放到main函数中，并引入glut库。

```c++
#define BUFFER_OFFSET(x) ((const void*)(x))
#include <GLUT/GLUT.h>
```

（参考：<http://markzhang.cn/blog/2014/01/06/hellogl-on-macosx/>，这也是一篇很好的教程，本文下面的内容有部分参考自此处）

之后删掉（如果不放心可以选择注释掉）#include “vgl.h”

#### 2.main函数

- 在main函数中增加一行代码：

```c++
cout << "Supported GLSL version is: " <<
     glGetString(GL_SHADING_LANGUAGE_VERSION) << endl;
```

这是为了获取目前openGL的版本，在程序运行后，就可以根据这个版本号去修改相关着色器的version了。

- 删除这些代码：

```c++
glutInitContextVersion(4, 3);
glutInitContextProfile(GLUT_CORE_PROFILE);

if(glewInit())
{
    cerr << "Unable to initialize GLEW ... exiting" << endl;
    exit(EXIT_FAILURE);
}
```

删除上面两行的理由是，我们这里没有使用书中的freeglut，因为Xcode自带了GLUT（应该是openglut），所以就没有必要再安装一个类似的库了。 而Xcode自带的GLUT，是没有上述的两个函数的。

删除下面一段的理由是，我们这里不使用glew库。

- （关键的地方），把

```
glutInitDisplayMode(GLUT_RGBA);
```

改成：

```
glutInitDisplayMode(GLUT_RGBA | GLUT_3_2_CORE_PROFILE);
```

这是因为在上面删掉了

```c++
glutInitContextVersion(4, 3);
glutInitContextProfile(GLUT_CORE_PROFILE);
```

之后，我们没有指定版本号，所以在显示模式里加以指定。

#### 3.改.frag和.vert

这个地方要感谢Stack Overflow上面的提问者[Matthew James Briggs](https://stackoverflow.com/users/2779792/matthew-james-briggs)和回答者[bames53](https://stackoverflow.com/users/365496/bames53)、[gonglong](https://stackoverflow.com/users/752131/gonglong)。

> 源问题链接：<https://stackoverflow.com/questions/34385540/opengl-red-book-with-mac-os-x/37561780#37561780?newreg=0bfd40bef37541a3bb68cc21b5b9af59>

虽然我不知道是否有更简单的做法，不过按照他们的过程确实做出了想要的结果。

- triangle.cpp

在头部常量定义的部分加上：

```c++
const char *pTriangleVert ="#version 410 core\n\
layout(location = 0) in vec4 vPosition;\n\
void\n\
main()\n\
{\n\
gl_Position= vPosition;\n\
}";

const char *pTriangleFrag = "#version 410 core\n\
out vec4 fColor;\n\
void\n\
main()\n\
{\n\
fColor = vec4(0.0, 0.0, 1.0, 1.0);\n\
}";
```

为什么要加上这些？因为似乎Xcode下面读文件有问题，也就是说，用.vert和.frag文件并写上相关的代码交给着色器，着色器并不买账。（我做出来的结果是：无论如何修改结点的fColor，三角形均为白色）

所以，我们相当于把两个文件中的内容改成了字符串，直接交给着色器去读，效果是一样的。

**注意：这个地方的#version 410 core要填成你自己的openGL版本号！**

修改改ShaderInfo：

```c++
ShaderInfo  shaders[] = {
        { GL_VERTEX_SHADER, pTriangleVert},
        { GL_FRAGMENT_SHADER, pTriangleFrag },
        { GL_NONE, NULL }
    };
```



- LoadShaders.h

把

```c++
#include <GL/gl.h>
```

改成：

```c++
#include <OpenGL/gl3.h>
```

并把ShaderInfo结构体里的 **filename **改成 **source **。

- LoadShaders.cpp

把与glew相关的代码删掉：

```c++
#define GLEW_STATIC
#include <GL/glew.h>
```

把所有的** _DEBUG** 改为 **DEBUG**。

之后删掉：

```c++
const GLchar* source = ReadShader( entry->filename );
if ( source == NULL ) 
{
    for ( entry = shaders; entry->type != GL_NONE; ++entry ) 
    {
        glDeleteShader( entry->shader );
        entry->shader = 0;
    }
    return 0;
}
glShaderSource( shader, 1, &source, NULL );
delete [] source;
```

改成

```c++
glShaderSource(shader, 1, &entry->source, NULL);
```

删掉

```c++
#ifdef GL_VERSION_4_1
    if ( GLEW_VERSION_4_1 ) {
        // glProgramParameteri( program, GL_PROGRAM_SEPARABLE, GL_TRUE );
    }
#endif /* GL_VERSION_4_1 */
```

---

为了方便，把做完所有修改后的代码粘在下面：

- triangle.cpp

```c++
//
//  triangle.cpp
//  gl_c1
//
//  Created by SillyDog on 2017/4/19.
//  Copyright © 2017年 Hope. All rights reserved.
//

//////////////////////////////////////////////////////////////////////////////
//
//  Triangles.cpp
//
//////////////////////////////////////////////////////////////////////////////

#include <iostream>
using namespace std;

//#include "vgl.h"
#include "LoadShaders.h"
#include <GLUT/GLUT.h>

#define BUFFER_OFFSET(x) ((const void*)(x))

enum VAO_IDs { Triangles, NumVAOs };
enum Buffer_IDs { ArrayBuffer, NumBuffers };

enum Attrib_IDs { vPosition = 0 };

GLuint VAOs[NumVAOs];
GLuint Buffers[NumBuffers];

const GLuint NumVertices = 6;
const char *pTriangleVert ="#version 410 core\n\
layout(location = 0) in vec4 vPosition;\n\
void\n\
main()\n\
{\n\
gl_Position= vPosition;\n\
}";

const char *pTriangleFrag = "#version 410 core\n\
out vec4 fColor;\n\
void\n\
main()\n\
{\n\
fColor = vec4(0.0, 0.0, 1.0, 1.0);\n\
}";

//----------------------------------------------------------------------------
//
// init
//

void init()
{
    glGenVertexArrays(NumVAOs, VAOs);
    glBindVertexArray(VAOs[Triangles]);
    
    GLfloat vertices[NumVertices][2] =
    {
        { -0.90, -0.90 },
        {  0.85, -0.90 },
        { -0.90,  0.85 },
        {  0.90, -0.85 },
        {  0.90,  0.90 },
        { -0.85,  0.90 }
    };
    
    glGenBuffers(NumBuffers, Buffers);
    glBindBuffer(GL_ARRAY_BUFFER, Buffers[ArrayBuffer]);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
    /*ShaderInfo shaders[] =
     {
     { GL_VERTEX_SHADER, "triangles.vert" },
     { GL_FRAGMENT_SHADER, "triangles.frag" },
     { GL_NONE, NULL }
     };*/
    
    ShaderInfo  shaders[] = {
        { GL_VERTEX_SHADER, pTriangleVert},
        { GL_FRAGMENT_SHADER, pTriangleFrag },
        { GL_NONE, NULL }
    };
    
    GLuint program = LoadShaders(shaders);
    glUseProgram(program);
    
    glVertexAttribPointer(vPosition, 2, GL_FLOAT, GL_FALSE, 0, BUFFER_OFFSET(0));
    glEnableVertexAttribArray(vPosition);
}

//----------------------------------------------------------------------------
//
// display
//

void display(void)
{
    glClear(GL_COLOR_BUFFER_BIT);
    
    glBindVertexArray(VAOs[Triangles]);
    glDrawArrays(GL_TRIANGLES, 0, NumVertices);
    
    //glutSwapBuffers();
    glFlush();
}

//----------------------------------------------------------------------------
//
// main
//

int main(int argc, char ** argv)
{
    glutInit(&argc, argv);
    
    glutInitDisplayMode(GLUT_RGBA | GLUT_3_2_CORE_PROFILE);
    
    //glutInitDisplayMode(GLUT_RGBA);
    
    glutInitWindowSize(512,512);
    
    //glutInitContextVersion(4, 3);
    //glutInitContextProfile(GLUT_CORE_PROFILE);
    
    glutCreateWindow(argv[0]);
    
    /*
     if(glewInit())
     {
     cerr << "Unable to initialize GLEW ... exiting" << endl;
     exit(EXIT_FAILURE);
     }
     */
    
    cout << "Supported GLSL version is: " <<
     glGetString(GL_SHADING_LANGUAGE_VERSION) << endl;
    
    init();
    glutDisplayFunc(display);
    
    glutMainLoop();
}
```



- LoadShaders.h

```c++
//////////////////////////////////////////////////////////////////////////////
//
//  --- LoadShaders.h ---
//
//////////////////////////////////////////////////////////////////////////////

#ifndef __LOAD_SHADERS_H__
#define __LOAD_SHADERS_H__

#include <OpenGL/gl3.h>
//#include <GL/gl.h>

#ifdef __cplusplus
extern "C" {
#endif  // __cplusplus

//----------------------------------------------------------------------------
//
//  LoadShaders() takes an array of ShaderFile structures, each of which
//    contains the type of the shader, and a pointer a C-style character
//    string (i.e., a NULL-terminated array of characters) containing the
//    entire shader source.
//
//  The array of structures is terminated by a final Shader with the
//    "type" field set to GL_NONE.
//
//  LoadShaders() returns the shader program value (as returned by
//    glCreateProgram()) on success, or zero on failure. 
//
    
/*
typedef struct {
    GLenum       type;
    const char*  filename;
    GLuint       shader;
} ShaderInfo;
*/
    
typedef struct
{
    GLenum       type;
    const char*  source;
    GLuint       shader;
} ShaderInfo;
    
GLuint LoadShaders( ShaderInfo* );

//----------------------------------------------------------------------------

#ifdef __cplusplus
};
#endif // __cplusplus

#endif // __LOAD_SHADERS_H__
```

- LoadShaders.cpp

```c++
//////////////////////////////////////////////////////////////////////////////
//
//  --- LoadShaders.cxx ---
//
//////////////////////////////////////////////////////////////////////////////

#include <cstdlib>
#include <iostream>

//#define GLEW_STATIC
//#include <GL/glew.h>
#include "LoadShaders.h"

#ifdef __cplusplus
extern "C" {
#endif // __cplusplus

//----------------------------------------------------------------------------

static const GLchar*
ReadShader( const char* filename )
{
#ifdef WIN32
	FILE* infile;
	fopen_s( &infile, filename, "rb" );
#else
    FILE* infile = fopen( filename, "rb" );
#endif // WIN32

    if ( !infile ) {
#ifdef DEBUG
        std::cerr << "Unable to open file '" << filename << "'" << std::endl;
#endif /* DEBUG */
        return NULL;
    }

    fseek( infile, 0, SEEK_END );
    int len = ftell( infile );
    fseek( infile, 0, SEEK_SET );

    GLchar* source = new GLchar[len+1];

    fread( source, 1, len, infile );
    fclose( infile );

    source[len] = 0;

    return const_cast<const GLchar*>(source);
}

//----------------------------------------------------------------------------

GLuint
LoadShaders( ShaderInfo* shaders )
{
    if ( shaders == NULL ) { return 0; }

    GLuint program = glCreateProgram();

    ShaderInfo* entry = shaders;
    while ( entry->type != GL_NONE ) {
        GLuint shader = glCreateShader( entry->type );

        entry->shader = shader;
/*
        const GLchar* source = ReadShader( entry->filename );
        if ( source == NULL ) {
            for ( entry = shaders; entry->type != GL_NONE; ++entry ) {
                glDeleteShader( entry->shader );
                entry->shader = 0;
            }

            return 0;
        }

        glShaderSource( shader, 1, &source, NULL );
        delete [] source;
*/
        glShaderSource(shader, 1, &entry->source, NULL);
        
        glCompileShader( shader );

        GLint compiled;
        glGetShaderiv( shader, GL_COMPILE_STATUS, &compiled );
        if ( !compiled ) {
#ifdef DEBUG
            GLsizei len;
            glGetShaderiv( shader, GL_INFO_LOG_LENGTH, &len );

            GLchar* log = new GLchar[len+1];
            glGetShaderInfoLog( shader, len, &len, log );
            std::cerr << "Shader compilation failed: " << log << std::endl;
            delete [] log;
#endif /* DEBUG */

            return 0;
        }

        glAttachShader( program, shader );
        
        ++entry;
    }
/*
#ifdef GL_VERSION_4_1
    if ( GLEW_VERSION_4_1 ) {
        // glProgramParameteri( program, GL_PROGRAM_SEPARABLE, GL_TRUE );
    }
#endif  GL_VERSION_4_1 */
    
    glLinkProgram( program );

    GLint linked;
    glGetProgramiv( program, GL_LINK_STATUS, &linked );
    if ( !linked ) {
#ifdef DEBUG
        GLsizei len;
        glGetProgramiv( program, GL_INFO_LOG_LENGTH, &len );

        GLchar* log = new GLchar[len+1];
        glGetProgramInfoLog( program, len, &len, log );
        std::cerr << "Shader linking failed: " << log << std::endl;
        delete [] log;
#endif /* DEBUG */

        for ( entry = shaders; entry->type != GL_NONE; ++entry ) {
            glDeleteShader( entry->shader );
            entry->shader = 0;
        }
        
        return 0;
    }

    return program;
}

//----------------------------------------------------------------------------
#ifdef __cplusplus
}
#endif // __cplusplus
```



工程目录：

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-4.png)

不出意外地，博主都写到这里了，怎么会不放运行结果呢：

![img](http://ohn6qfqhe.bkt.clouddn.com/gl1-5.png)

完毕！

在同学的一台机子上测试过方案，没有出现问题。

希望大家也能看到和上面一样的结果！

------

个人心得：

各种环境的配置过程到处都是，有很多写的很认真的文章和过程，但是仍然不能完全解决所有的问题。我相信本文也一样，你在配置完成后可能也没有看到想要的结果。但是，这不妨碍你一步步的去摸索，直到对整个程序和环境的框架了如指掌。投入时间总是没错的，在一条路走不通的情况下一定不能死磕。



End.