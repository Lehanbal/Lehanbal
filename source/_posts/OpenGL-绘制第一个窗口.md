title: OpenGL-绘制第一个窗口
author: Lehanbal
date: 2022-04-13 00:40:45
tags:
---
GLFW：配合 OpenGL 使用的轻量级工具程序库，缩写自 Graphics Library Framework（图形库框架）。

GLAD：对底层OpenGL接口的封装。

使用GLFW配合GLAD绘制的第一个窗口程序。

```c++
#include <iostream>
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#define HEIGHT 800
#define WIDTH 600

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processinput(GLFWwindow* window);

int main()
{
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);	//主要版本号设置为3
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);	//次版本号设置为3
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);	//使用核心模式（core profile）

	GLFWwindow* window = glfwCreateWindow(HEIGHT, WIDTH, "learnOpenGL", NULL, NULL);
	if (window == NULL)
	{
		std::cout << "Failed to create GLFW window" << std::endl;
		glfwTerminate();
		return -1;
	}
	//将当前窗口的上下文切换为当前线程的上下文
	glfwMakeContextCurrent(window);

	//通过GLAD初始化OpenGL的函数指针 注：调用OpenGL的函数前都需要初始化GLAD
	if (!gladLoadGLLoader((GLADloadproc) glfwGetProcAddress))
	{
		std::cout << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

	//告诉opengl渲染窗口大小  glViewport函数前两个参数控制窗口左下角的位置。第三个和第四个参数控制渲染窗口的宽度和高度
	glViewport(0, 0, WIDTH, HEIGHT);

	//对窗口注册一个回调函数，在窗口大小被改变的时候调用
	glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

	//渲染循环，让这个图一直循环下去
	/*
	 *  glfwWindowShouldClose函数在我们每次循环的开始前检查一次GLFW是否被要求退出，如果是的话该函数返回true然后渲染循环便结束了，之后为我们就可以关闭应用程序了。
	 *	glfwPollEvents函数检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数（可以通过回调方法手动设置）。
	 *	glfwSwapBuffers函数会交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色值的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上
	 * 
	 */
	while (!glfwWindowShouldClose(window))
	{
		processinput(window);

		//中间插入渲染指令
		// ......

		//下述两行代码可以理解为，设置清空颜色缓冲位后，使用glClearColor设置的rgba颜色填充颜色缓冲位
		glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
		//清空颜色的缓冲位
		glClear(GL_COLOR_BUFFER_BIT);
		
		glfwSwapBuffers(window);
		glfwPollEvents();
	}

	//释放资源
	glfwTerminate();

    return 0;
}

//动态调整viewport大小
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
}

//监控键盘输入，若是esc被摁下则通过glfwSetWindowShouldClose函数关闭窗口
void processinput(GLFWwindow* window)
{
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
	{
		glfwSetWindowShouldClose(window, true);
	}
}

```

