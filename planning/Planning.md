# Planning模块解析

## PlanningComponent

**功能：**Planning模块主入口

**输入：**

- Localization定位
- 障碍物Perception及Prediction信息
- 底盘信息chassis
- HD_map高精度地图

**输出：**

- ADCTrajectory轨迹信息
- RoutingRequest路径规划请求，用于给Routing模块重新进行路线规划导航
- PlanningLearningData *TODO*

**主要成员：**

- injector_: 车辆运行信息（即自动驾驶程序的运行信息）
- planning_base_: 规划器
- config_: 配置文件

**主要方法：**Init方法注册各类reader和writer, 进行消息监听和发送。监听器接收到消息之后就在Proc方法中进行Planning主流程：

- 检查是否Rerouting
- 输入数据载入local_view并检查
- planning_base_->RunOnce
- 发布ADCTrajectory并添加History



## OnLanePlanning: public PlanningBase

**功能：**规划器，分为几个类型，由对应FLAG决定，此处以OnLanePlanning为例解释

**输入：**

- injector车辆运行信息
- local_view各类客观信息（即PlanningComponent的输入）
- config配置

**输出：**ADCTrajectory

**主要成员：**Frame, 规划器通过维护frame来维护各种有用的信息

**主要方法：**

- Init初始化各类中间信息(history, status等)，加载地图等信息，并启动**参考线(ReferenceLine)**线程用以提供用作Planning基准规划线路的参考线。然后分配具体的Planner，并判断是否为E2E模式*(TODO)*
- RunOnce：
  - 初始化Frame
  - 判断是否符合交通规则*(TrafficDecider, TODO)*
  - 执行Planner的Plan, 生成ptr_trajectory_pb
  - 后处理ptr_trajectory_pb生成轨迹



## ReferenceLine*(TODO)*

**功能：**参考线用于提供一条基准路径，供Planning模块在基准路径基础上实现Plan



## TrafficDecider*(TODO)*

**功能：**根据当前客观条件判断是否符合若干个给定的交通规则



## PublicRoadPlanner: public Planner

**功能：**场景规划器，由规划器类型和配置决定，由OnLanePlannerDispatcher分配

**输入：**

- config配置
- planning_start_point开始规划的地点
- frame帧信息

**输出：**ptr_computed_trajectory轨迹信息

**主要成员：**

- scenario_manager, 用于推进场景更迭
- scenario_, 当前场景

**主要方法：**Plan: 

- scenario_manager->Update
- scenario_->Process
- 若scenario状态为STATUS_DONE, 则再一次scenario_manager->Update



## ScenarioManager

**功能：**场景(Scenario)表示了当前车辆处在的一个宽泛的状态，例如跟车、超车、红绿灯路口左（右）转等。ScenarioManager用于管理场景

**输入：**配置文件

**输出：**当前Scenario

**主要方法：**Update:

- Observe: 初始化first_encountered_overlap_map_*(TODO)*
- ScenarioDispatch: 
  - E2E模式检查
  - 更新场景：
    - 若非默认场景(LANE_FOLLOW)或PULL_OVER，则保持执行该场景
    - 若为默认场景，进行判断，从而决定是否进入某些特定场景*(TODO: Pad)*
  - 更新上下文

