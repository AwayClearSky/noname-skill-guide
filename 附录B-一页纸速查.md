# 附录 B 一页纸速查

## 怎么用这一份

这是全书的**压缩包** —— 把 31 章里"真正会天天用到"的东西抽出来。

**建议打印出来贴在显示器旁边。** 写技能时不再需要翻书，扫一眼就能找到：

> 时机叫什么？字段写哪个？这个方法怎么调？

⚠️ 三点说明：

- **它是索引，不是教程。** 每一项在正文里都有展开讲解，这里只留结论和"去哪一章"。
- **它按"你要做什么"组织**，不按章节顺序。所以看到表里的条目，是能直接用的写法。
- **拿不准就回到正文。** 表里写不下的边界条件，正文都写了。

---

## 一、回合骨架

一个回合长这样（自上而下）：

```text
回合开始
├─ phaseBefore / phaseBeforeStart / phaseBeforeEnd
├─ roundStart（只有本轮第一个回合才有）
├─ phaseBegin
├─ 六个阶段（依次执行）
│    ├─ 准备阶段    phaseZhunbeiBegin / phaseZhunbeiEnd
│    ├─ 判定阶段    phaseJudgeBegin / phaseJudgeEnd
│    ├─ 摸牌阶段    phaseDrawBegin1 / phaseDrawBegin2 / phaseDrawEnd
│    ├─ 出牌阶段    phaseUseBefore / phaseUseBegin / phaseUseEnd / phaseUseAfter
│    ├─ 弃牌阶段    phaseDiscardBegin / phaseDiscardEnd
│    └─ 结束阶段    phaseJieshuBegin / phaseJieshuEnd
├─ phaseEnd
└─ phaseAfter → （下一家的 phaseBefore）
```

**四个"读法"上的坑**：

| 规则 | 说明 |
| --- | --- |
| <strong>"结束阶段"用 `phaseJieshuBegin`</strong> | 别用 `phaseEnd` —— 那是<strong>整个回合跑完</strong>了 |
| <strong>"回合结束时"用 `phaseEnd`</strong>（`global`） | 它排在弃牌、结束阶段<strong>之后</strong>，能统计到本回合全部动作 |
| <strong>「于回合内」＝ `_status.currentPhase == player`</strong> | 不等于"出牌阶段内" |
| <strong>「出牌阶段结束时」用 `phaseUseEnd`</strong> | 每个出牌阶段只有一次 |

⭐ **记「本回合」别自己清标记**：`game.phaseNumber` 每回合开头 `+1`，拿它当"回合哨兵"——记下这个数字，跨回合自动失效。

---

## 二、时机名速查

⚠️ **第一条铁律**：时机名写错**不报错、不提示**，技能就是永远不触发。拿不准就在这张表里对一遍。

### 2.1 回合与阶段

| 时机 | 含义 | 常用 role |
| --- | --- | --- |
| `roundStart` / `roundEnd` | 新一轮开始 / 本轮结束 | `global` |
| `phaseBegin` | 回合开始 | `player` |
| `phaseZhunbeiBegin` / `phaseZhunbeiEnd` | 准备阶段 | `player` |
| `phaseJudgeBegin` / `phaseJudgeEnd` | 判定阶段 | `player` |
| `phaseDrawBegin1` | 摸牌阶段开始（**还能改摸牌数**） | `player` |
| `phaseDrawBegin2` | 摸牌阶段（改摸牌数用这个） | `player` |
| `phaseDrawEnd` | 摸牌阶段结束 | `player` |
| `phaseUseBegin` / `phaseUseEnd` | 出牌阶段开始 / 结束 | `player` |
| `phaseDiscardBegin` / `phaseDiscardEnd` | 弃牌阶段 | `player` |
| `phaseJieshuBegin` / `phaseJieshuEnd` | **结束阶段** | `player` |
| `phaseEnd` / `phaseAfter` | 回合结束 / 整回合结束 | `global` / `player` |
| `phaseChange` | **逐阶段**派发，可改写本回合阶段表 | `player` |

### 2.2 用牌与响应

| 时机 | 含义 | 常用 role |
| --- | --- | --- |
| `useCard` | 使用牌（含"使用后"） | `player` |
| `useCard0` / `useCard1` / `useCard2` | 使用牌的三个中间档（`useCard2`＝选完目标、**追加目标用这个**） | `player` |
| `useCardToPlayer` / `useCardToPlayered` | 结算至某角色时 / 后 | `player` / `target` |
| `useCardToTarget` / `useCardToTargeted` | 指定目标时 / 成为目标后 | `player` / `target` |
| `respond` / `respondSha` / `respondShan` | 打出牌 | `player` |
| `shaMiss` / `shaHit` | 【杀】被【闪】抵消 / 命中 | `player` / `target` |
| `useCard` / `respond`（`global`） | 别人用牌 / 打出牌也监听 —— **"全场有人用牌"就挂这个 role** | `global` |

### 2.3 伤害与体力

| 时机 | 含义 | 常用 role |
| --- | --- | --- |
| `damageBegin` | 伤害事件<strong>还没进 content</strong>（裸名，比 `damageBegin1` 还早） | `player` / `source` |
| `damageBegin1` / `2` | <strong>造成伤害时</strong>的 ①／② 小段（① 改伤害值／属性、② `trigger.cancel()`） | `source` |
| `damageBegin3` / `4` | <strong>受到伤害时</strong>的 ①／② 小段（① 改伤害值、② `trigger.cancel()`） | `player` |
| `damageEnd` | <strong>受到伤害后</strong> | `player` |
| `damageSource` | <strong>造成伤害后</strong> | `source` |
| `damageZero` | 伤害被减为 0 | `player` |
| `loseHp` | 失去体力后（⚠️ <strong>不触发"受伤后"</strong>） | `player` |
| `recover` / `recoverEnd` | 回复体力 | `player` |
| `changeHp` | 体力值变化（含护甲抵扣） | `player` |
| `loseMaxHp` / `gainMaxHp` | 体力上限变化 | `player` |
| `dying` | 濒死（<strong>排在求桃之前</strong>） | `player` |
| `die` / `dieAfter` | 死亡 / 死亡后（`die` 先、`dieAfter` 后） | `player` / `source` |

⚠️ **三条最常踩的**：

- <strong>"受到伤害后"＝`damageEnd`</strong>，"造成伤害后"＝`damageSource`（role 为 `source`）——<strong>两个不一样</strong>。
- <strong>`damageEnd` 只在伤害数 > 0 时派发</strong>；被完全防止／减为 0 时走的是 `damageZero`。
- <strong>改 `trigger.num` 必须做负数保护</strong>（0 点再减＝回复 1 点）。

### 2.4 牌的移动

| 时机 | 含义 | 常用 role |
| --- | --- | --- |
| `gain` / `gainAfter` | 获得牌 | `player` / `global` |
| `lose` / `loseAfter` | 失去牌 | `player` / `global` |
| `loseAsync` / `loseAsyncAfter` | **同时**失去（联机、多人结算） | `global` |
| `discard` / `discardAfter` | 弃置牌 | `player` / `global` |
| `equip` / `equipAfter` | 装备 / 装备后 | `player` |
| `addJudge` / `addJudgeAfter` | 置入判定区 | `player` |
| `addToExpansion` | 置入武将牌上 | `player` |
| `useCardAfter` | 用牌结算完（**此时实体牌还在处理区**） | `player` |

⭐ **"你交给或得到其他角色牌后"** 要用 `{ global: ["gainAfter", "loseAsyncAfter"] }` 两个一起听，再交叉比对。

### 2.5 技能与武将

| 时机 | 含义 | 常用 role |
| --- | --- | --- |
| `useSkill` / `useSkillAfter` | 发动技能 | `player` / `global` |
| `changeSkillsAfter` | 获得/失去技能后（**用 `addSkills` 复数才会派发**） | `player` |
| `showCharacterEnd` / `showCharacterAfter` | 明置武将（⚠️ **基础的 `showCharacter` 不派发**） | `player` |
| `enterGame` | 进入游戏 | `player` |
| `changeCharacterAfter` | 武将牌被交换/变更后 | `player` |

### 2.6 判定

| 时机 | 含义 |
| --- | --- |
| `judge` | 判定（可改判定牌） |
| `judgeFixing` | **判定牌生效时**（改结果用这个） |
| `judgeAfter` | 判定之后 |

⚠️ **改判**：命中"判定牌生效时"＝`judgeFixing`；**两个时机挂哪个都行、但作用不同** —— 挂错会**静默失效**（技能照打、牌没换）。第 14 章专门讲了这一对。

### 2.7 国战专有

| 时机 | 含义 |
| --- | --- |
| `showCharacter` 系 | 明置（用带后缀的 `End` / `After`） |
| `triggerHidden` / `triggerInvisible` | 暗置技 / 隐匿技触发（引擎内置询问明置） |
| `roundStart` / `roundEnd` | 势力状态每轮结算 |
| `changeCharacterAfter` | 换将（易位 / 变更副将 / 主副互换同一个核心事件） |

---

## 三、技能字段速查

### 3.1 触发技骨架（最小可跑）

```js
tutorial_moban: {
	trigger: { player: "phaseZhunbeiBegin" },
	frequent: true,
	filter(event, player) {
		return player.countCards("h") > 0;
	},
	async content(event, trigger, player) {
		await player.draw();
	},
},
```

⚠️ **带 `cost` 时，`check` 完全不生效** —— 要不要发动交给 `cost` 里那个选框自己的 `ai`。

### 3.2 主动技骨架

```js
tutorial_moban2: {
	enable: "phaseUse",
	filter(event, player) {
		return player.countCards("h") > 0;
	},
	selectCard: 1,
	position: "h",
	async content(event, trigger, player) {
		await player.draw();
	},
	ai: {
		order: 3,
		result: { player: 1 },
	},
},
```

⚠️ **`enable` 是字符串时只认两种**：暗号 `"phaseUse"`，或**一字不差等于那个框的事件名**。写成时机名（`"phaseUseBegin"`）⇒ **按钮永远不出现**。
⚠️ **不写 `ai.order`** ⇒ 记 -1 分 ⇒ AI **永远不会点**。

### 3.3 通用开关（最常写错的几个）

| 字段 | 作用 | 坑 |
| --- | --- | --- |
| `forced: true` | 锁定技：不询问 | 暗置时**照样失效**（还有一层询问） |
| `frequent` | "你可以"但 AI 自动 | 仍是普通技，能被"技能失效"封 |
| `direct: true` | 直接发动、跳过确认 | 暗置时不弹明置框 |
| `usable: n` | 一回合限 n 次 | 是**回合**口径，不是阶段 |
| `round: n` | 每 n 轮限一次 | 与"每阶段限一次"**不是一回事** |
| `locked: true` | 免疫"非锁定技失效" | 需与 `forced` 搭配 |
| `charlotte: true` | 别失效（内部技标记） | **拦不住 `removeSkill`** |
| `silent: true` | 别出声（日志/弹窗/优先级） | 与 `charlotte` 分工不同 |
| `unique: true` | 同一技能只算一次 | 用于多来源同名技 |
| `forceDie: true` | **阵亡后仍可触发** | **「死亡时」类技能必写** |
| `forceOut: true` | 移出游戏后仍可触发 | 与 `forceDie` 同理 |

### 3.4 子技与挂载（最容易"查得到却不触发"）

| 字段 | 作用 |
| --- | --- |
| `subSkill: { 名: {...} }` | **只注册**，展开后的 id ＝ `主技id_子技名` |
| `group: ["主技id_子技名"]` | **才挂载** —— 只写 `subSkill` 不写 `group` ⇒ 子技永不触发 |
| `global: "子技名"` | 登记为全局技（人人可用） |
| `init(player, skill)` | 技能被获得时跑一次（**持续 mod 常挂这里**） |
| `onremove: true` | 技能被移除时清 `storage`（**必写**，否则残留） |

### 3.5 表现类

| 字段 | 作用 |
| --- | --- |
| `mark: true` ＋ `marktext` ＋ `intro` | 标记三件套（**缺 `intro` 标记静默不显示**） |
| `markimage` | 用图片当标记 |
| `audio: 2` | 语音（扩展里要写字符串形式） |
| `logAudio` / `logAudio2` | 按情况挑语音 / 按武将挑语音 |
| `audioname2` | 按武将换整套语音（**键是武将 id**） |
| `skillAnimation` ＋ `animationColor` | 发动动画 |
| `dynamicTranslate(player, skill)` | 动态描述（**必须返回字符串**） |
| `ai: { order, result, ... }` | AI |
| `log: false` | 不自动打日志 |

### 3.6 类型标签

| 类型 | 字段 | 要注意 |
| --- | --- | --- |
| 锁定技 | `forced` | 见 3.3 |
| 限定技 | `limited` | content 里**必须** `player.awakenSkill(event.name)` |
| 觉醒技 | `limited` ＋ `forced` | 同上 |
| 转换技 | `zhuanhuanji` | content 里**必须** `changeZhuanhuanji("id")` |
| 持恒技 | `persevereSkill` | 免疫技能失效 |
| 使命技 | `dutySkill` | 官方固定用 `achieve` / `fail` 两个子技 |
| 阵法技 | `zhenfa: "siege"` 或 `"inline"` | **只是标签**，位置判定看 `siege()` / `inline()` |
| 主将技 / 副将技 | `mainSkill` / `viceSkill` | 国战专用；`init` 里查槽位 |
| 势力技 | `groupSkill: "wei"` | 只是标签，效果要自己校验 `player.group` |
| 契定技 | `qidingSkill` | 发动一次后自动变锁定技 |

---

## 四、常用 `player` 方法

### 4.1 牌与体力

| 想做什么 | 写法 |
| --- | --- |
| 摸牌 | `await player.draw(n)` |
| 弃牌 | `await player.discard(cards)` |
| 获得牌 | `await player.gain(cards, "gain2")` |
| 给牌 | `await player.give(cards, target)` |
| 弃置一定数量的手牌 | `await player.chooseToDiscard(n).forResult()` |
| 失去体力 | `await player.loseHp(n)` |
| 回复体力 | `await player.recover(n)` |
| 造成伤害 | `await player.damage(n, source)` |
| 加／减手牌上限 | `player.addMark("...")` / 改 `mod.maxHandcard` |
| 翻面 | `await player.turnOver()` |
| 横置／重置 | `player.link(true)` / `player.link(false)` |

### 4.2 技能

| 想做什么 | 写法 |
| --- | --- |
| 临时获得技能（到回合结束） | `player.addTempSkill(id)` |
| 永久获得 | `player.addSkill(id)` |
| 移除 | `player.removeSkill(id)` |
| 让技能暂时失效 | `player.tempBanSkill(id, expire)` |
| 把技能"废掉"（限定技用） | `player.awakenSkill(id)` |
| 恢复"已发动"状态 | `player.restoreSkill(id)` |
| 判断有没有某技能 | `player.hasSkill(id)` |
| 取技能列表（**含被失效的**） | `player.getSkills(null, false, false)` |
| 加技能封锁 | `player.addSkillBlocker(...)` |

### 4.3 标记

| 想做什么 | 写法 |
| --- | --- |
| 加标记 | `player.addMark(id, n)` |
| 取标记数 | `player.countMark(id)` |
| 移除标记 | `player.removeMark(id, n)` |
| 写自己的记录 | `player.setStorage(id, 值)` |
| 读记录（**务必给缺省值**） | `player.getStorage(id, 0)` |

⚠️ **`getStorage(key)` 缺省是 `[]`** —— 比数字必写 `getStorage(key, 0)`。

### 4.4 其它常用

| 想做什么 | 写法 |
| --- | --- |
| 判断是否自己的回合 | `_status.currentPhase == player` |
| 判断死亡 / 移出游戏 | `player.isDead()` / `player.isOut()` |
| 查历史（本回合） | `player.getHistory("useCard")` |
| 查历史（本轮） | `player.getRoundHistory("useCard")` |
| 攻击范围 | `player.inRange(target)` |
| 距离 | `get.distance(player, target)` |
| 手牌数 / 体力 | `player.countCards("h")` / `player.hp` |
| 加护甲 | `await player.changeHujia(n)` |

⚠️ <strong>`getHistory("damage")` 是"该角色受到伤害"</strong>；"以我为来源"要用 `"sourceDamage"`。
⚠️ <strong>`player.damage(num)` 不传来源时会把上层事件当事人当来源</strong> ⇒ 自伤是常态，须单独分支。

---

## 五、常用 `game` / `get` / `lib` / `_status`

### 5.1 `game`

| 想做什么 | 写法 |
| --- | --- |
| 打日志 | `game.log(player, "文字", card)` |
| 存活角色 | `game.filterPlayer(fn)` |
| 指定范围玩家 | `game.players` / `game.dead` |
| 延迟（等动画） | `game.delay()` / `game.delay(0, get.delayx(ms, ms))` |
| 建事件 | `game.createEvent(name, false, parent)` |
| 遍历所有牌 | `game.cardsGotoSpecial(...)` 等（第 15 章） |
| 回合号 | `game.phaseNumber` / `game.roundNumber` |

### 5.2 `get`

| 想做什么 | 写法 |
| --- | --- |
| 文案 | `get.translation(x)` |
| 技能描述 | `get.skillInfoTranslation(id, player)` |
| 牌名 | `get.name(card, player)` |
| 花色 / 点数 | `get.suit(card, player)` / `get.number(card, player)` |
| 牌的类型 | `get.type(card)` / `get.type2(card)` |
| 颜色 | `get.color(card)` |
| 牌的状态 | `get.position(card)` / `get.owner(card)` |
| 技能对象 | `get.info(skill)` |
| 是否锁定技 | `get.is.locked(skill, player)` |
| 效果估值（AI 用） | `get.effect(target, card, player, viewer)` |
| 智囊名单 | `get.zhinangs()` |
| 名词解释 | `get.poptip(id)` |

### 5.3 `lib` 与 `_status`

| 想取什么 | 写法 |
| --- | --- |
| 技能表 / 卡牌表 / 武将表 | `lib.skill` / `lib.card` / `lib.character` |
| 翻译表 | `lib.translate` |
| 全局技名单 | `lib.skill.global` |
| 六个阶段名 | `lib.phaseName` |
| 当前事件 / 玩家 | `_status.event` / `_status.currentPhase` |
| 是否联机 | `_status.connectMode` |
| 仁库 | `_status.renku` |

---

## 六、选框 API

| 想做什么 | 写法 |
| --- | --- |
| 要不要（是／否） | `player.chooseBool(prompt)` |
| 选一项（读 `result.index`） | `player.chooseControl()` ＋ `.set("choiceList", [...])` |
| 选牌 | `player.chooseCard(n, "h")` |
| 选牌（多选项版） | `player.chooseToDiscard(...)` |
| 选目标 | `player.chooseTarget(prompt, filterTarget)` |
| 选牌＋选人 | `player.chooseCardTarget({...})` |
| 使用牌 | `player.chooseToUse()` |
| 打出牌 | `player.chooseToRespond()` |
| 多人同时选牌 | `player.chooseCardOL(...)` |
| 议事 | `player.chooseToDebate(list)` |
| 发现 | `player.discoverCard(list)` |

**三条通用规矩**：

| 规矩 | 说明 |
| --- | --- |
| **一定要 `await` ＋ `.forResult()`** | 不 await ⇒ 下一步在它做完之前就跑了 |
| **取消时 `result.cards` 是空数组** | 别直接把它丢给 `discard` |
| **AI 最高分 ≤ 0 会自动取消** | 选框都会自动加取消项（`chooseControl` 除外） |

⚠️ <strong>`chooseBool` 不传 `ai` 恒答"是"</strong> —— 想倾向须显式给。
⚠️ <strong>`ai.result.player` ＝ 给发动者的收益</strong>，`.target` ＝ 给目标的收益（<strong>都别自己乘 `get.attitude`</strong>）。

---

## 七、卡牌字段

### 7.1 基础

| 字段 | 作用 |
| --- | --- |
| `type` | `"basic"` / `"trick"` / `"delay"` / `"equip"` |
| `subtype` | 装备子类（`"equip1"` 武器 等） |
| `enable` | 什么时候能用（同技能） |
| `filterCard` / `selectCard` / `position` | 选牌条件 |
| `filterTarget` / `selectTarget` | 选目标条件 |
| `content` | 结算 |
| `distance` | 武器攻击范围 |
| `skills` | 装备牌附带的技能 |

⚠️ **武器 / 防具牌只写 `type` `subtype` `skills`（武器再 `distance`）** —— 别手写 `enable` / `filterTarget` / `content`。

### 7.2 冷门但有用

| 字段 | 作用 |
| --- | --- |
| `allowDuplicate: true` | **允许判定区里重复存在**（蓄谋牌靠它） |
| `blankCard: true` | 判定区的牌**不给别人看详情** |
| `wuxieable: false` | 不可被【无懈可击】响应 |
| `destroy: "discardPile"` | 牌被"销毁"后去哪 |
| `fullskin` / `fullimage` | 卡面用整图（不叠边框） |
| `audio: true` | 使用时有语音 |
| `cardcolor: "spade"` | 指定牌的颜色（和花色分开） |
| `ai: { basic: {...} }` | 牌的 AI 估值 |

⭐ **模式专属牌要进"该模式自己的牌堆"** —— 只加通用牌包 `list` 无效。

---

## 八、排错四层

技能没反应时，**按顺序**问下去，别乱猜：

| 层 | 问什么 | 怎么问 |
| --- | --- | --- |
| **0 文件层** | 扩展开着吗？代码能解析吗？ | 看控制台红字 |
| **1 注册层** | `lib.skill` 里有吗？武将 `skills` 里写了吗？ | 控制台敲那两条咒语 |
| **2 触发层** | 日志里有"发动了"吗？ | 没有 ⇒ 查时机名 / role / `filter` / 次数 / 封锁 |
| **3 执行层** | 响了，但那段代码做了吗？ | 查 `await` / 报错 |

**四条自测咒语**（直接在控制台敲）：

```js
// 把 tutorial_chuyan 换成你的技能 id
lib.skill["tutorial_chuyan"]
lib.translate["tutorial_chuyan"]
lib.translate["tutorial_chuyan_info"]
game.me.getSkills().includes("tutorial_chuyan")
```

**引擎会主动告诉你的四句话**：

| 你看到的 | 它在说什么 |
| --- | --- |
| `缺少info的技能: xxx` | 有个地方用了 `xxx`，但 `lib.skill` 里没有 |
| 单独一行跳出一个技能 id | 这个 id 不存在，多半拼错了 |
| `孩子，你X的翻译传的是什么？！` | 那个技能的 `_info` 不是字符串 |
| 技能名显示成一串英文 id | 少了 `<id>` 这条译键 |

**三条最值钱的判据**：

1. **`lib.skill` 查到 ≠ 挂到人身上** —— 两条咒语要分开问。
2. **时机名对不上是头号死因** —— 差一个字母就是不相等，且完全静默。
3. **`enable` 的值不是时机名** —— `"phaseUse"` 是暗号，写成时机名按钮就不出现。

---

## 九、写代码时的固定约定

这几条不是引擎要求，是**这套教程推荐的习惯** —— 照着写，你的代码会和官方风格一致：

| 约定 | 说明 |
| --- | --- |
| **翻译键顶格写、不加注释符** | `id: "名",` / `id_info: "描述",`，只有分节标题行写 `//` |
| **只被调用一次的辅助逻辑内联** | 别抽成函数；≥ 2 次调用才做成 `lib.skill.<id>.<属性>` |
| **技能对象外不放独立函数** | 也不放模块级变量 |
| **异步方法一律 `await`** | 这是"技能看着发动了却没效果"的头号原因 |
| **`getStorage` 一定给缺省值** | 缺省是 `[]`，比数字会永远不成立 |
| **描述里写了"可以"，代码就要真的问** | 反之亦然 —— 这是对玩家的承诺 |
| **子技写了 `subSkill` 就必须写 `group`** | 否则永不触发 |

---

## 十、最后一张图：写一个技能的全流程

```text
① 读需求：把描述拆成「时机 + 条件 + 效果」
        ↓
② 定时机：在附录 B 第二章的时机表里找到它（这一分步最容易错）
        ↓
③ 定类型：触发技？主动技？持续效果（mod）？
        ↓
④ 写骨架：从第三章对应的骨架抄一份
        ↓
⑤ 填条件：filter / filterCard / filterTarget —— 与 content 用同一套判据
        ↓
⑥ 写效果：await player.xxx()
        ↓
⑦ 配 AI：触发技写 cost 里的 ai；主动技写 ai.order ＋ ai.result
        ↓
⑧ 补译键：id / id_info（顶格写）
        ↓
⑨ 进游戏试：看日志有没有"发动了"
        ↓
⑩ 没反应？翻回第八章，按四层查
```

⭐ **第 ② 步和第 ⑩ 步是新手 90% 的时间去向。** 所以这两张表（时机表、排错四层）值得你先单独记牢。

---

## 结语

全书到这里结束。

最后送你一句话，它概括了这 31 章 + 2 份附录的全部：

> **引擎很安静 —— 它不报错，只是不做事。**
> **所以写技能的本事，一半是"知道该写什么"，另一半是"知道它会怎么沉默"。**

祝你的技能一次跑通。
