**iOS热更新方案**

>JSPatch:  
>>热更新时，从服务器拉去js脚本。理论上可以修改和新建所有的模块，但是不建议这样做。建议用来做紧急的小需求和 修复严重的线上bug。
>>
>>
>
>lua脚本:
>>比如：wax。热更新时，从服务器拉去lua脚本,游戏开发经常用到。
>
>
>Weex:[weex地址](weex.apache.org/cn/)
>>跨平台，一套代码，iOS、Android都可以运行。用前端语法实现原生效果。比React Native更好用,weex基于vue.js，ReactNative使用React。ReactNative安装配置麻烦。 weex安装cli之后就可以使用。react模板JSX有一定的学习成本，vue和常用的web开发类似，模板是普通的html，
数据绑定用mustache风格，样式直接使用css。
>
>React Native [链接地址](reactnative.cn/)
>>不像Weex能一套代码多端运行，需要自己分别做修改。React Native 可以动态添加业务模块，但无法做到修改原生OC代码。JSPatch、lua 配合React Native可以让一个原生APP时刻处于可扩展可修改的状态。
>
>
>Hybrid
>>像PhoneGap之类的框架, 基本概念和web差不多, 通过更新js/html来实现动态化，没有原生的效果流畅。
>
>
>动态库
>>可以做demo用，真实使用的时候会被苹果禁止。因为 打包发到AppStore的ipa安装包 里的每个动态库 都有唯一的编码，iOS系统会进行验证，
所以动态通过网络获取 新的动态库 也用不了。
>
>
>rollout.io [链接地址](rollout.io/)
>>Rollout紧急修复线上bug。后端有相关的管理页面。因为是国外的网站，然后呢，要FQ才能使用。
>
>
> DynamicCocoa [链接地址](github.com/DynamicCoco)
>>滴滴iOS的一个框架，还没开源已经有1K+star和许多issue了，
与JSPatch比更加智能化，用OC在XCode中写完代码，用工具可以自动生成可以更新的js文件。