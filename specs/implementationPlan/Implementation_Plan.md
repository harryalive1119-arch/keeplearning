# Godot 推理解谜游戏实现计划

## 前言
本文档面向具有 Unity 基础但刚接触 Godot 的开发者，将以渐进的方式介绍如何实现一个推理解谜游戏。特别关注 Unity 到 Godot 的概念迁移，并提供详细的节点树结构和信号映射。

## 项目结构
```
project/
├── scenes/              # 场景文件 (.tscn)
│   ├── main/           # 主要场景
│   │   ├── game.tscn   # 主游戏场景
│   │   └── rooms/      # 房间场景
│   ├── ui/             # UI场景
│   │   ├── hud/        # 游戏HUD
│   │   └── menus/      # 各类菜单
│   └── prefabs/        # 预制体场景
│       ├── player/     # 玩家相关
│       └── interactables/ # 可交互物体
├── scripts/            # GDScript脚本
│   ├── autoload/      # 自动加载脚本
│   ├── resources/     # 资源类脚本
│   └── components/    # 组件脚本
└── assets/            # 资源文件
    ├── sprites/       # 2D图片
    ├── audio/        # 音频文件
    └── data/         # JSON配置文件
```

## Unity vs Godot 核心概念对比

### 1. 场景系统
- Unity的预制体（Prefab）→ Godot的场景（Scene）
- Unity的GameObject → Godot的Node
- Unity的组件（Component）→ Godot的节点（Node）
- Unity的Transform → Godot的Node2D/Node3D

### 2. 脚本系统
- Unity的MonoBehaviour → Godot的Node
- Unity的Start() → Godot的_ready()
- Unity的Update() → Godot的_process(delta)
- Unity的FixedUpdate() → Godot的_physics_process(delta)

## 详细实现步骤

### 1. 核心游戏状态机

#### 场景树结构
```
GameManager (AutoLoad Node)
└── StateManager (Node)
    └── SignalBus (Node)
```

#### 资源定义
```gdscript
# scripts/resources/GameState.gd
class_name GameStateResource extends Resource

@export var state_name: String
@export var input_enabled: bool = true
@export var ui_scene: PackedScene
@export var camera_behavior: String = "default"
@export var music_track: AudioStream
```

#### 信号映射
```gdscript
# scripts/autoload/GameManager.gd
extends Node

signal state_changed(from_state: GameState, to_state: GameState)
signal state_entered(state: GameState)
signal state_exited(state: GameState)
signal game_started
signal game_completed(success: bool)
signal game_reset

# 导出变量用于调优
@export_category("State Configuration")
@export var state_transition_time: float = 0.3
@export var input_lock_duration: float = 0.1
```

### 2. 玩家移动系统

#### 场景树结构
```
Player (CharacterBody2D)
├── CollisionShape2D
│   └── CapsuleShape2D
├── Sprite2D
├── AnimationPlayer
├── GroundDetectionRay (RayCast2D)          # 地面检测射线
├── LeftWallDetectionRay (RayCast2D)       # 左侧墙壁检测
├── RightWallDetectionRay (RayCast2D)       # 右侧墙壁检测
├── InteractionArea (Area2D)
│   └── CollisionShape2D
├── CameraAnchor (Node2D)
│   └── Camera2D
└── PlayerUI (CanvasLayer)
    └── InteractionPrompt (Control)
        ├── MarginContainer
        └── Label
```

#### 资源定义
```gdscript
# scripts/resources/PlayerMovementConfig.gd
class_name PlayerMovementConfig extends Resource

@export_category("Movement Parameters")
@export var base_speed: float = 200.0
@export var acceleration: float = 1000.0
@export var deceleration: float = 1200.0
@export var max_speed: float = 300.0
@export var dash_speed_multiplier: float = 1.5  # 加速移动时的速度倍率（按住Shift）

@export_category("Jump Parameters")
@export var jump_height: float = 80.0  # 基础跳跃高度（像素）
@export var jump_duration: float = 0.3  # 最大跳跃持续时间（秒）
@export var jump_force: float = 400.0  # 跳跃初始速度（像素/秒）
@export var jump_variable_height: bool = true  # 是否支持可变高度跳跃
@export var jump_cooldown: float = 0.1  # 跳跃冷却时间（秒）
@export var coyote_time: float = 0.1  # 边缘跳跃缓冲时间（秒）
@export var jump_buffer_time: float = 0.1  # 跳跃输入缓冲时间（秒）

@export_category("Physics Parameters")
@export var gravity: float = 800.0  # 重力加速度（像素/秒²）
@export var collision_shape_radius: float = 8.0
@export var collision_shape_height: float = 32.0
@export var collision_layer_mask: int = 0b11111  # 碰撞层掩码（5层）
@export var ground_detection_ray_length: float = 10.0  # 地面检测射线长度
@export var wall_detection_ray_length: float = 5.0  # 墙壁检测射线长度

@export_category("Interaction Parameters")
@export var interaction_range: float = 50.0
@export var auto_interaction_range: float = 80.0
@export var interaction_cooldown: float = 0.3

@export_category("Feedback Parameters")
@export var footstep_interval_fast: float = 0.25  # 快速移动脚步声间隔
@export var footstep_interval_slow: float = 0.5   # 慢速移动脚步声间隔
@export var jump_sound_delay: float = 0.1  # 跳跃音效延迟（秒）
@export var landing_sound_threshold: float = 100.0  # 落地音效速度阈值
@export var camera_shake_intensity: float = 2.0   # 相机震动强度
```

#### 信号映射
```gdscript
# scripts/components/Player.gd
signal player_moved(position: Vector2, velocity: Vector2)
signal player_stopped(position: Vector2)
signal player_jumped(jump_height: float, jump_duration: float)  # 玩家跳跃
signal player_landed(landing_velocity: float)  # 玩家落地
signal player_dash_started(direction: float)  # 玩家开始加速移动
signal player_dash_ended()  # 玩家结束加速移动
signal player_interaction_range_entered(object_type: String, object_id: String)
signal player_interaction_range_exited(object_type: String, object_id: String)
signal interaction_started(object_type: String, object_id: String)
signal interaction_completed(object_type: String, object_id: String, success: bool)
signal player_collided(collision_normal: Vector2, collider_name: String)
signal player_grounded_changed(is_grounded: bool)  # 玩家接地状态变化
signal player_wall_collided(wall_normal: Vector2, wall_name: String)  # 玩家撞墙
signal player_fell_off_bounds  # 玩家掉出边界（错误恢复）
```

### 3. 对话系统

#### 场景树结构
```
DialogueUI (CanvasLayer)
├── Control
│   ├── DialogueBox (Panel)
│   │   ├── MarginContainer
│   │   │   ├── VBoxContainer
│   │   │   │   ├── SpeakerInfo (HBoxContainer)
│   │   │   │   │   ├── Portrait (TextureRect)
│   │   │   │   │   └── SpeakerName (Label)
│   │   │   │   ├── DialogueText (RichTextLabel)
│   │   │   │   └── ChoicesContainer (VBoxContainer)
│   │   │   └── ContinueIndicator (TextureRect)
│   └── BackgroundDim (ColorRect)
```

#### 资源定义
```gdscript
# scripts/resources/DialogueData.gd
class_name DialogueData extends Resource

@export var dialogue_id: String
@export var speaker_id: String
@export var text: String
@export var emotion: String = "neutral"
@export var audio_clip: AudioStream
@export var duration: float = 0.0

@export_category("Dialogue Flow")
@export var choices: Array[DialogueChoice]
@export var next_dialogue_id: String
@export var required_evidence_id: String = ""
```

#### 信号映射
```gdscript
# scripts/autoload/DialogueManager.gd
signal dialogue_started(npc_id: String, dialogue_node_id: String)
signal dialogue_line_changed(speaker_id: String, text: String)
signal dialogue_choice_presented(choices: Array[String])
signal dialogue_ended(npc_id: String, completed: bool)
signal evidence_presentation_requested(npc_id: String, context: String)

@export_category("Dialogue Display")
@export var text_speed: float = 30.0
@export var choice_fade_time: float = 0.3
@export var portrait_switch_time: float = 0.2
```

### 4. 证物收集系统

#### 场景树结构
```
EvidenceUI (CanvasLayer)
├── Control
│   ├── EvidenceInventory (Panel)
│   │   ├── MarginContainer
│   │   │   ├── VBoxContainer
│   │   │   │   ├── InventoryHeader (HBoxContainer)
│   │   │   │   │   ├── Title (Label)
│   │   │   │   │   └── CloseButton (Button)
│   │   │   │   └── GridContainer
│   │   │   │       └── [EvidenceSlot x N]
│   └── EvidenceDetail (Panel)
        ├── MarginContainer
            ├── VBoxContainer
                ├── EvidenceImage (TextureRect)
                ├── EvidenceName (Label)
                └── EvidenceDescription (RichTextLabel)
```

#### 资源定义
```gdscript
# scripts/resources/EvidenceData.gd
class_name EvidenceData extends Resource

@export_category("Basic Info")
@export var evidence_id: String
@export var display_name: String
@export var description: String
@export var evidence_type: String
@export var icon: Texture2D

@export_category("Collection Info")
@export var scene_id: String
@export var hotspot_id: String
@export var is_collectable: bool = true
@export var is_critical: bool = false
@export var collection_order: int = 0

@export_category("Analysis Properties")
@export var can_rotate: bool = false
@export var can_zoom: bool = false
@export var analysis_time: float = 0.0
@export var related_evidence: Array[String] = []
```

#### 信号映射
```gdscript
# scripts/autoload/EvidenceManager.gd
signal evidence_collected(evidence_id: String, evidence_data: EvidenceData)
signal evidence_examined(hotspot_id: String, evidence_id: String)
signal evidence_collection_failed(reason: String)
signal evidence_analysis_started(evidence_id: String)
signal evidence_analysis_completed(evidence_id: String)

@export_category("Collection Effects")
@export var collection_animation_time: float = 0.5
@export var collection_sound_volume: float = 0.8
@export var highlight_intensity: float = 1.2
```

### 5. 谜底揭晓系统

#### 场景树结构
```
TruthRevealUI (CanvasLayer)
├── Control
│   ├── Background (ColorRect)
│   ├── NPCContainer (HBoxContainer)
│   │   └── [NPCSlot x N]
│   ├── QuestionPanel (Panel)
│   │   ├── MarginContainer
│   │   │   ├── VBoxContainer
│   │   │   │   ├── QuestionText (Label)
│   │   │   │   └── Statements (VBoxContainer)
│   ├── EvidencePresentation (Panel)
│   │   ├── MarginContainer
│   │   │   ├── VBoxContainer
│   │   │   │   ├── EvidenceGrid
│   │   │   │   └── PresentButton
│   └── TrustMeter (ProgressBar)
```

#### 资源定义
```gdscript
# scripts/resources/TruthRevelationData.gd
class_name TruthRevelationData extends Resource

@export_category("Session Config")
@export var session_id: String
@export var scene_background: Texture2D
@export var participants: Array[String]

@export_category("Questions")
@export var core_questions: Array[CoreQuestion]
@export var opening_dialogue: Array[DialogueLine]
@export var closing_dialogue: Array[DialogueLine]

@export_category("Trust System")
@export var initial_trust: float = 0.6
@export var trust_thresholds: Dictionary = {
    "high": 0.8,
    "medium": 0.5,
    "low": 0.3,
    "critical": 0.2
}
```

#### 信号映射
```gdscript
# scripts/autoload/TruthRevealManager.gd
signal truth_revelation_started(session_id: String)
signal statement_selected(statement_id: String, is_correct: bool)
signal evidence_presented(evidence_id: String, is_perfect: bool)
signal trust_changed(old_trust: float, new_trust: float)
signal revelation_completed(success: bool)

@export_category("Presentation Effects")
@export var statement_display_time: float = 0.8
@export var evidence_zoom_time: float = 0.5
@export var trust_change_animation_speed: float = 1.0
@export var dramatic_pause_duration: float = 1.5
```

## 测试场景设置

### 场景树结构
```
TestRoom (Node2D)
├── YSort
│   ├── Player (CharacterBody2D)
│   ├── NPCs (Node2D)
│   │   └── [NPC x N]
│   └── Interactables (Node2D)
│       └── [EvidenceHotspot x N]
├── Background (Sprite2D)
├── Collision (StaticBody2D)
│   └── [CollisionPolygon2D x N]
└── GameUI (CanvasLayer)
    ├── DialogueUI
    ├── EvidenceUI
    └── TruthRevealUI
```

## 性能优化建议

1. 使用对象池管理频繁创建/销毁的对象
2. 善用YSort节点处理2D深度排序
3. 对话文本使用RichTextLabel实现打字效果
4. 使用TextureAtlas合并小型UI图标
5. 证物检查时采用SubViewport预渲染

## 调试工具建议

1. 添加DEBUG标记的导出变量
```gdscript
@export_category("Debug")
@export var debug_mode: bool = false
@export var show_interaction_areas: bool = false
@export var infinite_trust: bool = false
```

2. 创建调试指令系统
```gdscript
# scripts/autoload/DebugCommands.gd
func _input(event: InputEvent) -> void:
    if not OS.is_debug_build():
        return
        
    if Input.is_key_pressed(KEY_F3):
        toggle_debug_overlay()
```

## 下一步建议

1. 使用Scene继承实现通用UI模板
2. 添加资源预加载系统
3. 实现存档/读档功能
4. 添加过场动画系统

## 注意事项

1. 所有数值配置必须使用@export暴露到Inspector
2. 信号连接优先使用Editor中的Signal面板
3. 善用Theme资源统一UI风格
4. 使用独立的AudioStreamPlayer管理音效

## 资源链接

- Godot官方文档：https://docs.godotengine.org/
- GDScript基础教程：https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/index.html
- 2D游戏教程：https://docs.godotengine.org/en/stable/tutorials/2d/index.html