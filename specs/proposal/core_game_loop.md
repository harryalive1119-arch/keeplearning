# 核心游戏循环与状态机

## Intent (设计意图)

设计一个清晰、可扩展的游戏状态机，管理玩家在侦探游戏中的核心体验循环。确保玩家能够在**自由探索**、**对话交互**、**证物收集**和**谜底揭晓**四个核心状态间无缝切换，同时保持游戏逻辑的模块化和数据驱动。

参考《逆转裁判》的流程设计：探索阶段收集线索 → 对话阶段获取证言 → 推理阶段构建证据链 → 最终揭晓真相。

## Mechanics (逻辑机制)

### 游戏状态定义
游戏包含四个主要状态，每个状态对应不同的玩家输入和UI界面：

1. **EXPLORATION** (探索状态)
   - 玩家使用方向键/WASD控制侦探角色在犯罪现场房间内自由移动
   - 按`E`键与可交互物体/人物进行互动
   - 按`Tab`键打开证据背包
   - 按`Space`键触发对话（当靠近NPC时）
   - 状态退出条件：进入对话或证据收集界面

2. **DIALOGUE** (对话状态)
   - 显示对话UI，呈现NPC的证言文本
   - 玩家通过鼠标点击或键盘方向键选择对话选项
   - 按`E`键快速跳过对话
   - 按`Esc`键退出对话返回探索状态
   - 特殊选项：出示证据（触发证据出示界面）
   - 状态退出条件：对话结束或玩家主动退出

3. **EVIDENCE_COLLECTION** (证物收集状态)
   - 进入场景互动界面，背景变暗聚焦
   - 玩家鼠标点击场景中的可收集物品
   - 收集到的证物自动加入证据背包
   - 按`Esc`键返回探索状态
   - 状态退出条件：收集完成或玩家主动退出

4. **TRUTH_REVELATION** (谜底揭晓状态)
   - 显示问题界面，呈现多个陈述句
   - 玩家选择其中一个陈述作为"真相"
   - 选择正确后，进入"证据出示"子状态
   - 玩家从证据背包中选择正确的证物/证言进行作证
   - 状态退出条件：成功揭晓真相或失败重试

### 状态转换规则
```
探索状态 → 对话状态：当玩家靠近NPC并按Space键
探索状态 → 证物收集状态：当玩家与可调查物体互动
对话状态 → 证据出示子状态：当玩家选择"出示证据"选项
任何状态 → 谜底揭晓状态：当满足所有线索收集条件（通过信号触发）
谜底揭晓状态 → 探索状态：当推理完成（成功或失败后）
```

### 游戏流程（Demo版本）
1. 游戏开始于探索状态，玩家在犯罪现场房间内
2. 与3个NPC对话获取证言
3. 收集5个关键证物
4. 所有线索收集完成后，自动进入谜底揭晓状态
5. 完成2轮问答（陈述选择 + 证据出示）
6. 揭晓真相，游戏结束

## Data Structure (数据结构)

### GameState (枚举)
```gdscript
enum GameState {
    EXPLORATION,      # 自由探索
    DIALOGUE,         # 对话交互
    EVIDENCE_COLLECTION, # 证物收集
    TRUTH_REVELATION  # 谜底揭晓
}
```

### GameManager (Resource)
```gdscript
# 游戏管理器资源属性
@export var current_state: GameState = GameState.EXPLORATION
@export var previous_state: GameState = GameState.EXPLORATION

# 进度跟踪
@export var collected_evidence_count: int = 0
@export var total_required_evidence: int = 5
@export var talked_to_npcs: Array[String] = []  # 已对话的NPC ID列表
@export var required_npcs: Array[String] = ["witness_1", "suspect_1", "detective_1"]

# 游戏状态标志
@export var is_truth_revelation_available: bool = false
@export var is_game_completed: bool = false

# 配置数据
@export var exploration_move_speed: float = 200.0
@export var interaction_range: float = 50.0  # 互动触发范围（像素）
```

### StateConfig (Resource)
```gdscript
# 状态配置资源，用于数据驱动设计
class_name StateConfig extends Resource

@export var state_name: String
@export var input_enabled: bool = true
@export var ui_scene: PackedScene  # 该状态对应的UI场景
@export var camera_behavior: String = "default"  # "follow_player", "fixed", "zoom_in"
@export var music_track: AudioStream
```

## Signals & API (信号与接口)

### 发出的信号
```gdscript
# 状态变化信号
signal state_changed(from_state: GameState, to_state: GameState)
signal state_entered(state: GameState)
signal state_exited(state: GameState)

# 游戏进度信号
signal evidence_collected(evidence_id: String, evidence_name: String)
signal npc_dialogue_completed(npc_id: String)
signal all_evidence_collected  # 所有必需证物收集完成
signal all_npcs_interviewed    # 所有必需NPC对话完成
signal truth_revelation_available  # 谜底揭晓可用

# 游戏流程信号
signal game_started
signal game_completed(success: bool)
signal game_reset
```

### 公共API方法
```gdscript
# 状态管理
func change_state(new_state: GameState) -> void
func get_current_state() -> GameState
func is_state_transition_allowed(from_state: GameState, to_state: GameState) -> bool

# 进度管理
func add_evidence(evidence_id: String) -> bool
func mark_npc_interviewed(npc_id: String) -> void
func check_progress() -> void  # 检查是否满足进入谜底揭晓的条件

# 游戏控制
func start_game() -> void
func reset_game() -> void
func complete_game(success: bool) -> void

# 数据访问
func get_collected_evidence() -> Array[String]
func get_remaining_evidence_count() -> int
func get_game_time() -> float
```

### 外部系统接口
1. **移动系统**：监听`state_changed`信号，在`EXPLORATION`状态启用移动输入
2. **对话系统**：在`DIALOGUE`状态激活，接收`npc_dialogue_completed`信号
3. **证据系统**：在`EVIDENCE_COLLECTION`状态激活，发射`evidence_collected`信号
4. **UI系统**：根据当前状态显示不同的UI界面
5. **音频系统**：根据状态变化切换背景音乐