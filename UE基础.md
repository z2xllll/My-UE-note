# 创建蓝图类

先创建c++类然后创建基于c++的蓝图类

## 调试

**临时日志输出:**

```cpp
UE_LOG(LogTemp, Warning, TEXT("Begin Play called!"));
```

临时日志中以警告方式出现
**屏幕输出**

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

注意:GetName()是获取实体名字的函数,*号是将FString转换成cString的运算符

**GetWorld()**
获取当前actor所在世界(UWorld*类型),如果没生成返回null

**GetActorLocation()**
获取当前actor所在位置信息,返回FVector.

**DrawDebugSphere(UWorld\*,Fvector,R(float),线段数量,Fcolor,持久性(bool)(false后面接浮点数秒数))**
画调试球体

**DrawDebugLine()**
**DrawDebugPoint()**
**SetActorLocation()**
传入Fvector();
**SetActorRotation(FRotator(0.f, 90.f, 0.f));**
改变角度
**AddActorWorldOffset()**
**AddActorWorldRotation()**
改变角度

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

## c++属性添加到蓝图

```cpp
//只能修改默认值,在蓝图中修改
UPROPERTY(EditDefaultOnly)
在想要设置的属性上一行加
//能修改单个实例属性
UPROPERTY(EditInstanceOnly)
//两边都能修改
UPROPERTY(EditAnywhere)
