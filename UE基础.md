# UE基础

## 基础操作

### 游戏运行时

E 上升 C 下降\
shift f1 退出游戏不中断\
shift ESC 退出游戏

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

- 摄像头速度=视角移动速度
- 坐标系用的是左手坐标系
- UE中一单位=1cm
- 头文件必须放在.generated.h前面

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

### 字符串格式化

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

## Actor

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

### 调试宏

```cpp
#pragma once
#include "DrawDebugHelpers.h"//这个头文件必须有

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

### 正弦函数产生循环

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

### c++属性添加到蓝图属性栏

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

### 暴露c++属性到蓝图图表中

要使用读写功能属性不能是私有
Category参数改变该属性所在蓝图属性栏目名称

```cpp
UPROPERTY(EditAnywhere,BlueprintReadOnly,Category="name")//只读
UPROPERTY(EditAnywhere,BlueprintReadWrite)//读写
UPROPERTY(EditAnywhere,BlueprintReadOnly,meta=(AllowPrivateAcess = "true"))//让私有变量可以在蓝图中被访问,UE5.5似乎没有该meta功能
```

### 暴露c++类函数到蓝图图表中

```cpp
UFUNCTION(BlueprintCallable)//适用于修改状态的操作(如触发事件、改变变量)
UFUNCTION(BlueprintPure)//适用于数据获取、计算、逻辑判断等无状态的操作
```

### C++Template

```cpp
Template <typename T>
T Avg(T a,T b)
{
    return (a+b)>>1;
}
int result = <Avg<int>(1,2);
```

### Component

```cpp
UPROPERTY(VisibleAnywhere)//参与反射回收,暴露给编辑器
UStaticMeshComponent* ItemMesh;

//工厂模式,相当于new,TEXT里面是名称
ItemMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMesh"));
//根组件被回收,ItemMesh成为根
RootComponent = ItemMesh;
```

实际使用一般是c++创组件,蓝图选择Mesh

## Pawn

### Capsule 胶囊组件

**头文件:**`#include "Components/CapsuleComponent.h"`

.h文件中

```cpp
private:
UPROPERTY(VisibleAnywhere)
UCapsuleComponent* Capsule;
```

.cpp文件中,构造函数里面

```cpp
Capsule = CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule"));
Capsule->SetCapsuleHalfHeight(20.f);//设置长度
Capsule->SetCapsuleRadius(15.f);//设置半径
SetRootComponent(Capsule);//比直接赋值应对场景多
```

### Forward Declaration

**定义:** 它允许你在文件的一部分声明一个类型(比如类、结构体、函数等),而不需要提供其完整的定义.这在大型项目中非常有用,可以减少头文件的相互依赖,加快编译速度.

防止在头文件中引用头文件导致引入过多不必要代码\
用法类似提前声明函数,后面实现\
.h文件中`class UCapsuleComponent;`\
之后便可以声明指针,在cpp文件中引用头文件

### Skeletal Mesh Components

骨骼网格组件

```cpp
//.h
class USkeletalMeshComponent;
UPROPERTY(VisibleAnywhere)
USkeletalMeshComponent* BirdMesh;
//.cpp
#include "Components/SkeletalMeshComponent.h"
ABird::ABird()
{
BirdMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("BirdMesh"));
BirdMesh->SetupAttachment(GetRootComponent());//使骨骼跟根组件移动
}
```

**增加动画:** 蓝图Animation中AnimationMode选用 Use Animation Asset可以使用动画资源

### 前后移动

UE编辑器里:Edit->Project Settings->Input->AxisMappings

```cpp
//.h
void MoveForward(float Value);//要重写

//.cpp
void ABird::MoveForward(float Value)
{
    if (Controller && Value != 0.f)
    {
        FVector Forward = GetActorForwardVector();
        AddMovementInput(Forward, Value);
    }
}

void ABird::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);

    PlayerInputComponent->BindAxis(FName("MoveForward"), this, &ABird::MoveForward);
}
```

### Camera And Spring Arm

**蓝图中:** 添加spring arm组件和camera组件,并将camera组件拖到arm上

```cpp
//.h
//先声明
class USpringArmComponent;
class UCameraComponent;

UPROPERTY(VisibleAnywhere)
USpringArmComponent* SpringArm;

UPROPERTY(VisibleAnywhere)
UCameraComponent* ViewCamera;

//.cpp
#include "GameFramework/SpringArmComponent.h"
#include "Camera/CameraComponent.h"

Bird::ABird()
{
    PrimaryActorTick.bCanEverTick = true;
    SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
    SpringArm->SetupAttachment(GetRootComponent());
    SpringArm->TargetArmLength = 300.f;

    ViewCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ViewCamera"));
    ViewCamera->SetupAttachment(SpringArm);

    AutoPossessPlayer = EAutoReceiveInput::Player0;
}
```

### Add Controller Input

**内容:** 用鼠标控制控制器的角度

Edit->Project Settings->Input->Axis Mappings中,鼠标x,y,对应鼠标
左右移动速度,前后移动速度,同时要使得控制器同步变化需再Pawn蓝图中勾选`Use Controller Rotation xxx`

c++
```cpp
//.h
void Turn(float Value);
void LookUp(float Value);

//.cpp
void ABird::Turn(float Value)
{
	AddControllerYawInput(Value);//更改yaw
}

void ABird::LookUp(float Value)
{
	AddControllerPitchInput(Value);//更改pitch
}

void ABird::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(FName("Turn"), this, &ABird::Turn);//吸附
	PlayerInputComponent->BindAxis(FName("LookUp"), this, &ABird::LookUp);
}
```

### 游戏模式
一般情况下游戏用的是默认模式,也就是说会有默认Pawn出现,要想改默认模式
就需要创建一个`game mode base`类,修改后将游戏模式指定为该模式就行\
**坏处:** 在大世界中,如果你拖入的你设定的pawn实例进入另一个区域后,
会导致pawn消失,除非你回到原来区域