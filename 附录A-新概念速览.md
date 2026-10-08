# 附录 A 新概念速览

## 这一份要做什么

正文 31 章讲完了。但你可能在别的地方（官方武将、别人的扩展）见过一些**没在正文里出现过的名词**：

> 「背水」「议事」「仁库」「蓄谋」「影」「整肃」「协力」「强令」……

这些名词有的是**真机制**（引擎内置了一套 API），有的只是**官方的一种写法约定**（引擎什么都不管，全靠自己写）。混在一起看，很容易以为"这一定是引擎的功能，只是我没找到"。

这一份附录把它们一次理清，分成五类：

| 类 | 引擎管不管 | 有哪些 | 在哪节 |
| --- | --- | --- | --- |
| **一、名词词典** | 管（有个现成注册表） | `lib.poptip` 里那一整套 | 一 |
| **二、内置 API** | 管（调 API 就行） | 议事 / 协力 / 谋弈 / 发现 / 护甲 / 智囊 / 仁库 | 二 |
| **三、写法约定** | **不管**（靠自己写） | 背水 / 整肃 / 强令 / 摧坚 / 妄行 / 施法 … | 三 |
| **四、卡牌机制** | 管（藏在卡牌字段里） | 蓄谋 / 【影】 | 四 |
| **五、扩展能做什么** | 管（开放给你用） | 注册自己的名词、注册自己的公共区域 | 五 |

⚠️ 这一份**不是教程**，是**索引**。每个名词只给"是什么 ＋ 代码落点"，需要细看时再回到对应章节或源码。

---

## 一、名词词典：`lib.poptip`

### 1.1 官方把冷门名词收在一处

游戏里有些词是"专有名词"，玩家看了不知道什么意思。官方做了一个统一的地方存它们的解释 —— 你写在**技能描述**里的这些词，玩家可以**点一下弹出解释框**。

这个注册表就是 `lib.poptip`，位置在 `src/noname/library/poptip.js`。它内置了下面这一批（这些就是官方认为"需要解释"的全部名词）：

| id | 名字 | 一句话 |
| --- | --- | --- |
| `rule_hujia` | 护甲 | 和体力类似，每点抵挡 1 点伤害，但不影响手牌上限 |
| `rule_suicong` | 随从 | 独立技能、手牌区、装备区，共享判定区；出场时替代主武将位置 |
| `rule_faxian` | 发现 | 从三张随机亮出的牌中选一张，无特殊说明则获得它 |
| `rule_xunengji` | 蓄能技 | 发动时可增大"黄色数字"，红色数字在该次结算中变为两倍 |
| `rule_zhinang` | 智囊 | 默认【过河拆桥】【无懈可击】【无中生有】【洞烛先机】 |
| `rule_renku` | 仁库 | 游戏外的公共区域，至多六张；溢出时把最早的牌置入弃牌堆 |
| `rule_chihengji` | 持恒技 | 拥有此标签的技能不会被其他技能无效 |
| `rule_chengshi` | 乘势 | 多分支效果中，其他选项的条件都满足时触发的附加效果 |
| `rule_beishui` | 背水 | 一种特殊选项：无法执行后果就不能选；选了则其余选项依次执行 |
| `rule_zhengsu` | 整肃 | 从擂进／变阵／鸣止中选一项令目标执行，未失败则得奖励 |
| `rule_xieli` | 协力 | 从同仇／并进／疏财／勠力中选一项，双方都完成则执行奖励 |
| `rule_rumo` | 入魔 | 每局限一次；入魔后每轮结束若未造成伤害则失去 1 体力 |
| `rule_bianshenji` | 变身技 | 满足条件得指示物，满上限可进入变身状态；归零则退出 |
| `rule_bianshen` | 变身 | 进入时弃置判定区所有牌；替换武将牌，两张牌血量单独计算 |
| `rule_shifa` | 施法 | 声明 X（1~3），之后第 X 个回合结束时执行效果 |
| `rule_gongtongpindian` | 共同拼点 | 多人同时亮牌，点数唯一最大者赢；无唯一最大则全员没赢 |
| `rule_qiangling` | 强令 | 发动时机固定为"出牌阶段开始时"；向一名其他角色发布任务 |
| `rule_cuijian` | 摧坚 | 发动时机固定为"使用伤害牌指定第一个目标后"；X 等于目标的非 `charlotte` 技能数 |
| `rule_wangxing` | 妄行 | 声明 X（1~4）；回合结束时弃 X 张牌或减 1 点体力上限 |
| `rule_boji` | 搏击 | 互相都在对方攻击范围内时，额外执行搏击后的效果 |
| `rule_youji` | 游击 | 目标在你攻击范围内、你不在其攻击范围内时，可执行额外效果 |
| `rule_jiang` | 激昂 | 昂扬技发动后失效，满足条件后恢复 |
| `rule_lizhan` | 历战 | 回合结束时，按本回合发动次数对效果做永久可叠加升级 |
| `rule_tongxin` | 同心 | 可与其他角色"同心"；技能发动后同心角色依次执行效果 |
| `sxrm_connect` | 连接 | 一种对手牌的动作：被连接的手牌对所有人可见；失去时全员依次弃置 |
| `sxrm_compare` | 延时拼点 | 拼点后不立刻公布，改为扣置移出游戏；满足条件才公开 |
| `sxrm_qidingSkill` | 契定技 | 发动一次后标签改为"锁定技"，并删掉描述里的"可以" |
| `rule_mamba` | 牢大 | 彩蛋 |

⭐ **这张表本身就是一份教材**：官方怎么定义一个新机制、解释怎么写（一句话讲清"是可选的还是强制的、边界在哪"），都在里面。

### 1.2 三个公开用法

`lib.poptip` 有三种用法，都很好用。

**用法一：在描述里嵌一个可点开的名词 —— `get.poptip(id)`**

它返回一段特殊 HTML，游戏渲染时会变成"可点击的名词"。注意：**它只能用在"能被当成 HTML 渲染"的地方** —— 技能描述的部分位置支持，`_info` 的描述正文里不一定支持（第 10 章讲过颜色标记的类似限制）。

```js
// 描述里出现「背水」这个词时，玩家可以点它看解释
tutorial_skill_info: `出牌阶段限一次，你可以${get.poptip("rule_beishui")}，然后摸两张牌。`,
```

⚠️ 这类写法要用**模板字符串**（反引号），因为里面插了一个函数调用。同时它意味着这条描述**不能写成死字符串**。官方在需要嵌这个的技能上，会把 `_info` 改成 **getter（取值函数）**：

```js
// 官方【五禽戏】的做法：把 _info 写成一个"每次读时才算"的属性
get wuling_wuqinxi_info() {
	return lib.poptip.getInfo("wl_wuqinxi");
},
```

**用法二：在代码里取名词的解释 —— `lib.poptip.getInfo(id)` / `getName(id)`**

`dynamicTranslate` 里想拼一句"名词＋解释"时用它。

```js
dynamicTranslate(player, skill) {
	if (!player) {
		return lib.translate[skill + "_info"];
	}
	return `${get.translation(skill)}：${lib.poptip.getInfo("rule_beishui")}`;
},
```

**用法三：注册你自己的名词 —— `lib.poptip.add({...})`**

这就是第 4 节的伏笔。见 4.1。

### 1.3 一个隐藏的好处

`lib.poptip.init()` 会把词典里的每一项<strong>同时写进 `lib.translate`</strong>：

```js
// 简化后的逻辑，示意用
_poptipMap.forEach((value, key) => {
	lib.translate[key] = value.name;
	lib.translate[key + "_info"] = value.info;
});
```

⇒ 于是 **`lib.translate["rule_beishui"]` 是"背水"、`lib.translate["rule_beishui_info"]` 是那段解释** —— 你可以像读普通译键一样读它们，也可以直接 `get.translation("rule_beishui")`。

---

## 二、内置 API：调一下就有

这一类的共同点：**引擎写好了一整套流程，你只要调一次函数**。

### 2.1 议事：`player.chooseToDebate(...)`

**是什么**：所有参与角色**同时**暗选一张手牌亮出，按**颜色**归类，看哪种颜色的人最多，得出一个"意见"。

**怎么调**：

```js
// 签名：player.chooseToDebate({ list: [...], args: [...] })
await player.chooseToDebate({ list: [player, ...targets], args: [] }).set("callback", async (event, trigger, player) => {
	const { debateResult: result } = event;
});
```

**回调里拿到什么**（`event.debateResult`）：

| 字段 | 含义 |
| --- | --- |
| `bool` | 这次议事有没有正常跑完 |
| `opinion` | **胜出的意见**：`"red"` / `"black"` / 某个颜色 / `null`（无结果） |
| `opinions` | 本次收集到的所有意见种类 |
| `targets` | 实际参与的角色 |
| `red` / `black` / `others` | 三种归类，每项是 `[角色, 牌]` 的数组 |

**官方范例**（读法：先看颜色，再分派效果）：

```js
async content(event, trigger, player) {
	const { targets } = event;
	await player.chooseToDebate([player].concat(targets.sortBySeat())).set("callback", async (event, trigger, player) => {
		const { debateResult: result } = event;
		const { opinion } = result;
		if (opinion == "red") {
			const cards = result.red.flatMap(i => i[1]).filter(card => get.itemtype(card) == "card");
			if (cards.length) {
				await player.gain(cards, "give");
			}
		} else if (opinion == "black") {
			const drawer = result.red.map(i => i[0]).unique().sortBySeat();
			await game.asyncDraw([player].concat(drawer));
		}
	});
},
```

**几个可调项**（都 `.set()` 上去）：

| 键 | 作用 |
| --- | --- |
| `fixedResult` | 预设结果（不参与抽牌，直接算进归类）—— 用来实现"某人已经决定好了" |
| `ai` / `aiCard` | 选牌倾向（不传就**随机选**） |
| `debateIgnore` | 非红非黑的牌算不算一种"意见"（默认算作 `others`） |
| `callback` | 结果出来后的回调，里面读 `debateResult` |

⭐ <strong>它自己会派发时机</strong>：`chooseToDebateBegin` / `chooseToDebateAfter`（给"全场"用），以及一个特别的 <strong>`debateShowOpinion`</strong>（在"归类完成、还没算胜负"的那一刻）—— 想做"改判别人的意见"这类技能，就挂这个。

⭐ 胜负规则：**取得票数"严格最多"的那种颜色**；`red` 和 `black` 打平就**没有结果**（`opinion` 为 `null`）。所以用它的技能必须处理 `opinion` 为 `null` 的分支。

### 2.2 协力：`cooperationWith` / `chooseCooperationFor` / `checkCooperationStatus`

**是什么**：和一名角色"结对"，双方各自去完成同一项任务，都完成才算成功。

**四个方法**（都在 `player` 上）：

| 方法 | 作用 |
| --- | --- |
| `chooseCooperationFor(target, "技能id")` | 让场上一位角色**选择**与谁协力、以及协力什么 |
| `cooperationWith(target, 名)` | 直接建立协力关系 |
| `checkCooperationStatus(target, "技能id")` | 查验"这项协力完成了没有" |
| `removeCooperation(target)` | 解除协力关系 |

**四选一的内容**（官方固定文案，来自词典 `rule_xieli`）：同仇（伤害值之和 ≥ 4）／并进（总计摸牌 ≥ 8）／疏财（弃置的牌含 4 种花色）／勠力（使用或打出的牌含 4 种花色）。

⚠️ 与议事不同：**协力没有"一键 API"** —— 上面四个方法只管"关系"和"查验"，**具体怎么算完成度要自己在规则技里写**（正文里讲过的"每阶段限一次""本回合内统计"那套本事就该上场了）。

### 2.3 谋弈：`player.chooseToDuiben(target)`

**是什么**：一对一的"猜拳"式博弈 —— 双方各选一项，按固定规则判胜负。和议事是两套东西（议事是多人按颜色分类，谋弈是两人对猜）。

### 2.4 发现：`player.discoverCard(list, ...)`

**是什么**：从**三张随机亮出的牌**里选一张。

⚠️ 注意名字：<strong>不是 `chooseToDiscover`，是 `discoverCard`</strong>（很容易记错）。

```js
const result = await player.discoverCard();          // 从整副牌堆里随机亮三张
const result2 = await player.discoverCard(list, { num: 3 });  // 从 list 里亮三张
```

**对象参数**（跟在 `list` 后面 —— 只有 `list` 还是位置参数）：

| 字段 | 作用 |
| --- | --- |
| `prompt` | 提示语 |
| `use` | 传 `true` 则**直接使用**选中的牌 |
| `nogain` | 传 `true` 则**不获得** |
| `num` | 亮出的张数 |
| `forced` | 是否强制（默认就是强制） |
| `ai` | AI 选牌倾向 |

### 2.5 护甲：`target.changeHujia(num, ...)`

**是什么**：一种"和体力类似的护盾"，每点抵挡 1 点伤害，但**不影响手牌上限**。

```js
await target.changeHujia(1);   // 加 1 点护甲
await target.changeHujia(-1);  // 减 1 点
```

### 2.6 智囊：`get.zhinangs()`

**是什么**：一份"特殊锦囊"名单，默认是【过河拆桥】【无懈可击】【无中生有】【洞烛先机】。**牌堆里没有的会被过滤**，玩家也可以在游戏设置里增减。

```js
if (get.zhinangs().includes(card.name)) {
	// 这是一张智囊牌
}
```

### 2.7 仁库：`_status.renku`

**是什么**：一个**游戏外的公共区域**，至多放 6 张牌，超出时最早的牌进弃牌堆。

**它不是某人的牌区，是个全局变量**：

```js
_status.renku                       // 数组，当前仁库里的牌
_status.renku.length                // 有几张
```

**怎么把牌放进去 / 拿出来** —— 走"区域名"，不是自己 `push`：

| 动作 | 写法 |
| --- | --- |
| 把牌置入仁库 | 用 `toRenku` 这个区域名（走"移动到公共区域"的那套流程） |
| 从仁库取牌 | 用 `fromRenku` 作为来源区域 |

⭐ 为什么不能自己 `_status.renku.push(card)`？因为**溢出处理和界面刷新都在引擎的钩子里**（放进去要判溢出、要打日志、要更新显示）。自己推数组会绕过这些，表现为"牌进去了但界面不对"。

⭐ **它背后是一个更通用的机制** —— 见 4.2。学会那个，你就能给扩展加一个**自己的公共区域**。

---

## 三、写法约定：引擎不管，全靠自己写

这一类的共同点：**你在引擎源码里搜不到它的专属 API** —— 因为它是**官方约好的一种写法**。

这类名词最容易让人误解成"引擎功能"。判断方法很简单：

> **在引擎源码里搜这个机制名，如果只有词典（`poptip.js`）里有，那一准是写法约定。**

### 3.1 背水：一种"全选"选项

<strong>是什么</strong>：给一个多选项技能加一个特殊选项 —— <strong>选了它，就把其余每个选项都执行一遍，再额外执行它的"后果"</strong>。而且<strong>后果做不到时，这个选项不该出现在列表里</strong>。

**引擎做了什么**：**什么都没做**。它就是普通的 `chooseControl` ＋ 一个约定好的选项名 `"背水！"`。

**官方范式**（这是本节最该记住的东西）：

```js
async cost(event, trigger, player) {
	const list = ["选项一", "选项二"];
	if (player.getEquip(1) || trigger.target.getEquip(1)) {
		list.push("背水！");
	}
	list.push("cancel2");
	const result = await player
		.chooseControl(list)
		.set("choiceList", ["选项一的效果", "选项二的效果", "背水！背水的后果，并执行所有选项"])
		.set("prompt", get.prompt(event.skill))
		.set("ai", function () {
			return "选项一";
		})
		.forResult();
	event.result = {
		bool: result.control != "cancel2",
		cost_data: result.control,
	};
},
```

**⭐ 关键技巧在 `content` 里：用 `!=` 而不是 `==`。**

```js
async content(event, trigger, player) {
	const result = event.cost_data;
	if (result == "背水！") {
		// 先做背水的后果
	}
	if (result != "选项二") {
		// 不是"选项二"就做选项一 ⇒ 背水时也会走到这里
	}
	if (result != "选项一") {
		// 不是"选项一"就做选项二 ⇒ 背水时也会走到这里
	}
},
```

逐字读一遍就懂了：

| 玩家选了 | 第一个 `if` | 第二个 `if` | 第三个 `if` |
| --- | --- | --- | --- |
| 选项一 | 跳过 | ✅ 做选项一 | 跳过 |
| 选项二 | 跳过 | 跳过 | ✅ 做选项二 |
| **背水！** | ✅ 做后果 | ✅ 做选项一 | ✅ 做选项二 |

**"背水 = 全做"这个语义，就是靠 `!=` 实现的** —— 一行判断都没多写。

**三个必守的规矩**：

1. <strong>后果做不到时，别把 `"背水！"` 放进 `list`</strong>（官方那句 `if (player.getEquip(1) ...)` 就是干这个的）。这就是词典里"若无法执行背水的后果，则无法选择背水"的落地。
2. <strong>选项名要带感叹号</strong>：`"背水！"`（官方一律这么写，别写成 `"背水"`）。
3. <strong>AI 也要认识它</strong> —— `ai` 里可以返回 `"背水！"`，但只在真的划算时返回（官方会算一遍收益再决定）。

### 3.2 其余约定：一张索引表

下面这些名词，**引擎都没有专属机制**，全是"官方固定写法"。做的时候照官方同类技能抄骨架，别自己发明。

| 名词 | 大概是什么 | 去哪找官方范例 |
| --- | --- | --- |
| **整肃** | 三选一任务（擂进／变阵／鸣止），未失败则发奖励 | 把 `zhengsu_*` 规则技挂在一个临时技上 |
| **强令** | 出牌阶段开始时向一名其他角色发布任务，之后判"完成没有" | 挂 `phaseUseBegin` ＋ 自己存任务内容 |
| **摧坚** | "使用伤害牌指定第一个目标后"触发，X = 目标的非 `charlotte` 技能数 | 目标技能数用 `getSkills(null, false, false)` 数（第 16 章） |
| **妄行** | 声明 X（1~4），回合结束时弃 X 张牌或减 1 点体力上限 | 两件事都用正文讲过的现成方法 |
| **搏击 / 游击** | 按"双方是否互相在攻击范围内"决定要不要执行额外效果 | `inRange` 互判 |
| **历战** | 回合结束时按本回合发动次数做**永久可叠加**升级 | 次数统计 ＋ 修改自身数值 |
| **同心** | 与其他角色结对，技能发动后他们也依次执行 | 类似协力的"关系 ＋ 依次执行" |
| **施法** | 声明 X，第 X 个回合结束时执行效果 | 记回合号（`game.phaseNumber`）＋ 到点触发 |
| **入魔** | 每局限一次；入魔后每轮结束若未造成伤害则失去 1 体力 | 限定技 ＋ 每轮结算 |
| **乘势 / 激昂** | 多分支全满足时 / 昂扬技失效后恢复 | 见第 17 章的昂扬技（`sunbenSkill`） |
| **共同拼点 / 延时拼点** | 多人拼点 / 拼点结果延后公布 | 拼点家族，官方有专门写法 |
| **连接（手牌）** | 被连接的手牌对全可见；失去时全员依次弃置 | 牌上的 `gaintag` ＋ 监听失去牌 |
| **变身 / 变身技** | 攒指示物，满上限进入变身（替换武将牌） | 属于"化身体系"，见第 16 章 |

> 💡 **一个偷懒但正确的办法**：先去第 1 节那张词典表里找到它的**官方解释**（一句话讲清了规则），然后按那句解释去写代码。官方解释本身就是"需求说明书"。

---

## 四、卡牌带来的机制：两个"伪装成技能"的卡

有些冷门机制的载体**不是技能，而是卡牌**。判断方法很简单：

> **在 `lib.skill` 里搜不到、在 `lib.card` 里能搜到 ⇒ 它是卡牌机制。**

### 4.1 蓄谋：靠卡牌字段实现的"判定区叠牌"

**是什么**：把一张牌当作"蓄谋牌"置入判定区；判定阶段开始时二选一 —— 使用它，或者把所有蓄谋牌丢进弃牌堆。

**它的两个特殊之处，全都来自卡牌字段**：

| 字段 | 作用 |
| --- | --- |
| `allowDuplicate: true` | **允许判定区里重复存在** |
| `blankCard: true` | 判定区的这张牌**不给别人看详情** |
| `wuxieable: false` | 不能被【无懈可击】响应 |

⭐ **`allowDuplicate` 是个通用字段，不止蓄谋用得上** —— 普通的延时锦囊，同名在同一个人的判定区里只能有一张（`addJudge` 会拦下来）；**想让"同名可叠"就用它**。这就是词典里那句"蓄谋牌可在判定区内重复存在"的全部实现。

⭐ **`blankCard` 则是"暗着的一张牌"** —— 判定区里那张牌不展示牌面细节。两者常一起用：既能叠好几张，又互相看不见是什么。

### 4.2 【影】：一张"打不出去"的牌

**是什么**：一种特殊的基本牌，**不能主动使用**，只作为"资源"存在于手上。

| 字段 | 作用 |
| --- | --- |
| `enable: false` | **不能主动使用**（把"能用"这条路堵死） |
| `cardcolor: "spade"` | 指定**颜色**（和花色是两个字段，别混） |
| `destroy: "discardPile"` | 被"销毁"后进弃牌堆 |
| `fullskin: true` | 卡面用整图 |

⭐ **`enable: false` 是这类牌的关键**：它让这张牌**只能被技能塞进你手里、不能当牌打出去** —— 于是它天然变成了一种"计数物"。想让某个资源"看得见、有牌面、但用不出去"，就这么做。

⭐ 官方还给它配了一个"批量造牌"的辅助方法 `getYing(count)`，一次生成 `count` 张。**卡牌对象里可以直接写方法** —— 这是"给卡牌加小工具"的官方用法。

### 4.3 还有哪些字段算这一类

| 字段 | 一句话 |
| --- | --- |
| `allowDuplicate` | 判定区可叠同名 |
| `blankCard` | 判定区里的牌不给别人看 |
| `wuxieable: false` | 不可被无懈响应 |
| `enable: false` | 不能主动使用 |
| `destroy: "discardPile"` | 销毁后去哪 |
| `cardcolor` | 强行指定颜色 |
| `fullskin` / `fullimage` | 卡面用整图 |

📖 更完整的卡牌字段表见**附录 B 第七章**。

---

## 五、扩展能做什么：两个开放接口

这一节是"进阶彩蛋"—— 上面两个机制（名词词典、公共区域）**官方是开放给你用的**。

### 4.1 注册你自己的名词

你的扩展里如果有个自造名词（比如"誓约""印记"），可以注册进词典，让它也成为**可点开**的：

```js
// 在你的扩展的 content / precontent 里调用
lib.poptip.add({
	id: "my_shiyue",
	name: "誓约",
	info: "你的誓约对象回合开始时，你摸一张牌。",
});
```

**参数**：

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| `name` | ✅ | 名词本身（会写进 `lib.translate`） |
| `id` | 建议写 | 不写会自动生成一个随机 id（那就没法在描述里引用了） |
| `info` | 建议写 | 解释正文（会写进 `lib.translate` 的 `<id>_info`） |
| `type` | 可选 | 默认 `"rule"`；也可以是 `"skill"` / `"card"` / `"character"` |
| `dialog` | 可选 | 自定义"点开时显示什么"的函数（不写就显示 `info` 文字） |

⚠️ **`type` 会影响显示格式**：`"skill"` 会渲染成〖〗、`"card"` 渲染成【】、其余（含默认的 `"rule"`）就是纯名词。

⚠️ **`type` 只有那几种，写别的会当场抛错**（源码里是 `throw new Error("未注册的poptip类型: ...")`）。想加新类别，得先注册：

```js
lib.poptip.addType("mytype");
```

⚠️ **注册的时机**：要在"游戏加载你的扩展"这一步做（`game.import` 的回调里，或 `precontent`）。注册太晚（比如进游戏之后），描述早就渲染完了，就来不及了。

⚠️ 顺带一个提醒：`type` 写 `"skill"` / `"card"` 时，源码会 `console.warn` 提示你"请于 `lib.skill` / `lib.card` 中显式注册" —— **技能和卡牌不应该走 `poptip` 注册**，它们在 `lib.skill` / `lib.card` 里天然就有条目。`poptip` 是留给**规则名词**的。

### 4.2 注册你自己的公共区域

前面说"仁库的背后是个更通用的机制"—— 就是 `lib.commonArea`。

**它长什么样**（仁库自己的定义就是这么写的，结构一目了然）：

```js
// 简化示意：一个公共区域的"五件套"
lib.commonArea.set("renku", {
	translate: "仁库",              // 界面上显示的名字
	areaStatusName: "renku",        // 存牌的地方（_status.renku）
	toName: "toRenku",              // "移进去"时用的区域名
	fromName: "fromRenku",          // "取出来"时用的来源区域名
	async addHandeler(event, trigger, player) {
		// 牌被放进来时做什么（仁库在这里处理"超过 6 张就溢出"）
	},
	async removeHandeler(event, trigger, player) {
		// 牌被拿出去时做什么
	},
});
```

**五个字段的分工**：

| 字段 | 作用 |
| --- | --- |
| `translate` | 界面／日志里怎么称呼它 |
| `areaStatusName` | 牌存在哪个全局变量里（`_status.<这个名字>`） |
| `toName` / `fromName` | 移动牌时用的<strong>参数名</strong> —— 这是它接入"移动牌的整套流程"的关键 |
| `addHandeler` / `removeHandeler` | 进／出时你自己的处理逻辑（<strong>注意官方拼写就是 `Handeler`</strong>） |

⭐ **价值**：注册之后，你的公共区域就能走**引擎那套移动牌的流程**（`player.lose` / `gain` / `cardsGotoSpecial`…），从而白拿到日志、界面刷新、溢出处理、联机同步。

⚠️ 但说实话 —— **普通扩展基本用不到这个**。写"一个新的公共区域"是官方级别的改造，绝大多数需求用 `player.storage` ＋ 标记就够了。这里写出来，是为了让你**在别人的代码里遇到 `commonArea` 时不至于懵**。

---

## 动手改一改

### 练习 1

下面这个技能想实现"背水"：选项一是"摸一张牌"，选项二是"回复 1 点体力"，背水的后果是"失去 1 点体力"。请补全 `content`，让**选了"背水！"时三件事都发生**。

```js
tutorial_beishui: {
	enable: "phaseUse",
	async cost(event, trigger, player) {
		const result = await player
			.chooseControl({ controls: ["选项一", "选项二", "背水！", "cancel2"] })
			.set("choiceList", ["摸一张牌", "回复1点体力", "背水！失去1点体力，并执行以上所有选项"])
			.set("prompt", get.prompt(event.skill))
			.set("ai", function () {
				return "选项一";
			})
			.forResult();
		event.result = {
			bool: result.control != "cancel2",
			cost_data: result.control,
		};
	},
	async content(event, trigger, player) {
		const result = event.cost_data;
		// ★ 在这里写三个分支
	},
},
```

### 练习 2

你在自己的扩展里造了个名词"誓约"，想让它在技能描述里**可以点开看解释**。请写出注册代码，并说明注册要放在哪里、为什么不能放在 `content` 里。

---

## 参考答案（先自己做完再看）

### 练习 1

用 `!=` 三个分支：

```js
async content(event, trigger, player) {
	const result = event.cost_data;
	if (result == "背水！") {
		await player.loseHp();
	}
	if (result != "选项二") {
		await player.draw();
	}
	if (result != "选项一") {
		await player.recover();
	}
},
```

对照着读：

| 玩家选了 | 第一个 `if` | 第二个 `if` | 第三个 `if` |
| --- | --- | --- | --- |
| 选项一 | 跳过 | 摸一张 | 跳过 |
| 选项二 | 跳过 | 跳过 | 回复 1 点 |
| 背水！ | 失去 1 点 | 摸一张 | 回复 1 点 |

⭐ **这就是"背水 = 全做"的全部实现**。别写成三个 `else if` —— 那样背水时只会走一个分支。

### 练习 2

注册代码：

```js
lib.poptip.add({
	id: "my_shiyue",
	name: "誓约",
	info: "你的誓约对象回合开始时，你摸一张牌。",
});
```

描述里引用它（注意要用**模板字符串**）：

```js
tutorial_shiyue_info: `出牌阶段限一次，你可以与一名其他角色${get.poptip("my_shiyue")}。`,
```

**放在哪里**：扩展加载期 —— 也就是 `game.import` 的回调里，或者 `precontent` / `content` 这些**加载阶段**的运行点。

<strong>为什么不能放 `content`</strong>：这里的 `content` 指的是<strong>技能对象的 `content`</strong>（技能发动时才跑）。描述文字在<strong>游戏加载、渲染技能详情时</strong>就已经算完了 —— 等技能发动时再去注册，界面早就画好了，这个名词既不会被识别、也不会成为可点开的链接。<strong>注册必须早于"第一次渲染描述"。</strong>

> 💡 顺带一个配套技巧：`_info` 里嵌了 `get.poptip(...)` 之后，它就不再是个死字符串了。官方在这种情况下会把 `_info` 写成 **getter**：
> ```js
> get tutorial_shiyue_info() {
> 	return `出牌阶段限一次，你可以与一名其他角色${get.poptip("my_shiyue")}。`;
> },
> ```
> 这样每次读描述时都重新算一遍，注册顺序就不会成为问题。

---

**下一份**：附录 B「一页纸速查」—— 把全书最常用的表浓缩到几页，可以打印出来贴在显示器旁边。
