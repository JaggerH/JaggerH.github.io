---
id: 47cce2cd-974f-49a3-a525-fe7e92c9e875
title: Operate in Bull Market
created_time: 2024-04-01T11:42:00.000Z
last_edited_time: 2024-04-01T14:43:00.000Z
tags:
  - Stock
date: '2024-04-01'
status: Ready

---

我觉得比较符合近期的市场

*   早上大涨要卖；

*   早上大跌要买；

*   下午大涨不追；

*   下午大跌次日买；

可以对近一个星期的数据做回测

我当前觉得市场的问题是，我当前无法判断市场的热钱效应，无法判断市场的打板成功率

就我个人的感觉而言，我觉得市场板块连续性比较差，因为我最近的3次打板，只成功了一次，这种情况下我个人觉得很难获得丰厚回报，甚至很难脱离成本区间，我觉得这可能和近期市场的波动规律有一定关系，因为显然没有进入主升浪，市场参与者也是持续在做轮动的（这中间应该是存在一个笛卡尔集的），在这种情况下的交易规则，我觉得比较适合以上的做法，就是震荡规则。

基于此我需要先检索震荡市的常见策略，第二是看能不能跟打板做一定的结合，因为前期涨停的票，波动率高，我觉得期望应该更好。

收敛我的目标

1、从第一次进入板块日涨幅TOP10，会延续多久

2、多久后开始下跌，下跌幅度和盘整时间到下次再次上涨需要

会延续多久到多久开始下跌是一个问题

所以我该统计的是，从第一次进入日涨幅TOP10之后，板块

很复杂，没有想清楚

重新设立目标

1、首先是排除掉涨幅前100的股票，因为大概率是涨停的，取100-120的股票做测试，收盘买入；

2、假设第二天开始作为买入标的；

    - 早上大涨要卖；

    - 早上大跌要买；

    - 下午大涨不追；

    - 下午大跌次日买；

    回测开始，首日开盘等权重买入股票（同时将等权重的票池作为benchmark），个票波动3%作为大涨或大跌的标准，上午涨幅超过3%卖出，下午跌幅超过3%次日买入。