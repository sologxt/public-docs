## 流量分析功能使用简介
我的网站/应用每天使用、离开的用户成千上万，我到底要怎么去分析这些数据呢？
举例来说，某电商在双十一前期做了各种渠道的宣传，那么在双十一当天，该如何使用我们的流量分析功能呢？

_注：采集数据源存放在“web端页面埋点”表中。_

### 自定义指标，选择业务关注的流量指标
点击自定义指标，然后在下拉弹出的区域中,选择自定义关注的指标。选择分析的指标：访客数（UV），IP数，浏览量（PV），访问次数（VV），总访问时长，总点击量。点击确定，则可以在页面中看到选择指标的分析情况。
![](/assets/traffic/6.png)
注意，为了能够更舒适地分析指标，我们设置了分析的最大数量为6。那么，您可以根据需求选择1-6个流量指标进行分析。
#### 指标解释
* 访客数（UV）：有多少不同的用户访问了网站/应用。
_影响UV数量的因素：使用场所如网吧、学校、公司等公用相同IP的场所会使UV值变小。_
* IP数：使用不同IP地址访问网站/应用的用户数量。
_影响IP数量的因素：动态的IP地址如ADSL拨号上网会使IP数值变大。_
* 浏览量（PV）：页面的浏览次数，衡量网站/应用用户访问的网页数量。
* 访问次数（VV）：记录所有访客访问了网站/应用的次数。_指访客的访问行为（完成浏览并关掉网站的所有页面）。_
* 总访问时长：记录所有访客访问网站/应用的页面浏览总时长。
* 总点击量：记录所有访客访问网站/应用中进行操作点击的总次数。
* 平均访问时长：平均每次访问的页面浏览时长。_即在每一次的访问中，浏览页面的平均时长_
* 平均浏览页数： 平均每次访问的页面浏览次数。_即在每一次的访问中，浏览页面的平均次数_
* 平均点击量：平均每次访问的操作点击次数。_即在每一次的访问中，进行点击操作的平均次数。_
* 人均访问时长：平均每人的页面浏览时长。
* 人均点击量：平均每人进行点击操作的次数。

### 实时概况，观察当天的数据发生情况
在实时概况中，每分钟更新一次实时概况区域的数据，这时我们可以看到当前在线的用户数量，以及对应指标今天的数据增长，亦可与昨天指标的数据进行对比。同时，在每个小时都会预测一次今日全天的指标数值。

如何预测？当我们的数据量低于三周，则数果智能使用简单移动平均法（前三天同个时刻数值的平均占比）来预测今日全天的指标数值；而当我们的数据量超过三周，数果智能提供更精准的预测方法（前三周同期同时刻数值的平均占比）来预测今日全天的指标数值。

那么预测值有什么作用呢？预测值的大小能够提供一定的建议给运营人员，是否应该要对今日的推广力度及运营的策略进行调整了。
![](/assets/traffic/7.png)
点击展开数据，我们可以参考对比更多的数据