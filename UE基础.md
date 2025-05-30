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
- 命令行`show collison`查看碰撞体
- `slomo+float`放缓时间到float倍

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
- 如果是全项目范围,修改 Project Settings > Maps & Modes 的 Default Pawn Class.
- 如果是单个关卡范围,修改 World Settings > Game Mode Override.
- 如果是运行时动态切换,使用蓝图或 C++ 实现.

## Character
角色默认拥有网格和骨骼

取消`Use Controller Rotation x,y,x`
```cpp
bUseControllerRotationPitch = false;
bUseControllerRotationYaw = false;
bUseControllerRotationRoll = false;
```

不取消会使角色和镜头一起移动、
**旋转弹簧臂而不使角色旋转:** `Use Pawn Controller Rotation`\
右移
```cpp
void ASlashCharacter::MoveRight(float Value)
{
	if (Controller && Value != 0.f)
	{
		FVector Right = GetActorRightVector();
		AddMovementInput(Right, Value);
	}
}
```

了解旋转矩阵概念

### Controller Directions
控制器方向同步
```cpp
void ASlashCharacter::MoveForward(float Value)
{
	if (Controller && Value != 0.f)
	{
		// find out which way is forward
		const FRotator ControllerRotation = GetControlRotation();//得到控制器方向
		const FRotator YawRotation(0.f, ControllerRotation.Yaw, 0.f);//得到单个方向

		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);//转换成Vector,向右移动改成EAxis::Y即可
		AddMovementInput(Direction, Value);
	}
}
```
同步角色朝向和控制器方向
```cpp
#include "GameFramework/CharacterMovementComponent.h"
ASlashCharacter::ASlashCharacter()
{
	PrimaryActorTick.bCanEverTick = true;
	GetCharacterMovement()->bOrientRotationToMovement = true;//使角色朝向跟随移动方向改变
	GetCharacterMovement()->RotationRate = FRotator(0.f, 400.f, 0.f);//改变旋转速度

}
```

### Hair And Eyebrows
添加头发和眉毛到角色\
**添加前:** 在Slash.Build.cs文件中,添加`"HairStrandsCore"`在
`PublicDependencyModuleNames.AddRange`中\
生成文件(可删除)
- Binaries
- Intermediate
- Saved

每次往构建文件中添加东西,最好删掉这些再重新编译\删除后右键点击`Slash.uproject`生成解决方案,然后双击打开该项目重建模块游戏打开后关闭ue5用vs重新编译\
更改头发颜色(更改材质)

## Animation

1. 先创建动画蓝图(`Animation`->`AnimationBP`->选择骨架)
2. 打开蓝图编辑器,进入`Mesh`视图,切换`Animation Mode`->`Use Animation Blueprint`->选择刚创建的动画蓝图
3. 添加对象变量类型'BP Slash Character`
4. 添加`Event Blueprint Initialize Animation`事件,将`Try Get Pawn Owner`连到`Cast To BP_SlashCharacter`上进行转换后连到`Set Character`上,再连到`Get Character Movement`上面,创建`Character Movement Component`类型变量,最后连到`Set MovementComponent`变量上,Set相当于给变量赋值
5. ps:没找到`Get Character Movement`是因为开了`Context Sensitive`
6. ![图示](./Image/image1.png)//漏连set了
7. ![获取速度](./Image/image.png)
8. 7将Event update连到set

然后建立状态机,建立状态,加入状态动画,加入转换规则,动画记得选中循环播放

### Anim Instance

父类改为 Anim Instance\
编译任何动画实例修改前先关闭编辑器,蓝图中的update函数是一直运行的,会影响c++,出现终端删除缓存文件删除`.vs`,`sln`,启动ue5后重新打开解决方案,关闭ue5再强制编译`(ctrl+f5)`

```cpp
//.h
#pragma once

#include "CoreMinimal.h"
#include "Animation/AnimInstance.h"
#include "SlashAnimInstance.generated.h"

/**
 * 
 */
UCLASS()
class SLASH_API USlashAnimInstance : public UAnimInstance
{
	GENERATED_BODY()

public:
	virtual void NativeInitializeAnimation() override;
	virtual void NativeUpdateAnimation(float DeltaTime) override;
	
	UPROPERTY(BlueprintReadOnly)
	class ASlashCharacter* SlashCharacter;

	UPROPERTY(BlueprintReadOnly, Category = "Movement")
	class UCharacterMovementComponent* SlashCharacterMovement;

	UPROPERTY(BlueprintReadOnly, Category = "Movement")
	float GroundSpeed;
};
```

```cpp
//.cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Characters/SlashAnimInstance.h"
#include "Characters/SlashCharacter.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "Kismet/KismetMathLibrary.h"

void USlashAnimInstance::NativeInitializeAnimation()
{
	Super::NativeInitializeAnimation();

	SlashCharacter = Cast<ASlashCharacter>(TryGetPawnOwner());//类型转换
	if (SlashCharacter)
	{
		SlashCharacterMovement = SlashCharacter->GetCharacterMovement();//获取移动组件
	}
}

void USlashAnimInstance::NativeUpdateAnimation(float DeltaTime)
{
	Super::NativeUpdateAnimation(DeltaTime);

	if (SlashCharacterMovement)
	{
		GroundSpeed = UKismetMathLibrary::VSizeXY(SlashCharacterMovement->Velocity);//获取速度
	}
}
```

### Jump
使用`Action Mappings`,适用于单次操作触发的动作,同时会生成一个 `Jump`蓝图时间类属`Action Event`\
**蓝图:** 直接在角色蓝图中将`Jump`函数连接到内置的`Character::Jump`函数上\
**c++:**`SlashCharacter`中绑定`Jump`函数,用系统自带函数获取IsFalling变量信息,将移动的状态机存在缓存中,在创建主状态机包含地面状态和空中状态.

### Inverse Kinematics
解决上楼梯脚步悬空问题\
先创建一个`Animation`的Controll Rig,在`Forward Slove Graph`导入相应角色`Mesh`,在里面创建函数`IKFootTrace`,![alt text](image.png)
不会碰到Bone可能是因为以本身做碰撞检测
![alt text](image-1.png)
![alt text](image-2.png)
![alt text](image-3.png)
![alt text](image-4.png)
最低点是离地最近的\
将`ShouldDoIKTrace`设为全局可看
![alt text](image-5.png)
![alt text](image-7.png)
`settings`那里改成`pin to input`
![alt text](image-6.png)
微调需要在`FootTrace`函数里面调增碰撞检测起点和终点\
**脚掌贴地面**未实现

## Section 9

### Collision Presets
1. 拖动网格体进世界会自动生成`Actor`类,默认带网格体组件,网格体组件里面有`Collision Presets`选项
- `No Collsion`有碰撞
- `Query`仅可以进行空间查询,上一节检测脚到地面距离,属于简单碰撞,可阻挡
- `Physic`无碰撞
- `Enable`两者都发生

### Collision Responses
对各种对象的碰撞反应

### OverlapEvents
重叠事件
`On Component Begin Overlap`某个物体发生重叠时发生的事件
![alt text](image-8.png)

### On Component Begin Overlap
1. 解决方案中搜素`PrimitiveComponent.h`文件中找到对应委托的参数列表
2.  声明函数
3.  绑定函数
```cpp
//Item.h
class USphereComponent;
UPROPERTY(VisibleAnywhere)
USphereComponent* Sphere;

//Item.cpp
#include "Components/SphereComponent.h"

AItem::AItem()
{
	rimaryActorTick.bCanEverTick = true;
	Sphere = CreateDefaultSubobject<USphereComponent>(TEXT("Shere"));
	Sphere->SetupAttachment(GetRootComponent());

}

void AItem::BeginPlay()
{
	Super::BeginPlay();

	//绑定回调函数到委托
	Sphere->OnComponentBeginOverlap.AddDynamic(this, &AItem::OnSphereOverlap);
}

void AItem::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FString OtherActorName = OtherActor->GetName();
	if (GEngine)
	{
		//打印名字在屏幕上
		GEngine->AddOnScreenDebugMessage(1, 30.f, FColor::Red, OtherActorName);
	}
}
```
`End`同理

## Weapon

### The Weapon Class

`Weapon`继承`Item`,继承的函数必须是虚函数,且重载的
函数不能加`UFUNCTION`

### Sockets
插槽,找到骨骼,找到手右键`add socket`添加插槽,可以预览加上武器动画,和动作

### Mixamo
一个获取动画资源网站,分为角色和动画,角色页面可以获取网格体,又有`mixamo`的网格体才能用对应动画,下载对应动画后可以在游戏内导入,导入动画时要选择SK即骨骼,是`mixamo`里下的,

### IK Rig
将一个骨架动画转移另一个`Retargeting`,需要用到`IK Rig`, 动画->重定向->IK绑定,右键`Animation->IK Rig->选择IK mesh`,`retarget root`反向目标根节点是绑定其他所有骨骼的参考框架,右键骨骼`Set`,`chain`为骨骼链,
![alt text](image-9.png)
再给想要转移动画的角色重复该操作,同样要创建IK,需要与`Maxamo IK`绑定中创建的链条对应的链条,只有一个骨骼的链叫做根,`retarget root`位置和`mixamo`骨骼位置一致,`root chain`是最上面的骨骼,链名字要一样,`twist`骨骼会打断`chain`,手动修改起点和终点

### IK Retargeter
重定向动画到角色
1. 在`Rigs`里面创建`Retargeter`
2. 点击进去设置`target`为想要转移到的角色上面
3. 如果角色初始姿势不对,左上角`Auto Align`自动对齐动作
4. 导出重定向动画到一个文件夹,以后角色就可以用里面的动画了

### Attaching the Sword
捡起武器,蓝图:
![alt text](image-10.png)
```cpp
#include "Characters/SlashCharacter.h"
void AWeapon::OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{	
	Super::OnSphereBeginOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);

	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);
	if (SlashCharacter)
	{
		FAttachmentTransformRules FATR(EAttachmentRule::SnapToTarget, true);
		ItemMesh->AttachToComponent(SlashCharacter->GetMesh(), FATR, FName("RightHandSocket"));
	}
}
```

### Picking Up Items

**捡起物体,思路:** 在`SlashCharacter`中创建一个变量存储重叠的物体类型,
在.h文件中创建一个`Set`函数设置类型,在重叠开始后设置对应类型,结束后设置为
空,每次按e判断变量属于哪个类型,执行相应操作

```cpp
void AWeapon::Equip(USceneComponent* InParent, FName InSocketName)
{
	FAttachmentTransformRules FATR(EAttachmentRule::SnapToTarget, true);
	//吸附对象,枚举类型,插槽名称
	ItemMesh->AttachToComponent(InParent, FATR, InSocketName);
}

//强制内联
FORCEINLINE void SetOverlappingItem(AItem* Item) { OverlappingItem = Item; }
```

### Enum for Character State
枚举角色状态
`enum class`作用域枚举 
单独放在一个头文件里面,减少耦合度同时避免头文件不必要的引入,\
`SlashInstance.h 和 SlashCharacter.h`中需要引入,同时在关键\
动作发生时更新对应状态,UE枚举类型的命名规范
```cpp
UENUM(BlueprintType)//在蓝图中当类型使用
enum class ECharacterState : uint8//8位无符号整型
{
	ECS_Unequipped UMETA(DisplayName = "Unequipped"),//更改在蓝图中名称
	ECS_EquippedOneHandedWeapon UMETA(DisplayName = "Equipped One-Handed Weapon"),
	ECS_EquippedTwoHandedWeapon UMETA(DisplayName = "Equipped Two-Handed Weapon")
};
```

### Switch Animation Poses

![alt text](image-11.png)

### Equiped Animations
`mixamo`带移动的动画要想保持原地移动需要勾选`in place`选项 
`controll rig`导致的下半身不动的问题未解决(已解决,\
忘记开启动画循环了)

### Multiple Animation Blueprints
使用多个动画蓝图处理不同内容.
1. 创建新动画蓝图,复制部分原蓝图过去
2. `tips:`复制过来的蓝图可能还有某些未声明的变量,\
此时可以通过编译查错快速定位未声明变量位置,\
然后右键变量快速创建变量
3. 一个蓝图链接到另一个蓝图
4. ![](image-12.png)
`Linked Anim Graph`
5. 绑定两者之间相同变量
6. ![alt text](image-13.png)
7. 传入缓存动画,`Input Pose`
8. ![alt text](image-14.png)
9. ![alt text](image-15.png)

## Montage

### Animation Montages

先创建`Animation Montages`,然后在里面添加动画,
蓝图:
![alt text](image-16.png)
```cpp

#include "Animation/AnimMontage.h"
void ASlashCharacter::Attack()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance)
	{
		AnimInstance->Montage_Play(AttackMontage);
		int32 Selection = FMath::RandRange(1, 2);
		FName SectionName;
		switch (Selection)
		{
		case 1:
			SectionName = "Attack1";
			break;
		case 2:
			SectionName = "Attack2";
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(SectionName,AttackMontage);
	}

	UPROPERTY(EditDefaultsOnly, Category = "Montages")
UAnimMontage* AttackMontage;
}
记得给蓝图中的`AttackMontage`赋值.
```

### Animation Notifies

在`Notify`那一栏右键可以创建,
要在蓝图中访问枚举量必须在枚举类型前加`UENUM(BlueprintType)`
![alt text](image-17.png)
修改成只有在装备武器时能发起攻击

### Item State

给物体添加状态
```cpp
if (ActionState == EActionState::EAS_Attacking)return;
```
禁止移动攻击

### Audio

添加声音,在蒙太奇里添加特殊通知播放声音,点击\
通知选择声音导入,想要调大声音可以更改资产属性`Volume`,\
`pich`是声调,想要不改变资产更改声音属性,添加`Sound Cue`,
![alt text](image-18.png)
虚幻5新增`Metasounds`
![alt text](image-19.png)
声音数组,未做用力攻击声音
![alt text](image-20.png)
同样可以在通知里面加入`niagara`粒子特效
![alt text](image-21.png)
`foot_l`是粒子生成的地方

### Fix Foot Placement
解决罗圈腿问题(原因不明)
![alt text](image-22.png)

### Putting the Sword Away
装备与卸下武器,寻找动画,制作蒙太奇,导入动画,设置`SectionName`,设置相应变量,实施相应动画,在`BP_SlashCharacter`里面设置蒙太奇,注意置空`OverlappingItem`

### Attaching the Sword to Back
1. 在背部某个骨骼上建立插槽
2. 边调整插槽位置边对比动画
3. 在`EquipMontage`添加合适的通知点
4. 在c++中完成转换插槽的函数
5. 在`ABP_SlashCharacter`事件中实现调用

### Equip Sounds

捡起金属声音
![alt text](image-23.png)

设置碰撞预设
```cpp
	if (Sphere)
	{
		Sphere->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	}
```

c++添加声音
```cpp
//weapon.h
UPROPERTY(EditAnywhere, Category = "Weapon Properties")
USoundBase* EquipSound = nullptr;
//weapon.cpp
#include "Kismet/GameplayStatics.h"
void AWeapon::Equip(USceneComponent* InParent, FName InSocketName)
{
	ItemState = EItemState::EIS_Equipped;
	AttachMeshToSocket(InParent, InSocketName);
	if (EquipSound)
	{
		UGameplayStatics::PlaySoundAtLocation(this, EquipSound, GetActorLocation());
	}
	if (Sphere)
	{
		Sphere->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	}
}
```

### Edit Animation
修改动画\
在动画里选中骨骼后点击+key修改关键帧
![alt text](image-24.png)

保存修改的动画
![alt text](image-25.png)

## 处理武器碰撞

### Trace
追踪技术,`On component Begin Overlap`需要右键对应组件添加
蓝图:
![alt text](image-26.png)
如果有一个角色既有网格体又有胶囊会出bug,只和胶囊发生重叠,将`Object Type`改为 `World Dynamic`,同时开启`Generate Overlapping`,`Visibility`开启,`Weapon`设置ignore`pawn`类型防止和胶囊触发重叠事件
```cpp
//Weapon.h
class UBoxComponent;

public:
	AWeapon();

protected:
	virtual void BeginPlay() override;

UFUNCTION()
void OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

	UPROPERTY(VisibleAnywhere, Category = "Weapon Properties")
	UBoxComponent* WeaponBox = nullptr;

	UPROPERTY(VisibleAnywhere)
	USceneComponent* BoxTraceStart = nullptr;

	UPROPERTY(VisibleAnywhere)
	USceneComponent* BoxTraceEnd = nullptr;

//Weapon.cpp
AWeapon::AWeapon()
{
	WeaponBox = CreateDefaultSubobject<UBoxComponent>(TEXT("WeaponBox"));
	WeaponBox->SetupAttachment(GetRootComponent());
	WeaponBox->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
	//设置对全部通道的碰撞响应为重叠
	WeaponBox->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Overlap);
	//设置对玩家的碰撞响应为无视
	WeaponBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Ignore);

	BoxTraceStart = CreateDefaultSubobject<USceneComponent>(TEXT("Box Trace Start"));
	BoxTraceStart->SetupAttachment(GetRootComponent());
	BoxTraceEnd = CreateDefaultSubobject<USceneComponent>(TEXT("Box Trace End"));
	BoxTraceEnd->SetupAttachment(GetRootComponent());
}

void AWeapon::BeginPlay()
{
	Super::BeginPlay();

	WeaponBox->OnComponentBeginOverlap.AddDynamic(this, &AWeapon::OnBoxOverlap);
}

void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();
	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.Add(this);
	FHitResult BoxHit;
	UKismetSystemLibrary::BoxTraceSingle(
		this,
		Start,
		End,
		FVector(5.f, 5.f, 5.f), //	Box Extent
		BoxTraceStart->GetComponentRotation(), //	Orientation
		ETraceTypeQuery::TraceTypeQuery1, //	Trace Type
		false, //	Trace Complex
		ActorsToIgnore, //	Actors to Ignore
		EDrawDebugTrace::ForDuration, //	Debug Type
		BoxHit, //	Out Hit Result
		true //	IbIgnoreSelf
		);
}
```

### interface 
被碰撞对象要触发某种事件时用接口,接口只包含函数声明,而不实现函数
```cpp
/**
 * 在这里写入想要当作接口的函数.
 */
class SLASH_API IHitInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	virtual void GetHit() = 0;
};
```