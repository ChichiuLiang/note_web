> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wwd0501/article/details/78657319)

**IBM®HeapAnalyzer**

创建者：Jinwoo Hwang（[JinwooHwang.com](http://jinwoohwang.com/)）

欢迎来到 IBM HeapAnalyzer。IBM HeapAnalyzer 允许您使用其获得专利的启发式搜索引擎查找可能的 Java™堆泄漏，并分析 Java 堆转储。

**介绍**

Heapdump 包含堆中所有对象的列表。此工具分析 Java SDK 1.3.1,1.4.x，5.0，6.0 和 7.0 的堆转储

IBM HeapAnalyzer“按原样” 提供。

**条件**

· Java 运行时环境 7 或更高（IBM Java 运行时环境 7 或 IBM Java 系统转储分析更高）  
  
  
  
如果使用 Java 虚拟机的旧版本下面将引发异常：  
  
在线程异常 “主” java.lang.NoClassDefFoundError：的 java / UTIL / 正则表达式 / PatternSyntaxException

· 用于大型 Java 堆转储的 64 位 Java 运行时环境，如果需要超过 4GB 的内存

**定义**

· **根对象**一个没有（不同）对象持有引用的对象。

· **父对象**的对象（例如，A），其保持至少一个参考一些 （不同的）对象（例如，B）。在这种情况下，A 被认为是 B 的父亲。   
  

· **所有者对象**如果一个对象有多个父对象，则选择一个父对象作为所有者对象。 总大小仅用所有者对象计算。   
  

· **子对象**至少有一个（不同的）对象 （例如 A）拥有引用的对象（例如 B）。在这种情况下，B 被认为是 A 的孩子。   
  

· **键入**相同对象的集合

· **大小**对象的大小是在内存中保存该对象所需的内存量。

· **总大小**一个对象的子树大小是它的大小和 从其子节点到达的所有对象大小的总和。请注意，每个 对象在处理期间都被分配了一个唯一的父代和根。 如果 家长和孩子的总体积存在显着差异，则称为总体规模下降。   

· **泄漏大小**泄漏嫌疑人的总大小。如果泄漏嫌疑人包含另一名泄漏嫌疑人，泄漏大小将被嵌套泄漏嫌疑人的总大小减去

**特征**

· 从表视图和树视图的父视图和子视图

· 地址搜索

· 根列表视图

· 同类型查看

· 配置恢复并保存

· IBM，Solaris® 和 HP-UX®Java ascii / 二进制 Java 堆转储支持

· IBM Java 系统转储支持：IBM 的 Java 7 或更高版本需要处理的最新 IBM Java 系统转储文件格式

· 外观和感觉选项

· 自动检测截断的堆转储

· Java Profile v1.0.1 ascii 和二进制 Java 堆转储支持

· Solaris hprof 1.0.1 二进制 Java 堆转储支持

· heapdump 文件命令行支持。请参阅 “如何运行此工具” 部分

· 支持 IBM 便携式堆转储. phd 格式以及 IBM 堆转储. txt 和. txt.gz

· 自动检测文件格式

· 支持控制台窗口

· 从 heapdump 创建一棵树

· 计算每个对象的大小

· 计算每个子树的总大小

· 查找子树中的总大小下降

· 按大小显示对象之间的差距

· 按大小显示对象

· 按总大小显示对象

· 通过孩子的数量显示对象

· 按大小显示类型

· 按计数 / 频率显示类型

· 显示类型的字母顺序

· 显示差距分布图

· 显示对象的详细信息

· 用正则表达式查找类型

· 在输入字段，树和文本中拖放支持

· 树形导航中的书签

· 找到 可能的泄漏嫌疑人

图标

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/ab18a02d-f155-458e-a011-fd233d56a552/media/class.gif)类  
  
![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/aa4c511f-e7e7-4f8b-8d13-c1970e8e37d8/media/object.gif)实例  
  
![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/357a3970-801a-4f84-9382-4ebcbb1307d8/media/array.gif)数组

**如何运行这个工具**

您需要使用 Java 2 Platform Standard Edition 6 或更高版本的 Java 运行时环境（JRE）来运行此工具。

用法 <Java 路径> java -Xmx [heapsize] -jar ha <HeapAnalyzer 版本 > .jar <heapdump 文件 >

例如，/ usr / java60 / bin / java -Xmx1000m -jar ha146.jar heapdump1234.txt

如果在处理堆转储时存在 java.lang.OutOfMemoryError，请尝试增加最大堆大小（-Xmx）值，以便 JVM 获得更多内存。如果你是从 Windows PC 运行的，这里是一个批处理（.bat）文件（由 James Stroud 编写），可以帮助你运行这个文件，帮助你在你的 PC 上选择特定版本的 Java。这是为了您的 PC 上安装了多个版本的 Java 的情况下，典型情况下。请注意，您需要修改 bat 文件中的目录路径，作为 bat 文件中的注释说明。[下载批处理文件](https://www.ibm.com/developerworks/mydeveloperworks/files/form/anonymous/api/library/9572e0a3-e664-4175-86aa-0996f37c2a13/document/d9563a7d-569a-4c1a-b713-8dce1d1f6c6e/media/run-ibm-heap.bat)

由于性能问题，最大堆大小不应大于此工具可用物理内存大小的大小。

1. 启动 HeapAnalyzer

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/592cc6f1-7863-410e-80ce-ff1676d57757/media/image001.png)

[初始画面]

2. 选择文件 - > 打开并选择一个堆转储文件

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/087274d0-347a-4be4-9966-8d67abd529e0/media/image002.jpg)

选择 IBM 堆转储文本文件，IBM 便携式堆转储，Java Profile V1.0.1 ascii / binary Heap [Dump](https://so.csdn.net/so/search?q=Dump&spm=1001.2101.3001.7020) 或 HeapAnalyzer 处理文件以处理堆转储。

3. 在处理堆转储期间显示进度。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/library/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/document/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/7ae6c515-e86c-43a6-9bcc-2e8e60cf7681/media/image003.png)

4. 如果您正在处理大型堆转储，则需要很长时间。以下是处理完成时的屏幕。请不要关闭这个窗口，直到你不需要这个 heapdump。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/ad3cf518-324a-4014-b947-93c296620872/media/image004.png)

内存泄漏嫌疑人在树视图的 “传票泄漏嫌疑人” 菜单下编译。

如果没有内存泄漏嫌疑，树视图默认情况下不显示。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/ff6ab14a-d194-4fdc-85e9-08fec7656008/media/image005.jpg)

点击 Analysis 菜单并选择 Tree view 菜单项来显示堆树。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/0afcb470-b296-481a-9e8c-123b49ca5dcf/media/image006.jpg)

5. 以下是 heapdump 的树视图。

图标，![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)表示它已经被包含在树形视图中作为所有者对象的子对象

每个树节点的格式如下所示：

TotalSize（TotalSize / HeapSize％）[ObjectSize] NumberOfChildObject（根对象数）Name 地址

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/ed2b22ea-8e5d-48c7-867c-71801086682a/media/image007.jpg)

6. 在树形视图中，您可以看到节点的详细信息，您可以搜索父节点和子节点之间的总大小降落

或者您可以通过选择一个节点并单击鼠标右键来查找地址。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/136993a4-1e5d-44ca-a842-05d38e5b02fa/media/image008.jpg)

“搜索总大小下降” 会发现父母的总大小之间的大小下降

和父母孩子的最大总规模。

如果从 “搜索总大小下降” 菜单中找不到任何大小的下拉菜单，则需要减少

选项中搜索的最小总大小下降。

您可以通过选择菜单 “查找地址” 在树形视图中查找地址

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/0b34744a-bb5b-4e68-ac04-4ae0c463ff8e/media/image009.jpg)

以下是地址搜索的结果：

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/0a4832ac-d38d-4753-b106-584ebde86883/media/image010.jpg)

7. 如果要查看关于节点的详细信息，只需单击，您将在右窗格中获得有关信息。或者您可以选择以下菜单。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/5b428117-2131-4f75-ac5a-ebc2c2ffb4d0/media/image011.jpg)

以下是 heapdump 树中详细节点信息的屏幕

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/7a6c6e1d-5103-4d18-9b83-227e2e015c20/media/image012.jpg)

8. 您可以通过选择 “添加书签” 来放置书签来保存位置并继续导航树。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/eb67fefc-5e21-4c6c-8ff3-febac5c20812/media/image013.jpg)

书签存储在 “树状视图” 菜单栏的 “转到书签” 菜单中。

你可以通过点击书签去树中保存的位置

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

通过点击 “删除书签” 菜单中的书签可以删除书签。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/92688a43-2b66-4d9f-812c-0092215a1b86/media/image015.jpg)

9. 如果树上有很多孩子，可以在树的底部看到 “NNNN more children”。

通过展开节点可以看到更多的孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/d5e8702d-1619-45fb-8026-b24ca69000eb/media/image016.jpg)

您也可以使用父节点中的 “显示更多的孩子” 菜单。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/d302e785-d7f9-4e2a-83df-16c907aa80c1/media/image017.jpg)

现在你可以看到更多的孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/72e076cc-7945-4bbb-8657-4bf3e2c85300/media/image018.jpg)

10. 您可以使用 “查找泄漏嫌疑人” 菜单找到可能的泄漏区域。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/e32c22e1-8160-418c-a3ec-2b87a7d05cdc/media/image019.jpg)

怀疑泄漏区域显示如下。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/607c0ffa-eb86-40dc-9af4-64a4253752a7/media/image020.jpg)

图标 “，![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)” 表示它不是根对象 / 类，并且此节点有父项。

如果你想显示更多的父母，你可以选择菜单 “显示更多的父母”

如果你想从根对象显示，你可以选择菜单 “显示从根”

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/6a604a6a-3d1a-4382-a988-4cdd9a0523f3/media/image021.jpg)

如果要查看所有名称相同的对象，请从菜单中选择 “列出相同类型”：

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/e79602a5-330d-481d-ad95-1210f2076d0d/media/image022.jpg)

以下是查询的结果：

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/61ded263-0166-4a80-baf9-8e19ff27c72d/media/image023.jpg)

通过点击鼠标右键，你可以在树形视图中找到一个特定的对象，你可以找到一个特定对象的父母或孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/25cad41f-6782-430a-89a1-7e8bbbb75543/media/image024.jpg)

11. 显示按总尺寸排序的对象。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

对象按总大小进行显示和排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/d478af67-9a49-49dd-a346-3a8d5e5f5a34/media/image026.jpg)

通过点击鼠标右键，你可以在树形视图中找到一个特定的对象，你可以找到一个特定对象的父母和孩子。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

如果您单击 TotalSize 的列标题，该列将按相反顺序排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/016c6749-ac9f-4607-9afe-b4ca5def501a/media/image028.jpg)

12. 列出按大小排序的对象

对象按大小排序，方法是单击 “大小” 列标题。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/83a23cd0-21fc-4c27-957b-353f3a383c2e/media/image029.jpg)

13. 列出按子项数排序的对象

对象通过点击 “否” 子列标题按子项数进行排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/e70066b9-b2cb-44ae-bd23-f5be9b20ded4/media/image030.jpg)

点击地址栏标题按地址顺序排序对象。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/906f56ba-0db7-4d18-8198-a882744d7c0b/media/image031.jpg)

单击对象列标题按字母顺序对对象进行排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/bf7d3870-ad5f-4fdc-aac2-e1e525792ff1/media/image032.jpg)

您也可以将每列移动到不同的位置。Address 列在下面的屏幕中移到了 TotalSize 列的旁边。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/d689958b-148f-4d12-9d5b-a050689be576/media/image033.jpg)

14. 按大小排序的列表类型

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/e44a4282-1390-4855-9f78-8b4d1bc45ba3/media/image034.jpg)

类型按大小的总和排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/2822d63b-cb31-4da0-8fcf-66c83c596c5c/media/image035.jpg)

通过单击鼠标右键，可以找到具有特定对象的相同名称的对象。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/85d82019-f080-4007-a310-abaabb927b6e/media/image036.jpg)

15. 按频率 / 计数排序的列表类型

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/fddee721-a6bb-4dc5-883a-9ba9219880ff/media/image037.jpg)

16. 按字母顺序排列的列表类型

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/72eef376-5672-4461-84f9-376bfab69a03/media/image038.jpg)

选择根目录来显示根对象。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/9911ba5a-f583-4294-86a1-32e5f645059f/media/image039.jpg)

所有的根对象显示如下：

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/73df6370-8f98-408c-a149-eb63c2af068b/media/image040.jpg)

通过点击鼠标右键，你可以在树形视图中找到一个特定的对象，你可以找到一个特定对象的相同类型，父母和孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/d8147590-d32a-4b44-bddb-000aaa98eaf6/media/image041.jpg)

16. 列出按大小排序的对象之间的间隔

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/99d376cc-e602-4257-a076-7acbee09f2a3/media/image042.jpg)

以下是按大小查看对象之间的差距

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/7fb80cbc-f8fd-4c8d-a8ff-93a6d0af4d59/media/image043.jpg)

17. 显示差距统计。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/45ac0ded-419d-4fa5-8367-3080d9e2cc31/media/image044.jpg)

差距分布图。不要认为差距是自由空间。差距实际上是两个对象之间的差距。Java 堆转储没有任何有关它们的信息。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/a7df71d1-c173-481d-9937-9c0a05c2dce6/media/image045.jpg)

18. 您可以通过选择查看 - > 选项菜单来配置设置

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/1df9c957-f695-4d03-b570-e1f557c37354/media/image046.jpg)

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/f3d48004-c224-4fff-9fd2-74e82a2793d3/media/image047.png)

19. 搜索对象 / 类型并按总大小进行排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/08f4e221-7d51-4d01-9902-121bc45c46bf/media/image048.jpg)

在正则表达式中输入对象 / 类型名称。大写 / 小写事项！

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/e67f6501-dd60-4330-aee3-53a74f2d5f71/media/image049.jpg)

以下是名称中包含 “字节” 的类型列表。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

20. 搜索对象 / 类型并按大小排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/94780ada-e362-46b1-bc0a-fed82e61403d/media/image051.jpg)

以下是名称中有 “字节” 的类型列表，按照子项的数量排序。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/6a1d04c8-e67b-4639-9226-e22f07f4bd11/media/image052.jpg)

通过点击鼠标右键，你可以在树形视图中找到一个特定的对象，你可以找到一个特定对象的相同类型，父母和孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/731d9ea2-bedd-46f8-9d7f-2e4f55ce84d3/media/image053.jpg)

通过输入确切的名称，我们可以查看有关特定类型的详细信息。

例如，您可以输入一个类型的确切名称：byte []

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/aefbdf2c-19c4-4da9-829f-0074102a01e6/media/image054.jpg)

以下是 byte [] 类型的列表以及有关 byte [] 的详细信息

列标题包含更多信息。

TotalSize 的总和是 117,712 字节。总和的大小是 117,712 字节。

有 140 个字节的对象 []

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/8b5cc38f-cab7-4a98-beca-3168a4913f33/media/image055.jpg)

您可以通过选择搜索地址找到一个特定的地址

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/c2f9c378-1cff-4e76-8326-4dcc1eddf9b9/media/image056.jpg)

输入一个对象的地址

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/b04278b9-90d8-4524-8cd2-4e232e036c71/media/image057.jpg)

以下是地址搜索的结果

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

通过点击鼠标右键，你可以在树形视图中找到一个特定的对象，你可以找到一个特定对象的相同类型，父母和孩子。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/40c04c1f-4157-4ffd-8a35-4231fd047f52/media/image059.jpg)

22. 你可以隐藏 / 显示状态栏。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/df3065f6-d8f6-411a-8cdf-0ed07c55dcb8/media/image060.jpg)

23. 你可以隐藏 / 显示控制台。

![](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/ae0bef3f-d332-4fbb-9b48-b10a2fb29f72/page/4c1e6ecc-9441-4b34-b863-f0c46de196f0/attachment/96b6e8e7-14f8-47d9-9955-2692088e1789/media/image061.jpg)

**来自 HeapAnalyzer 的异常 / 错误**

· OutOfMemoryError   
  
  
  
将 - Xmx 增加到您的环境的最大可用大小。如果您在 32 位 Java 虚拟机上达到 32 位地址空间限制，则建议对大型 Java 堆转储使用 64 位 Java 虚拟机。在大多数情况下，对于 Windows® 操作系统上的 32 位 Java 虚拟机上的 - Xmx，大小约为 1500MB。例如，您可能希望将 64 位 Java 虚拟机用于文件大小为 500MB 的 PHD Java 堆转储。

· 线程 “main” 中的异常 java.lang.NoClassDefFoundError：java / util / regex / PatternSyntaxException   
  
  
  
HeapAnalyzer 需要 Java 虚拟机版本 5 或更高版本。  
  
如果使用较早版本的 Java 虚拟机，则会引发异常：

· java.lang.StringIndexOutOfBoundsException：字符串索引超出范围：0   
  
在 java.lang.String.charAt（未知源）  
  
在 com.ibm.jinwoo.heap.FileTask $ ActualTask​​。<init>（FileTask.java:386）  
  
at com.ibm.jinwoo.heap.FileTask $ 1.construct（FileTask.java:794）  
  
at com.ibm.jinwoo.heap.SwingWorker $ 2.run（SwingWorker.java:45）  
  
at java.lang.Thread.run（Unknown Source ）

处理损坏的 heapdump 或截断的内容时，您可以看到此异常。截断或损坏的 heapdump 不可靠。

· 异常在解析线 9：0x0x50003070 [1000] 的 java / 郎 / 字符串  
  
了 java.lang.RuntimeException   
  
在 com.ibm.jinwoo.heap.FileTask $ ActualTask <初始化>（FileTask.java:321）。  
  
在 com.ibm.jinwoo .heap.FileTask $ 1.construct（FileTask.java:794）  
  
at com.ibm.jinwoo.heap.SwingWorker $ 2.run（SwingWorker.java:45）  
  
at java.lang.Thread.run（Unknown Source）  
  
  
  
一些旧的 IBM SDK 在堆转储中生成无效的地址。用 0x 替换 0x0x 后，HeapAnalyzer 可以处理堆转储。

· 解析第 10 行时出现格式错误：0x50004050 0x50004050 heapdump 中的  
  
  
  
意外格式。可能是已损坏的 heapdump。进一步的分析是不可靠的。

文章来源：[https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/W3b463571efc8_4f02_99af_3cbc0da42ddc/page/IBM%20HeapAnalyzer%20Information](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/W3b463571efc8_4f02_99af_3cbc0da42ddc/page/IBM%20HeapAnalyzer%20Information)