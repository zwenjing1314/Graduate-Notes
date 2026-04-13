# 介绍

**pytesseract** 是一个基于 Google Tesseract-OCR 引擎的 Python 库，主要用于从图像中提取文字。它支持多种语言，易于使用，并且可以与其他图像处理库（如 OpenCV 和 PIL）结合使用。

# 常见API

## image_to_data()

###  函数签名与参数

```py
def image_to_data(
    image,                    # PIL Image 对象或文件路径
    lang: str = None,         # 识别语言，如 'eng', 'chi_sim'
    config: str = '',         # Tesseract 配置字符串
    output_type: Output = Output.STRING,  # 输出格式
    ...
) -> Union[str, dict]:
```

### 内部执行流程（源码级别）

```py
# pytesseract/pytesseract.py 的简化实现

def image_to_data(image, lang=None, config='', output_type=Output.STRING):
    """
    对图片执行 OCR，返回详细的结构化数据
    """
    
    # ===== 步骤 1：准备临时文件 =====
    if isinstance(image, Image.Image):
        # 如果是 PIL Image 对象，保存为临时文件
        temp_name = save_image(image)  # 生成 /tmp/tess_XXXXX.png
    else:
        # 如果已经是文件路径，直接使用
        temp_name = image
    
    # ===== 步骤 2：构建 Tesseract 命令行 =====
    # Tesseract 的 TSV 输出模式
    cmd = [
        tesseract_cmd,           # tesseract 可执行文件路径
        temp_name,               # 输入图片
        stdout_prefix,           # 输出前缀（使用 stdout）
        "--psm", "6",           # 页面分割模式（可被 config 覆盖）
        "--oem", "3",           # OCR 引擎模式（可被 config 覆盖）
        "tsv"                   # 输出格式：TSV (Tab-Separated Values)
    ]
    
    # 添加语言参数
    if lang:
        cmd.extend(["-l", lang])
    
    # 添加额外配置
    if config:
        cmd.extend(shlex.split(config))
    
    # ===== 步骤 3：执行系统调用 =====
    try:
        # 运行 Tesseract 命令行工具
        proc = subprocess.Popen(
            cmd,
            stdout=subprocess.PIPE,    # 捕获标准输出
            stderr=subprocess.PIPE,    # 捕获错误信息
            stdin=subprocess.DEVNULL   # 不需要标准输入
        )
        
        # 等待执行完成，获取输出
        stdout, stderr = proc.communicate()
        
        # 检查返回码
        if proc.returncode != 0:
            raise TesseractError(proc.returncode, stderr.decode('utf-8'))
    
    finally:
        # 清理临时文件
        if isinstance(image, Image.Image):
            os.remove(temp_name)
    
    # ===== 步骤 4：解析 TSV 输出 =====
    tsv_output = stdout.decode('utf-8')
    
    if output_type == Output.DICT:
        # 将 TSV 转换为字典格式
        return parse_tsv_to_dict(tsv_output)
    elif output_type == Output.DATAFRAME:
        # 转换为 Pandas DataFrame
        import pandas as pd
        return pd.read_csv(io.StringIO(tsv_output), sep='\t')
    else:
        # 返回原始字符串
        return tsv_output
```

### Tesseract 的 TSV 输出格式

Tesseract 执行后输出的 TSV 格式如下：

```py'
level	page_num	block_num	par_num	line_num	word_num	left	top	width	height	conf	text
1	1	0	0	0	0	0	0	1654	2339	-1	
2	1	1	0	0	0	50	100	800	1200	-1	
3	1	1	1	0	0	50	100	800	150	-1	
4	1	1	1	1	0	50	100	800	50	-1	
5	1	1	1	1	1	50	100	120	45	95.5	发票
5	1	1	1	1	2	180	100	80	45	92.3	号码
5	1	1	1	1	3	270	100	40	45	88.7	:
4	1	1	1	2	0	50	160	600	50	-1	
5	1	1	1	2	1	50	160	150	45	96.2	No.123456
```

层级结构：

```py
Level 1: Page（整个页面）
  └─ Level 2: Block（文本块，如段落、表格）
      └─ Level 3: Paragraph（段落）
          └─ Level 4: Line（行）
              └─ Level 5: Word（词）
```

### parse_tsv_to_dict() 的实现

```py
def parse_tsv_to_dict(tsv_output: str) -> dict[str, list]:
    """
    将 TSV 字符串解析为字典格式
    
    输入：
        level\tpage_num\t...\ttext\n
        5\t1\t1\t1\t1\t1\t50\t100\t120\t45\t95.5\t发票\n
        ...
    
    输出：
        {
            'level': [1, 2, 3, 4, 5, 5, ...],
            'page_num': [1, 1, 1, 1, 1, 1, ...],
            'block_num': [0, 1, 1, 1, 1, 1, ...],
            'par_num': [0, 0, 1, 1, 1, 1, ...],
            'line_num': [0, 0, 0, 1, 1, 2, ...],
            'word_num': [0, 0, 0, 0, 1, 2, ...],
            'left': [0, 50, 50, 50, 50, 180, ...],
            'top': [0, 100, 100, 100, 100, 100, ...],
            'width': [1654, 800, 800, 800, 120, 80, ...],
            'height': [2339, 1200, 150, 50, 45, 45, ...],
            'conf': [-1, -1, -1, -1, 95.5, 92.3, ...],
            'text': ['', '', '', '', '发票', '号码', ...]
        }
    """
    
    lines = tsv_output.strip().split('\n')
    
    # 第一行是表头
    headers = lines[0].split('\t')
    
    # 初始化字典，每个键对应一个空列表
    result = {header: [] for header in headers}
    
    # 解析每一行数据
    for line in lines[1:]:
        if not line.strip():
            continue
        
        values = line.split('\t')
        
        # 将每个值添加到对应的列表中
        for header, value in zip(headers, values):
            # 根据列名转换类型
            if header in ['left', 'top', 'width', 'height', 
                         'page_num', 'block_num', 'par_num', 
                         'line_num', 'word_num']:
                result[header].append(int(value) if value else 0)
            elif header == 'conf':
                result[header].append(float(value) if value else -1.0)
            else:
                result[header].append(value)
    
    return result
```

