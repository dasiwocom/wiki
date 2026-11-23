# 英语词汇分类规则

依据 **Princeton WordNet supersenses**（Jurafsky & Martin《Speech and Language Processing》列出名词 26 类、动词 15 类；副词 1 类）；形容词采用 Tsvetkov et al. 2014（LREC 2014, L14-1073）的 13 类分类。多词表达（MWE）与功能词按语言学常规单独归类。

## 目录结构

一级六类：名词 / 动词 / 形容词 / 副词 / 短语 / 功能词
二级：supersense 或表达类型，文件名格式 `类别码 中文.md`
标准类别全部建立，即使当前无词条也保留空文件（便于后续补词、保持结构完整）。

## 新增词条规则

1. 判断是否为多词表达（含空格，如 `never mind`、`ice cream`）→ 进入 `短语/`
2. 判断是否为功能词（介词、连词、疑问词等）→ 进入 `功能词/`
3. 判断词性 → 进入对应一级目录
4. 判断语义类别 → 进入对应 supersense 文件
5. 一词多义可写入多个文件（如 `fly` 入 `名词/ANIMAL`、`动词/MOTION`）
6. 不确定时先放入最接近的类别，并在词条后标注待定

## Supersense 清单

### 名词（WordNet 26 类，全部建立）
ACT 行为 / ANIMAL 动物 / ARTIFACT 人工物 / ATTRIBUTE 属性 / BODY 身体 / COGNITION 认知 / COMMUNICATION 交际 / EVENT 事件 / FEELING 情感 / FOOD 食物 / GROUP 群体 / LOCATION 地点 / MOTIVE 动机 / OBJECT 自然物 / PERSON 人 / PHENOMENON 现象 / PLANT 植物 / POSSESSION 所有物 / PROCESS 过程 / QUANTITY 数量 / RELATION 关系 / SHAPE 形状 / STATE 状态 / SUBSTANCE 物质 / TIME 时间 / TOPS 顶层

> 注：WordNet 名词没有 `MOTION` 类（MOTION 是动词类）；也没有 `NATURAL EVENT` / `NATURAL OBJECT`，正确写法为 `EVENT` / `OBJECT`。`TOPS` 为抽象顶层类，通常留空。

### 动词（WordNet 15 类，全部建立）
BODY 身体动作 / CHANGE 变化 / COGNITION 认知 / COMMUNICATION 交际 / COMPETITION 竞争 / CONSUMPTION 消费 / CONTACT 接触 / CREATION 创造 / EMOTION 情绪 / MOTION 运动 / PERCEPTION 感知 / POSSESSION 所有物 / SOCIAL 社交 / STATIVE 状态 / WEATHER 天气

### 形容词（Tsvetkov et al. 2014，13 类，全部建立）
PERCEPTION 感知（颜色、味道、光洁） / SPATIAL 空间（尺寸、形状） / TEMPORAL 时间（年龄、周期） / MOTION 运动（快慢） / SUBSTANCE 物质（质地、温度） / WEATHER 天气 / BODY 身体（健康、体况） / FEELING 情感 / MIND 心智 / BEHAVIOR 行为（性格） / SOCIAL 社会 / QUANTITY 数量 / MISC 其他

### 副词（WordNet 1 类）
ALL 全部

### 短语（按结构/功能）
动词短语 / 固定表达（习语） / 副词短语 / 量词短语 / 比较表达 / 复合名词（含国名、节日名等专名）

### 功能词
虚词（介词、连词、限定词、疑问词、比较级等）

## 数据说明

- 共 62 个分类文件、2482 条词条；26+15+13+1+6+1 个标准类别文件全部有词条，无空文件。
- 一词多义（如 `mop` 拖把/拖地、`second` 第二/秒）分别保留，不视为重复。
- 词条来源：原 1871 条 + 新增 611 条日常高频词。新增侧重母语对话与真实英文环境（标牌、菜单、机场、邮件、购物、App），覆盖功能词、常见动词/名词、日常短语；并填满此前为空的 8 个标准类别（`名词/FEELING`、`名词/GROUP`、`名词/MOTIVE`、`名词/OBJECT`、`名词/PROCESS`、`名词/SUBSTANCE`、`名词/TOPS`、`动词/WEATHER`）。
- 历次修正：速度形容词（fast/quick 等）从 TEMPORAL 移到 MOTION；`dry` 从名词移到形容词/SUBSTANCE；`deep-fry`、`stir-fry` 从名词移到动词/CREATION；副词性短语（every day、on time 等）从复合名词移到副词短语；`evacuate`、`shelter` 归位；非标准文件 `NATURAL EVENT` 合并入标准 `EVENT`。
- 备份：`.workbuddy/backup/英语词汇_全面优化前_20261124_0047/`、`.workbuddy/backup/英语词汇_结构补全前_20261124_0055/`、`.workbuddy/backup/英语词汇_新增实用词前_20261124_0102/`，均 SHA256 校验通过。
