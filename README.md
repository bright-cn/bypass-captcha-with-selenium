# 使用 Selenium 在 Python 中绕过 CAPTCHA

[![Promo](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/)

本指南讲解如何在 Selenium 中绕过 CAPTCHA：

- [常见 CAPTCHA 类型](#常见-captcha-类型)
- [Selenium 处理 CAPTCHA：分步教程](#selenium-处理-captcha分步教程)
  - [第 1 步：创建新的 Python 项目](#第-1-步创建新的-python-项目)
  - [第 2 步：安装 Selenium](#第-2-步安装-selenium)
  - [第 3 步：设置 Selenium 脚本](#第-3-步设置-selenium-脚本)
  - [第 4 步：添加浏览器自动化逻辑](#第-4-步添加浏览器自动化逻辑)
  - [第 5 步：安装 Selenium Stealth 插件](#第-5-步安装-selenium-stealth-插件)
  - [第 6 步：配置 Stealth 以避免 CAPTCHAs](#第-6-步配置-stealth-以避免-captchas)
  - [第 7 步：重复机器人检测测试](#第-7-步重复机器人检测测试)
- [如果上述方案无效怎么办](#如果上述方案无效怎么办)

## 什么是 CAPTCHA？

[CAPTCHA](https://www.bright.cn/blog/web-data/what-is-a-captcha)（Completely Automated Public Turing test to tell Computers and Humans Apart）用于区分人类用户与机器人。它提供人类易做、机器难做的挑战。常见提供商包括 Google reCAPTCHA、hCaptcha 和 BotDetect。

## 常见 CAPTCHA 类型

- 文本型：输入被扭曲的字母/数字
- 图片型：在网格中识别特定物体
- 音频型：根据音频片段输入词语
- 拼图型：完成简单拼图（如滑块）

![Text CAPTCHA example](https://github.com/bright-cn/bypass-captcha-with-selenium/blob/main/images/Text-CAPTCHA-example.png)

CAPTCHA 常出现在表单提交的最后一步：

![CAPTCHA in form submission](https://github.com/bright-cn/bypass-captcha-with-selenium/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

其作用是阻止自动化机器人完成操作。虽然有解题服务，但因用户体验较差，强制固定显示 CAPTCHA 的做法并不常见。

CAPTCHA 也常是更广泛安全措施（如 Web 应用防火墙 WAF）的一部分：

![Web Application Firewall](https://github.com/bright-cn/bypass-captcha-with-selenium/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

当检测到可疑行为时，这些系统会触发 CAPTCHA。要绕过它们，机器人必须模拟人类行为，这通常需要频繁更新脚本。

## Selenium 处理 CAPTCHA：分步教程

[Selenium](https://www.selenium.dev/) 是最常用的浏览器自动化库之一，非常适合在控制浏览器的同时模拟人类行为。下面用 Python 脚本演示如何在 Selenium 中尽量避免触发 CAPTCHA。

### 第 1 步：创建新的 Python 项目

请先在本地安装 Python 3 和 Chrome。

如果你已经有 Selenium 抓取或测试脚本，可跳过前三步。否则，创建一个演示项目并进入：

```bash
mkdir selenium_demo
cd selenium_demo
```

在其中创建一个新的 Python 虚拟环境：

```bash
python -m venv venv
```

用你喜欢的 Python IDE 打开项目文件夹，创建 `script.py` 文件。

### 第 2 步：安装 Selenium

激活虚拟环境：

Windows:

```powershell
venv\Scripts\activate
```

Linux 或 macOS:

```bash
source venv/bin/activate
```

安装 Selenium：

```bash
pip install selenium
```

### 第 3 步：设置 Selenium 脚本

在 `script.py` 中导入 Selenium：

```python
from selenium import webdriver
```

创建一个 [`ChromeOptions`](https://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.chrome.options) 对象，将 Chrome 配置为无头模式启动：

```python
options = webdriver.ChromeOptions()
options.add_argument("--headless")
```

用上述配置初始化一个 [Chrome WebDriver](https://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.chrome.webdriver)，最后用 `quit()` 关闭它。当前的 `script.py` 如下：

```python
from selenium import webdriver

# 配置 Chrome 使用无头模式
options = webdriver.ChromeOptions()
options.add_argument("--headless")

# 启动一个 Chrome 实例
driver = webdriver.Chrome(options=options)

# 浏览器自动化逻辑...

# 关闭浏览器并释放资源
driver.quit()
```

上述脚本会以无头模式启动 Chrome，然后关闭浏览器。

### 第 4 步：添加浏览器自动化逻辑

为测试 Selenium 的 CAPTCHA 规避逻辑，脚本将访问 [bot.sannysoft.com](https://bot.sannysoft.com/) 并截取整页截图。该页面会运行多项测试以判断访问者是人还是机器人。用本机标准浏览器访问时应能全部通过。

使用 [`get()`](https://selenium-python.readthedocs.io/api.html#selenium.webdriver.remote.webdriver.WebDriver.get) 访问目标页面：

```python
driver.get("https://bot.sannysoft.com/")
```

Selenium 无内置整页截图功能。变通做法是将窗口尺寸调整为 `<body>` 尺寸后再截图：

```python
# 获取页面的宽高
full_width = driver.execute_script("return document.body.parentNode.scrollWidth")
full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# 将浏览器窗口设为页面宽高
driver.set_window_size(full_width, full_height)

# 截取整页截图
driver.save_screenshot("screenshot.png")

# 恢复原始窗口大小
driver.set_window_size(original_size["width"], original_size["height"])
```

这样，`screenshot.png` 将包含整页截图。

合并完整逻辑如下：

```python
from selenium import webdriver

# 配置 Chrome 使用无头模式
options = webdriver.ChromeOptions()
options.add_argument("--headless")

# 启动一个 Chrome 实例
driver = webdriver.Chrome(options=options)

# 访问目标页面
driver.get("https://bot.sannysoft.com/")

# 记录当前窗口大小
original_size = driver.get_window_size()

# 获取页面的宽高
full_width = driver.execute_script("return document.body.parentNode.scrollWidth")
full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# 将浏览器窗口设为页面宽高
driver.set_window_size(full_width, full_height)

# 截取整页截图
driver.save_screenshot("screenshot.png")

# 恢复原始窗口大小
driver.set_window_size(original_size["width"], original_size["height"])

# 关闭浏览器并释放资源
driver.quit()
```

运行脚本：

```bash
python script.py
```

脚本会在无头模式下启动 Chromium，访问目标页面、截图，然后关闭浏览器。项目根目录会生成 `screenshot.png` 文件。

![screenshot.png file example](https://github.com/bright-cn/bypass-captcha-with-selenium/blob/main/images/screenshot.png-file-example-206x1024.png)

如图中红框所示，无头模式下的 Chrome 未通过多项检测，这意味着脚本很可能被标记为机器人，从而在受保护站点上遭遇 CAPTCHA。

为降低被检测并避免 CAPTCHA，下一步使用 Stealth 插件。

### 第 5 步：安装 Selenium Stealth 插件

[Selenium Stealth](https://github.com/diprajpatra/selenium-stealth) 是一个 Python 包，可在 Selenium 控制下减少 Chrome/Chromium 被识别为机器人的概率。它通过修改浏览器属性来避免自动化特征泄露，从而绕过反机器人检测。

其工作方式类似 Puppeteer Stealth，但面向 Selenium。

使用 `pip` 安装：

```sh
pip install selenium-stealth
```

在 `script.py` 中引入：

```python
from selenium_stealth import stealth
```

### 第 6 步：配置 Stealth 以避免 CAPTCHAs

调用 `stealth()` 注册并配置 Chrome WebDriver，以规避 CAPTCHA：

```python
stealth(
driver,
user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36",
languages=["en-US", "en"],
vendor="Google Inc.",
platform="Win32",
webgl_vendor="Intel Inc.",
renderer="Intel Iris OpenGL Engine",
fix_hairline=True,
)
```

可按需调整参数；以上设置已足以绕过多数基础反爬检测。配置完成后，Selenium 控制的浏览器会更像真实用户的浏览器。

### 第 7 步：重复机器人检测测试

最终的 `script.py` 如下：

```python
from selenium import webdriver
from selenium_stealth import stealth

# 配置 Chrome 使用无头模式
options = webdriver.ChromeOptions()
options.add_argument("--headless")

# 启动一个 Chrome 实例
driver = webdriver.Chrome(options=options)

# 使用 Selenium Stealth 配置以减少被检测
stealth(
driver,
user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36",
languages=["en-US", "en"],
vendor="Google Inc.",
platform="Win32",
webgl_vendor="Intel Inc.",
renderer="Intel Iris OpenGL Engine",
fix_hairline=True,
)

# 访问目标页面
driver.get("https://bot.sannysoft.com/")

# 记录当前窗口大小
original_size = driver.get_window_size()

# 获取页面的宽高
full_width = driver.execute_script("return document.body.parentNode.scrollWidth")
full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# 将浏览器窗口设为页面宽高
driver.set_window_size(full_width, full_height)

# 截取整页截图
driver.save_screenshot("screenshot.png")

# 恢复原始窗口大小
driver.set_window_size(original_size["width"], original_size["height"])

# 关闭浏览器并释放资源
driver.quit()
```

再次执行：

```bash
python script.py
```

打开 `screenshot.png`，你会看到所有机器人检测测试均已通过：

![All bot detection tests passed on the new screenshot.png](https://github.com/bright-cn/bypass-captcha-with-selenium/blob/main/images/All-bot-detction-tests-passed-on-the-new-screenshot-249x1024.png)

## 如果上述方案无效怎么办

反机器人系统关注的不仅是浏览器设置，IP 信誉同样关键。仅靠免费库更换 IP 往往无效，你需要在 Selenium 中集成代理。

即便浏览器配置到位，CAPTCHA 仍可能出现。对于基础的 reCAPTCHA v2，可尝试 [selenium-recaptcha-solver](https://pypi.org/project/selenium-recaptcha/)，但这些库更新滞后、能力有限。

面对 Cloudflare 等复杂系统，基础方法会失效。更稳妥的方案是使用 Bright Data 的网页抓取工具，支持 reCAPTCHA、hCaptcha、px_captcha、SimpleCaptcha、GeeTest、FunCaptcha、Cloudflare Turnstile、AWS WAF Captcha、KeyCAPTCHA 等。

[Bright Data 的 CAPTCHA Solver](https://www.bright.cn/products/web-unlocker/captcha-solver) 可与任意 HTTP 客户端或浏览器自动化工具（包括 Selenium）配合使用。

## 结论

借助 Selenium Stealth，可以覆盖 Chrome 的默认配置以降低被检测概率。但这并非万能方案，高级反爬工具仍可能拦截你。更现实的做法是通过可解锁的 API 连接目标站点，直接获取去除 CAPTCHA 的页面 HTML。

其中一个选择是 [Web Unlocker](https://www.bright.cn/products/web-unlocker)。该抓取 API 会在每次请求时自动通过代理轮换出口 IP，并处理浏览器指纹、自动重试与 CAPTCHA 解决，让处理 CAPTCHA 变得前所未有的轻松。

立即注册，开启免费试用。
