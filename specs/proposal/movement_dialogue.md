# 移动与对话系统

## Intent (设计意图)

为侦探游戏提供沉浸式的角色移动和自然流畅的对话交互体验。移动系统旨在让玩家在2D侧视角的犯罪现场中自由探索，参考《蔚蓝》(Celeste)的精确移动手感，同时保持侦探游戏的节奏感。对话系统参考《逆转裁判》的证言呈现方式，提供分支对话、证据出示和关键信息记录功能。

核心设计目标：
1. **移动的流畅性**：平滑的加速/减速，精确的碰撞检测，与环境物体的自然交互
2. **对话的沉浸感**：逐字显示文本，角色表情变化，对话选项的逻辑分支
3. **系统的集成性**：移动与对话状态无缝切换，证据系统深度集成
4. **数据的可配置性**：所有对话内容通过资源文件驱动，便于内容迭代

## Mechanics (逻辑机制)

### 移动系统机制

#### 基础移动控制
- **输入设备**：键盘方向键（←→）或A/D键进行水平移动
- **移动轴**：纯水平移动（X轴），Y轴用于跳跃重力物理
- **跳跃控制**：Space键进行跳跃，支持按住时间影响跳跃高度
- **加速移动**：Shift键按住时增加移动速度（暂时不介入跑步动画）
- **移动速度**：基础速度200像素/秒，加速后300像素/秒，可通过配置调整
- **加速度**：从静止到最大速度需0.2秒（参考Celeste的平滑加速）
- **减速度**：释放按键后0.15秒内停止（惯性效果）

#### 跳跃机制
- **跳跃高度**：基础跳跃高度80像素
- **跳跃持续时间**：最大跳跃持续时间0.3秒
- **跳跃变量**：按住Space时间越长，跳跃高度越高（最大1.5倍基础高度）
- **跳跃冷却**：跳跃后0.1秒内不能再次跳跃（防止连跳）
- **跳跃缓冲**：允许在落地前最多0.1秒内提前输入跳跃（Coyote Time）
- **跳跃取消**：在空中松开Space键可提前结束跳跃（减少高度）

#### 物理特性
- **碰撞检测**：使用Godot的CharacterBody2D + CollisionShape2D（支持平台物理）
- **碰撞层**：
  - Layer 1: 玩家角色
  - Layer 2: 环境障碍（墙壁、家具）
  - Layer 3: NPC角色（可穿透但会触发对话）
  - Layer 4: 可调查物体
  - Layer 5: 平台边缘（用于边缘检测）
- **碰撞形状**：胶囊形碰撞体，高度32像素，宽度16像素
- **重力系统**：启用重力（800像素/秒²），模拟真实跳跃物理
- **地面检测**：使用射线检测判断是否在地面，支持斜坡和台阶
- **墙壁检测**：左右两侧射线检测，用于墙壁跳跃（预留功能）

#### 交互机制
- **互动范围**：以玩家为中心，半径50像素的圆形区域
- **互动提示**：当玩家靠近可交互物体时，物体显示高亮轮廓
- **互动触发**：
  1. 自动触发：进入NPC对话范围时显示"按Space对话"提示
  2. 手动触发：面对可调查物体时按`E`键进行调查
- **视角管理**：2D侧视角固定相机，跟随玩家但有限制边界（房间边界）

#### 移动反馈
- **脚步声**：根据移动速度播放不同频率的脚步声音频
- **灰尘粒子**：快速移动时在脚下生成灰尘粒子效果
- **屏幕微震**：与重要物体碰撞时产生轻微相机震动

### 对话系统机制

#### 对话触发
- **NPC对话**：玩家进入NPC的交互范围（半径80像素）并按`Space`键
- **物体调查**：玩家面对可调查物体按`E`键，显示物体描述文本
- **自动对话**：特定剧情触发时强制进入对话状态

#### 对话流程
1. **对话开始**：显示对话UI，背景模糊，播放进入音效
2. **文本显示**：逐字显示文本，速度可调（默认30字符/秒）
3. **角色信息**：显示说话者头像、姓名、当前情绪状态
4. **玩家响应**：在分支点显示2-4个对话选项
5. **对话结束**：播放退出音效，返回探索状态

#### 对话选项类型
1. **普通询问**：继续当前话题，获取更多信息
2. **关键质问**：质疑对方的证言，可能触发证据出示环节
3. **出示证据**：选择此选项进入证据出示界面
4. **结束对话**：礼貌结束对话，记录关键信息

#### 证据出示机制（对话内）
1. **触发条件**：玩家选择"出示证据"选项或质疑关键证言时
2. **界面切换**：对话UI下方显示证据背包网格
3. **选择证据**：玩家使用方向键或鼠标选择证据
4. **反馈系统**：
   - 正确证据：NPC反应激烈，提供新信息，播放"正确"音效
   - 错误证据：NPC否认，对话可能提前结束，播放"错误"音效
5. **返回对话**：证据出示后继续原对话流程

#### 对话记录
- **自动记录**：所有关键证言自动添加到"侦探笔记"
- **重点标记**：玩家可以手动标记重要对话段落
- **证据关联**：对话中提到的证据自动建立链接关系

## Data Structure (数据结构)

### PlayerMovementConfig (Resource)
```gdscript
# 玩家移动配置资源
class_name PlayerMovementConfig extends Resource

@export_category("移动参数")
@export var base_speed: float = 200.0
@export var acceleration: float = 1000.0  # 加速度（像素/秒²）
@export var deceleration: float = 1200.0  # 减速度（像素/秒²）
@export var max_speed: float = 300.0
@export var dash_speed_multiplier: float = 1.5  # 加速移动时的速度倍率（按住Shift）

@export_category("跳跃参数")
@export var jump_height: float = 80.0  # 基础跳跃高度（像素）
@export var jump_duration: float = 0.3  # 最大跳跃持续时间（秒）
@export var jump_force: float = 400.0  # 跳跃初始速度（像素/秒）
@export var jump_variable_height: bool = true  # 是否支持可变高度跳跃
@export var jump_cooldown: float = 0.1  # 跳跃冷却时间（秒）
@export var coyote_time: float = 0.1  # 边缘跳跃缓冲时间（秒）
@export var jump_buffer_time: float = 0.1  # 跳跃输入缓冲时间（秒）

@export_category("物理参数")
@export var gravity: float = 800.0  # 重力加速度（像素/秒²）
@export var collision_shape_radius: float = 8.0
@export var collision_shape_height: float = 32.0
@export var collision_layer_mask: int = 0b11111  # 碰撞层掩码（5层）
@export var ground_detection_ray_length: float = 10.0  # 地面检测射线长度
@export var wall_detection_ray_length: float = 5.0  # 墙壁检测射线长度

@export_category("交互参数")
@export var interaction_range: float = 50.0
@export var auto_interaction_range: float = 80.0  # NPC自动提示范围
@export var interaction_cooldown: float = 0.3  # 互动冷却时间（秒）

@export_category("反馈参数")
@export var footstep_interval_fast: float = 0.25  # 快速移动脚步声间隔
@export var footstep_interval_slow: float = 0.5   # 慢速移动脚步声间隔
@export var jump_sound_delay: float = 0.1  # 跳跃音效延迟（秒）
@export var landing_sound_threshold: float = 100.0  # 落地音效速度阈值
@export var camera_shake_intensity: float = 2.0   # 相机震动强度
```

### DialogueData (Resource)
```gdscript
# 单句对话数据
class_name DialogueLine extends Resource

@export var speaker_id: String  # 说话者ID（如"witness_1"）
@export var text: String        # 对话文本
@export var emotion: String = "neutral"  # 情绪状态（neutral, angry, surprised等）
@export var audio_clip: AudioStream  # 语音音频（可选）
@export var duration: float = 0.0  # 自动播放时长（0表示手动前进）
```

### DialogueNode (Resource)
```gdscript
# 对话节点（支持分支）
class_name DialogueNode extends Resource

@export var node_id: String  # 节点唯一标识
@export var lines: Array[DialogueLine]  # 连续对话行
@export var choices: Array[DialogueChoice]  # 对话选项（为空则表示线性对话）

# 下一节点映射（根据选择跳转）
@export var next_node_map: Dictionary = {}  # key: 选项索引, value: 下一节点ID
@export var default_next_node: String = ""  # 无选择时的默认下一节点
```

### DialogueChoice (Resource)
```gdscript
# 对话选项
class_name DialogueChoice extends Resource

@export var text: String  # 选项文本
@export var requirement_evidence: String = ""  # 需要的证据ID（空表示无要求）
@export var triggers_evidence_presentation: bool = false  # 是否触发证据出示
@export var is_critical_question: bool = false  # 是否为关键质问
```

### NPCData (Resource)
```gdscript
# NPC角色数据
class_name NPCData extends Resource

@export var npc_id: String
@export var display_name: String
@export var default_dialogue_node: String  # 默认对话节点ID
@export var position_in_scene: Vector2  # 场景中的初始位置
@export var interaction_radius: float = 80.0
@export var portraits: Dictionary = {}  # key: 情绪状态, value: 纹理资源
```

### DialogueManager (Resource)
```gdscript
# 对话管理器资源属性
class_name DialogueManager extends Resource

@export var current_dialogue_node: String = ""
@export var current_line_index: int = 0
@export var dialogue_history: Array[Dictionary] = []  # 历史记录

# 对话状态
@export var is_in_dialogue: bool = false
@export var is_waiting_for_choice: bool = false
@export var is_presenting_evidence: bool = false

# 当前对话上下文
@export var current_speaker_id: String = ""
@export var current_evidence_required: String = ""  # 当前需要的证据ID
```

## Signals & API (信号与接口)

### 移动系统信号
```gdscript
# 玩家移动相关
signal player_moved(position: Vector2, velocity: Vector2)
signal player_stopped(position: Vector2)
signal player_jumped(jump_height: float, jump_duration: float)  # 玩家跳跃
signal player_landed(landing_velocity: float)  # 玩家落地
signal player_dash_started(direction: float)  # 玩家开始加速移动
signal player_dash_ended()  # 玩家结束加速移动
signal player_interaction_range_entered(object_type: String, object_id: String)
signal player_interaction_range_exited(object_type: String, object_id: String)

# 交互触发
signal interaction_started(object_type: String, object_id: String)
signal interaction_completed(object_type: String, object_id: String, success: bool)

# 物理事件
signal player_collided(collision_normal: Vector2, collider_name: String)
signal player_grounded_changed(is_grounded: bool)  # 玩家接地状态变化
signal player_wall_collided(wall_normal: Vector2, wall_name: String)  # 玩家撞墙
signal player_fell_off_bounds  # 玩家掉出边界（错误恢复）
```

### 对话系统信号
```gdscript
# 对话流程
signal dialogue_started(npc_id: String, dialogue_node_id: String)
signal dialogue_line_changed(speaker_id: String, text: String, line_index: int)
signal dialogue_choice_presented(choices: Array[String])
signal dialogue_choice_selected(choice_index: int, choice_text: String)
signal dialogue_ended(npc_id: String, completed: bool)

# 证据出示
signal evidence_presentation_requested(npc_id: String, context: String)
signal evidence_selected_for_presentation(evidence_id: String)
signal evidence_presentation_result(correct: bool, feedback_text: String)

# 记录与进度
signal key_statement_recorded(statement_id: String, text: String, npc_id: String)
signal contradiction_found(statement_id_1: String, statement_id_2: String)
```

### 公共API方法

#### 移动系统API
```gdscript
# 移动控制
func set_movement_enabled(enabled: bool) -> void
func teleport_player(position: Vector2) -> void
func get_player_position() -> Vector2
func get_player_velocity() -> Vector2
func get_is_grounded() -> bool  # 获取玩家是否在地面
func get_is_dashing() -> bool  # 获取玩家是否正在加速移动

# 跳跃控制
func jump() -> bool  # 执行跳跃，返回是否成功
func cancel_jump() -> void  # 取消当前跳跃（减少高度）
func can_jump() -> bool  # 检查当前是否可以跳跃

# 加速移动控制
func start_dash() -> bool  # 开始加速移动，返回是否成功
func stop_dash() -> void  # 停止加速移动

# 交互控制
func force_interaction(object_id: String) -> bool  # 强制触发交互
func get_nearby_interactables() -> Array[String]  # 返回附近可交互物体ID列表

# 配置管理
func update_movement_config(config: PlayerMovementConfig) -> void
func get_movement_config() -> PlayerMovementConfig  # 获取当前移动配置
```

#### 对话系统API
```gdscript
# 对话控制
func start_dialogue(npc_id: String, node_id: String = "") -> bool
func advance_dialogue() -> bool  # 前进到下一行/下一节点
func select_choice(choice_index: int) -> bool
func exit_dialogue() -> void

# 证据出示
func present_evidence(evidence_id: String) -> Dictionary  # 返回结果{correct: bool, feedback: String}
func get_available_evidence_for_presentation() -> Array[String]  # 返回可出示的证据列表

# 数据查询
func get_current_dialogue_text() -> String
func get_current_speaker() -> String
func get_dialogue_history() -> Array[Dictionary]
func get_npc_data(npc_id: String) -> NPCData

# 内容管理
func load_dialogue_graph(json_path: String) -> bool  # 从JSON加载对话图
func save_dialogue_progress() -> void
```

### 外部系统集成接口
1. **游戏状态机**：监听`dialogue_started`/`dialogue_ended`信号来切换游戏状态
2. **证据系统**：通过`evidence_presentation_requested`信号请求证据列表
3. **UI系统**：接收`dialogue_line_changed`和`dialogue_choice_presented`信号更新界面
4. **音频系统**：根据对话情绪播放对应的语音和音效
5. **存档系统**：通过`save_dialogue_progress`方法保存对话进度