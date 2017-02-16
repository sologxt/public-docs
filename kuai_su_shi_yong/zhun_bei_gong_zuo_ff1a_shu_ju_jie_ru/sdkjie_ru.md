### SDK接入 {#sdk}

SDK接入主要是应用在做可视化埋点上，主要是采集web、Android和IOS的埋点数据，并且收集埋点数据和相应的维度指标数据，用于做诸如用户路径等分析。

请您邀请工程师接入一段 SDK 代码，作为接下来数据采集和分析的准备：WEB SDK使用文档／Android SDK使用文档／IOS SDK使用文档。

在接入 SDK 前，选择创建项目还是应用时，需要注意的是：

如果需要统一管理不同平台 web／iOS／Android 的产品，可以作为不同应用，放在同一个项目下。 同一项目下，如果某个平台 web／iOS／Android 有多个应用，在概览里的数据将会合并显示。

如果两个项目完全没有分析的交集，可以建立两个单独场景项目。

注意：项目下的应用如果想要进行合并和迁移，历史数据是无法迁移的。