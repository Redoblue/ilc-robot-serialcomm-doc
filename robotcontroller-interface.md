# **RobotController 接口类**

------

**RobotController**类通过实现对Windows串口通讯的封装，提供了一个控制第三代机器人底盘的接口。

| Properties | Information |
|--------:|:------------------|
| Header: | \#include "robotcontroller.h" |
| Inherits:     | None |
| Inherited By  | None |  
|Version|1.0|

###**静态常量**

|Type|Name|Value|Description|
|:---|:---|:----|:----------|
|float |LOW_SPEED|0.2|低速度|
|float |MIDDLE_SPEED|0.5|中速度|
|float |HIGH_SPEED|0.8|高速度|

###**公开类型**

| Type | Name |
|-----:|------|
| enum | **SpeedMode** { LOW, MIDDLE, HIGH } |
| enum | **RotationDirection** { CLOCKWISE, COUNTER_CLOCKWISE } |
| enum | **LiftDirection** { UP, DOWN} |
| enum | **LimitState** { LIMIT_NONE_TOGGLED, LIMIT_TOP_TOGGLED, LIMIT_BOTTOM_TOGGLED, LIMIT_UNKNOWN_STATE } |
| enum | **OpenPortState** { PORT_OPEN_SUCCESS, PORT_OPEN_FAILURE } 
###**公开方法**

| Type | Name |
|-----:|------|
|| RobotController() |
|| ~RobotController() |
| RobotController::OpenPortState | InitPort(const int port) const |
| RobotController::OpenPortState | ResetPort(const int port) const |
| void | ClosePort() const |
| void | SetWalkSpeedMode(RobotController::SpeedMode mode) |
|RobotController::SpeedMode| GetWalkSpeedMode()|
|void|SetRotateSpeedMode(RobotController::SpeedMode mode)|
|RobotController::SpeedMode|GetWalkSpeedMode()|
|void|SetLiftSpeedMode(RobotController::SpeedMode mode)|
|RobotController::SpeedMode|GetLiftSpeedMode()|
|void|Walk(WalkFrame &walkFrame)|
|void|Walk(const float normalizedLineSpeed, const float directionAngle, const float normalizedAngularSpeed)|
|void|WalkToward(const float directionAngle, const float normalizedLineSpeed = 0.0f)|
|void|WalkTo(const float directionAngle, const float length, const float normalizedLineSpeed = 0.0f)|
|void|RotateToward(RotateDirection direction, const float normalizedAngularSpeed = 0.0f)|
|void|RotateTo(RotateDirection direction, const float angle, const float normalizedAngularSpeed = 0.0f)|
|void|Lift(LiftFrame &liftFrame)|
|void|Lift(const float normalizedLineSpeed)
|void|LiftToward(LiftDirection direction, const float normalizedLineSpeed = 0.0f)|
|void|LiftTo(LiftDirection direction, const float length, const float normalizedLineSpeed = 0.0f)|
|void|StopWalk()|
|void|StopRotate()|
|void|StopLift()|
|void|StopAll()|
|RobotController::LimitSwitchState|GetLimitSwitchState()|


------

###**详细描述**

**动作**：
**RobotController**类提供了控制底盘移动、旋转、升降的一些方法，这些方法主要有两种定义方式：一种是**只指定方向不指定运动区间**（长度、角度等）的方式，使用这种方式时，底盘将会沿着指定的方向作持续的运动，除非收到停止命令或者其他相互排斥的运动命令（相互排斥的意思是两个动作不能同时执行，如向左走和向右走是一对相互排斥的动作，但向左走和向右转不是一对相互排斥的动作，因此可以同时执行）；第二种方式**既指定方向又指定运动区间**，在这种方式下，底盘会在执行完指定的运动区间后停止运动，等待新的指令下达。

**速度**
在所有关于运动的方法中，都有两种指定运动速度的方法：一种是通过使用**SetWalkSpeedMode()**、**SetRotateSpeedMode()**、**SetLiftSpeedMode()**这些方法分别设定底盘的移动、旋转、升降运动的**全局速度模式**，一旦速度模式被指定，这些运动会默认地运动在相应的速度模式，除非在参数中特别指定运动速度；第二种方式需要在每次调用方法时，**手动指定**该次运动的速度。需要注意的是：在所有公用方法中使用的速度都应该是**归一化速度**，即只需要指定所需速度占最大速度的比率即可，这个值应该在（0.0f~1.0f）之间，为了简化这个过程，类中同时还定义了9个**关于速度的静态变量**，它们分别对应于移动、旋转和升降这三个动作,在指定速度时只需要调用这些静态变量即可。如果既不设定速度模式，又不指定速度，则底盘将使用默认的低速模式。

**坐标**
机器人的行走和旋转动作都可以等效为一个在其自身坐标系中的向量。默认的坐标系是以机器人自身为坐标原点，正右方为弧度0的**极坐标系**，(theta, distance)代表该坐标系中的一个向量（以原点为起点），机器人的每个行走和旋转动作都可以等效表示为这样一个向量，如(PI/2, 0.5f)表示向前行走0.5m。在公开的方法里，有一些函数用后缀**To**修饰,如WalkTo，这些方法表示运动到坐标系中一个指定的点。而在另外一些用**Toward**修饰的方法中，参数一般对应一个坐标导数空间中的点，如(theta, diff(distance, time))，其中第二个参数为行走速度，在只指定方向和速度的情况下，机器人会沿着指定方向持续运动。

**帧**
此版本接口中只有两种硬件组件，分别对应**行走旋转**、**升降**，相应地有两种帧：**WalkFrame**，负责行走和旋转相关的动作设置；**LiftFrame**，负责升降杆动作设置。这两种帧结构非常简单，使用时只需要给相应的属性赋值即可。具体参考**serialframe.h**头文件。

------

###**成员类型说明**

> enum RobotController::SpeedMode

这个枚举常量定义的是底盘运动的全局速度模式。
|Constant|Value|Description|
|:-------|:----|:-----|
|RobotController::LOW|0|低速模式，全局速度为最高速度的20%|
|RobotController::MIDDLE|1|中速模式，最高速度的50%|
|RobotController::HIGH|2|高速模式，最高速度的80%|


------

>enum RobotController::RotateDirection

该枚举变量定义的是底盘旋转的方向，假设从底盘的顶端竖直向下看。
|Constant|Value|Description|
|:----|:----|:----|
|RobotController::CLOCKWISE|0|顺时针旋转|
|RobotController::COUNTER_CLOCKWISE|1|逆时针计旋转|


------

>enum RobotController::LiftDirection

定义升降杆的运动方向
|Constant|Value|Description|
|:----|:----|:----|
|RobotController::UP|0|升降杆上升|
|RobotController::DOWN|1|升降杆下降|


------

>enum LimitSwitchState

定义限位开关的状态
|Constant|Value|Description|
|:----|:----|:----|
|RobotController::LIMIT_NONE_TOGGLED|0|上下限位开关都没有触发|
|RobotController::LIMIT_TOP_TOGGLED|1|上端限位开关被触发|
|RobotController::LIMIT_BOTTOM_TOGGLED|2|下端限位开关被触发|
|RobotController::LIMIT_UNKNOW_STATE|3|限位开关处于不明状态，这里多数会由于硬件干扰而导致|


------

>enum OpenPortState

该枚举变量定义的是打开串口方法所返回的结果
|Constant|Value|Description|
|:----|:-----|:-----|
|RobotController::PORT_OPEN_SUCCESS|0|打开串口成功|
|RobotController::PORT_OPEN_FAILURE|1|打开串口失败|


------

###**成员函数说明**

>RobotController::RobotController()

默认的构造函数。

>RobotController::~RobotController()

默认的析构函数。

>RobotController::OpenPortState RobotController::InitPort(const int port) const

初始化串口操作，接收要使用的串口号作为输入，并返回执行结果。

>RobotController::OpenPortState RobotController::ResetPort(const int port) const

重置指定的串口，并返回重置操作的结果。

>void RobotController::ClosePort() const

关闭串口连接。

>void RobotController::SetWalkSpeedMode(RobotController::SpeedMode mode)

设定底盘移动的速度模式，在调用方法而不指定速度时，将默认使用该速度模式指定行走速度。

>RobotController::SpeedMode RobotController::GetWalkSpeedMode()

获得当前的移动速度模式。

>void RobotController::SetRotateSpeedMode(RobotController::SpeedMode mode)

设置底盘旋转的速度模式，在调用旋转方法而不指定速度时，将默认使用该速度模式指定旋转速度。

>RobotController::SpeedMode RobotController::GetRotateSpeedMode()

获得当前的旋转速度模式。

>void RobotController::SetLiftSpeedMode(RobotController::SpeedMode mode)

设置升降杆升降的速度模式，如果在调用升降方法时不指定速度，将默认使用该速度模式指定速度模式。

>RobotController::SpeedMode RobotController::GetLiftSpeedMode()

获得当前的升降速度模式。

>void RobotController::Walk(WalkFrame &walkFrame)

执行底盘行走动作帧，WalkFrame包含可以同时控制底盘**直线行走速度(m_lineSpeed)、直线行走方向(m_directionAngle)、旋转角速度(m_angularSpeed)**这三个属性，使用时，可以通过这三个属性分别赋值，实现自定义的运动方式。

>void RobotController::Walk(const float normalizedLineSpeed, const float directionAngle, const float normalizedAngularSpeed)

在不构建行走动作帧的情况下，直接使用参数控制底盘运动。

>void RobotController::WalkToward(const float directionAngle, const float normalizedLineSpeed = 0.0f)

朝着指定的方向角**directionAngle**持续前进的方法。速度为可选参数，是归一化后的速度，使用时除了手动指定一个在0.0f~1.0f之间的浮点型数据外，还可以直接调用接口提供的静态速度**LOW_SPEED、MIDDLE_SPEED、HIGH_SPEED**。如果不指定速度，则使用速度模式指定的速度，而如果速度模式也没有设置，则使用默认的低速模式。

>void RobotController::WalkTo(const float directionAngle, const float length, const float normalizedLineSpeed = 0.0f)

行走到一个指定的点，坐标为(directionAngle, length)，即沿着directionAngle方向行走length长度。速度的指定方式见WalkToward函数。

>void RobotController::RotateToward(RotateDirection direction, const float normalizedAngularSpeed = 0.0f)

朝着指定的方向**direction**旋转，速度指定方式和WalkToward相同。

>void RobotController::RotateTo(RotateDirection direction, const float angle, const float normalizedAngularSpeed = 0.0f)

朝着指定的方向旋转某个角度。

>void RobotController::Lift(LiftFrame &liftFrame)

执行升降帧LiftFrame，LiftFrame包含一个浮点型属性**m_lineSpeed**，指定升降杆运动的速度。

>void RobotController::Lift(const float normalizedLineSpeed)

直接使用速度控制升降杆的运动，速度如果为正，则向上运动，否则向下。

>void RobotController::LiftToward(LiftDirection direction, const float normalizedLineSpeed = 0.0f)

升降杆沿着指定方向持续运动,速度指定方式和WalkToward相同。

>void RobotController::LiftTo(LiftDirection direction, const float length, const float normalizedLineSpeed = 0.0f)

升降杆沿着某个方向运动一段距离。

>void RobotController::StopWalk()

停止行走。

>void RobotController::StopRotate()

停止旋转。

>void RobotController::StopLift()

停止升降。

>void RobotController::StopAll()

停止所有动作。

>LimitSwitchState RobotController::GetLimitSwitchState()

获得限位开关状态。参考**enum LimitSwitchState**。

------

**作者**： 包利强
**电话**： 17184037406
**工号**： 11632976

------
