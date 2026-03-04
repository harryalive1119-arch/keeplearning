# 证物收集系统

## Intent (设计意图)

设计一个直观且富有沉浸感的证物收集系统，让玩家在犯罪现场通过调查和互动发现关键线索。系统参考《逆转裁判》的证据收集机制，强调"发现-收集-分析-使用"的完整证据链构建过程。核心目标是：

1. **发现乐趣**：通过细致的场景调查发现隐藏线索
2. **收集满足感**：证物收集提供明确的视觉和听觉反馈
3. **逻辑关联**：证据之间可以建立逻辑关系，形成证据网络
4. **推理支持**：收集的证物在谜底揭晓阶段发挥关键作用

系统需要支持多种证据类型（物理证物、照片、文档、录音等），并提供证据分析功能（放大查看、旋转、组合分析）。

## Mechanics (逻辑机制)

### 证物收集流程

#### 1. 调查模式进入
- **触发方式**：在探索状态下，玩家靠近可调查物体按`E`键
- **界面切换**：游戏进入"调查模式"，场景背景变暗/模糊，聚焦于调查区域
- **视角控制**：相机拉近到物体，提供详细视图（可配置缩放级别1.5x-3.0x）
- **状态限制**：玩家不能移动，只能进行调查交互

#### 2. 现场调查机制
- **热点检测**：可调查物体上存在多个"热点区域"（Hotspot）
- **光标变化**：鼠标移动到热点区域时，光标变为放大镜图标
- **调查提示**：热点区域显示半透明高亮（alpha: 0.3）和简短描述
- **多层调查**：某些物体支持多层调查（如：先调查桌子，再调查桌上的信件）

#### 3. 证物收集交互
- **点击收集**：点击热点区域收集证物，播放收集动画
- **收集反馈**：
  - 视觉：证物飞入证据背包图标，屏幕边缘闪烁提示
  - 听觉：独特的收集音效（纸张声、金属声、拍照声等）
  - 文本：屏幕下方显示"已获得：[证物名称]"
- **无法收集**：某些线索只能观察不能收集（如墙上的血迹），系统自动记录到侦探笔记

#### 4. 证据分析功能
- **背包查看**：按`Tab`键打开证据背包，网格化显示所有收集的证物
- **详细查看**：点击证物进入详细分析界面：
  - 3D物体：支持360度旋转、缩放（鼠标拖拽+滚轮）
  - 文档类：可滚动阅读全文，支持关键字高亮
  - 照片类：可放大查看细节，支持亮度/对比度调整
  - 录音类：播放控制（播放/暂停/快进）
- **证据组合**：某些谜题需要组合多个证物（如：将钥匙与锁孔配对）

#### 5. 证据链构建
- **自动关联**：系统自动检测逻辑相关的证据（如：凶器与伤口描述）
- **手动链接**：玩家可以在侦探笔记中手动连接相关证据
- **证据网络**：可视化显示证据之间的关系图（节点-边图）

### 证物类型与处理

#### 物理证物（Physical Evidence）
- **示例**：凶器、指纹、衣物纤维、钥匙
- **收集方式**：直接点击收集
- **分析功能**：3D查看、测量工具、材质分析
- **特殊处理**：某些证物需要"取证工具"（如指纹粉、紫外线灯）

#### 文档证物（Documentary Evidence）
- **示例**：信件、日记、合同、时间表
- **收集方式**：点击文档收集
- **分析功能**：文本高亮、关键字搜索、笔迹分析
- **特殊处理**：加密文档需要解密小游戏

#### 影像证物（Visual Evidence）
- **示例**：照片、监控录像、手机截图
- **收集方式**：使用"虚拟相机"拍摄或直接收集
- **分析功能**：图像增强、人脸识别、时间戳提取
- **特殊处理**：模糊图像需要"清晰化"处理

#### 音频证物（Audio Evidence）
- **示例**：录音、电话记录、环境声音
- **收集方式**：使用"录音设备"录制或收集存储设备
- **分析功能**：声谱分析、语音识别、背景噪音过滤
- **特殊处理**：损坏音频需要修复

#### 数字证物（Digital Evidence）
- **示例**：手机数据、电脑文件、社交媒体记录
- **收集方式**：通过"数字取证工具"提取
- **分析功能**：数据恢复、元数据分析、时间线重建
- **特殊处理**：密码保护设备需要破解

### 调查进度管理

#### 完成度追踪
- **区域调查率**：每个场景显示调查完成百分比
- **关键证据标记**：必须收集的证据用金色边框标记
- **可选证据**：额外线索提供背景故事但不影响主线

#### 调查提示系统
- **轻度提示**：玩家长时间无进展时，显示"也许该调查[X区域]"
- **中度提示**：可调查物体显示更明显的高亮
- **重度提示**：直接显示热点区域位置（辅助模式）

#### 调查日志
- **自动记录**：所有调查活动按时间顺序记录
- **关键发现**：重要证据自动添加书签
- **时间线**：基于证据时间戳自动生成事件时间线

## Data Structure (数据结构)

### EvidenceData (Resource)
```gdscript
# 证物基础数据
class_name EvidenceData extends Resource

@export_category("基本信息")
@export var evidence_id: String  # 唯一标识符（如"knife_001"）
@export var display_name: String  # 显示名称（如"染血的水果刀"）
@export var description: String  # 详细描述
@export var evidence_type: String  # 类型："physical", "document", "visual", "audio", "digital"

@export_category("收集信息")
@export var scene_id: String  # 所在场景ID
@export var hotspot_id: String  # 热点区域ID
@export var is_collectable: bool = true  # 是否可收集
@export var is_critical: bool = false  # 是否关键证据（必须收集）
@export var collection_order: int = 0  # 建议收集顺序（0表示无顺序）

@export_category("内容数据")
@export var texture: Texture2D  # 证物图标/图片
@export var model_3d: PackedScene  # 3D模型（可选）
@export var document_text: String  # 文档内容（如信件全文）
@export var audio_clip: AudioStream  # 音频文件
@export var metadata: Dictionary = {}  # 额外元数据

@export_category("分析属性")
@export var can_rotate: bool = false  # 是否可旋转（3D物体）
@export var can_zoom: bool = false  # 是否可缩放
@export var analysis_time: float = 0.0  # 建议分析时间（秒）
@export var related_evidence: Array[String] = []  # 相关证据ID列表
```

### HotspotData (Resource)
```gdscript
# 热点区域数据
class_name HotspotData extends Resource

@export var hotspot_id: String
@export var parent_object_id: String  # 所属物体ID
@export var position: Vector2  # 在物体上的相对位置
@export var size: Vector2 = Vector2(32, 32)  # 热点区域大小
@export var shape: String = "circle"  # 形状："circle", "rectangle", "polygon"

@export_category("交互属性")
@export var is_visible: bool = true  # 是否可见
@export var requires_tool: String = ""  # 需要的工具ID（空表示不需要）
@export var investigation_text: String  # 调查时显示的文本
@export var action_type: String = "collect"  # 动作类型："collect", "examine", "use_item"

@export_category("证物关联")
@export var evidence_id: String = ""  # 关联的证据ID（如可收集）
@export var alternative_evidence_id: String = ""  # 替代证据ID（如工具不同）
@export var spawns_after_event: String = ""  # 在特定事件后出现
```

### EvidenceCollectionState (Resource)
```gdscript
# 证据收集状态
class_name EvidenceCollectionState extends Resource

@export var collected_evidence: Array[String] = []  # 已收集的证据ID列表
@export var examined_hotspots: Array[String] = []  # 已调查的热点ID列表
@export var evidence_notes: Dictionary = {}  # key: 证据ID, value: 玩家笔记

@export_category("进度追踪")
@export var scene_completion: Dictionary = {}  # key: 场景ID, value: 完成度0-1
@export var last_collected_evidence: String = ""  # 最后收集的证据ID
@export var collection_timestamps: Dictionary = {}  # key: 证据ID, value: 收集时间

@export_category("分析状态")
@export var analyzed_evidence: Array[String] = []  # 已详细分析的证据ID
@export var evidence_combinations: Array[Dictionary] = []  # 已尝试的证据组合
```

### InvestigationTool (Resource)
```gdscript
# 调查工具
class_name InvestigationTool extends Resource

@export var tool_id: String  # 工具ID（如"uv_light", "fingerprint_powder"）
@export var display_name: String  # 显示名称
@export var description: String  # 工具描述
@export var icon_texture: Texture2D  # 工具图标

@export_category("功能属性")
@export var tool_type: String  # 类型："visual", "physical", "digital", "audio"
@export var reveals_hotspot_type: String = ""  # 可显示的热点类型
@export var usage_limit: int = -1  # 使用次数限制（-1表示无限）
@export var cooldown_time: float = 0.0  # 冷却时间（秒）

@export_category("效果")
@export var effect_texture: Texture2D  # 使用时的特效纹理
@export var sound_effect: AudioStream  # 使用音效
@export var success_rate: float = 1.0  # 成功率（0-1）
```

### EvidenceNetwork (Resource)
```gdscript
# 证据关系网络
class_name EvidenceNetwork extends Resource

@export var nodes: Array[EvidenceNode] = []  # 证据节点
@export var edges: Array[EvidenceEdge] = []  # 证据关系边

class EvidenceNode:
    var evidence_id: String
    var position: Vector2  # 在网络图中的位置
    var importance: float = 1.0  # 重要性权重（1-5）
    var category: String = ""  # 分类标签

class EvidenceEdge:
    var from_evidence_id: String
    var to_evidence_id: String
    var relationship_type: String  # 关系类型："supports", "contradicts", "timeline", "location"
    var strength: float = 1.0  # 关系强度（0-1）
    var description: String = ""  # 关系描述
```

## Signals & API (信号与接口)

### 证据收集信号
```gdscript
# 收集事件
signal evidence_collected(evidence_id: String, evidence_data: EvidenceData)
signal evidence_examined(hotspot_id: String, evidence_id: String)  # 调查但未收集
signal evidence_collection_failed(reason: String, hotspot_id: String)

# 进度事件
signal evidence_collection_progress_updated(scene_id: String, progress: float)
signal all_critical_evidence_collected  # 所有关键证据收集完成
signal evidence_analysis_started(evidence_id: String)
signal evidence_analysis_completed(evidence_id: String, insights: Array[String])

# 工具事件
signal investigation_tool_used(tool_id: String, hotspot_id: String, success: bool)
signal investigation_tool_changed(current_tool_id: String)
signal hotspot_revealed(hotspot_id: String, tool_id: String)  # 使用工具后显示新热点
```

### 证据管理信号
```gdscript
# 背包事件
signal evidence_backpack_opened
signal evidence_backpack_closed
signal evidence_selected_in_backpack(evidence_id: String)
signal evidence_deselected_in_backpack

# 分析事件
signal evidence_rotated(evidence_id: String, rotation_degrees: float)
signal evidence_zoomed(evidence_id: String, zoom_level: float)
signal evidence_combination_attempted(evidence_ids: Array[String], success: bool)

# 关系事件
signal evidence_relationship_discovered(evidence_id_1: String, evidence_id_2: String, relationship_type: String)
signal evidence_contradiction_found(evidence_id_1: String, evidence_id_2: String)
signal evidence_timeline_updated(timeline_events: Array[Dictionary])
```

### 调查模式信号
```gdscript
# 模式切换
signal investigation_mode_entered(scene_id: String, focus_object_id: String)
signal investigation_mode_exited(scene_id: String)
signal investigation_camera_zoomed(zoom_level: float)
signal investigation_camera_panned(position: Vector2)

# 热点交互
signal hotspot_hover_started(hotspot_id: String)
signal hotspot_hover_ended(hotspot_id: String)
signal hotspot_clicked(hotspot_id: String, mouse_button: int)
```

### 公共API方法

#### 证据收集API
```gdscript
# 收集控制
func collect_evidence(hotspot_id: String) -> Dictionary  # 返回{success: bool, evidence_id: String}
func examine_evidence(hotspot_id: String) -> void  # 仅调查不收集
func force_collect_evidence(evidence_id: String) -> bool  # 强制收集（调试用）

# 进度查询
func get_collected_evidence_count() -> int
func get_total_evidence_count(scene_id: String = "") -> int
func get_collection_progress(scene_id: String = "") -> float  # 返回0-1
func is_evidence_collected(evidence_id: String) -> bool
func is_critical_evidence_all_collected() -> bool

# 证据访问
func get_evidence_data(evidence_id: String) -> EvidenceData
func get_all_collected_evidence() -> Array[EvidenceData]
func get_evidence_by_type(evidence_type: String) -> Array[EvidenceData]
func get_evidence_by_scene(scene_id: String) -> Array[EvidenceData]
```

#### 调查工具API
```gdscript
# 工具管理
func equip_tool(tool_id: String) -> bool
func get_current_tool() -> String
func use_tool(hotspot_id: String) -> Dictionary  # 返回{success: bool, revealed_hotspots: Array}
func get_available_tools() -> Array[String]

# 工具状态
func get_tool_usage_count(tool_id: String) -> int
func is_tool_available(tool_id: String) -> bool
func recharge_tool(tool_id: String, amount: int = 1) -> void
```

#### 证据分析API
```gdscript
# 分析控制
func analyze_evidence(evidence_id: String) -> void
func rotate_evidence(evidence_id: String, degrees: float) -> void
func zoom_evidence(evidence_id: String, level: float) -> void
func combine_evidence(evidence_id_1: String, evidence_id_2: String) -> Dictionary

# 关系分析
func find_evidence_relationships(evidence_id: String) -> Array[Dictionary]
func check_for_contradictions() -> Array[Dictionary]  # 返回矛盾对列表
func build_evidence_timeline() -> Array[Dictionary]  # 返回时间线事件
func generate_evidence_network() -> EvidenceNetwork
```

#### 调查模式API
```gdscript
# 模式控制
func enter_investigation_mode(scene_id: String, focus_object_id: String = "") -> bool
func exit_investigation_mode() -> void
func is_in_investigation_mode() -> bool

# 热点管理
func get_hotspots_in_scene(scene_id: String) -> Array[HotspotData]
func get_hotspots_on_object(object_id: String) -> Array[HotspotData]
func reveal_hotspot(hotspot_id: String) -> void
func hide_hotspot(hotspot_id: String) -> void

# 相机控制
func set_investigation_zoom(level: float) -> void
func pan_investigation_camera(delta: Vector2) -> void
func reset_investigation_camera() -> void
```

### 外部系统集成接口
1. **游戏状态机**：监听`investigation_mode_entered`/`exited`信号切换游戏状态
2. **对话系统**：通过`evidence_collected`信号更新对话选项可用性
3. **UI系统**：接收证据收集进度信号更新UI显示
4. **存档系统**：通过证据收集状态资源保存进度
5. **音频系统**：播放证据收集和分析相关音效
6. **成就系统**：跟踪证据收集完成度和特殊收集成就