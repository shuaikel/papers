### 1. 什么是状态管理

从2013年React发布至今已进6个年头，前端框架逐渐形成React、Vue、Angular三足鼎立之势，几年前还在争论单向绑定还是双向绑定，现在三大框架已经不约而同选择单向绑定，双向绑定沦为单纯的语法糖，框架间的差异越来越小，加上Ant-Design/Fusion-Design/NG-ZORRO/ElementUI组件库的成熟，选择任一你熟悉的框架都能高效的完成任务。

> 简单应用使用组件内State方便快捷，但随着应用复杂度上升，会发现数据散落在不同的组件，组件通信会变得异常复杂，我们先后尝试过原生Redux，分形的Fractal的思路、自研类Mbox框架、Angular Service，最终认为Redux依旧是复杂应用数据流处理最佳选项之一。
> 
> 庆幸的是除了React社区，Vue社区有类似的Vuex，Angular社区有NgRX也提供了几乎同样的能力，甚至NgRx还可以无缝使用Redux-devtools来调试状态变化。
> 

![](https://user-gold-cdn.xitu.io/2019/2/11/168dcce274c2fdc8?imageView2/0/w/1280/h/960/ignore-error/1)

无论如何优化，始终要遵循Redux三原则：

原则|方法|引发的问题|
----|---|-------|
Single source of truth| 组件Statless，数据来源于Store| 如何组织Store|
State is read-only| 只能通过触发action来改变State|action 数量膨胀，大量样板代码|
Change are made with pure functions| Reducer 是纯函数| 副作用如何处理，大量样板代码|



```
==============================================================
=                        Request Begin                       =
==============================================================

URL:	http://mall-api-test2.ewgvip.com/wechat/activity-spell-group/my-list.html

Params:		{
  "status" : 0,
  "device" : "IOS",
  "pageSize" : 20,
  "language" : "CN",
  "version" : "2.2.0",
  "page" : 1,
  "access_token" : "a_8d180a0f2c58f53ffa6c81ad492370b6"
}

Result:
	{
  "pageSize" : 20,
  "result" : true,
  "code" : 0,
  "pageCount" : 1,
  "currentCount" : 1,
  "list" : [
    {
      "status" : 1,
      "join_list" : [
        {
          "total_money" : "0.01",
          "order_sn" : "SH190304105027464315247751",
          "user_id" : 152477,
          "nickname" : "shuaike",
          "is_head" : 1,
          "avatar" : "https:\/\/wx.qlogo.cn\/mmopen\/vi_32\/Q0j4TwGTfTLXumYX0XoyNnPib4KQZKQhjq1DUicxX3RCgCCG9u0eibx6cIib3bE97icVfEpJib0Qibu67ls5OnNBVGm7w\/132"
        }
      ],
      "user_id" : 152477,
      "nickname" : "shuaike",
      "market_price" : "0.01",
      "unfinished_people" : 1,
      "create_time" : "2019-03-04 10:50:27",
      "remain_time" : 3090,
      "completed_people" : 1,
      "left_time" : "08:51:30",
      "total_money" : "0.01",
      "goods_title" : "test220",
      "option_name" : "",
      "goods_count" : 1,
      "order_sn" : "SH190304105027464315247751",
      "product_price" : "2.00",
      "active_hour" : 1,
      "avatar" : "https:\/\/wx.qlogo.cn\/mmopen\/vi_32\/Q0j4TwGTfTLXumYX0XoyNnPib4KQZKQhjq1DUicxX3RCgCCG9u0eibx6cIib3bE97icVfEpJib0Qibu67ls5OnNBVGm7w\/132",
      "people" : 2,
      "goods_thumb" : "http:\/\/ewg-cdn.oss-cn-shenzhen.aliyuncs.com\/ewg_group\/upload\/2019\/02\/de8dae713cbaf7612cf62c4dd7680912.jpg",
      "team_id" : 128
    }
  ],
  "page" : "1",
  "red_packet" : {
    "is_red_packet" : false
  },
  "listCount" : 1
}

==============================================================
=                        Request End                         =
==============================================================


```

















