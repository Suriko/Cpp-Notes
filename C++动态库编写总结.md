## C++动态库接口编写

### 动态库包含

#### 1. 头文件

接口文件，包含导入导出定义

```c++
#ifdef WIN32
#ifdef HEART_BEAT_API_EXPORTS
#define HEART_BEAT_API __declspec(dllexport)
#else
#define HEART_BEAT_API __declspec(dllimport)
#pragma comment(lib, "HeartBeat.lib")
#endif // HEART_BEAT_API_EXPORTS
#else
#define HEART_BEAT_API
#endif // WIN32
```

此段为windows下动态库导入导出代码段

#### 2.C风格接口

下面两个函数其实是对类的创建和销毁

```c++
extern "C" {
	HEART_BEAT_API IHeartBeat* CreateHeartBeat();
	HEART_BEAT_API void ReleaseHeartBeat(IHeartBeat* pHeart);
}
```

`extern "C"` 为**让C++编译器按照C语言的方式进行名称修饰** 不要用C++的方式修饰函数名 提高接口的兼容性 （不同C++编译器名称修饰规则不同）

##### extern "C"作用

* 确保动态库的**二进制兼容性**
* 支持**动态加载（LoadLibrary/GetProcAddress）**
* 让C++库能被其他语言调用

#### 3. 接口实现

接口实现放在子类的实现中

此处使用了单例模式，但是并非线程安全，因此还是只适用于单线程中调用

单例模式的作用是**控制实例数量** 确保唯一性

```c++
static IHeartBeat* g_pHeartBeat = NULL;

extern "C" {
	IHeartBeat* CreateHeartBeat()
	{
		if(!g_pHeartBeat){
			g_pHeartBeat = new HeartBeatService();
		}
		return g_pHeartBeat;
	}

	void ReleaseHeartBeat(IHeartBeat* pHeart)
	{
		if(pHeart != g_pHeartBeat){
			return;
		}

		delete pHeart;
		g_pHeartBeat = NULL;
	}
}
```

此处还是需加上 extern "C"  和声明保持一致，不一致可能会出现源文件按照C++编译器来修饰名称导致找不到函数报错。

