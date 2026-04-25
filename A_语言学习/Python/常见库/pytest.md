# 介绍

pytest是一个非常成熟的全功能的Python测试框架



# 常用知识点

## 装饰器

### @pytest.fixture 

@pytest.fixture 是 pytest 框架中最核心的功能之一，用于定义可复用的测试资源。下面详细解释它的作用和工作原理：

一、核心作用

1. 提供可复用的测试数据/环境

```py
@pytest.fixture
def temp_output_dir(self, tmp_path):
    """提供临时输出目录"""
    return tmp_path / "output_images"
```

作用：
定义一个名为 temp_output_dir 的测试夹具（fixture）
其他测试方法可以通过参数名直接使用它
避免在每个测试中重复创建临时目录

二、工作原理详解

数据流向图

```
@pytest.fixture 定义
       ↓
┌─────────────────────────┐
│ def temp_output_dir():  │
│     return tmp_path /   │
│            "output_images" │
└────────────┬────────────┘
             │ 注册为 fixture
             ↓
测试方法声明依赖
       ↓
┌──────────────────────────────┐
│ def test_xxx(self,          │
│              temp_output_dir)│  ← 参数名匹配 fixture 名
│     ...                      │
└──────────────┬───────────────┘
               │ pytest 自动注入
               ↓
        执行测试时
               ↓
┌──────────────────────────────┐
│ 1. 调用 temp_output_dir()    │
│ 2. 获取返回值                │
│ 3. 注入到测试方法的参数中     │
└──────────────────────────────┘
```

三、实际使用示例

不使用 fixture（代码重复）

```py
class TestRenderPdfToImages:
    def test_render_single_page_pdf(self, single_page_pdf, tmp_path):
        # ❌ 每个测试都要手动创建目录
        temp_output_dir = tmp_path / "output_images"
        temp_output_dir.mkdir()
        
        result = render_pdf_to_images(single_page_pdf, temp_output_dir, dpi=300)
        assert len(result) == 1
    
    def test_render_multi_page_pdf(self, sample_pdf_path, tmp_path):
        # ❌ 重复代码
        temp_output_dir = tmp_path / "output_images"
        temp_output_dir.mkdir()
        
        result = render_pdf_to_images(sample_pdf_path, temp_output_dir, dpi=300)
        assert len(result) == 2
    
    def test_different_dpi(self, single_page_pdf, tmp_path):
        # ❌ 又是重复代码
        temp_output_dir = tmp_path / "output_images"
        temp_output_dir.mkdir()
        
        result = render_pdf_to_images(single_page_pdf, temp_output_dir, dpi=150)
        assert len(result) == 1
```

使用 fixture（代码复用）

```py
class TestRenderPdfToImages:
    @pytest.fixture
    def temp_output_dir(self, tmp_path):
        """只需定义一次"""
        return tmp_path / "output_images"
    
    def test_render_single_page_pdf(self, single_page_pdf, temp_output_dir):
        # ✅ 直接使用，无需关心如何创建
        result = render_pdf_to_images(single_page_pdf, temp_output_dir, dpi=300)
        assert len(result) == 1
    
    def test_render_multi_page_pdf(self, sample_pdf_path, temp_output_dir):
        # ✅ 同样的 fixture 可以复用
        result = render_pdf_to_images(sample_pdf_path, temp_output_dir, dpi=300)
        assert len(result) == 2
    
    def test_different_dpi(self, single_page_pdf, temp_output_dir):
        # ✅ 干净简洁
        result = render_pdf_to_images(single_page_pdf, temp_output_dir, dpi=150)
        assert len(result) == 1
```

### @pytest.mark.parametrize

一、核心作用

让同一个测试函数使用多组不同的输入数据自动运行多次，避免编写重复的测试代码。

二、工作原理详解

装饰器结构解析

```py
@pytest.mark.parametrize("dpi,expected_min_size", [
    (72, 500),      # 第1组参数
    (150, 1000),    # 第2组参数
    (300, 2000),    # 第3组参数
    (600, 4000),    # 第4组参数
])
def test_different_dpi_settings(self, single_page_pdf, temp_output_dir, dpi, expected_min_size):
    """测试不同 DPI 设置对输出图片尺寸的影响"""
    result = render_pdf_to_images(single_page_pdf, temp_output_dir, dpi=dpi)
    
    from PIL import Image
    with Image.open(result[0]) as img:
        width, height = img.size
        assert width >= expected_min_size
        assert height >= expected_min_size
```

三、执行流程（数据流）

```py
原始测试函数（写1次）
       ↓
@pytest.mark.parametrize 装饰
       ↓
pytest 自动展开为 4 个独立测试
       ↓
┌─────────────────────────────────────────────────┐
│ 测试1: test_different_dpi_settings[72-500]      │
│   dpi = 72                                      │
│   expected_min_size = 500                       │
│   → 执行测试逻辑                                 │
│   → 验证图片尺寸 >= 500px                        │
└─────────────────────────────────────────────────┘
       ↓
┌─────────────────────────────────────────────────┐
│ 测试2: test_different_dpi_settings[150-1000]    │
│   dpi = 150                                     │
│   expected_min_size = 1000                      │
│   → 执行测试逻辑                                 │
│   → 验证图片尺寸 >= 1000px                       │
└─────────────────────────────────────────────────┘
       ↓
┌─────────────────────────────────────────────────┐
│ 测试3: test_different_dpi_settings[300-2000]    │
│   dpi = 300                                     │
│   expected_min_size = 2000                      │
│   → 执行测试逻辑                                 │
│   → 验证图片尺寸 >= 2000px                       │
└─────────────────────────────────────────────────┘
       ↓
┌─────────────────────────────────────────────────┐
│ 测试4: test_different_dpi_settings[600-4000]    │
│   dpi = 600                                     │
│   expected_min_size = 4000                      │
│   → 执行测试逻辑                                 │
│   → 验证图片尺寸 >= 4000px                       │
└─────────────────────────────────────────────────┘
```



## pytest 类启动



| 运行范围                     | 命令格式                               | 示例                                              |
| ---------------------------- | -------------------------------------- | ------------------------------------------------- |
| **运行指定类中的所有方法**   | `pytest <文件名.py>::<类名>`           | `pytest test_sample.py::TestMyClass`              |
| **运行类中的特定方法**       | `pytest <文件名.py>::<类名>::<方法名>` | `pytest test_sample.py::TestMyClass::test_method` |
| **运行指定目录下的所有测试** | `pytest <目录名>`                      | `pytest tests/`                                   |
| **运行整个测试文件**         | `pytest <文件名>.py`                   | `pytest test_sample.py`                           |
| **运行所有默认命名的测试**   | `pytest`                               | 在项目根目录下直接执行                            |
| **通过表达式筛选**           | `pytest -k '<表达式>'`                 | `pytest -k "MyTestClass and not slow"`            |
| **通过标记筛选**             | `pytest -m <标记名>`                   | `pytest -m smoke`                                 |
| **从包中运行测试**           | `pytest --pyargs <包名>`               | `pytest --pyargs pkg.testing`                     |
| **从文件读取参数 (v8.2+)**   | `pytest @<文件名>`                     | `pytest @tests_to_run.txt`                        |

### 核心定位方式：`文件名::类名::方法名`

这是 pytest 中最核心、最精确的**节点ID（nodeid）**定位语法。你可以把它理解成一个路径：

- `pytest test_mod.py::TestClass`：运行指定类中的所有测试方法。
- `pytest test_mod.py::TestClass::test_method`：运行该类中的某一个特定方法。
- `pytest test_mod.py::TestClass::test_method[param]`：更加精确，可以运行参数化测试的特定实例。

### 运行类时的常用参数

在运行命令时，你还可以附加各种参数来控制测试行为：

- **控制输出**：`-v` (更详细信息)，`-s` (显示print内容)。
- **决定停止时机**：`-x` (首次失败则停止)，`--maxfail=2` (失败两次后停止)。
- **调试与报告**：`--lf` (仅运行上次失败的用例)，`--pdb` (失败时进入调试器)，`--durations=10` (展示最慢的10个测试)。

### 额外技巧

1. **查看完整帮助**：在终端运行 `pytest --help` 即可查看所有可用选项。
2. **🔁 组合使用**：你可以将类路径和各种参数组合，实现更灵活的测试执行，例如 `pytest tests/test_sample.py::TestMyClass -v -s --maxfail=3`。