# UE基础

## 基础操作

### 游戏运行时

E 上升 C 下降\
shift f1 退出游戏不中断\
shift ESC 退出游戏\

### 编辑器

`ctrl space`显示内容栏

`按住鼠标左键+前后移`=前后移动\
`左右移`=视角左右移动\
**水平高度不变**

按住右键改变视角角度不改变移动

`同时按住左右键+前后移`=上升,下降\
`左右移`=左右移动,视角方向不变

`按住鼠标左或右键+c`=放大中心点视角,松开鼠标还原

`g`:切换游戏视图

**Bookmarks**\
ctrl+(0-9)设置书签,即固定摄像头位置

`选中物体+按住alt键拖动物体`可复制出一个同样的物体,也可旋转复制\
`按住shift点击多个物体`可对多个物体同时进行操作

`f`:选中物体点击f后可快速贴近观察物体

## 基础知识

摄像头速度=视角移动速度

坐标系用的是左手坐标系

UE中一单位=1cm

## Visual Studio设置

### 注意

- **一定要取消热重载**
- **最好取消实时编译**
**容易出问题的地方**
- 改变`UPROPERTY`参数时使用热重载会崩溃
- 改变蓝图类结构(添加成员)使用热重载可能会崩溃

**解决办法:** 退出UE编辑器,在vs中编译并启动UE

## 创建蓝图类

先创建c++类然后创建基于c++的蓝图类

## 调试

- **临时日志输出**:

```cpp
UE_LOG(LogTemp, Warning, TEXT("Begin Play called!"));
```

临时日志中以警告方式出现

- **屏幕输出**

```cpp
if (GEngine)
{
 GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Red, FString("String on Screen!"));
}
```

1是键值,对应屏幕输出位置

## 字符串格式化

```cpp
UE_LOG(LogTemp, Warning, TEXT("DeltaTime: %.2f"), DeltaTime);

if (GEngine)
{
 FString Message = FString::Printf(TEXT("DeltaTime: %f"), DeltaTime);
 GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Red, Message);
}

FString Name = GetName();
FString Message = FString::Printf(TEXT("DeltaTime: %s"), *Name);
```

**注意**:GetName()是获取实体名字的函数,*号是将FString转换成cString的运算符

- **GetWorld()**
获取当前actor所在世界(UWorld*类型),如果没生成返回null

- **GetActorLocation()**
获取当前actor所在位置信息,返回FVector.

- **DrawDebugSphere(UWorld\*,Fvector,R(float),线段数量,Fcolor,持久性(bool)(false后面接浮点数秒数))**
画调试球体

1. **DrawDebugLine()**
2. **DrawDebugPoint()**
3. **SetActorLocation()**

传入Fvector();
**SetActorRotation(FRotator(0.f, 90.f, 0.f));**

改变角度\
**AddActorWorldOffset()**\
**AddActorWorldRotation()**

## 调试宏

```cpp
#pragma once
#include "DrawDebugHelpers.h"

#define DRAW_SPHERE(Location) if(GetWorld()) { DrawDebugSphere(GetWorld(), Location, 50.f, 12, FColor::Red, true, -1.f); };
#define DRAW_SPHERE_SingleFrame(Location) if(GetWorld()) { DrawDebugSphere(GetWorld(), Location, 50.f, 12, FColor::Red, false, -1.f); };
#define DRAW_LINE(Start, End) if(GetWorld()) { DrawDebugLine(GetWorld(), Start, End, FColor::Red, true, -1.f); };
#define DRAW_LINE_SingleFrame(Start, End) if(GetWorld()) { DrawDebugLine(GetWorld(), Start, End, FColor::Red, false, -1.f); };
#define DRAW_POINT(Location) if(GetWorld()) { DrawDebugPoint(GetWorld(), Location, 10.f, FColor::Red, true, -1.f); };
#define DRAW_POINT_SingleFrame(Location) if(GetWorld()) { DrawDebugPoint(GetWorld(), Location, 10.f, FColor::Red, false, -1.f); };
#define DRAW_VECTOR(Start,End) if(GetWorld()) { DrawDebugDirectionalArrow(GetWorld(), Start, End, 50.f, FColor::Red, true, -1.f); };
#define DRAW_VECTOR_SingleFrame(Start,End) if(GetWorld()) { DrawDebugDirectionalArrow(GetWorld(), Start, End, 50.f, FColor::Red, false, -1.f); };
```

```cpp
void AItem::Tick(float DeltaTime)
{
 Super::Tick(DeltaTime);
 float speed = 50.f;
 float swift = 45.f;
 AddActorWorldRotation(FRotator(swift * DeltaTime, 0.f,  0.f));
 //避免帧数不同导致单位之间内位移量不同
 AddActorWorldOffset(FVector(speed * DeltaTime, 0.f, 0.f));
 DRAW_SPHERE_SingleFrame();
 DRAW_VECTOR_SingleFrame();
}
```

## 正弦函数产生循环

```cpp
void AItem::Tick(float DeltaTime)
{
 Super::Tick(DeltaTime);
 RunningTime += DeltaTime;
 float DeltaZ = Amplitude * FMath::Sin(TimeConstant * RunningTime);
 AddActorWorldOffset(FVector(0.f, 0.f, DeltaZ));
 DRAW_SPHERE_SingleFrame();
 DRAW_VECTOR_SingleFrame();
}
```

## c++属性添加到蓝图属性栏

```cpp
//只能修改默认值,在蓝图中修改
UPROPERTY(EditDefaultsOnly)
//能修改单个实例属性
UPROPERTY(EditInstanceOnly)
//两边都能修改
UPROPERTY(EditAnywhere)
//只能查看不能修改 
UPROPERTY(VisibleDefaultsOnly)
```

## 暴露c++属性到蓝图图表中

要使用读写功能属性不能是私有
Category参数改变该属性所在蓝图属性栏目名称

```cpp
UPROPERTY(EditAnywhere,BlueprintReadOnly,Category="name")//只读
UPROPERTY(EditAnywhere,BlueprintReadWrite)//读写
UPROPERTY(EditAnywhere,BlueprintReadOnly,meta=(AllowPrivateAcess = "true"))//让私有变量可以在蓝图中被访问,UE5.5似乎没有该meta功能
```

## 暴露c++类函数到蓝图图表中

```cpp
UFUNCTION(BlueprintCallable)//适用于修改状态的操作(如触发事件、改变变量)
UFUNCTION(BlueprintPure)//适用于数据获取、计算、逻辑判断等无状态的操作
```

## C++Template

```cpp
Template <typename T>
T Avg(T a,T b)
{
    return (a+b)>>1;
}
int result = <Avg<int>(1,2);
```
