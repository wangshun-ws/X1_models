# X1（无皮肤版本）模型说明

这个目录同时包含 `URDF` 版本和 `MJCF` 版本的 X1 机器人模型。

## URDF 部分

### 文件组成

- `X1.urdf`
  - 原始 URDF 模型文件。
- `meshes/`
  - URDF 和 MJCF 共用的 STL 网格资源。

### 介绍

`URDF` 部分主要用于：

- 查看原始机器人连杆和关节结构
- 查阅惯量、质量、关节轴等基础参数
- 作为后续整理 `MJCF` 时的参考来源
```
<mujoco>
    <compiler meshdir="meshes/" balanceinertia="true" discardvisual="false"/>
</mujoco>
```
- `balanceinertia=true`表明导出mjcf时对惯性参数进行修正，原本从solidworks里面导出的惯性，有些不满足惯性不等式

### 关节介绍

`X1.urdf` 中的原始关节可以按功能分成 4 组：

| 分组 | 关节 | 类型 | 说明 |
| --- | --- | --- | --- |
| 下肢左腿 | `left_hip_yaw_joint`、`left_hip_roll_joint`、`left_hip_pitch_joint`、`left_knee_joint`、`left_ankle_pitch_joint`、`left_ankle_roll_joint` | revolute | 左腿 6 自由度 |
| 下肢右腿 | `right_hip_yaw_joint`、`right_hip_roll_joint`、`right_hip_pitch_joint`、`right_knee_joint`、`right_ankle_pitch_joint`、`right_ankle_roll_joint` | revolute | 右腿 6 自由度 |
| 躯干 | `waist_yaw_joint`、`torso_joint`、`head_joint` | revolute + fixed | 腰部偏航和躯干、头部连接 |
| 上肢 | `left_shoulder_pitch_joint`、`left_shoulder_roll_joint`、`left_shoulder_yaw_joint`、`left_elbow_joint`、`left_hand_roll_joint`、`right_shoulder_pitch_joint`、`right_shoulder_roll_joint`、`right_shoulder_yaw_joint`、`right_elbow_joint`、`right_hand_roll_joint` | revolute | 双臂共 10 个转动关节 |

如果只看可动关节，原始 URDF 主要包含：

- 下肢 12 个 revolute joint
- 腰部 1 个 revolute joint
- 上肢 10 个 revolute joint

总计 23 个转动关节，另外还有 `torso_joint` 和 `head_joint` 两个固定关节。

### 碰撞体优化（mjcf也是这样的）

![alt text](image/碰撞体.png "碰撞体")

## MJCF 部分

### 文件组成

- `X1.xml`
  - 当前使用的 MuJoCo 主模型文件。
- `scene.xml`
  - MuJoCo 场景入口文件，包含地面、光照和 `X1.xml`。



### 当前模型状态

当前 `X1.xml` 不是对 `X1.urdf` 的原样转换，而是已经按 MuJoCo 调试需求做过修改：

- 根节点加入了 `freejoint`，机器人可以作为自由体下落
- 根节点初始高度设置为 `pos="0 0 0.97"`，避免初始埋地
- 腰部和上肢关节已经删除，这些部分现在是固定连接
- 当前只保留下肢 12 个关节：
  - 左腿：`left_hip_yaw_joint`、`left_hip_roll_joint`、`left_hip_pitch_joint`、`left_knee_joint`、`left_ankle_pitch_joint`、`left_ankle_roll_joint`
  - 右腿：`right_hip_yaw_joint`、`right_hip_roll_joint`、`right_hip_pitch_joint`、`right_knee_joint`、`right_ankle_pitch_joint`、`right_ankle_roll_joint`
- 所有保留下来的下肢关节统一继承：
  - `damping="0.1"`
  - `armature="0.01"`
  - `frictionloss="0.2"`

### 关节介绍

当前 `X1.xml` 的关节配置和原始 URDF 不同，保留情况如下：

| 关节组 | 当前状态 | 说明 |
| --- | --- | --- |
| `root_freejoint` | 保留 | 机器人整体作为自由体运动，用于下落和基础动力学仿真 |
| 下肢 12 个关节 | 保留 | 当前主要控制对象，也是 actuator 作用的关节 |
| `waist_yaw_joint` | 删除 | 腰部固定，不再单独转动 |
| 双臂 10 个关节 | 删除 | 上肢固定，不再单独转动 |

当前保留的下肢 12 个关节作用如下：

| 关节名 | 作用 |
| --- | --- |
| `left_hip_yaw_joint` / `right_hip_yaw_joint` | 控制髋关节绕竖直轴旋转 |
| `left_hip_roll_joint` / `right_hip_roll_joint` | 控制髋关节左右侧倾 |
| `left_hip_pitch_joint` / `right_hip_pitch_joint` | 控制大腿前后摆动 |
| `left_knee_joint` / `right_knee_joint` | 控制膝关节屈伸 |
| `left_ankle_pitch_joint` / `right_ankle_pitch_joint` | 控制踝关节前后摆动 |
| `left_ankle_roll_joint` / `right_ankle_roll_joint` | 控制踝关节左右翻转 |

下肢关节的力矩范围如下，来源于 `/home/wangshun/X1/rl_locomotion/rl_locomotion/legged_lab/assets/X1/urdf/X1.urdf`：

| 关节名 | 力矩范围 |
| --- | --- |
| `left_hip_yaw_joint` / `right_hip_yaw_joint` | `[-126, 126]` |
| `left_hip_roll_joint` / `right_hip_roll_joint` | `[-63, 63]` |
| `left_hip_pitch_joint` / `right_hip_pitch_joint` | `[-252, 252]` |
| `left_knee_joint` / `right_knee_joint` | `[-332.64, 332.64]` |
| `left_ankle_pitch_joint` / `right_ankle_pitch_joint` | `[-228.06, 228.06]` |
| `left_ankle_roll_joint` / `right_ankle_roll_joint` | `[-25.2, 25.2]` |

这些关节在当前 MJCF 中统一使用：

- `damping="0.1"`
- `armature="0.01"`
- `frictionloss="0.2"`

### 执行器配置

当前 `X1.xml` 中已经为下肢 12 个关节配置了 `motor actuator`。

特点如下：

- actuator 类型为 `motor`
- `gear="1"`，控制输入可近似看成直接输出关节力矩
- `ctrlrange` 与 `forcerange` 按各关节力矩范围设置


### 场景文件

`scene.xml` 在 `X1.xml` 基础上补充了场景要素：

- 地面平面
- 基础光照
- 天空盒和地面材质
- 更合适的显示中心和范围

实际使用时，推荐在 MuJoCo 中直接打开 `scene.xml`，不要直接打开 `X1.xml`。


### 已知注意事项

1. 当前 MJCF 中的关节数量已经不同于 URDF。
2. 如果你有外部控制程序按关节下标访问状态，需要同步更新索引。
3. 当前模型主要适合结构调试和基础仿真，后续如果要做稳定站立或步态控制，通常还需要继续检查碰撞体和接触参数。

## 参考关系

- `URDF` 基础参考：`X1.urdf`
- `MJCF` 主模型：`X1.xml`
- `MJCF` 场景入口：`scene.xml`
