# 谜底揭晓系统（真相揭示环节）

## Intent (设计意图)

设计一个符合侦探召集所有人揭示真相场景的推理高潮环节。侦探将嫌疑人、证人和相关人物聚集在犯罪现场，通过一系列逻辑推理步骤逐步揭示真相。系统参考《逆转裁判》的推理节奏，但场景设定为侦探主持的真相揭示会，而非法庭辩论。

核心设计目标：
1. **叙事整合**：将分散的线索通过侦探的推理串联成完整故事
2. **渐进推理**：通过3-5个核心问题的逐步证明，引导玩家理解真相全貌
3. **证据引导**：每个问题有唯一正确答案，但提供次证据作为提示线索
4. **信任度管理**：玩家的推理准确度影响NPC的信任度，决定最终结局走向
5. **戏剧呈现**：通过对话、证据出示和角色反应营造紧张氛围

## Mechanics (逻辑机制)

### 真相揭示场景设置

#### 场景描述
- **地点**：犯罪现场房间中央，所有相关人物围成一圈
- **人物**：侦探（玩家）、3-4个嫌疑人/证人、可能还有警长等旁观者
- **氛围**：紧张、严肃，背景音乐为低沉悬疑的钢琴曲
- **视角**：2D侧视角，类似视觉小说布局，但保留角色站位和表情变化

#### 流程结构
1. **开场陈述**：侦探简要说明召集目的和初步结论
2. **核心问题循环**（3次，对应demo）：
   - 侦探提出一个关键问题（陈述选择）
   - 玩家从3-5个选项中选择最合理的陈述
   - 选择后触发特定对话，质疑或支持该陈述
   - 对话中自动引出"需要证据证明"的环节
   - 玩家从证据背包中选择证据进行证明
   - 根据证据选择结果，更新信任度并继续
3. **真相揭晓**：所有问题解决后，侦探完整揭示犯罪过程和凶手
4. **结局分支**：根据信任度水平决定不同结局（合作/反抗/自首等）

### 核心问题证明机制

#### 阶段一：陈述选择（Statement Selection）
1. **问题呈现**：侦探提出如"凶手进入房间的方式是？"的问题
2. **选项显示**：显示3-5个可能的陈述（如：A.从窗户进入 B.有钥匙 C.门没锁）
3. **选择交互**：玩家使用方向键或鼠标选择一项
4. **即时反馈**：
   - 正确选择：侦探点头，播放轻微肯定音效，继续对话
   - 错误选择：侦探皱眉，播放轻微否定音效，但允许继续（影响信任度）
5. **设计原则**：每个问题有且只有一个逻辑上完全正确的答案

#### 阶段二：对话展开（Dialogue Expansion）
1. **自动对话**：选择陈述后，触发预设的对话序列
2. **角色反应**：不同NPC对该陈述有不同反应（支持、反对、质疑）
3. **逻辑缺口**：对话中会自然暴露出"需要证据支持"的点
4. **证据提示**：NPC可能会说"但这需要证据证明"或"你有证据吗？"

#### 阶段三：证据出示（Evidence Presentation）
1. **触发时机**：对话进行到证据需求点时自动进入证据出示界面
2. **界面设计**：证据背包以网格形式显示，当前场景变暗聚焦
3. **证据分类**：
   - **完全正确证据**（1个）：能直接且充分证明该陈述
   - **次证据**（1-2个）：相关但不充分，选择后会提供进一步提示
   - **无关证据**（其余）：完全无关，选择后降低信任度
4. **选择反馈**：
   - 完全正确：侦探自信展示证据，NPC震惊/承认，信任度大幅提升
   - 次证据：侦探犹豫，NPC质疑"这不能完全证明"，提供线索提示
   - 无关证据：侦探尴尬，NPC嘲笑，信任度明显下降
5. **提示系统**：选择次证据后，侦探会自言自语"这还不够，也许应该找..."提供方向性提示

#### 阶段四：信任度更新（Trust Update）
1. **信任度系统**：代表NPC对侦探推理能力的信任程度（0-100%）
2. **影响因素**：
   - 陈述选择正确：+15%信任度
   - 陈述选择错误：-10%信任度
   - 证据选择完全正确：+25%信任度
   - 证据选择次证据：+5%信任度（因提供线索）
   - 证据选择无关：-20%信任度
3. **信任度阈值**：
   - >80%：高信任，NPC积极配合，可能自首
   - 50%-80%：中等信任，需要更多说服
   - <50%：低信任，NPC可能反抗或逃跑
4. **视觉表示**：屏幕上方显示信任度条，颜色从红（不信任）到绿（信任）

### 流程示例（Demo版本）

#### 问题1：凶器是什么？
- 选项：A.水果刀 B.绳索 C.花瓶 D.手枪
- 正确答案：A.水果刀（证据：刀上的指纹和血迹匹配）
- 完全正确证据：染血的水果刀（证据ID: knife_001）
- 次证据：现场血迹照片（提示：血迹形状显示锐器造成）
- 无关证据：嫌疑人的不在场证明

#### 问题2：凶手进入房间的方式？
- 选项：A.窗户 B.正门钥匙 C.门没锁 D.通风管道
- 正确答案：B.正门钥匙（证据：只有嫌疑人有钥匙副本）
- 完全正确证据：钥匙复制收据（证据ID: receipt_002）
- 次证据：窗户锁完好照片（提示：排除窗户可能性）
- 无关证据：嫌疑人的雨伞

#### 问题3：作案时间？
- 选项：A.晚上8点 B.晚上10点 C.凌晨1点 D.早上6点
- 正确答案：C.凌晨1点（证据：监控时间戳和证言矛盾）
- 完全正确证据：监控录像截图（证据ID: cctv_003）
- 次证据：邻居证言（提示：听到声音的时间段）
- 无关证据：天气预报

### 容错与引导机制

#### 信任度作为容错系统
- **初始值**：60%信任度（基于侦探的声誉）
- **失败保护**：信任度降至30%时触发"最后机会"提示
- **恢复机制**：连续正确选择可快速恢复信任度
- **临界效果**：信任度<20%时，NPC可能提前离场（游戏失败）

#### 渐进提示系统
1. **一级提示（轻度）**：信任度<50%时，陈述选项显示逻辑倾向性（如：某个选项轻微高亮）
2. **二级提示（中度）**：连续两次错误后，侦探自言自语提示关键证据类型
3. **三级提示（重度）**：信任度<30%时，显示证据与陈述的关联度星级（1-5星）

#### 重试机制
- **有限重试**：每个问题允许一次"重新考虑"机会（消耗信任度）
- **跳过选项**：仅在教学模式或最低难度可用
- **自动存档**：每个问题阶段自动存档，可加载重试

### 视觉与音频设计

#### 视觉反馈
- **角色表情**：根据信任度变化显示不同表情（信任→认真倾听，不信任→怀疑眼神）
- **证据展示**：出示证据时全屏显示证据大图，附带侦探解说文字
- **信任度条**：屏幕上方动态变化，颜色渐变，重要变化时闪烁
- **气氛效果**：关键证据出示时屏幕轻微震动，角色表情特写

#### 音频设计
- **背景音乐**：紧张悬疑的钢琴曲，随着推理进展变化强度
- **音效反馈**：
  - 正确选择：清脆的"叮"声
  - 错误选择：低沉的"嗡"声
  - 证据出示：纸张翻动声、投影仪切换声
  - 信任度变化：上升-上扬音阶，下降-下降音阶
- **语音提示**：侦探的关键推理语句有语音配音

## Data Structure (数据结构)

### TruthRevelationSession (Resource)
```gdscript
# 真相揭示会话
class_name TruthRevelationSession extends Resource

@export_category("会话配置")
@export var session_id: String
@export var scene_background: Texture2D  # 背景图像
@export var participants: Array[String]  # 参与者NPC ID列表
@export var participant_positions: Dictionary  # key: NPC ID, value: 位置Vector2

@export_category("问题流程")
@export var core_questions: Array[CoreQuestion]  # 核心问题列表（按顺序）
@export var opening_dialogue: Array[DialogueLine]  # 开场对话
@export var closing_dialogue: Array[DialogueLine]  # 结束对话

@export_category("信任度配置")
@export var initial_trust: float = 0.6  # 初始信任度（0-1）
@export var trust_thresholds: Dictionary = {  # 信任度阈值
    "high": 0.8,
    "medium": 0.5,
    "low": 0.3,
    "critical": 0.2
}
@export var max_questions: int = 3  # 最大问题数（demo为3）
```

### CoreQuestion (Resource)
```gdscript
# 核心问题
class_name CoreQuestion extends Resource

@export var question_id: String
@export var question_text: String  # 问题文本（如："凶器是什么？"）
@export var narrative_context: String  # 叙事上下文

@export_category("陈述选项")
@export var statements: Array[QuestionStatement]  # 3-5个陈述选项
@export var correct_statement_index: int  # 正确陈述索引

@export_category("证据映射")
@export var perfect_evidence_id: String  # 完全正确证据ID（唯一）
@export var secondary_evidence_ids: Array[String]  # 次证据ID列表（1-2个）
@export var irrelevant_evidence_ids: Array[String]  # 无关证据ID列表（自动生成）

@export_category("对话流程")
@export var post_selection_dialogue: Array[DialogueLine]  # 选择后的对话
@export var evidence_request_dialogue: String  # 请求证据的对话文本
@export var perfect_evidence_dialogue: Array[DialogueLine]  # 完美证据出示后对话
@export var secondary_evidence_dialogue: Array[DialogueLine]  # 次证据出示后对话
@export var irrelevant_evidence_dialogue: Array[DialogueLine]  # 无关证据出示后对话

@export_category("信任度影响")
@export var trust_impact_perfect: float = 0.25  # 完美证据信任度影响
@export var trust_impact_secondary: float = 0.05  # 次证据信任度影响
@export var trust_impact_irrelevant: float = -0.20  # 无关证据信任度影响
@export var trust_impact_correct_statement: float = 0.15  # 正确陈述信任度影响
@export var trust_impact_incorrect_statement: float = -0.10  # 错误陈述信任度影响
```

### QuestionStatement (Resource)
```gdscript
# 问题陈述选项
class_name QuestionStatement extends Resource

@export var statement_id: String
@export var statement_text: String  # 陈述文本
@export var is_correct: bool = false  # 是否为正确陈述

@export_category("逻辑属性")
@export var logical_weight: float = 1.0  # 逻辑权重（用于提示系统）
@export var contradiction_with: Array[String] = []  # 与此陈述矛盾的证据ID
@export var supported_by: Array[String] = []  # 支持此陈述的证据ID

@export_category("反馈内容")
@export var selection_feedback: String  # 选择此陈述时的反馈文本
@export var npc_reactions: Dictionary = {}  # key: NPC ID, value: 反应文本
```

### TrustSystem (Resource)
```gdscript
# 信任度系统配置
class_name TrustSystem extends Resource

@export_category("基础配置")
@export var max_trust: float = 1.0
@export var min_trust: float = 0.0
@export var initial_trust: float = 0.6

@export_category("影响因子")
@export var statement_correct_multiplier: float = 1.0
@export var evidence_perfect_multiplier: float = 1.0
@export var consecutive_correct_bonus: float = 0.1  # 连续正确奖励
@export var consecutive_penalty_reduction: float = 0.05  # 连续错误惩罚递减

@export_category("阈值效果")
@export var effects_at_thresholds: Dictionary = {
    "high": {
        "npc_attitude": "cooperative",
        "dialogue_variation": "respectful",
        "unlock_options": true
    },
    "medium": {
        "npc_attitude": "neutral",
        "dialogue_variation": "standard",
        "unlock_options": false
    },
    "low": {
        "npc_attitude": "suspicious",
        "dialogue_variation": "doubtful",
        "unlock_options": false
    },
    "critical": {
        "npc_attitude": "hostile",
        "dialogue_variation": "challenging",
        "early_exit": true  # NPC可能提前离场
    }
}

@export_category("视觉反馈")
@export var trust_bar_colors: Dictionary = {
    "high": Color.GREEN,
    "medium": Color.YELLOW,
    "low": Color.ORANGE,
    "critical": Color.RED
}
@export var trust_change_animation: String = "pulse"  # 变化动画类型
```

### TruthRevelationState (Resource)
```gdscript
# 真相揭示状态
class_name TruthRevelationState extends Resource

@export_category("会话进度")
@export var current_session_id: String = ""
@export var current_question_index: int = 0  # 当前问题索引
@export var completed_questions: Array[String] = []  # 已完成的问题ID
@export var selected_statement_indices: Dictionary = {}  # key: 问题ID, value: 选择的陈述索引
@export var presented_evidence: Dictionary = {}  # key: 问题ID, value: 出示的证据ID

@export_category("玩家状态")
@export var current_trust: float = 0.6  # 当前信任度（0-1）
@export var consecutive_correct: int = 0  # 连续正确次数
@export var consecutive_incorrect: int = 0  # 连续错误次数
@export var hints_used: int = 0  # 已使用的提示次数
@export var retries_used: int = 0  # 已使用的重试次数

@export_category("临时数据")
@export var available_evidence: Array[String] = []  # 当前可用的证据ID列表
@export var current_stage: String = "selection"  # 当前阶段："selection", "dialogue", "evidence", "transition"
@export var dialogue_progress: int = 0  # 当前对话进度（行索引）
```

### HintSystem (Resource)
```gdscript
# 提示系统配置
class_name HintSystem extends Resource

@export_category("提示级别")
@export var hint_levels: Array[HintLevel] = []

class HintLevel:
    var level_name: String  # "subtle", "moderate", "direct"
    var trigger_condition: String  # 触发条件
    var hint_text_template: String  # 提示文本模板
    var trust_cost: float = 0.0  # 信任度成本

@export_category("自适应提示")
@export var enable_adaptive_hints: bool = true
@export var min_trust_for_hints: float = 0.3  # 使用提示的最低信任度
@export var hint_cooldown: float = 60.0  # 提示冷却时间（秒）

@export_category("证据提示")
@export var evidence_hint_types: Array[String] = ["type", "location", "character", "time"]
@export var secondary_evidence_hint: String = "这个证据相关但不充分，考虑寻找更直接的证据。"
```

## Signals & API (信号与接口)

### 真相揭示流程信号
```gdscript
# 会话控制
signal truth_revelation_session_started(session_id: String)
signal truth_revelation_session_ended(session_id: String, success: bool, final_trust: float)
signal core_question_started(question_id: String, question_index: int)
signal core_question_completed(question_id: String, success: bool, trust_change: float)

# 阶段转换
signal statement_selection_phase_entered(question_id: String, statements: Array[String])
signal dialogue_phase_entered(question_id: String, selected_statement_index: int)
signal evidence_presentation_phase_entered(question_id: String, evidence_request_text: String)
signal phase_transition_completed(from_phase: String, to_phase: String)

# 玩家选择
signal statement_selected(question_id: String, statement_index: int, is_correct: bool, feedback: String)
signal evidence_presented(question_id: String, evidence_id: String, evidence_type: String, is_perfect: bool)
signal evidence_presentation_result(question_id: String, result_type: String, feedback: String, trust_impact: float)
```

### 信任度系统信号
```gdscript
# 信任度变化
signal trust_changed(old_trust: float, new_trust: float, reason: String)
signal trust_threshold_crossed(threshold_name: String, direction: String)  # direction: "above" or "below"
signal trust_critical_warning(current_trust: float)  # 信任度低于临界值警告

# NPC反应
signal npc_attitude_changed(npc_id: String, old_attitude: String, new_attitude: String)
signal npc_dialogue_variation_applied(npc_id: String, variation_type: String)
signal npc_early_exit_warning(npc_id: String, remaining_questions: int)  # NPC可能提前离场
```

### 提示与引导信号
```gdscript
# 提示系统
signal hint_available(hint_level: String, hint_text: String)
signal hint_used(hint_level: String, hint_text: String, trust_cost: float)
signal hint_cooldown_started(duration: float)
signal hint_cooldown_ended

# 引导反馈
signal secondary_evidence_hint_provided(evidence_id: String, hint_text: String)
signal logical_inconsistency_highlighted(statement_id: String, evidence_id: String)
signal progress_stall_detected(time_elapsed: float, suggested_action: String)
```

### 叙事与结局信号
```gdscript
# 真相揭晓
signal truth_revelation_final_started  # 最终真相揭晓开始
signal truth_revelation_final_completed  # 最终真相揭晓完成
signal crime_timeline_revealed(timeline_events: Array[Dictionary])
signal culprit_revealed(culprit_id: String, confession: bool)

# 结局分支
signal ending_determined(ending_type: String, trust_level: String)
signal ending_cutscene_started(ending_id: String)
signal ending_credits_started
```

### 公共API方法

#### 会话控制API
```gdscript
# 流程控制
func start_truth_revelation(session_id: String) -> bool
func advance_to_next_question() -> bool
func restart_current_question() -> bool
func exit_truth_revelation(force: bool = false) -> bool

# 阶段控制
func enter_statement_selection_phase() -> void
func enter_dialogue_phase(statement_index: int) -> void
func enter_evidence_presentation_phase() -> void
func advance_dialogue() -> bool  # 前进对话到下一行

# 状态查询
func get_current_question() -> CoreQuestion
func get_current_phase() -> String
func get_remaining_questions() -> int
func is_session_completed() -> bool
```

#### 玩家选择API
```gdscript
# 陈述选择
func select_statement(statement_index: int) -> Dictionary  # 返回{success: bool, is_correct: bool, feedback: String}
func get_available_statements() -> Array[QuestionStatement]
func get_statement_by_index(index: int) -> QuestionStatement

# 证据出示
func present_evidence(evidence_id: String) -> Dictionary  # 返回{result_type: String, is_perfect: bool, feedback: String, trust_impact: float}
func get_available_evidence_for_question() -> Array[String]
func get_evidence_recommendation() -> Array[String]  # 返回推荐证据ID列表（基于当前问题）
func classify_evidence_for_question(evidence_id: String) -> String  # 返回"perfect", "secondary", "irrelevant"
```

#### 信任度管理API
```gdscript
# 信任度操作
func get_current_trust() -> float
func modify_trust(delta: float, reason: String) -> float
func reset_trust_to_initial() -> void
func set_trust_cap(min_cap: float, max_cap: float) -> void

# 阈值检查
func check_trust_thresholds() -> Array[String]  # 返回当前达到的阈值列表
func get_trust_level() -> String  # 返回"high", "medium", "low", "critical"
func is_trust_critical() -> bool

# NPC态度
func update_npc_attitudes_based_on_trust() -> void
func get_npc_attitude(npc_id: String) -> String
func force_npc_attitude(npc_id: String, attitude: String) -> void
```

#### 提示系统API
```gdscript
# 提示控制
func request_hint(hint_level: String = "subtle") -> Dictionary  # 返回{hint_text: String, trust_cost: float}
func get_available_hints() -> Array[String]
func is_hint_available(hint_level: String) -> bool
func get_hint_cooldown_remaining() -> float

# 证据提示
func provide_secondary_evidence_hint(evidence_id: String) -> String
func suggest_evidence_category() -> String  # 返回证据类型建议
func highlight_logical_inconsistencies() -> Array[Dictionary]
```

#### 数据管理API
```gdscript
# 问题管理
func load_core_question(question_id: String) -> CoreQuestion
func validate_question_prerequisites(question_id: String) -> bool
func get_question_progress(question_id: String) -> Dictionary  # 返回完成状态和选择记录

# 证据过滤
func filter_evidence_for_question(question_id: String, evidence_list: Array[String]) -> Array[String]
func get_perfect_evidence_for_question(question_id: String) -> String
func get_secondary_evidence_for_question(question_id: String) -> Array[String]

# 状态持久化
func save_current_state() -> Dictionary
func load_state(state_data: Dictionary) -> bool
func export_session_statistics() -> Dictionary  # 返回会话统计数据
```

#### 叙事集成API
```gdscript
# 真相揭晓
func trigger_final_revelation() -> void
func generate_crime_timeline() -> Array[Dictionary]
func determine_culprit() -> String  # 返回凶手ID
func play_confession_scene(culprit_id: String) -> void

# 结局管理
func determine_ending() -> String  # 基于信任度返回结局ID
func play_ending_sequence(ending_id: String) -> void
func calculate_ending_variants() -> Array[String]  # 返回所有可能的结局

# 新游戏+
func prepare_truth_revelation_ngplus(previous_trust: float) -> Dictionary
```

### 外部系统集成接口
1. **游戏状态机**：通过`truth_revelation_session_started`/`ended`信号协调游戏状态
2. **证据系统**：通过`get_available_evidence_for_question`获取玩家收集的证据
3. **对话系统**：通过`dialogue_phase_entered`信号触发预设对话序列
4. **UI系统**：接收阶段转换和选择反馈信号更新界面显示
5. **音频系统**：根据信任度变化和选择结果播放对应的音效
6. **存档系统**：通过状态保存/加载接口持久化进度
7. **成就系统**：跟踪完美推理、高信任度等成就
8. **叙事系统**：通过结局管理接口触发不同的叙事分支