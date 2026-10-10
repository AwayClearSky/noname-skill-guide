# 第 12 章 `player` 对象

## 这一章要做什么

第 11 章讲了五个工具箱。但你翻回自己的技能代码，会发现出现最多的名字**不是那五个**，而是这个：

```js
player.draw();
player.countCards("h");
player.hasSkill("tutorial_zhutai");
player.addMark("tutorial_jishi_effect", 2);
player.awakenSkill(event.name);
```

**`player`。**

它是技能代码里出现频率最高的名字 —— 官方技能里 `player.xxx` 的调用超过 **3 万次**，比第 11 章那五个工具箱加起来还多。原因很简单：

> **`get` / `game` / `lib` / `ui` / `_status` 都是"环境"，而 `player` 是"人"。**
>
> 技能写的都是"某个人做什么事"，所以主角永远是 `player`。

**这一章就把这个主角讲透。**

它身上挂了**三百多个**方法 —— 比第 11 章那五个对象加起来还多。别慌：**真正要背的只有三十来个**，剩下的用到再查。这一章按"**你想做什么**"分成 8 组，每组先给必背表，再挑一两个最容易踩的地方展开。

> 💡 **和上一章一样的读法**：第一遍通读、混个眼熟；写技能卡住时回来查。

### 先分清"属性"和"方法"

`player` 身上有两类东西：

| 类型 | 长什么样 | 例子 |
| --- | --- | --- |
| **属性** | 直接写，不加括号 | `player.hp`（当前体力）、`player.maxHp`（体力上限）、`player.group`（势力） |
| **方法** | 要加括号，可能带参数 | `player.draw()`、`player.countCards("h")` |

**怎么区分？** 记住一个粗糙但好用的判据：

> **"要算一下才得到的"都是方法，"摆在那儿的事实"多是属性。**

- "他手上有几张牌" —— 要**数** → `player.countCards("h")`
- "他现在几点体力" —— 是**摆着的** → `player.hp`

⚠️ **两个最容易写错的**：

| 想表达 | ✅ 对 | ❌ 错 |
| --- | --- | --- |
| 是不是男性 | `player.hasSex("male")` | `player.sex === "male"` |
| 手牌上限 | `player.getHandcardLimit()` | `player.hp` |

**为什么"性别"要用方法？** 因为有些技能会改性别。`player.sex` 是"原始值"，`hasSex` 才是"算上各种修正之后的真实答案"。**手牌上限同理** —— 它是一个方法而不是一个属性，因为〖筑台〗这类技能会改它（第 9 章你写过一个）。

> ⭐ **一条通用规律：凡是"可能被技能修改"的东西，都用方法去问，别直接读属性。**
>
> 这条规律在本章会反复出现（距离、攻击范围、手牌上限、能不能用某张牌……）。

---

## 一、①牌：读牌与动牌

这是用得最多的一组。

### 1.1 必背：怎么看牌

| 方法 | 作用 | 例子 |
| --- | --- | --- |
| `countCards(position, filter)` | **数**有几个 | `countCards("h")` → 手牌数 |
| `getCards(position, filter)` | **拿**出这些牌（返回数组） | `getCards("h")` → 手牌数组 |
| `hasCard(filter, position)` | **有没有**满足条件的牌 | `hasCard(c => get.suit(c) == "club", "he")` |
| `hasCards(position, filter)` | **有没有**牌（同上，判据在前） | `hasCards("h")` |
| `getEquips(subtype)` | 装备区里的牌 | `getEquips(1)` → 武器 |

⭐ **`filter` 有两种写法**（引擎对两个方法的参数说明是权威的）：

> `position`：牌区，`h`手牌区、`e`装备区、`j`判定区、`x`扩展区、`s`特殊区（木牛流马牌的位置）。
> `filter`：可以是**牌名**、**牌名数组**、**属性对象**、或**过滤函数**。

```js
player.countCards("h", { color: "red" });            // 红色手牌数
player.getCards("he", card => get.type(card) === "basic");   // 手牌+装备里的基本牌
player.countCards("e", "sha");                        // 装备区里叫"杀"的（基本为 0）
```

⚠️ **`position` 能拼**：`"he"` = 手牌+装备、`"hes"` = 再加特殊区、`"hej"` = 含判定区。**不写 = 只数手牌。**

⚠️ **两个顺序相反的坑**（真的很容易记反）：

| 方法 | 参数顺序 |
| --- | --- |
| `hasCard(filter, position)` | **判据在前** |
| `hasCards(position, filter)` | **位置在前** |
| `countCards(position, filter)` | **位置在前** |
| `getCards(position, filter)` | **位置在前** |

⭐ **记法**：**带 `s` 的那个（`hasCards`）跟兄弟们一样"位置在前"；光杆的 `hasCard` 是特例。**

### 1.2 必背：怎么动牌

| 方法 | 作用 |
| --- | --- |
| `gain(cards, ...)` | **获得**牌 |
| `discard(cards)` | **弃置**牌（不询问，直接弃） |
| `modedDiscard(cards)` | 弃置牌，但**先过一遍"不可弃置"检查** |
| `loseToDiscardpile(cards)` | **置入弃牌堆**（≠弃置） |
| `give(cards, target, visible)` | **交给**某个角色 |

**"从哪来"写在 `source` 字段上** —— 这是 `gain` 最常被问的地方：

```js
await player.gain({ cards, animate: "gain2" });
// animate 是"获得方式"，主要影响动画与日志的措辞

await player.gain({ cards, source: target, animate: "give", bySelf: true });
// 从 target 那里获得（明示来源），走"给"的动画
```

⭐ **`animate` 最常用的几个关键字**：`"gain2"`（凭空获得，最通用）、`"draw"`（摸牌）。

**`discard` 与 `modedDiscard` 的区别**（第 11 章速查提过，这里说清）：

> **`discard` 是"我就是要弃这些牌"，`modedDiscard` 是"我要弃这些牌，但如果有技能说不能弃，那就别弃"。**

```js
// 锁定技效果，无条件弃置
await player.discard({ cards });

// "弃置其所有牌" —— 尊重"不可弃置"类技能，用这个
await player.modedDiscard({ cards: player.getCards("he") });
```

⭐ **什么时候用哪个？** 记住：**玩家的主动弃牌用 `discard`；描述里写"弃置…所有牌"这种大规模动作，用 `modedDiscard`。**

### 1.3 必背：动"别人"的牌

这三个是跨角色拿牌/弃牌的专用方法。**它们和第 1.1 节的方法最大的区别是：会弹框问玩家选。**

| 方法 | 做什么 |
| --- | --- |
| `discardPlayerCard(...)` | **弃置**某人区域里的一张牌 |
| `gainPlayerCard(...)` | **获得**某人区域里的一张牌 |
| `choosePlayerCard(...)` | **只选不动**（选了之后你自己决定干什么） |

```js
// 弃置目标装备区或判定区的一张牌（强制，不让他拒绝）
await player.discardPlayerCard("ej", true, event.target);

// 获得目标的一张手牌或装备牌（强制）
await player.gainPlayerCard(target, "he", true);

// 只选：选完之后我自己处理这张牌
const result = await player.choosePlayerCard(target, "he", true).forResult();
```

⭐ **注意这三段代码的参数顺序——它们看起来不一样，但都对。** 为什么？**这是一个重要的机制，下一章（第 13 章）会专门讲**。这里你先记住结论：

> **这几个方法的参数是"按类型认领"的，跟顺序无关。**
> `player` → 目标、`"he"`/`"ej"` → 牌区、`true` → 强制、字符串 → 提示语。**引擎自己看类型决定往哪儿放。**

### 1.4 必背：用牌

| 方法 | 作用 |
| --- | --- |
| `useCard(card, targets)` | **使用**一张牌 |
| `canUse(card, target, distance, includecard)` | **能不能**对他用这张牌 |
| `hasUseTarget(card, distance, includecard)` | **有没有**任何一个能用的目标 |
| `getUseValue(card, distance, includecard)` | 用这张牌**值多少**（AI 用） |

**`canUse` 的第 3、4 个参数**是最容易忽略的地方：

```js
player.canUse("sha", target);
// 默认：既看距离、也看出杀次数限制

player.canUse("sha", target, false);
// 第 3 参 = false ⇒ 不看距离

player.canUse("sha", target, true, false);
// 第 4 参 = false ⇒ 也不看出杀次数
```

⭐ **规律：这两个参数都是"要不要受限制"，写 `false` 就是"豁免"。** 写"无视距离使用一张【杀】"这类效果时用得上。

**`hasUseTarget` 的典型用法**（写"视为使用"类技能前的兜底）：

```js
if (!player.hasUseTarget({ name: "sha", isCard: true })) {
	return;         // 一张【杀】都打不出去，别弹框
}
await player.chooseUseTarget({ card: { name: "sha", isCard: true }, forced: true, addCount: false });
```

> ⚠️ **忘了这个兜底，会出现"弹了个框但什么都不能选"的空框。**

### 1.5 常用

| 方法 | 作用 |
| --- | --- |
| `showCards(cards, str, flashAnimation, isFlash)` | **展示**牌（第 3 参不传 = 纯展示，牌不动） |
| `showHandcards(str)` | 展示所有手牌 |
| `viewHandcards(target)` | **查看**某人的手牌（只有你自己看得到） |
| `equip(card, draw)` | 装备一张牌 |
| `addJudge(card, cards)` | 把牌放进判定区 |
| `removeGaintag(tag, cards)` | 去掉牌上的某个标记 |
| `countDiscardableCards(player, position, filter)` | 他有几张**可弃置**的牌 |
| `countGainableCards(player, position, filter)` | 他有几张**可被获得**的牌 |

**最后两个是"判断"用的** —— 写"弃置其一张牌"前先问一句：

```js
if (target.countDiscardableCards(player, "he")) {
	await target.chooseToDiscard({ position: "he", forced: true });
}
```

⭐ **这样写就不会出现"他一张牌都没有，却要求他弃牌"的尴尬。**

---

## 二、②体力：血与牌

### 2.1 必背：加减体力

| 方法 | 作用 | 注意 |
| --- | --- | --- |
| `draw(num)` | **摸** `num` 张牌 | 不写 `num` 就是 1 张 |
| `drawTo(num)` | **摸到** `num` 张 | 已经够了就什么都不做 |
| `recover(num)` | **回复** `num` 点体力 | 不写就是 1 点 |
| `recoverTo(num)` | **回复到** `num` 点 | 超额自动空转 |
| `loseHp(num)` | **失去** `num` 点体力 | 不写就是 1 点 |
| `damage(num)` | **受到** `num` 点伤害 | 不写就是 1 点 |
| `gainMaxHp(num)` | 体力上限 +num | |
| `loseMaxHp(num)` | 体力上限 -num | |

**"摸牌"和"回复体力"都是"到某个数为止"更常用** —— 比如官方〖遗计〗类效果、〖仁德〗。

```js
await target.drawTo(Math.min(5, num));   // 摸到 5 张（或更少，如果已经超过）
await player.recoverTo(num);             // 回复到 num 点
```

### 2.2 ⭐ 必背：`damage` 的"来源"

这是本组最容易踩的地方。

`player.damage()` 的完整写法是给它一个**参数对象**：

```js
await target.damage({ num: 2, source: player, card: trigger.card, nature: "fire" });
```

**其中 `source`（伤害来源）非常关键** —— 因为：

> ⭐ **不写 `source` 时，引擎会把"当前事件的当事人"当成来源。**

**后果**：你在一张伤害类技能里写 `player.damage()`，本意是"他受伤"，但如果此刻正在跑的事件主角就是 `player`，**那这次伤害的来源就成了他自己** —— 于是"造成伤害后"类的技能会被他自己的伤害触发，各种奇怪的连锁就来了。

**两种正确的写法**：

```js
// ① 明确指定来源（推荐）
await _status.currentPhase.damage(1, player);       // 简写：damage(num, source)
await target.damage({ num: 1, source: player });    // 完整写法

// ② 明确"没有来源"（写"失去体力"而不是"受到伤害"时用）
await player.damage("nosource");
```

⭐ **判据**：描述里写"**受到伤害**" → 给来源；写"**失去体力**" → 其实该用 `loseHp()`（它天然无来源），**别用 `damage("nosource")` 代替**。

> 💡 **`loseHp` 和 `damage` 是两回事**：
>
> | | 触发"受到伤害后"类技能 | 触发器 |
> | --- | --- | --- |
> | `loseHp()` | ❌ 不触发 | "失去体力后"类 |
> | `damage()` | ✅ 触发 | "受到伤害后"类 |
>
> 描述写"失去 1 点体力"就用 `loseHp`，写"受到 1 点伤害"才用 `damage`。**别混。**

### 2.3 必背：查体力的状态

| 方法 | 作用 |
| --- | --- |
| `player.hp` / `player.maxHp` | 当前体力 / 体力上限 |
| `isDamaged()` | **已受伤**（hp < maxHp） |
| `getDamagedHp()` | **已失去几点体力**（maxHp − hp） |
| `needsToDiscard(add, filter, pure)` | 手牌数**超过**手牌上限多少 |
| `isIn()` | **还在场上**（没阵亡、没被移出） |
| `isAlive()` | **活着**（含已阵亡但仍在处理中的情况） |

**`isDamaged` 和 `getDamagedHp` 的区别**：

```js
if (player.isDamaged()) { }        // 只问"有没有受伤"（对错）
const n = player.getDamagedHp();   // 问"伤了多少"（数字）
```

**`needsToDiscard()` 的用法** —— 判"要不要弃牌"：

```js
if (player.needsToDiscard()) { }       // 手牌超了，得弃
const n = player.needsToDiscard();     // 超出几张
```

⭐ **别自己写 `player.countCards("h") > player.getHandcardLimit()`** —— `needsToDiscard` 里还考虑了"某些牌不计入手牌上限"（比如〖筑台〗那种被技能标记的牌）。**用现成的。**

### 2.4 ⚠️ `isIn` 还是 `isAlive`？

这两个很像，但有区别：

| 方法 | 什么时候用 |
| --- | --- |
| `isAlive()` | 想知道"他是不是活着"（**一般用这个**） |
| `isIn()` | 想知道"他还在不在场上"（排除"已阵亡但正在处理中"的中间态） |

**实用建议**：**判"还能不能对他做事"，用 `isIn()`** —— 因为死亡结算过程中，角色可能"已经死了但牌还在"，这时 `isAlive` 的答案可能让你误判。

> 💡 **另外提醒一句**（第 11 章速查里提过）：**用 `game.filterPlayer()` / `game.countPlayer()` 数人头时，不用再叠 `isIn()`** —— 那两个方法遍历的是 `game.players`，而阵亡者已经被移进 `game.dead` 了，**天然不含死者**。
>
> `isIn()` 只在"**你手里已经握着一个角色引用**"（`event.source`、`trigger.player`、storage 里存的）时才需要。

---

## 三、③技能：增删查

### 3.1 必背

| 方法 | 作用 | 活多久 |
| --- | --- | --- |
| `hasSkill(skill)` | 有没有这个技能 | — |
| `addSkill(skill)` | **永久**获得 | 整局 |
| `removeSkill(skill)` | 永久失去 | — |
| `addTempSkill(skill, expire)` | **临时**获得 | 到 `expire` 为止 |
| `addSkills(skill, popup)` | 一次加一批 | 整局 |
| `awakenSkill(skill)` | **觉醒**（限定技/觉醒技已发动） | — |

**`addTempSkill` 的第二个参数**是"什么时候失效"：

```js
player.addTempSkill("tutorial_mark", "phaseUseEnd");    // 到本出牌阶段结束
player.addTempSkill("tutorial_mark", "roundEnd");       // 到本轮结束
player.addTempSkill("tutorial_mark");                   // 不写 = 到本回合结束
```

⭐ **不写第二参就是"本回合结束"** —— 这是最常用的场景，所以引擎把它当默认。

⚠️ **`awakenSkill` 不是"让他觉醒"的意思**！它的真实作用是：

> **把一个限定技/觉醒技标记为"已发动"，并且顺手把它从技能列表里摘掉。**

所以第 9 章你写的〖积势〗里那句：

```js
player.awakenSkill(event.name);
```

读作：<strong>"把〖积势〗标记为已用"</strong>。—— 少了它，限定技可以无限发动。

### 3.2 ⭐ 技能标签：`hasSkillTag`

引擎给技能准备了一批"**标签**"（tag），用来快速问"他有没有某类能力"：

```js
if (target.hasSkillTag("noe")) { }            // 他是不是"不能被过河拆桥"
if (player.hasSkillTag("jueqing")) { }        // 他是不是"伤害视为失去体力"
```

⭐ **标签是引擎的"通用约定"** —— 你写了某个标签，**其他技能就知道怎么对待你**。比如：

| 标签 | 含义 |
| --- | --- |
| `jueqing` | "对你造成的伤害视为失去体力" |
| `noe` | 不能被弃置装备 |
| `nodraw` | 摸牌数不受影响 |
| `mingzhi_yes` / `mingzhi_no` | 国战：要不要明置（第 26 章细讲） |
| `forceMajor` | 国战：势力视为大势力（第 27 章细讲） |

**怎么用的？** 在你自己的技能里声明：

```js
ai: {
	jueqing: true,        // ⇒ 别人 hasSkillTag("jueqing") 会得到 true
},
```

> ⚠️ **技能标签全部写在 `ai: { … }` 里**（这是引擎的历史遗留），不是写在顶层。看到 `ai` 里有个你没见过的键，先想想"是不是个标签"。

### 3.3 ⭐ "视为拥有技能"：`addAdditionalSkill`

第 11 章速查里见过一句"视为拥有技能 = `addAdditionalSkill`"。它是这么用的：

```js
player.addAdditionalSkill("tutorial_pijian", "tutorial_pijian_effect");
//                      └─ 主人技能 id ─┘  └─ 要"视为拥有"的技能 ─┘

player.removeAdditionalSkill("tutorial_pijian");
```

**它和 `addSkill` 的区别**：

| | `addSkill` | `addAdditionalSkill` |
| --- | --- | --- |
| 写进 `player.skills` 吗 | ✅ 会 | ❌ 不会 |
| 拆开时怎么拆 | `removeSkill` 单独拆 | 按"主人 id"**整批**拆 |

⭐ **"按条件切换"的场景用它最方便** —— 比如"若你的装备区有牌，视为拥有〖A〗；否则视为拥有〖B〗"：

```js
// 条件变了就把旧的整批摘掉、换新的上去
player.removeAdditionalSkill("tutorial_pijian");
player.addAdditionalSkill("tutorial_pijian", player.countCards("e") ? "tutorial_pijian_a" : "tutorial_pijian_b");
```

### 3.4 常用

| 方法 | 作用 |
| --- | --- |
| `tempBanSkill(skill, expire)` | 让某技能**暂时失效**（不传 expire = 到本回合结束） |
| `getSkills(...)` | 拿到他的技能列表（第 3 个参数传 `false` 能看见"被废掉"的技能） |
| `changeZhuanhuanji(skill)` | **转换技**翻面（第 17 章细讲）；翻面后武将牌上的 ☯ 图标会**自动转到对应朝向** |
| `addCharge()` / `countCharge()` / `removeCharge()` | 蓄力技的充能（第 17 章细讲） |

---

## 四、④标记与存储：记住一个数

第 9 章你写过〖积势〗的标记。这里把整套补齐。

### 4.1 必背：标记

| 方法 | 作用 |
| --- | --- |
| `addMark(skill, num, log)` | 加 `num` 枚标记（不写 `num` 就是 1） |
| `countMark(skill)` | 现在有几枚 |
| `removeMark(skill, num, log)` | 减 `num` 枚 |
| `hasMark(skill)` | 有没有（≥1） |
| `setMark(skill, num, log)` | **直接设成** `num` 枚 |

⭐ **第三个参数 `log`** 控制"要不要写日志"：不传 = 写（默认会弹一行"XX 获得了 1 枚「势」"），传 `false` = 悄悄加。

```js
player.addMark("tutorial_jishi_effect", 2);          // 会打日志
player.addMark("tutorial_jishi_effect", 1, false);   // 不加日志
```

> 💡 **什么时候不加日志？** 当标记是"内部计数器"、玩家不需要看见时。**能看见的标记就让它打日志** —— 那是玩家的信息来源。

### 4.2 ⭐ `getStorage` 与 `player.storage` —— 两套并存的写法

第 11 章提过一句"这两套并存"，这里说清。

| 写法 | 特点 |
| --- | --- |
| `player.storage.<技能id>` | **直接读写属性**，官方老技能大量使用 |
| `player.getStorage(name, defaultValue = [])` / `player.setStorage(name, value, mark)` | 走引擎的**缺省值机制** |

```js
// 写法 A：直接属性（要先判存在）
if (player.storage.hujiaing) {
	// 已经记过了
}
player.storage.hujiaing = true;

// 写法 B：getStorage（有缺省值，不用判存在）
const n = player.getStorage("tutorial_chaji", 0);   // 没存过就得到 0
player.setStorage("tutorial_chaji", n + 1);
```

⚠️ **`getStorage` 的缺省值不写时是空数组**：

```js
player.getStorage("tutorial_chaji");       // 没存过 → [] （空数组！）
player.getStorage("tutorial_chaji", 0);    // 没存过 → 0
```

⭐ **这是个经典坑**：拿它跟数字比较时，**必须显式写第二参**：

```js
// ❌ 错的：[] > 3 永远是 false，这个判断恒不成立
if (player.getStorage("tutorial_chaji") > 3) { }

// ✅ 对的
if (player.getStorage("tutorial_chaji", 0) > 3) { }
```

**该用哪套？** 记住：

> **需要"没存过时有个合理默认值" → 用 `getStorage`；只是记个开关 → 直接 `player.storage` 也行。**

**新写的技能推荐 `getStorage`** —— 少一个"判空"的步骤。

### 4.3 常用：批量记录

| 方法 | 作用 |
| --- | --- |
| `markAuto(skill, info)` | **自动记录**一组数据（并自动维护标记显示） |
| `unmarkAuto(skill, info)` | 从记录里移除 |

```js
player.markAuto("tutorial_jifeng", [target]);
// 记住"target 这个人"，并且自动在标记里显示出来

player.unmarkAuto("tutorial_jifeng", [target]);
```

⭐ **`markAuto` 的特别之处**：它记的东西会**自动变成标记的显示内容**（技能上会显示"锋：曹操"这类文字）。用它比 `addMark` 更适合"记人名"。

```js
// 读回来
player.getStorage("tutorial_jifeng").includes(target)
```

> ⚠️ **`markAuto` 记的是"数组"，读的时候用 `.includes` 判在不在。** 这和 `addMark` 的"计数"是两种用途。

### 4.4 武将牌上的牌：`addToExpansion` / `getExpansions`

这是"〖屯田〗的『田』、〖寄锋〗的『锋』"那类机制。

| 方法 | 作用 |
| --- | --- |
| `addToExpansion(params)` | 把牌**置于武将牌上** |
| `getExpansions(tag)` | 拿回那些牌 |
| `loseToDiscardpile(cards)` | 把牌从武将牌上移进弃牌堆 |

```js
await player.addToExpansion({
	cards: event.cards,
	animate: "gain2",
	gaintag: ["tutorial_jifeng"],
});
//        └─ "称为『锋』"就是靠它 ─┘

const cards = player.getExpansions("tutorial_jifeng");
```

⭐ **`gaintag` 就是"称为 X"的实现** —— 它是一张牌的"标签"，用来区分"武将牌上的哪些牌是哪批"。配套还要在 `translate` 里写一条：

```js
tutorial_jifeng: "锋",
```

否则界面上显示的是一串英文。（第 10 章讲的"翻译键"在这里也管用。）

> 💡 **第 15 章会专门讲这一族机制**（武将牌上的牌、明置手牌、分配牌）。这里先认识这三个方法。

---

## 五、⑤询问：让玩家选

### 5.1 一张索引表

这一组**在第 7 章已经系统讲过**（`chooseToDiscard` / `chooseCard` / `chooseTarget` / `chooseBool` / `chooseControl`），第 13 章还会补上另一半。这里只列个索引，方便你查：

| 想干什么 | 用什么 |
| --- | --- |
| 让他选牌（并弃掉） | `chooseToDiscard` |
| 让他选牌（只是选） | `chooseCard` |
| 让他选人 | `chooseTarget` |
| 让他选一项 | `chooseControl` |
| 问他"要不要" | `chooseBool` |
| 让他用一张牌 | `chooseToUse` |
| 让他打出一张牌响应 | `chooseToRespond` |
| 让他把牌交给你 | `chooseToGive` |
| 拼点 | `chooseToCompare` |
| 一排按钮里挑 | `chooseButton` |
| **一次选牌 + 选人** | `chooseCardTarget` |
| **拖动移动牌** | `chooseToMove` |

### 5.2 三步套路是通用的

⭐ **所有 `chooseXxx` 的用法都一样**（第 7 章的三步套路）：

```js
const result = await player.chooseXxx("提示语", ...条件).forResult();
if (!result.bool) {
	return;          // 玩家取消了
}
// 用 result.cards / result.targets / result.control …
```

---

## 六、⑥历史与统计：过去发生了什么

### 6.1 必背

| 方法 | 问什么 |
| --- | --- |
| `getHistory(key, filter, last)` | 某个历史的**事件列表** |
| `hasHistory(key, filter, last)` | 有没有满足条件的历史（返回对错） |
| `getStat(key)` | 统计数值（技能发动次数、造成了多少伤害……） |
| `countUsed(card, type)` | 本回合**用过几张**某牌 |
| `getRoundHistory(key, filter, num, keep, last)` | 本轮的历史 |
| `isPhaseUsing()` | **现在是不是在他的出牌阶段内** |
| `getLastUsed(num)` | 他最后一次用的牌 |

### 6.2 ⭐ `getHistory` 的常用 key

| key | 装什么 |
| --- | --- |
| `"useCard"` | 他**使用**过的牌 |
| `"respond"` | 他**打出**过的牌 |
| `"damage"` | 他**受到**的伤害 ⚠️ |
| `"sourceDamage"` | 他**造成**的伤害 |
| `"gain"` | 他获得过的牌 |
| `"lose"` | 他失去过的牌 |
| `"skipped"` | 他跳过过的阶段 |
| `"useSkill"` | 他发动过的技能 |
| `"custom"` | 自定义的（配合 `.set("customHistory", ...)`） |

⚠️ <strong>`"damage"` 是"他受到的"，不是"他造成的"</strong> —— 这是最经典的看反。<strong>"造成"要加 `source`</strong>：

```js
// ❌ 读反了：这是"他挨了多少打"
player.getHistory("damage");

// ✅ "他打出去多少"
player.getHistory("sourceDamage");
```

⭐ <strong>一句话记法</strong>：<strong>`getHistory("damage")` 是"我挨的打"，加 `source` 才是"我打的"。</strong>

### 6.3 ⭐ `getHistory` 的第三个参数 `last`

第三个参数传一个事件，表示"**只算到这个事件为止**"：

```js
player.getHistory("useCard", evt => evt.card.name === "sha", 某个事件);
```

**用途**：判"**这是他这回合第一次**用【杀】吗"：

```js
const evtx = event.getParent();     // 本次使用牌的事件
return !player.hasHistory("useCard", evt => evt !== evtx && get.name(evt.card) === "sha", evtx);
//                                                    └─ ⚠ 必须写 evt !== evtx ─┘
```

⚠️ **`last` 是"包含"截止事件的** —— 所以**必须写 `evt !== evtx` 把自己排掉**，否则会匹配到自己，判断恒不成立（技能永不发动）。

> 💡 **为什么不用 `usable: 1` 来做"每回合首次"？** 因为 `usable` 记的是"**发动次数**" —— 玩家发动后取消（比如cost取消），`usable` 就可能被消耗掉或没消耗，行为跟"首次使用"不是一回事。**要判"首次"就老老实实查历史。**

### 6.4 `getStat` 与技能自己的历史

```js
// 我发动过多少次这个技能（按玩家分开记）
const n = player.getStat("skill")["tutorial_lunzhan"] || 0;
```

⭐ **描述里写"X 为本回合此技能发动次数"时**，官方习惯写成 `.length + 1`：

```js
const n = player.getHistory("useSkill", evt => evt.skill === "tutorial_lunzhan").length + 1;
//                                                                                 └─ 那个 +1 ─┘
```

**为什么 `+1`？** 因为技能发动时日志是**在效果跑完之后**才记的 —— 所以此刻读到的是"不含本次"的次数。**加 1 才是"含本次"。**

---

## 七、⑦判定·距离·目标

### 7.1 必背

| 方法 | 作用 |
| --- | --- |
| `judge(card)` | **进行判定**（返回可 await 的结果） |
| `inRange(target)` | 目标**在不在攻击范围内** |
| `inRangeOf(source)` | 我**在不在**别人的攻击范围内（反过来的问法） |
| `distanceTo(target)` | 到目标的**距离** |
| `canCompare(target)` | **能不能跟他拼点** |
| `hasValueTarget(card, distance, includecard)` | 用这张牌**有没有划算的目标**（AI 用） |
| `hasUsableCard(name, type)` | 他手上有没有一张**能当这张牌**用出去的牌 |
| `getHandcardLimit()` | **手牌上限** |
| `getAttackRange()` | **攻击范围** |
| `getEquipRange()` | 装备提供的攻击范围 |

**`inRange` 和 `distanceTo` 的区别**：

```js
player.inRange(target);        // "够得着吗" —— 对错
player.distanceTo(target);     // "多远" —— 数字
```

⭐ **别自己拿距离和攻击范围比大小** —— 引擎会在中间套一层 `globalFrom`/`globalTo` 修正（第 9 章的〖马术〗就是改这个的）。**用 `inRange` 一步到位。**

⭐ **反过来的问法也有现成的：`player.inRangeOf(source)`** ＝ "我在不在 source 的攻击范围内"（源码就一句 `return source.inRange(this)`）。写"**你**在其攻击范围内"这类条件时用它，读起来正好和描述一致。

**`canCompare`（拼点前置校验）**：

```js
if (player.canCompare(target)) {
	// 双方便都可拼点：非自己、双方都有手牌、都没被"禁止拼点"
}
```

⭐ **它一个方法就顶三个判断** —— 第 13 章讲拼点时会重点用。写拼点技能的 `filterTarget` 时**直接写 `player.canCompare(target)` 就行**，别再叠"不是自己""有手牌"。

**想判断他手上有没有某张牌能用**，用 `hasUsableCard(name, type)`：

```js
if (player.hasUsableCard("sha")) { }              // 他手上有没有能当【杀】用的牌
if (player.hasUsableCard("shan", "respond")) { }  // 他能不能打出【闪】
```

⭐ **第一参数是牌名**，不是"牌"本身 —— 判据是 `get.name(那张牌, 他)`，所以"在他手里会被视为【杀】"的牌也算数。

⭐ **第二参数管场合**：不传 ⇒ 按"能不能用"算（还要过使用次数那道门）；传 `false` ⇒ 不管次数，只要是这张牌就算；传 `"use"` / `"respond"` ⇒ 分别问"能用" / "能打出"。

### 7.2 全场最多 / 最少

这一族是"问一个相对位置"，**判"手牌最多""体力最少"时用**：

| 方法 | 问什么 |
| --- | --- |
| `isMaxHandcard(only, filter)` | 我的**手牌数**是不是全场最多 |
| `isMinHandcard(only, filter)` | …**最少** |
| `isMaxHp(only, raw, filter)` | 我的**体力**是不是全场最高 |
| `isMinHp(only, raw, filter)` | …**最低** |

```js
if (player.isMaxHandcard()) { }        // 并列最多也算
if (player.isMaxHandcard(true)) { }    // 第 1 参传 true = 必须"唯一"最多
```

⭐ **默认"并列也算"** —— 传 `true` 才是"唯一最…"。很多人在这里搞错（以为默认是唯一）。

### 7.3 常用

| 方法 | 作用 |
| --- | --- |
| `hasEmptySlot(type)` | 有没有空置的装备栏 |
| `getEquips(subtype)` | 装备区里的牌（`1` = 武器） |
| `getKnownCards(other, filter)` | **`other` 所知**的、我的手牌（AI 用） |
| `isAllCardsKnown(other)` | 我的手牌他是不是全知道 |

**最后两个是"为 AI 服务"的** —— 写 AI 时想知道"对手知道我的什么牌"，用它们。第 13 章讲拼点 AI 时会用上。

---

## 八、⑧演出与提示

这一组**不改变游戏状态，只影响玩家看到什么**。写得好，技能会"有质感"。

### 8.1 必背

| 方法 | 做什么 |
| --- | --- |
| `logSkill(name, targets)` | **播报技能发动**（打日志 + 播语音 + 放技能动画） |
| `line(target, config)` | 角色之间**划线** |
| `popup(name, className)` | 头顶**弹一个词** |
| `say(str)` | 让角色**说话** |
| `chat(str)` | 往聊天区**发一句**（AI 用得多） |
| `addTip(index, message, isTemp, css, nobroadcast)` | 界面左上角**加一条提示** |

> 💡 `addTip` 后面还有三个可选参数：`isTemp`（`true` = 回合结束自动消失；也可以填一个自定义时机名）、`css`（自定义样式）、`nobroadcast`（传 `true` 就**只在本机显示**、不广播给别人 —— 名字和 `addSkill` 的第 3 个参数撞车，含义也一样）。
>
> ⚠️ `nobroadcast` 是 **`1.11.5.1` 才加的**。在那之前 `addTip` 一律无条件全场广播，没有"只给自己看"这一档。

### 8.2 ⭐ `logSkill` —— 大多数时候别手写

⚠️ **这是最容易"画蛇添足"的地方**：

> **技能正常发动时，引擎会自动调用 `logSkill`。你再手写一次，就打了**两遍**日志。**

**什么时候才需要自己写？** 只有两种：

1. **子技被别的技能"点名"发动时**（第 23 章讲全局技时会遇到）
2. **你想给日志指定一个不同的目标**（比如"看起来像是他发动的，其实是另一个人"）

```js
// 正常情况：什么都不用写，引擎会记
async content(event, trigger, player) {
	await player.draw();
},
```

```js
// 特殊需求：以另一个名字记日志
player.logSkill("nzry_lijun");
```

⭐ **判据**：**你的技能是"正常触发/正常点按钮发动"的 → 不写 `logSkill`。** 只有"绕过了正常发动流程"才自己补。

### 8.3 划线、弹出与说话

```js
player.line(source, "green");        // 从他line到 source，绿线
player.popup("tianxiang");           // 头顶弹出"天香"（会去查翻译）
player.say("还钱！");                 // 让他说一句
target.chat(`我选${get.translation(event.choice)}`);   // 聊天区发言
```

**`line` 的第二个参数是颜色** —— 常用 `"green"`（友方/正面）、`"red"`（敌方/负面）、`"yellow"`。

⭐ **`popup` 会自动翻译** —— 传技能 id 或牌名，它会去 `lib.translate` 查中文：

```js
player.popup("tianxiang");    // 头顶显示"天香"
player.popup("sha");          // 头顶显示"杀"
```

**`popup` 的第二个参数是样式类**，不写就是 `"water"`（水色）。

### 8.4 进阶：`player.when(...)`

`player.when(时机名)` 是「**给自己挂一个一次性的时机监听**」—— 写"过一会儿再做一件事"时特别省事。它内部就是**现场造一个临时技能**（名字随机，形如 `player_when_xxxxxxxx`）挂到你身上，**触发一次就自动摘掉**。

**它能收几种「时机名」**（和技能 `trigger` 一样）：

| 你写 | 等于 |
| --- | --- |
| `when("phaseJieshuBegin")` | `{ player: "phaseJieshuBegin" }` |
| `when(["phaseJieshuBegin", "phaseZhunbeiBegin"])` | `{ player: [...] }` |
| `when({ global: "phaseJieshuBegin" })` | 原样（旁听别人的回合也行） |

**基本形态**：

```js
// 到"本回合结束阶段开始"时执行一次，然后自动摘掉
player.when({ player: "phaseJieshuBegin" }).step(async (event, trigger, player) => {
	player.unmarkSkill("rehuomo");        // 官方〖祸末〗（clan/skill.js）原样
});
```

⭐ **链式上能挂的几个方法**：

| 方法 | 干什么 |
| --- | --- |
| `.step(fn)` | 挂"到点时执行的内容"，**支持闭包**（能直接用外面的变量）—— 首选 |
| `.filter(fn)` | 加条件：返回假 ⇒ 这次不执行；**而且机会不会被消耗** —— 下个时机会再判一次 |
| `.then(fn)` | 早期写法，也能挂内容；但普通函数会被转成字符串再编译，**闭包里的变量会丢** —— 想用闭包就写 `.step` |
| `.popup(str)` / `.translation(str)` | 给这个临时技能一个弹字／显示名 |
| `.assign(obj)` | 往临时技能上补别的字段（如 `mark`、`intro`） |
| `.finish()` | 配合第 2 个参数 `false` 用（见例④） |

**多给几个例子**：

```js
// ① 带条件：只有手牌多于 3 张才补牌；不满足就留着，下次时机再判
player.when("phaseJieshuBegin")
	.filter((event, player) => player.countCards("h") > 3)
	.step(async (event, trigger, player) => {
		await player.draw();
	});
```

```js
// ② 一次挂两个时机：哪个先到就在哪执行，且只执行一次
player.when(["phaseJieshuBegin", "phaseDrawBegin1"]).step(async (event, trigger, player) => {
	await player.draw();
});
```

```js
// ③ 给它起个名字（弹字／显示名），玩家能看懂这是啥
player.when("phaseJieshuBegin")
	.translation("余韵")
	.popup("余韵")
	.step(async (event, trigger, player) => {
		await player.recover();
	});
```

```js
// ④ 先造出来、晚点再决定挂不挂：第 2 个参数 instantlyAdd = false，最后 .finish() 才生效
const listener = player.when("phaseJieshuBegin", false);
if (get.attitude(player, target) > 0) {
	listener.step(async (event, trigger, player) => {
		await target.draw();
	});
	listener.finish();      // 现在才真正挂上去
}
```

⚠️ **`.filter()` 不满足，不等于作废**：这个临时技能只在**真的发动**那一刻才被标记"已用掉"（`src/noname/library/element/content.ts` 里那句 `lib.skill[event.skill].triggered = true`）；条件不满足时它还在身上，下个时机会重新判。这正是①②能"挂了不一定响"的道理。

**它和你写 `subSkill` 的区别**：

| | `subSkill` | `player.when` |
| --- | --- | --- |
| 写在哪 | 技能对象里（静态） | `content` 里（动态挂） |
| 活多久 | 技能在就一直在 | **触发一次就摘掉** |
| 条件 | `filter` 常驻 | `.filter()` 临时加 |

⭐ **"只此一次"的后续效果用 `player.when` 最省事** —— 不用另开子技，也不用自己管"用完删掉"。官方几十个技能在用（`src/character/` 里搜 `.when(`，如〖祸末〗`rehuomo`、〖通博〗`tongbo`）。

> ⚠️ **但它不能永久存在** —— 它是一次性的：**触发一次，引擎就把这个临时技删掉**（`src/noname/library/element/content.ts` 里那句注释「删除 when 生成的临时技能」＝ `player.removeSkill(event.skill)`，并顺带清掉 `lib.skill` / `lib.translate`）。

> ⚠️ **别把这条理解反了**：**它并不是说过完这个回合就没了** —— 在**触发之前**，这个监听一直挂在你身上，**可以跨好几个回合**（你挂 `roundStart`，就得等到下一轮的轮开始才响，这一整轮里它都待着）。所以"一次性"说的是**它只响一次**，不是说它寿命只有一回合。

> 💡 真想要**长期持续生效**（响很多次、或一直挂着），才得改用 `addTempSkill` / `addSkill` ＋ 子技（第 9 章那套）。

---

### 8.5 想查某个方法时去哪找

`player` 的全部方法都实现在 **`src/noname/library/element/player.js`** —— 这个文件里**每个方法上面通常有一行注释说明参数**，是查签名最可靠的地方（比任何教程都准）。

另外 `src/noname/library/element/Player/type.d.ts` 里有**参数对象的类型定义** —— 想知道 `damage()` / `gain()` 这些"收对象的方法"到底能写哪些字段时，看它最快。

---

## 九、串起来：一个技能里用了多少个 `player` 方法

把第 13 章要写的〖论战〗先放一半在这儿，数一数它用了哪些：

```js
tutorial_lunzhan: {
	enable: "phaseUse",
	usable: 1,
	filter(event, player) {
		return game.hasPlayer(current => player.canCompare(current));
	},
	filterTarget(card, player, target) {
		return player.canCompare(target);
	},
	async content(event, trigger, player) {
		const target = event.targets[0];
		const result = await player.chooseToCompare(target).forResult();
		if (result.winner) {
			await result.winner.draw();
		}
	},
},
```

| 这行 | 用到谁 | 在做什么 |
| --- | --- | --- |
| `game.hasPlayer(...)` | `game` | 全场找"有没有能拼点的人" |
| `player.canCompare(current)` | **`player`** | 问他"能不能跟他拼点" |
| `event.targets[0]` | 事件 | 取选中的目标 |
| `player.chooseToCompare(target)` | **`player`** | 发起拼点 |
| `result.winner` | 结果 | **赢家是谁**（平局时没有） |
| `result.winner.draw()` | **`player`** | 让赢家摸一张牌 |

⭐ **注意 `result.winner` 这个字段的价值**：

> 描述写的是"<strong>赢的角色</strong>" —— 它可能是你、也可能是对手。<strong>`winner` 直接告诉你答案，不用自己比点数。</strong>

这就是"读懂 result 的形状"能省下的功夫。**第 13 章会把拼点的 result 讲全。**

---

## 动手改一改

### 练习 1

下面每句话该用哪个方法？

1. 我的**手牌数**。
2. 我**已经失去几点体力**。
3. 我**本回合用过几张【杀】**。
4. 我**能不能和对方拼点**。
5. 目标**装备区里有没有牌**。
6. 我**手牌数是不是全场最多**（并列不算）。

> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

### 练习 2

判断对错，并说明理由。

1. `player.sex === "male"` 和 `player.hasSex("male")` 等价。
2. `player.getHandcardLimit()` 可以直接用 `player.hp` 代替。
3. `player.getStorage("tutorial_chaji")` 没存过时返回 `0`。
4. `player.getHistory("damage")` 读的是"我造成的伤害"。
5. `player.damage()` 不写来源没关系，引擎会自己判断。
6. `player.awakenSkill("xxx")` 会让角色"觉醒"（获得新技能）。

> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

### 练习 3

下面这段代码想让"目标弃置一张牌"，但目标可能一张牌都没有。加一句守卫。

```js
async content(event, trigger, player) {
	const target = event.targets[0];
	await target.chooseToDiscard({ position: "he", forced: true });
},
```

> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

### 练习 4

下面这个技能想判"**这回合我第一次受伤前，有没有用过【杀】**"，但写错了。找出来并改正：

```js
filter(event, player) {
	return player.getHistory("sourceDamage", evt => evt.card && get.name(evt.card) === "sha");
},
```

> 💡 提示：想想 `"damage"` 和 `"sourceDamage"` 的区别。  
> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

### 练习 5

下面这个技能的 `filter` 想让"只有手牌超过手牌上限时才发动"，但恒不成立。为什么？

```js
filter(event, player) {
	return player.countCards("h") > player.getStorage("tutorial_ceshi");
},
```

> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

### 练习 6

给下面这段代码补上正确的"伤害来源"，使它读作"**你对目标造成 2 点火焰伤害**"。

```js
async content(event, trigger, player) {
	await event.targets[0].damage(2);
},
```

> 💡 参考答案在本文末尾 —— 建议先自己做完，再去对答案。

---

## 本章小结

一句话概括这一章：

> **`player` 是技能代码里的主角 —— 它身上有三百多个方法，但按"你想做什么"分 8 组，每组必背的就那么几个。**

**8 组的分工：**

| # | 组 | 必背的那几个 |
| --- | --- | --- |
| 1 | **牌** | `countCards` / `getCards` / `hasCard` / `hasCards` / `gain` / `discard` / `modedDiscard` / `give` / `discardPlayerCard` / `gainPlayerCard` / `canUse` / `hasUseTarget` |
| 2 | **体力** | `draw` / `drawTo` / `recover` / `recoverTo` / `loseHp` / `damage` / `gainMaxHp` / `loseMaxHp` / `isDamaged` / `getDamagedHp` / `needsToDiscard` / `isIn` |
| 3 | **技能** | `hasSkill` / `addSkill` / `removeSkill` / `addTempSkill` / `awakenSkill` / `hasSkillTag` / `addAdditionalSkill` / `tempBanSkill` |
| 4 | **标记与存储** | `addMark` / `countMark` / `removeMark` / `getStorage` / `setStorage` / `markAuto` / `addToExpansion` / `getExpansions` |
| 5 | **询问** | 见第 7、13 章 |
| 6 | **历史与统计** | `getHistory` / `hasHistory` / `getStat` / `countUsed` / `isPhaseUsing` |
| 7 | **判定·距离·目标** | `judge` / `inRange` / `distanceTo` / `canCompare` / `getHandcardLimit` / `isMaxHandcard` / `isMinHp` |
| 8 | **演出与提示** | `logSkill`（少写）／`line` / `popup` / `say` / `player.when` |

**七条最容易踩的：**

| # | 结论 |
| --- | --- |
| 1 | ⭐ <strong>凡是"可能被技能修改"的东西都用方法问</strong> —— `hasSex` 而不是 `sex`、`getHandcardLimit()` 而不是 `hp`、`inRange` 而不是自己比距离。 |
| 2 | ⭐ <strong>`hasCard(filter, position)` 是"判据在前"，`hasCards(position, filter)` 是"位置在前"</strong> —— 两个顺序相反，记牢。 |
| 3 | ⭐ <strong>`damage()` 不写 `source` 时，引擎会把"当前事件当事人"当来源</strong> ⇒ 自伤是常态。<strong>要"失去体力"就用 `loseHp`，别用 `damage` 代替。</strong> |
| 4 | ⭐ <strong>`getStorage(name, 0)` —— 缺省值不写是空数组 `[]`</strong>，跟数字比永远不成立。 |
| 5 | ⭐ <strong>`getHistory("damage")` 是"我挨的打"</strong>，"我打的"要写 `"sourceDamage"`。 |
| 6 | ⭐ <strong>`awakenSkill` 是"标记限定技已用"，不是"让他觉醒"</strong> —— 限定/觉醒技的 content 里必须有它。 |
| 7 | ⭐ <strong>正常发动的技能别手写 `logSkill`</strong> —— 引擎会自动记，手写会打两遍。 |

**"该用哪个"速查：**

| 想知道 | 用 |
| --- | --- |
| 手牌数 / 装备数 / 判定区牌数 | `countCards("h"/"e"/"j")` |
| 手牌上限 | `getHandcardLimit()` |
| 超没超手牌上限 | `needsToDiscard()` |
| 已失去几点体力 | `getDamagedHp()` |
| 是不是全场最多 | `isMaxHandcard()` / `isMaxHp()` |
| 够不够得着 | `inRange(target)` |
| 能不能拼点 | `canCompare(target)` |
| 本回合用过几张某牌 | `countUsed("sha")` |
| 这个技能发动过几次 | `getHistory("useSkill", …).length + 1` |
| 记一个数 | `addMark` / `getStorage` / `markAuto` |
| 记一批牌 | `addToExpansion` |

**这一章也没有新增武将技能** —— 它和第 11 章合起来，是进阶篇的"工具箱"部分。

工具箱讲完了。**下一章开始动真格的**：

> **第 7 章你学了四张最常用的选择框。但官方库里有 30 种 `chooseXxx`** —— 拼点、按钮框、选别人的牌、拖动移动牌、多人同时选……
>
> **这些"另一半"里，藏着写复杂技能的钥匙。**
>
> 尤其是**拼点** —— 全库用了 100 多次，是仅次于"选牌/选目标"的第三大交互。而它有一个**非常反直觉的坑**：双方用的是**同一个 AI 函数**。

下一章全讲这个。

👉 [第 13 章 · 选择框的另一半](13-选择框的另一半.md)

---

## 参考答案（先自己做完再看）

### 练习 1

1. <strong>`player.countCards("h")`</strong>
2. <strong>`player.getDamagedHp()`</strong> —— 注意不是 `player.maxHp - player.hp` 那种算法，虽然结果一样；用方法更稳（体力上限也可能被技能改）。
3. <strong>`player.countUsed("sha")`</strong> —— 它读的是"本回合真实使用"的次数（打出不计）。
4. <strong>`player.canCompare(target)`</strong> —— 一个方法顶三个判断：不是自己、双方都有手牌、都没被禁止拼点。
5. <strong>`target.countCards("e")`</strong>（或者 `target.hasCards("e")` 判有无）—— ⚠️ 注意 `countCards` 是"位置在前"。
6. <strong>`player.isMaxHandcard(true)`</strong> —— 第 1 参传 `true` 才是"唯一最多"；不传的话并列也算。

### 练习 2

1. ❌ <strong>错。</strong> `player.sex` 是"原始值"，而 `hasSex` 会<strong>算上技能的修正</strong>（有些技能会改性别）。<strong>规则：可能被改的东西都用方法问。</strong>
2. ❌ <strong>错。</strong> `player.hp` 只是体力值。手牌上限会被〖筑台〗这类 `mod` 技能修改，<strong>必须用 `getHandcardLimit()`</strong> 才能拿到修正后的结果。
3. ❌ <strong>错。</strong> <strong>缺省值不写时返回空数组 `[]`</strong>，不是 `0`。要数字就得显式写 `player.getStorage("tutorial_chaji", 0)` —— 不写的话，跟数字比较会恒不成立。
4. ❌ <strong>错。</strong> <strong>反了。</strong> `"damage"` 是"<strong>我受到</strong>的伤害"，`"sourceDamage"` 才是"<strong>我造成</strong>的"。这是最经典的看反。
5. ❌ <strong>错。</strong> 不写来源时，引擎会把<strong>当前事件的当事人</strong>当成来源。如果你正在跑的事件主角就是 `player` 自己，那这次伤害的来源就成了他自己 —— <strong>自伤是常态</strong>。要明确写 `source`，或用 `"nosource"` 表示"没有来源"。
6. ❌ <strong>错。</strong> `awakenSkill` 的作用是"<strong>把这个限定技/觉醒技标记为已发动，并从技能列表里摘掉</strong>"。它不会给角色新技能。想让角色获得新技能要用 `addSkill` / `addTempSkill`（官方〖魂姿〗就是"`awakenSkill` 掉旧的 + `addSkill` 加新的"）。

### 练习 3

加一句"他有没有可弃的牌"的守卫：

```js
async content(event, trigger, player) {
	const target = event.targets[0];
	if (target.countDiscardableCards(player, "he")) {
		await target.chooseToDiscard({ position: "he", forced: true });
	}
},
```

**为什么用 `countDiscardableCards` 而不是 `countCards("he")`？** 因为前者会过一遍"这张牌能不能被弃置"的技能修正（有些技能让牌"不可弃置"）。**判"能不能让他弃"要用前者。**

> 💡 另一个更省事的写法：把 `chooseToDiscard` 的 `forced` 传 `false`（可取消），让他自己决定。但那会让描述里的"弃置"变成"可以弃置" —— **描述和代码必须一致**（第 10 章的规矩）。所以还是写守卫。

### 练习 4

错在<strong>用 `sourceDamage` 去读"我受到的伤害"</strong>。

先理清需求：要判的是"**这回合第一次受伤前**有没有用过【杀】" —— 那要查的是"我**使用过**的牌"，不是伤害记录：

```js
filter(event, player) {
	return player.hasHistory("useCard", evt => get.name(evt.card) === "sha");
},
```

**两个要点：**

- **`useCard` 才是"使用过的牌"** —— `damage` 系是"伤害记录"，跟"用过什么牌"是两回事，别拿它当替身。
- **判牌名用 `get.name(evt.card)`，别用 `evt.card.name`** —— 牌可能被转化（第 11 章讲过）。而且 `evt.card` 有时是虚拟牌，直接用 `get.name` 才稳。

<strong>如果确实要判"造成过【杀】的伤害"</strong>（那是另一个需求），才轮到 `sourceDamage` —— 且<strong>必须判 `evt.num > 0`</strong>（0 点伤害或被防止的不算）：

```js
player.getHistory("sourceDamage", evt => evt.num > 0 && evt.card && get.name(evt.card) === "sha");
```

### 练习 5

因为 **`getStorage` 的缺省值是空数组**。

```js
player.countCards("h") > player.getStorage("tutorial_ceshi")
//                     └─ 没存过时是 [] ─┘
```

`数字 > []` 这种比较在 JS 里会走"把数组转成字符串再比"的诡异路径，结果是<strong>恒为 `false`</strong>（或恒为 `true`，取决于具体数字）—— 总之<strong>不是你想要的那个意思</strong>，而且<strong>不报错</strong>。

改正 —— **显式给缺省值**：

```js
filter(event, player) {
	return player.countCards("h") > player.getStorage("tutorial_ceshi", 0);
},
```

> 💡 **更好的写法**：如果只是想判"手牌超过手牌上限"，直接用引擎现成的：
>
> ```js
> return player.needsToDiscard() > 0;
> ```
>
> 它还额外考虑了"某些牌不计入手牌上限"。

### 练习 6

补上 `source` 和属性：

```js
async content(event, trigger, player) {
	await event.targets[0].damage({ num: 2, source: player, nature: "fire" });
},
```

**三个要点：**

- **`source: player`** —— 明确"是你打的"。不写的话来源会变成"当前事件当事人"，很容易变成自伤。
- **`nature: "fire"`** —— 属性伤害要写属性。不写就是普通伤害。（雷用 `"thunder"`。）
- **`num: 2`** —— 伤害值。

> 💡 **简写形式**：`damage` 也支持"位置参数"，`target.damage(2, player)` 等于 `{ num: 2, source: player }`。
>
> **但属性没地方放**，所以涉及属性伤害时**一律用对象写法**。
