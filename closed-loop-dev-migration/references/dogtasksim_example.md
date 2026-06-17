# DogTaskSim 迁移实例

> 本文档记录了将闭环研发方法论迁移到 DogTaskSim 项目的完整过程。
> 作为迁移 skill 的参考示例。

---

## 项目概况

**DogTaskSim** 是一个四足机器人（Go2）+ 机械臂（D1）的移动抓取仿真平台。

**技术栈**：
- 仿真引擎：MuJoCo 3.9.0
- 调度后台：FastAPI (port 8200)
- 仿真服务器：FastAPI (port 8100)
- 导航算法：NavigationController
- 抓取算法：GraspPlanner + IK Solver
- RL Policy：18 DOF (12 腿 + 6 臂)
- 配置：TOML (config/robots/sim_go2_d1.toml)
- 环境：conda `mower`

**任务流程**：
导航到目标 → 检测目标 → 抓取 → 返回起点

---

## 迁移步骤

### Step 1: 架构分析 → 信号链 4 层

```
第一层：调度/FSM 层
  数据源：scheduler.log, task_run.jsonl, SSE 事件
  判断：FSM 状态转换是否正确？超时？重试？

第二层：规划/算法层（导航 + IK + 抓取规划）
  数据源：nav_trajectory, ik_solutions, grasp_plan
  判断：导航路径是否合理？IK 是否可解？

第三层：PD 控制/执行层
  数据源：joint_positions, joint_velocities, PD targets
  判断：PD 跟踪是否到位？关节速度/力矩是否超限？

第四层：MuJoCo 物理层
  数据源：qpos, qvel, contact forces, body positions
  判断：接触力是否异常？是否有穿模？
```

### Step 2: PR 里程碑设计

| PR | 目标 | 门控条件 |
|----|------|---------|
| PR0 | 基础仿真跑通 | 行走不摔倒，相机输出正常 |
| PR1 | 导航精度 | 到达误差 < 0.3m，朝向误差 < 15° |
| PR2 | 感知检测 | 检测到目标，坐标变换误差 < 5cm |
| PR3 | 抓取全流程 | IK 成功率 > 80%，抓取动作完整 |
| PR4 | 闭环任务 | FSM FINISHED，总耗时 < 120s |
| PR5 | Sim2Real 对齐 | 仿真与实机行为一致 |
| PR6 | 场景泛化 | 多场景成功率 > 70% |
| PR7 | 鲁棒性 | 异常注入后能恢复 |

### Step 3: 视频录制方案

使用 MuJoCo native rendering (EGL headless)：
- 第三视角（tracking camera）：观察机器人整体行为
- 第一视角（front_cam）：观察感知和抓取细节
- 参数：640x480, 30fps, H.264 / yuv420p
- 脚本：`scripts/record_video.py`（MultiViewRecorder）

### Step 4: 打分准则设计

10 项结构化检查（总分 100）：

| 检查项 | 权重 | 关键指标 |
|--------|------|---------|
| 行走稳定性 | 20 | 步态自然、机身水平、转向平滑 |
| 导航精度 | 15 | 到达误差、朝向误差、路径效率 |
| 目标准确性 | 10 | 检测成功、框准确 |
| 机械臂运动 | 15 | IK 成功、运动平滑、无奇异 |
| 抓取执行 | 20 | 夹爪闭合、物体稳定、提起成功 |
| 返航能力 | 10 | 返回起点、路径合理 |
| 碰撞检测 | 5 | 无穿模、无碰撞 |
| 整体协调性 | 5 | 底盘臂协调、切换平滑 |
| 异常扣分 | -50~0 | 摔倒、卡死、抖动、掉落 |
| 改进建议 | - | 3-5 项优先改进方向 |

### Step 5: 视频审查子 Agent

使用 mimo-v2.5 多模态模型（Codex 本地代理）：
- 从视频中提取 5 帧关键帧
- 调用 mimo-v2.5 API 审查
- 输出结构化 JSON 报告
- 脚本：`scripts/video_review.py`

### Step 6: 数值分析器

5 层分析（脚本：`scripts/analyze_episode.py`）：
- FSM 分析：状态转换正确性
- 导航分析：到达误差、路径效率
- 抓取分析：IK 成功率、抓取成功率
- 关节跟踪分析：跟踪误差、速度平滑性
- 稳定性分析：基座高度、倾斜角、是否摔倒

---

## 产出文件结构

```
scripts/
    __init__.py
    record_video.py        # 视频录制（MultiViewRecorder）
    run_eval_episode.py    # 一键评估运行器
    analyze_episode.py     # 数值分析器
    video_review.py        # 多模态 AI 视频审查
    scoring_rubric.md      # 打分准则文档
AGENT.md                   # 长期记忆文件
logs/eval_episodes/        # 评估日志目录
```

---

## 关键发现

1. **视频录制**：MuJoCo EGL headless 模式可在无显示器环境下录制高质量视频
2. **关键帧提取**：从视频中均匀提取 5 帧足以覆盖主要行为阶段
3. **打分准则**：10 项检查覆盖了从行走到抓取的全流程
4. **子 Agent 隔离**：确保视频审查的客观性，不受主 Agent 判断影响
5. **三层证据交叉**：数值分析 + 视频审查 + 关键帧目视，三者交叉验证
