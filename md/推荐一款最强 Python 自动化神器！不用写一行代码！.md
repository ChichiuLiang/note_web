> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/338091702) ![](https://pic4.zhimg.com/v2-dcaefc874363856b43de21570b509d6b_r.jpg)

  
搞过自动化测试的小伙伴，相信都知道，在 Web 自动化测试中，有一款自动化测试神器工具: `selenium`。结合标准的 WebDriver API 来编写 Python 自动化脚本，可以实现解放双手，让脚本代替人工在 Web 浏览器上完成指定的操作。  
虽然`selenium`有完备的文档，但也需要一定的学习成本，对于一个纯小白来讲还是有些门槛的。  
最近，**微软开源**了一个非常强大的自动化项目叫**「playwright-python」**，项目地址：  
[https://github.com/microsoft/playwright-python](https://github.com/microsoft/playwright-python)  
它支持主流的浏览器，包含：`Chrome`、`Firefox`、`Safari`、`Microsoft Edge` 等，同时支持以**无头模式**、**有头模式运行**，并提供了`同步`、`异步`的 API，可以结合 `Pytest` 测试框架使用，并且支持浏览器端的自动化脚本录制。  
而对于 Python 爱好者来说，还有一个更大的福利，这个项目是针对 Python 语言的纯自动化工具，**可以做到，连一行代码都不用写，就能实现自动化功能**。听起来，简直太碉堡了！  

![](https://pic1.zhimg.com/v2-c2a8209f9999efca193457600711a8b4_b.jpg)

  
可能你会觉得有点不可思议，真的不用写一行代码吗？但它真的就是这么厉害。下面我们一起看下这个神器。  
**1.** **Playwright 介绍**  
Playwright 是一个强大的 Python 库，仅用一个 API 即可自动执行`Chromium`、`Firefox`、`WebKit`等主流浏览器自动化操作，并同时支持以**无头模式**、**有头模式**运行。  
Playwright 提供的自动化技术是绿色的、功能强大、可靠且快速，支持`Linux`、`Mac`以及`Windows`操作系统。  

![](https://pic1.zhimg.com/v2-320efaed36d77af4f2b7d4445b45a7e0_r.jpg)

  
**官网：**  
[https://playwright.dev/](https://playwright.dev/)  

![](https://pic3.zhimg.com/v2-cfda50656dd2bbe85f2c89cab7ad523e_r.jpg)

  
从官网上来看，官方给`Playwright`定位是一款真正意义上的 Web 端到端测试工具。  
**2.** **Playwright 使用**  
**2.1 安装**  
Playwright 功能强大，但它的安装步骤，非常简单，只需要 2 步:  
**第 1 步，安装 playwright-python 依赖库** (需要注意的是，playwright 库需要依赖 Python3.7 + 以上)  
# 安装依赖库 ➜ ~ pip3 install playwright Looking in indexes: [https://pypi.douban.com/simple](https://pypi.douban.com/simple) Collecting playwright Downloading [https://pypi.doubanio.com/packages/08/f0/9f937ccff3221685d4a8bd406649c85855b9b6a2fafe75920b02151b48e0/playwright-0.162.2-py3-none-macosx_10_13_x86_64.whl](https://pypi.doubanio.com/packages/08/f0/9f937ccff3221685d4a8bd406649c85855b9b6a2fafe75920b02151b48e0/playwright-0.162.2-py3-none-macosx_10_13_x86_64.whl) (58.2 MB) |████████████████████████████████| 58.2 MB 1.6 MB/s Collecting greenlet==1.0a1 Downloading [https://pypi.doubanio.com/packages/aa/74/6e93515873829a8d894863bbae1d709405bdd50d66fdf239480cc9db0598/greenlet-1.0a1-cp38-cp38-macosx_10_9_x86_64.whl](https://pypi.doubanio.com/packages/aa/74/6e93515873829a8d894863bbae1d709405bdd50d66fdf239480cc9db0598/greenlet-1.0a1-cp38-cp38-macosx_10_9_x86_64.whl) (86 kB) |████████████████████████████████| 86 kB 6.9 MB/s Collecting typing-extensions Downloading [https://pypi.doubanio.com/packages/60/7a/e881b5abb54db0e6e671ab088d079c57ce54e8a01a3ca443f561ccadb37e/typing_extensions-3.7.4.3-py3-none-any.whl](https://pypi.doubanio.com/packages/60/7a/e881b5abb54db0e6e671ab088d079c57ce54e8a01a3ca443f561ccadb37e/typing_extensions-3.7.4.3-py3-none-any.whl) (22 kB) Collecting pyee>=8.0.1 Downloading [https://pypi.doubanio.com/packages/0d/0a/933b3931107e1da186963fd9bb9bceb9a613cff034cb0fb3b0c61003f357/pyee-8.1.0-py2.py3-none-any.whl](https://pypi.doubanio.com/packages/0d/0a/933b3931107e1da186963fd9bb9bceb9a613cff034cb0fb3b0c61003f357/pyee-8.1.0-py2.py3-none-any.whl) (12 kB) Installing collected packages: greenlet, typing-extensions, pyee, playwright Successfully installed greenlet-1.0a1 playwright-0.162.2 pyee-8.1.0 typing-extensions-3.7.4.3  
可以在`https://pypi.org/project/playwright/`查看它的依赖版本信息。  
**第 2 步，安装主流的浏览器驱动**  
这样，会将 Chromeium、Firefox、Webkit 浏览器驱动下载到本地  
# 安装浏览器驱动（安装过程稍微有点慢，请耐心等待） ➜ ~ python3 -m playwright install Downloading chromium v827102 - 121.3 Mb [====================] 100% 0.0s chromium v827102 downloaded to /Users/xxx/Library/Caches/ms-playwright/chromium-827102 Downloading firefox v1205 - 74.1 Mb [ ] 1% 37767.9s  
3. **如果想查看`Playwright`支持的功能**， 可以直接在命令行输入：  
➜ ~ python3 -m playwright help Usage: index [options] [command] Options: -V, --version output the version number -b, --browser <browserType> browser to use, one of cr, chromium, ff, firefox, wk, webkit (default: "chromium") --color-scheme <scheme> emulate preferred color scheme, "light" or "dark" --device <deviceName> emulate device, for example "iPhone 11" --geolocation <coordinates> specify geolocation coordinates, for example "37.819722,-122.478611" --lang <language> specify language / locale, for example "en-GB" --proxy-server <proxy> specify proxy server, for example "[http://myproxy:3128](http://myproxy:3128)"or"socks5://myproxy:8080"--timezone <time zone> time zone to emulate, for example"Europe/Rome"--timeout <timeout> timeout for Playwright actions in milliseconds (default:"10000") --user-agent <ua string> specify user agent string --viewport-size <size> specify browser viewport size in pixels, for example"1280, 720" -h, --help display help for command Commands: open [url] open page in browser specified via -b, --browser cr [url] open page in Chromium ff [url] open page in Firefox wk [url] open page in WebKit codegen [options] [url] open page and generate code for user actions screenshot [options] <url> <filename> capture a page screenshot pdf [options] <url> <filename> save page as pdf install Ensure browsers necessary for this version of Playwright are installed help [command] display help for command  
从命令行帮助信息中可以看出，`Playwright`支持的功能相当丰富！  
**3. 实操演示**  
开篇就提到，使用 Playwright 无需写一行代码，我们只需手动操作浏览器，它会录制我们的操作，然后自动生成代码脚本。  
**3.1 录制脚本**  
我们先查看录制脚本的命令说明  
➜ ~ python3 -m playwright codegen --help Usage: index codegen [options] [url] open page and generate code for user actions Options: -o, --output <file name> saves the generated script to a file --target <language> language to use, one of javascript, python, python-async, csharp (default: "python") -h, --help display help for command Examples: $ codegen $ codegen --target=python $ -b webkit codegen https://example.com  
**其中**  

*   python -m playwright codegen 录制脚本
*   --help 帮助文档
*   -o 生成自动化脚本的目录
*   --target 脚本语言，包含 JS 和 Python，分别对应值为：python 和 javascript
*   -b 指定浏览器驱动

比如，我要在 [http://baidu.com](http://baidu.com) 搜索，用 chromium 驱动，将结果保存为 mikezhou.py 的 python 文件。  
# 我们通过下面命令打开 Chrome 浏览器开始录制脚本 # 指定生成语言为: Python（默认 Python，可选） # 保存的文件名：mikezhou.py（可选） # 浏览器驱动：webkit（默认 webkit，可选） # 最后跟着要打开的目标网站（默认仅仅是打开浏览器，可选） python3 -m playwright codegen --target python -o 'mikezhou.py' -b chromium [https://www.baidu.com](https://www.baidu.com)  
命令行输入后会自动打开浏览器，然后可以看见在浏览器上的一举一动都会被自动翻译成代码，如下所示:  

![](https://pic4.zhimg.com/v2-32a3944fb40ec85be20cd1eae92ab33f_r.jpg)![](https://pic1.zhimg.com/v2-6ce896ae8fbfe9e0b24bf02032304de8_r.jpg)

  
最后，自动化脚本会自动生成，保存到文件中`mikezhou.py`, 且上述所有的人工操作，都会被自动转化成代码：  
from playwright import sync_playwright def run(playwright): browser = playwright.chromium.launch(headless=False) context = browser.newContext() # Open new page page = context.newPage() # Go to [https://www.baidu.com/](https://www.baidu.com/) page.goto("[https://www.baidu.com/](https://www.baidu.com/)") # Click input[] page.click("input[name=\"wd\"]") # Fill input[] page.fill("input[name=\"wd\"]"," 禾目大 ") # Press CapsLock page.press("input[name=\"wd\"]","CapsLock") # Fill input[] page.fill("input[name=\"wd\"]"," 自动化测试实战宝典 ") # Press Enter page.press("input[name=\"wd\"]","Enter") # assert page.url() =="[https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&rsv_idx=1&tn=baidu&wd=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95%E5%AE%9E%E6%88%98%E5%AE%9D%E5%85%B8%20&fenlei=256&rsv_pq=af40e9aa00012d5a&rsv_t=c659gpz2%2Fjri1SAoIXdT9gP%2BmrqufXzRtMSSAL0n0fv7GSoLF5vaiNVPA3U&rqlang=cn&rsv_enter=1&rsv_dl=tb&rsv_sug3=38&rsv_sug1=22&rsv_sug7=100&rsv_sug2=0&rsv_btype=i&inputT=8034&rsv_sug4=9153](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&rsv_idx=1&tn=baidu&wd=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95%E5%AE%9E%E6%88%98%E5%AE%9D%E5%85%B8%20&fenlei=256&rsv_pq=af40e9aa00012d5a&rsv_t=c659gpz2%2Fjri1SAoIXdT9gP%2BmrqufXzRtMSSAL0n0fv7GSoLF5vaiNVPA3U&rqlang=cn&rsv_enter=1&rsv_dl=tb&rsv_sug3=38&rsv_sug1=22&rsv_sug7=100&rsv_sug2=0&rsv_btype=i&inputT=8034&rsv_sug4=9153)" # Close page page.close() # --------------------- context.close() browser.close() with sync_playwright() as playwright: run(playwright)  
**3.2 支持同步**  
同步的关键字为：`sync_playwright`  
比如，我们依次使用三个浏览器内核打开浏览器，然后百度一下，接着对在搜索界面截图，最后关闭浏览器  
from time import sleep from playwright import sync_playwright # 注意：默认是无头模式 with sync_playwright() as p: # 分别对应三个浏览器驱动 for browser_type in [p.chromium, p.firefox, p.webkit]: # 指定为有头模式，方便查看 browser = browser_type.launch(headless=False) page = browser.newPage() page.goto('[http://baidu.com](http://baidu.com)') # 执行一次搜索操作 page.fill("input[name=\"wd\"]"," 自动化测试实战宝典 ") with page.expect_navigation(): page.press("input[name=\"wd\"]","Enter") # 等待页面加载完全 page.waitForSelector("text = 搜索工具 ") # 截图 page.screenshot(path=f'test-{browser_type.name}.png') # 休眠 3s sleep(3) # 关闭浏览器 browser.close()  
需要指出的是，`playwright-python` 内置的 API 基本上囊括常见的自动化操作  
**3.3 支持异步**  
异步步的关键字为：`async_playwright`，异步操作可结合 asyncio 同时进行三个浏览器操作。  
import asyncio from playwright import async_playwright # 异步执行 async def main(): async with async_playwright() as p: for browser_type in [p.chromium, p.firefox, p.webkit]: # 指定为有头模式，方便查看 browser = await browser_type.launch(headless=False) page = await browser.newPage() await page.goto('[http://baidu.com](http://baidu.com)') # 执行一次搜索操作 await page.fill("input[name=\"wd\"]"," 自动化测试实战宝典 ") await page.press("input[name=\"wd\"]","Enter") # 等待页面加载完全 await page.waitForSelector("text = 搜索工具 ") # 截图 await page.screenshot(path=f'test-{browser_type.name}.png') await browser.close() asyncio.get_event_loop().run_until_complete(main())  
**3.4 支持移动端**  
更厉害的是，playwright 还可支持移动端的浏览器模拟。下面是官方文档提供的一段代码，模拟在给定地理位置上手机 iphone 11 pro 上的 Safari 浏览器，首先导航到 [http://maps.google.com](http://maps.google.com)，然后执行定位并截图。  
from playwright import sync_playwright with sync_playwright() as p: iphone_11 = p.devices['iPhone 11 Pro'] browser = p.webkit.launch(headless=False) context = browser.newContext( **iphone_11, locale='en-US', geolocation={ 'longitude': 12.492507, 'latitude': 41.889938 }, permissions=['geolocation'] ) page = context.newPage() page.goto('[https://maps.google.com](https://maps.google.com)') page.click('text="Your location"') page.screenshot(path='colosseum-iphone.png') browser.close()  
**3. 5 支持 Pytest 框架**  
另外，还可以配合 pytest 插件一起使用，给出一段官网示例:  
def test_playwright_is_visible_on_google(page): page.goto("[https://www.google.com](https://www.google.com)") page.type("input[name=q]","Playwright GitHub") page.click("input[type=submit]") page.waitForSelector("text=microsoft/Playwright")  
当然，除了上面列举出来的特性，还有更多有意思的用法，感兴趣的读者可以自行探索一下。  
**4. 最后**  
playwright 相比已有的自动化测试框架来说，具有有很多优势，比如：  

*   跨浏览器，支持 Chromium、Firefox、WebKit
*   跨操作系统，支持 Linux、Mac、Windows
*   可提供录制生成代码功能，解放双手
*   可用于移动端

目前存在的缺点就是生态和文档还不是非常完备，比如没有 API 中文文档、没有较好的教程和示例供学习。不过相信，随着知道的人越来越多，未来会越来越好。  
最后，再说一个小秘密，Playwright 是一个跨语言的自动化框架，除了支持 Python，也支持 Java、JS 等，更加详细的功能可以通过官方项目去解锁!  
如果你觉得文章还不错，请大家 **点赞、分享、留言** 下，因为这将是我持续输出更多优质文章的最强动力！