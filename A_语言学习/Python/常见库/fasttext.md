## 安装

```
pip install fasttext
```

## FastText原理与应用介绍

FastText 是 Facebook AI Research (FAIR) 于 2016 年开源的一个轻量级自然语言处理库，用于文本分类和词向量表示学习。它的核心思想是基于词袋模型和 n-gram 特征，结合了 神经网络 的高效性和传统机器学习的可解释性。

### 核心思想

子词信息利用：FastText 不仅考虑完整的词，还考虑词内部的字符 n-gram（如 "apple" 的 3-gram 包括 "<ap", "app", "ppl", "ple", "le>"），这有助于处理未登录词 (OOV) 和形态丰富的语言。

高效的训练算法：使用 Hierarchical Softmax 和负采样技术加速训练过程，特别是在处理大规模语料和多分类问题时。

简单而强大的架构：采用浅层神经网络结构，输入层通过词向量和 n-gram 向量的平均表示文本，输出层进行分类或词向量学习。

### FastText代码实现示例

以下是使用 Python FastText 库进行文本分类和词向量训练的示例：

```py
import fasttext

# 1. 文本分类示例

# 准备训练数据（格式：__label__类别 文本）
train_data = """
__label__体育 篮球是一项受欢迎的运动。
__label__体育 足球比赛非常激烈。
__label__科技 人工智能发展迅速。
__label__科技 区块链技术有广泛应用。
__label__娱乐 这部电影很精彩。
__label__娱乐 音乐会现场气氛热烈。"""

# 保存训练数据到文件
with open('train.txt', 'w', encoding='utf-8') as f:
    f.write(train_data)

# 训练分类模型
model = fasttext.train_supervised(input="train.txt", lr=0.1, epoch=25, wordNgrams=2, dim=100)

# 测试模型
text = "智能手机技术日新月异"
prediction = model.predict(text)
print(f"预测类别：{prediction[0][0]}")
print(f"置信度：{prediction[1][0]:.4f}")

# 保存模型
model.save_model("text_classifier.bin")

# 2.词向量训练示例
# 准备训练语料（每行一个句子）
corpus = """
篮球是一项受欢迎的运动。
足球比赛非常激烈。
人工智能发展迅速。
区块链技术有广泛应用。
这部电影很精彩。
音乐会现场气氛热烈。
智能手机技术日新月异。"""

# 保存语料到文件
with open('corpus.txt', 'w', encoding='utf-8') as f:
    f.write(corpus)

# 训练词向量模型
word_model = fasttext.train_unsupervised('corpus.txt', model='skipgram', dim=100)

# 获取词向量
vector = word_model.get_word_vector("人工智能")
print(f"词向量维度：{len(vector)}")

# 查找相似词
similar_words = word_model.get_nearest_neighbors("篮球")
print("与‘篮球’最想似的词：")
for score, word in similar_words:
    print(f"{word}:{score:.4f}")
```

## GPT提问

### fasttext库以后工作中有用吗？它可以作为基准吗？看看展开来学习一下这个库？

有用，而且**很适合当“强而快”的基准（baseline）**。但它更像是你工具箱里的“轻量级拳头”，不是要取代 Transformer 的那种“重武器”。

1) #### fastText 以后工作中有用吗？

**有用的场景很实际：**

- **小中型文本分类/多标签分类**：垃圾/色情/涉政初筛、投诉分类、工单路由、意图识别、舆情主题等。
- **延迟/成本敏感**：需要毫秒级推理、CPU 跑、QPS 高、预算紧。
- **数据不算特别大、但要快速迭代**：业务早期先做 MVP、快速试错。
- **词形变化多的语言/噪声文本**：fastText 的**字符 n-gram 子词**机制对拼写错误、变体、未登录词更友好。

它常见的价值是：**你可以很快得到一个“不丢人”的效果 + 极强的吞吐**，并且工程上很省心（模型小、部署简单）。

2. #### 它可以作为基准吗？

**可以，而且是很好的 baseline**，原因：

- 比“TF-IDF + 线性分类器”通常更强（尤其是含大量 OOV、拼写噪声、短文本时）。
- 训练/调参成本低，适合作为“每个新任务先跑一下”的固定对照组。
- 很多团队会把 baseline 分层：
  - **Tier-0**：多数类/规则/词典
  - **Tier-1**：TF-IDF + LR / SVM
  - **Tier-2**：**fastText**（强/快/稳）
  - **Tier-3**：BERT/LLM fine-tune（效果上限更高、成本更高）

**注意**：如果你的任务依赖复杂语义、长依赖、推理（比如细粒度关系抽取、跨句推断），fastText 往往会被 Transformer 拉开差距——但 baseline 仍然有意义：你能清楚知道“重模型到底带来多少增益”。

3. #### 要不要展开学习？怎么学最划算？

建议学，而且用“能落地的方式”学：**先会用，再理解原理，再做工程化**。

**A. 先掌握：最小可用（1 天内）**

你只需要会两件事：

1. **监督分类（supervised）**格式：每行一个样本
    `__label__类名 文本内容...`
2. 常用参数：
   - `lr` 学习率
   - `epoch` 轮数
   - `wordNgrams`（1~2/3 常用）
   - `dim` 向量维度（50~200）
   - `minCount` 低频截断
   - `loss`（softmax/hs/ova；多标签常用 ova）

一个最小示例（Python）：

```py
import fasttext

model = fasttext.train_supervised(
    input="train.txt",
    lr=0.5,
    epoch=25,
    wordNgrams=2,
    dim=100,
    minCount=2,
    loss="softmax"
)

# print(model.test("test.txt"))
print(model.predict("这是一条投诉工单", k=3))
model.save_model("ft.bin")
```

**B. 再理解：它为啥强（半天）**

抓住三个关键词就够了：

- **Subword（字符 n-gram）**：解决未登录词、拼写变体
- **CBOW/Skip-gram 的高效实现**：训练快
- **Hierarchical softmax / Negative sampling**：大类别时更高效

理解到“它相当于带子词特征的线性模型/浅层模型”就能解释大多数现象。

**C. 工程化：让它真能当 baseline（1~2 天）**

把下面这套流程固定下来，以后任何文本分类项目都能复用：

- 数据切分：train/valid/test（防止泄露）
- 指标：Accuracy + Macro-F1（类别不均衡就用 Macro-F1/PR-AUC）
- 误差分析：top 混淆对、长尾类召回、OOV 比例
- 训练策略：
  - 类别不均衡：重采样/阈值/分层指标
  - 多标签：`loss="ova"` + 设阈值
- 部署策略：
  - **量化（quantize）**：模型更小更快（适合线上）

4. #### 什么时候不该把 fastText 当主力？

- 需要**深语义/复杂推理**、跨句信息、结构化知识
- 需要**更强的可控性/可解释性**（虽然 fastText 比深度模型更可解释一点，但仍不如规则/线性特征可控）
- 数据量很大且你追求 SOTA：通常会转 Transformer/LLM

### 选择单标签，你能不能生成的完整一套流程，加大一点数据量，进行分词，做层级标签，量化。模拟做一套符合工作中使用fasttext的完整流程

下面给你一套**“单标签（A）+ 分词 + 层级标签（两级）+ 训练/评估 + 量化上线”**的完整工作流模拟。你照着把脚本放进一个目录里跑，就能复用到真实项目（把数据生成那步换成你们真实工单即可）。

0) #### 目录结构（建议直接照抄）

```
ft_ticket_demo/
  data/
    raw/
      tickets_raw.tsv
    processed/
      train_parent.txt
      valid_parent.txt
      test_parent.txt
      train_child__logistics.txt
      valid_child__logistics.txt
      test_child__logistics.txt
      ...（每个父类一套 child 数据）
  models/
    parent.bin
    parent.q.bin
    child__logistics.bin
    child__logistics.q.bin
    ...
  scripts/
    00_generate_synth_data.py
    01_preprocess_tokenize.py
    02_train_fasttext.sh
    03_eval_hierarchy.py
    04_quantize_fasttext.sh
    05_infer_demo.py
```

#### 依赖与安装（两条路，选一种）

**A. 用 fastText 官方命令行（更常用、更稳定）**

```bash
# 获取 fastText 并编译（Linux/macOS）
git clone https://github.com/facebookresearch/fastText.git
cd fastText && make
# 编译完会生成 ./fasttext 可执行文件
```

**DeepSeek解答**
下载好后，这编译操作是怎么回事？生成的可执行文件有什么用？
cd fastText && make

**1. 为什么需要编译？**

fastText 是使用 **C++ 语言**编写的，计算机无法直接理解源代码（一堆 `.cc` 和 `.h` 文件）。`make` 命令会自动调用编译器（如 `g++`）将这些源代码转换成机器可以执行的二进制指令，最终产出一个可执行文件。

**2. 编译后生成的可执行文件有什么作用？**

编译成功后，你会在 `fastText` 目录下看到一个名为 **`fasttext`** 的文件（没有后缀，或者可能是 `fasttext.exe` 在 Windows 下）。这是一个**命令行工具**，你可以直接在终端中使用它来执行 fastText 的各种功能，而**不需要编写 Python 代码**。

例如，通过命令行你可以：

- **训练词向量**：
  `./fasttext skipgram -input corpus.txt -output model`
- **训练文本分类器**：
  `./fasttext supervised -input train.txt -output classifier`
- **查询词向量**：
  `./fasttext print-word-vectors model.bin < words.txt`
- **量化模型以减小体积**等。

这个可执行文件是 fastText 的核心工具，它提供了完整的 fastText 功能，效率高且不依赖 Python 环境。

**3. 和你之前 Python 代码的关系**

你之前运行的 Python 脚本（`FastText 库进行文本分类和词向量训练的示例.py`）实际上是通过 Python 绑定的方式调用了 fastText 的底层 C++ 库。如果你从源码编译，还可以通过以下命令安装 Python 绑定：

bash

```
pip install .
```



这样，你的 Python 环境中就会安装上这个源码编译的 fasttext 模块，可能解决之前遇到的兼容性问题。

**4. 总结**

- **`make` 是编译操作**，将源代码变成可执行程序。
- **生成的 `fasttext` 可执行文件**是一个强大的命令行工具，可以直接完成词向量训练、分类等任务。
- 如果你只想在 Python 中使用，可以编译后执行 `pip install .` 来安装 Python 包；如果你需要批量处理或集成到脚本中，命令行工具也很方便。

编译过程一般只需要执行一次，成功后你就可以根据需要使用这个可执行文件了。如果在编译过程中遇到缺少依赖（如 `g++`、`make`）的错误，请根据提示安装相应工具（例如在 Ubuntu 上执行 `sudo apt install build-essential`，macOS 安装 Xcode Command Line Tools）。

**B. Python 方式（如果你环境能装得上）**

```bash
pip install jieba scikit-learn pandas
# fasttext 的 pip 包在一些环境里可能需要编译依赖；装不上就用 A 方案
```

下面我默认你用 **A（命令行）** 来训练/量化，最贴近生产。

#### Step 1：生成“较大规模”模拟工单数据（单标签 + 层级标签）

scripts/00_generate_synth_data.py

```py
import random
import pandas as pd

random.seed(42)

PARENTS = {
  "logistics": ["delay", "tracking", "lost", "address_change"],
  "after_sale": ["refund", "return", "exchange", "warranty"],
  "payment": ["pay_failed", "charged_twice", "refund_status", "invoice"],
  "account": ["login", "security", "unbind_phone", "coupon_bind"],
  "product": ["damaged", "wrong_item", "missing_parts", "quality_issue"],
  "promotion": ["coupon", "price_diff", "activity_rule", "points"],
  "service": ["rude", "slow_response", "no_solution", "complaint_channel"],
}

TEMPLATES = {
  ("logistics","delay"): [
    "包裹{days}天没更新了 什么时候到",
    "物流一直停在{city} 我着急用",
    "快递延迟了 能催一下吗",
  ],
  ("logistics","tracking"): [
    "单号{track}查不到信息",
    "物流轨迹不对 显示已签收但我没收到",
    "帮我查下运单{track}现在在哪",
  ],
  ("logistics","lost"): [
    "快递丢了 一直找不到",
    "显示派送失败多次 我怀疑丢件",
    "包裹疑似遗失 需要处理",
  ],
  ("logistics","address_change"): [
    "地址填错了 能改成{city}{addr}吗",
    "订单还没发货 帮我修改收货地址",
    "能把收件电话改成{phone}吗",
  ],

  ("after_sale","refund"): [
    "我想退款 订单{order}怎么处理",
    "不想要了 直接退款可以吗",
    "申请退款 一直没通过",
  ],
  ("after_sale","return"): [
    "想退货 退货流程怎么走",
    "商品不合适 我要退回去",
    "退货地址给我一下",
  ],
  ("after_sale","exchange"): [
    "尺码不对 想换货",
    "能换成别的颜色吗",
    "换货大概多久能到",
  ],
  ("after_sale","warranty"): [
    "用了{days}天就坏了 质保怎么走",
    "保修需要提供什么材料",
    "售后维修怎么申请",
  ],

  ("payment","pay_failed"): [
    "支付失败 提示{err}",
    "付款一直转圈 没成功",
    "银行卡支付不通过 怎么办",
  ],
  ("payment","charged_twice"): [
    "我被扣了两次款",
    "重复扣费 需要退回一笔",
    "支付显示失败但扣款了",
  ],
  ("payment","refund_status"): [
    "退款进度怎么查",
    "退款多久到账",
    "退款到哪张卡",
  ],
  ("payment","invoice"): [
    "怎么开发票",
    "电子发票能补开吗",
    "发票抬头填错了能改吗",
  ],

  ("account","login"): [
    "登录不上 一直提示验证码错误",
    "账号进不去 帮我看看",
    "换手机后登录不了",
  ],
  ("account","security"): [
    "账号疑似被盗 有异常登录",
    "我想改密码",
    "账号安全怎么设置",
  ],
  ("account","unbind_phone"): [
    "手机号不用了 怎么解绑",
    "换绑手机需要什么",
    "原号码收不到验证码怎么办",
  ],
  ("account","coupon_bind"): [
    "优惠券绑定失败",
    "券到账了吗 我找不到",
    "领券提示活动不可用",
  ],

  ("product","damaged"): [
    "收到就是破损的",
    "包装烂了 商品坏了",
    "开箱发现碎了",
  ],
  ("product","wrong_item"): [
    "发错货了 不是我买的型号",
    "收到的颜色不对",
    "寄来的不是这个商品",
  ],
  ("product","missing_parts"): [
    "少了配件 缺{part}",
    "少发东西了",
    "包装里没有说明书/充电线",
  ],
  ("product","quality_issue"): [
    "质量很差 有瑕疵",
    "做工问题 很明显",
    "用了一次就出问题",
  ],

  ("promotion","coupon"): [
    "优惠券怎么用",
    "券提示不可用",
    "下单没抵扣优惠券",
  ],
  ("promotion","price_diff"): [
    "刚买就降价 能补差价吗",
    "价格变了 订单按哪个算",
    "活动价没生效",
  ],
  ("promotion","activity_rule"): [
    "活动规则怎么看",
    "满减怎么计算",
    "活动时间到什么时候",
  ],
  ("promotion","points"): [
    "积分怎么用",
    "积分没到账",
    "积分兑换失败",
  ],

  ("service","rude"): [
    "客服态度很差 我要投诉",
    "服务太差了 说话不礼貌",
    "客服敷衍我",
  ],
  ("service","slow_response"): [
    "客服一直不回复",
    "排队太久了",
    "等了很久没人处理",
  ],
  ("service","no_solution"): [
    "客服没解决问题",
    "一直推脱 不给方案",
    "沟通半天没有结果",
  ],
  ("service","complaint_channel"): [
    "投诉渠道在哪里",
    "我要升级处理/找主管",
    "怎么提交正式投诉",
  ],
}

def rnd_track():
    return "YT" + "".join(random.choice("0123456789") for _ in range(10))

def rnd_order():
    return "".join(random.choice("0123456789") for _ in range(12))

def rnd_phone():
    return "1" + "".join(random.choice("0123456789") for _ in range(10))

def rnd_city():
    return random.choice(["上海","北京","广州","深圳","杭州","成都","武汉","西安","南京","重庆"])

def rnd_addr():
    return random.choice(["中山路88号","人民路100号","软件园A座","国贸中心3楼","XX小区5栋"])

def rnd_part():
    return random.choice(["充电线","说明书","螺丝","遥控器","数据线","电池"])

def rnd_err():
    return random.choice(["网络异常","余额不足","交易受限","风控拦截","系统繁忙"])

def fill(t):
    return (t.replace("{track}", rnd_track())
             .replace("{order}", rnd_order())
             .replace("{phone}", rnd_phone())
             .replace("{city}", rnd_city())
             .replace("{addr}", rnd_addr())
             .replace("{part}", rnd_part())
             .replace("{err}", rnd_err())
             .replace("{days}", str(random.randint(1,14))))

def sample_one():
    parent = random.choice(list(PARENTS.keys()))
    child = random.choice(PARENTS[parent])
    text = fill(random.choice(TEMPLATES[(parent, child)]))
    # 加一点噪声（更像真实数据）
    if random.random() < 0.25:
        text += random.choice([" 麻烦尽快", " 很急", " 谢谢", " 订单别搞错", " 速度处理"])
    return parent, child, text

def main(n=80000, out_path=r"/Users/zhouwenjing/Documents/PythonProjects/CodeRepository/GPT-generiert/FastText-in-Practice/ft_ticket_demo/data/raw/tickets_raw.tsv"):
    rows = []
    for _ in range(n):
        parent, child, text = sample_one()
        rows.append({"parent": parent, "child": child, "text": text})  # ["key: value, key:value, key:value", ... 8000]
    df = pd.DataFrame(rows)
    df.to_csv(out_path, sep="\t", index=False)
    print("saved:", out_path, "rows=", len(df))

if __name__ == "__main__":
    main()
```

运行：

```bash
mkdir -p data/raw
python scripts/00_generate_synth_data.py
```

**DeepSeek提问**

df = pd.DataFrame(rows)
    df.to_csv(out_path, sep="\t", index=False)
    print("saved:", out_path, "rows=", len(df))
这里面的DataFrame和to_csv的用法是能不能写一下？

`pd.DataFrame()` 是 pandas 中用于创建二维表格数据结构（类似 Excel 表格或 SQL 表）的构造函数。这里的 `rows` 通常是**可迭代对象**（如列表、元组、字典等），提供了表格的数据。

常见的 `rows` 形式有：

- **列表的列表**：每个内层列表代表一行数据。

```
rows = [['Alice', 25], ['Bob', 30]]
df = pd.DataFrame(rows, columns=['Name', 'Age'])
```

- **字典的列表**：每个字典的键为列名，值为该行对应列的值。

```
rows = [{'Name': 'Alice', 'Age': 25}, {'Name': 'Bob', 'Age': 30}]
df = pd.DataFrame(rows)
```

- **NumPy 数组**：二维数组直接转换为 DataFrame。
- **单个字典**：如果 `rows` 是单个字典，可以指定 `index` 参数。

创建后的 `df` 是一个 DataFrame 对象，包含行和列，可以方便地进行数据操作。

`to_csv()` 是 DataFrame 的方法，用于将数据写入逗号分隔值（CSV）文件或其他分隔符文件。

参数解释：

- **`out_path`**：字符串，要保存的文件路径（如 `'data.tsv'` 或 `'/path/to/file.csv'`）。
- **`sep="\t"`**：指定字段分隔符。`"\t"` 表示制表符（Tab），因此生成的文件实际上是 TSV（Tab-Separated Values）文件。如果不指定，默认为逗号 `','`。
- **`index=False`**：表示不将 DataFrame 的行索引写入文件。默认情况下 `index=True`，会将行索引（0,1,2...）作为第一列写入，通常我们不需要它，所以设置为 `False`。

#### Step 2：分词 + 生成 fastText 训练文件（父类/子类）

scripts/01_preprocess_tokenize.py

```py
import re
import random
import pandas as pd
import jieba

random.seed(42)

def normalize(text: str) -> str:
    text = text.strip()
    text = re.sub(r"\s+", " ", text)
    text = re.sub(r"\d{6,}", " <NUM> ", text)  # 大数字归一（订单号/单号）
    text = re.sub(r"\s+", " ", text).strip()
    return text

def tokenize(text: str) -> str:
    # jieba 分词：fastText 要求 token 用空格分隔
    return " ".join([w for w in jieba.cut(text) if w.strip()])

def write_fasttext(lines, path):
    with open(path, "w", encoding="utf-8") as f:
        for lab, txt in lines:
            f.write(f"__label__{lab} {txt}\n")

def split_df(df, train=0.8, valid=0.1):
    df = df.sample(frac=1.0, random_state=42).reset_index(drop=True)
    n = len(df)
    n_train = int(n * train)
    n_valid = int(n * valid)
    return df[:n_train], df[n_train:n_train+n_valid], df[n_train+n_valid:]

def main(raw_path="data/raw/tickets_raw.tsv", out_dir="data/processed"):
    df = pd.read_csv(raw_path, sep="\t")
    # print(df.shape)
    # print(df.head())
    df["text_norm"] = df["text"].astype(str).map(normalize).map(tokenize)
    # print(df.shape)
    # print(df.head())
    tr, va, te = split_df(df)

    # 1) 父类数据
    parent_tr = list(zip(tr["parent"], tr["text_norm"]))
    parent_va = list(zip(va["parent"], va["text_norm"]))
    parent_te = list(zip(te["parent"], te["text_norm"]))

    import os
    os.makedirs(out_dir, exist_ok=True)
    write_fasttext(parent_tr, f"{out_dir}/train_parent.txt")
    write_fasttext(parent_va, f"{out_dir}/valid_parent.txt")
    write_fasttext(parent_te, f"{out_dir}/test_parent.txt")

    # 2) 子类数据：按 parent 分组，每个 parent 一套 child 模型（更符合层级约束）
    for parent, g in df.groupby("parent"):
        tr_g, va_g, te_g = split_df(g)

        child_tr = list(zip(tr_g["child"], tr_g["text_norm"]))
        child_va = list(zip(va_g["child"], va_g["text_norm"]))
        child_te = list(zip(te_g["child"], te_g["text_norm"]))

        write_fasttext(child_tr, f"{out_dir}/train_child__{parent}.txt")
        write_fasttext(child_va, f"{out_dir}/valid_child__{parent}.txt")
        write_fasttext(child_te, f"{out_dir}/test_child__{parent}.txt")

    print("processed saved to:", out_dir)

if __name__ == "__main__":
    main()
```

运行：

```bash
pip install jieba pandas
mkdir -p data/processed
python scripts/01_preprocess_tokenize.py
```

**CodeArts Agent提问**

.astype(str).map(normalize).map(tokenize)这里面的函数都是什么意思？

这段代码是典型的 **Pandas 数据预处理流水线** 操作。它利用了 Pandas 的**链式调用**（Method Chaining）特性，对 DataFrame 中的文本数据按顺序进行了类型转换、规范化处理和分词处理。

具体来说，`df["text_norm"] = df["text"].astype(str).map(normalize).map(tokenize)` 这行代码的执行逻辑如下：

**1. `df["text"]`**

- **含义**：从 DataFrame `df` 中选取名为 `"text"` 的那一列数据（这是一个 Pandas Series 对象）。

**2. `.astype(str)`**

- **含义**：**类型转换**。
- **作用**：将 `"text"` 列中的**每一个元素**都强制转换为字符串类型。
- **为什么这样做**：在实际数据中，文本列可能混杂着数字、空值（NaN）或其他类型。直接处理这些非字符串数据可能会导致后续的字符串操作报错，所以先统一转为字符串是最安全的做法。

**3. `.map(normalize)`**

- **含义**：**元素级映射**。

- **作用**：将 `normalize` 函数应用到 Series 的每一个元素上。

- `normalize` 是什么

  ：这通常是一个

  自定义函数

  （在代码的其他地方定义）。在 NLP（自然语言处理）任务中，它通常用于

  文本规范化

  ，例如：

  - 去除文本中的特殊符号、标点、HTML 标签。
  - 将全角字符转为半角。
  - 统一转换为小写（如果是英文）。
  - 去除多余的空格。

**4. `.map(tokenize)`**

- **含义**：**元素级映射**。

- **作用**：将 `tokenize` 函数应用到上一步处理好的每一个元素上。

- `tokenize` 是什么

  ：这也是一个

  自定义函数

  。它的作用是

  分词

  ，即把连续的字符串切分成一个个独立的词元列表。

  - 例如：`"我爱编程"` 经过 tokenize 可能会变成 `["我", "爱", "编程"]`。

**5. `df["text_norm"] = ...`**

- **含义**：**赋值**。
- **作用**：将经过上述三步处理后的最终结果，保存到 DataFrame 中的一个新列，列名为 `"text_norm"`（normalized text，规范化文本）。

**总结**

这行代码完成了一个完整的数据清洗流程：
**原始文本** -> **转字符串** -> **清洗/规范化** -> **分词** -> **保存结果**。

.**逐步处理过程解析**

- **第一步：`text = text.strip()`**
  - **操作**：去除字符串**开头和结尾**的所有空白字符（包括空格、换行符 `\n`、制表符 `\t` 等）。
  - **目的**：清理文本边界。
- **第二步：`text = re.sub(r"\s+", " ", text)`**
  - **操作**：使用正则表达式 `\s+` 匹配文本内部**所有连续的空白字符**，并将它们统一替换为**单个空格**。
  - **目的**：消除文本中间多余的空格、换行或缩进，统一格式。
- **第三步：`text = re.sub(r"\d{6,}", " <NUM> ", text)`**
  - **操作**：使用正则表达式 `\d{6,}` 匹配**连续 6 位及以上的数字**，并将它们替换为字符串 `" <NUM> "`（注意前后带有空格）。
  - **目的**：将长数字（如订单号、身份证号片段、金额等）泛化为一个特殊标记 `<NUM>`，防止模型学习到无意义的特定数字规律。
- **第四步：`text = re.sub(r"\s+", " ", text).strip()`**
  - **操作**：再次执行“将连续空白替换为单个空格”，并去除首尾空格。
  - **目的**：这是为了修复第三步可能带来的副作用。例如，如果原文是 `abc123456def`，替换后变成 `abc <NUM> def`，这是没问题的。但如果原文是 `abc 123456 def`，替换后可能变成 `abc <NUM> def`（出现双空格）。这一步确保最终文本的空格也是干净的。

**这里面的sample函数的具体用法是什么？还有rest_index函数**

这段代码中使用了 Pandas 的两个非常核心的方法：`sample()` 和 `reset_index()`。它们通常配合使用，用于在划分数据集前进行“洗牌”操作。

下面我为你详细解释这两个函数的具体用法和参数含义。

1. `df.sample()` —— 随机抽样/打乱

`sample()` 函数主要用于从 DataFrame 中随机抽取行。在这个代码中，它的作用是**打乱数据顺序**。

**语法：** `df.sample(n=None, frac=None, replace=False, weights=None, random_state=None, axis=None)`

**代码中的用法：** `df.sample(frac=1.0, random_state=42)`

- **

  ```
  frac=1.0
  ```

  **：

  - **含义**：抽取的比例。
  - **解释**：`1.0` 表示 100%，即抽取所有行。如果写 `0.5` 则抽取一半的行。
  - **作用**：因为抽取了所有行且是随机的，所以实际上实现了**对整个 DataFrame 的行顺序进行随机打乱**（Shuffle）。

- **

  ```
  random_state=42
  ```

  **：

  - **含义**：随机种子。
  - **解释**：这是一个固定整数（42 是计算机科学中的经典梗数字）。
  - **作用**：保证**可复现性**。如果不设置这个参数，每次运行代码时，打乱的顺序都会不一样；设置了 42 之后，无论运行多少次，打乱的顺序都是完全一样的。这对于调试模型和复现实验结果非常重要。

------

2. `reset_index()` —— 重置索引

`reset_index()` 函数用于重置 DataFrame 的索引（Index）。当数据经过筛选、删除或打乱后，索引通常会变得不连续（例如：0, 5, 3, 9...），这个方法可以将其恢复为标准的 0, 1, 2, 3...

**语法：** `df.reset_index(level=None, drop=False, inplace=False, ...)`

**代码中的用法：** `.reset_index(drop=True)`

- **

  ```
  drop=True
  ```

  **：

  - **含义**：是否丢弃原索引。

  - 解释

    ：

    - 如果是 `False`（默认），原来的索引会变成一列新的数据，列名为 `index`。
    - 如果是 `True`，原来的索引会被直接丢弃，不会保留在数据中。

  - **作用**：在这里，原来的索引（0, 1, 2...）只是位置标记，没有实际意义，打乱后变成乱序的旧索引也没用了，所以用 `drop=True` 把它扔掉，保持数据整洁。

------

3. 为什么要这样组合使用？

这行代码 `df = df.sample(frac=1.0, random_state=42).reset_index(drop=True)` 的完整逻辑是：

1. **打乱数据**：防止原始数据是有序的（例如前 100 行都是类别 A，后 100 行都是类别 B）。如果不打乱直接切分，会导致训练集只有类别 A，测试集只有类别 B，模型就会训练失败。
2. **重置索引**：打乱后，数据的行号变成了乱序（比如第 0 行其实是原来的第 50 行数据）。为了方便后续使用切片（如 `df[:n_train]`）来切分数据集，必须把索引重新整理成 0, 1, 2, 3... 这样连续的顺序。

**总结：**

- `sample(frac=1.0, ...)`：负责**洗牌**（打乱顺序）。
- `reset_index(drop=True)`：负责**整理牌面**（把牌序号重新排好）。

`zip()` 是 Python 中一个非常实用且常用的内置函数。简单来说，它的作用是**“打包”**或**“配对”**。

1. `zip()` 的核心用法

它接受多个可迭代对象（如列表、元组、Pandas Series 等），将它们中**对应位置**的元素一个个地配对，组成一个个元组，最后返回一个包含这些元组的迭代器。

**形象比喻：**
就像衣服的**拉链**一样，两边的齿扣（元素）一一对应地扣在一起。

**简单示例：**

```python
a = [1, 2, 3]
b = ['a', 'b', 'c']

zipped = zip(a, b)
print(list(zipped))
```

```python
[(1, 'a'), (2, 'b'), (3, 'c')]
```

可以看到，列表 `a` 的第 1 个元素和列表 `b` 的第 1 个元素组成了一个新的元组 `(1, 'a')`，以此类推。

#### Step 3：训练（父类模型 + 每个父类一个子类模型）

scripts/02_train_fasttext.sh

```bash
#!/usr/bin/env bash
set -e

FASTTEXT=./fastText/fasttext   # 改成你的 fasttext 可执行文件路径
DATA=data/processed
OUT=models

mkdir -p ${OUT}

# 通用超参（单标签 softmax）
LR=0.5
EPOCH=25
DIM=100
NGRAM=2
MINCOUNT=2
THREAD=8
MINN=2
MAXN=5

# 1) 父类模型
${FASTTEXT} supervised \
  -input ${DATA}/train_parent.txt \
  -output ${OUT}/parent \
  -lr ${LR} -epoch ${EPOCH} -dim ${DIM} \
  -wordNgrams ${NGRAM} -minCount ${MINCOUNT} \
  -minn ${MINN} -maxn ${MAXN} \
  -loss softmax -thread ${THREAD}

# 2) 子类模型：每个 parent 一个
for parent in logistics after_sale payment account product promotion service
do
  ${FASTTEXT} supervised \
    -input ${DATA}/train_child__${parent}.txt \
    -output ${OUT}/child__${parent} \
    -lr ${LR} -epoch ${EPOCH} -dim ${DIM} \
    -wordNgrams ${NGRAM} -minCount ${MINCOUNT} \
    -minn ${MINN} -maxn ${MAXN} \
    -loss softmax -thread ${THREAD}
done

echo "training done"
```

运行：

```bash
chmod +x scripts/02_train_fasttext.sh
scripts/02_train_fasttext.sh
```

#### Step 4：层级评估（父类 accuracy + 子类 accuracy + 端到端 accuracy）

fastText 自带 `test` 只给 top1 precision/recall，不够“层级端到端”。我们用一个评估脚本更贴近工作。

scripts/03_eval_hierarchy.py

```

```
