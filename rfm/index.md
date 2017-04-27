# RFM客户细分应用场景



  某公司有一份每天的用户订单记录数据，主要记录了用户ID、购买时间、购买金额、购买商品数等字段。最近其市场部需要针对不同价值的用户群来制定相应的营销活动策略，希望能够通过RFM模型来快速锁定出客户的价值，该公司市场部是怎样使用数果智能来操作RFM客户细分的呢？


## 第一步：设置RFM场景数据  
首先，在 “维度管理” 界面下的 点击 “场景数据设置” 按钮，在弹开的界面上，选择“RFM数据设置”选项卡，在对应数据区域里面设置相应的字段。例如这里设置为：  

- 设置所属项目为： RFM_data
- 选择购买日期维度为：购买日期
- 选择购买金额维度为：购买金额
- 选择客户ID维度： 用户ID  

最后点击“提交”按钮，即可完成RFM客户细分模型场景数据设置。

![](/assets/rfm/rfm_6.png)


##  第二步：新建RFM客户细分项目 
在 “场景应用” 界面下的选择“RFM客户细分”选项， 然后点击“新增RFM客户细分”按钮，即可新建一个RFM项目。

![](/assets/rfm/rfm_7.png)

然后在RFM客户细分界面，首先输入相应的RFM名称，例如输入：RFM_1, 因为提前在RFM场景数据设置相应的字段，这部分的字段自动映射到“预设相应的维度”区域，然后在“配置参数”里面默认将最近一次消费相隔的天数（R）、消费频率（F）、累计消费金额(M)进行2分位划分。然后点击“查询”按钮，  
即可对客户的价值智能划分为： 2 \* 2 \* 2 = 8 份  
可以在列表中查看到各个细分客户群的用户数以及各个2分位所处于的数值。最后点击“保存”按钮，即可完成RFM功能设置。

![](/assets/rfm/rfm_8.png)


###  结果解读
从划分的8个RFM细分客户群来看，目前这群用户中，各个用户群的分布如下：  
1. 重要价值客户（0<=R<=50，F>1，M>140）有：702人，占所有用户的 4.9%；
2. 重要保持客户（R>50，F>1，M>140) 有 : 586人,占所有用户的4.09%；
3. 重要发展客户（0<=R<=50，0<=F<=1，M>140) 有 : 3102人,占所有用户的21.65%；
4. 重要挽留客户（R>50，0<=F<=1，M>140) 有 : 2771人,占所有用户的19.34%；
5. 一般价值客户（0<=R<=50，F>1，0<M<=140) 有 : 440人,占所有用户的3.07%；
6. 一般保持客户（R>50，F>1，0<M<=140) 有 :353人,占所有用户的2.46%；
7. 一般发展客户（0<=R<=50，0<=F<=1，0<M<=140) 有 :2989人,占所有用户的20.86%；
8. 一般挽留客户（R>50，0<=F<=1，0<M<=140) 有 :3383人,占所有用户的23.61%；  

市场运营人员即可洞察不同的细分价值客户群的规模以及对应的指标，能够快速帮助企业做出相应的精细化营销策略。


- 二分位解释：例如目前RFM的客户群里面有100个人，对这群用户的消费频率进行2分位划分的时候，需要自动划分为2份，每份各50人，那么前面50人的消费频次切割点为3次，那么此时相当于对F划分为：0<= F <= 3 , F > 3 。而切割点就是3。 其他类似的三分位、四分位都类似这样切割。  


### 自定义RFM分组

如果企业市场运营人员对目前企业的运营指标比较熟悉，能够快速知道RFM的各个分层经验值，我们也提供自定义的RFM分组，能够帮助企业结合自身行业经验进行划分RFM客户群。

![](/assets/rfm/rfm_9.png)


### 另外，我们还可以针对RFM的细分客户群，添加到用户行为追踪里面，帮助企业了解每个客户的轨迹明细。




