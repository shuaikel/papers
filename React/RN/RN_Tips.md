+ 在static中使用this方法

> 首先需要在componentDidMount(){}方法中动态的添加点击事件

```
属性给params
componentDidMount(){

    this.props.navigation.setParams({
        title:'自定义Header',
        navigatePress:this.navigatePress
    })
}

<!--实现这个函数-->
navigatePress = () => {
    alert('点击headerRight');
    console.log(this.props.navigation);
}

接下来我们就可以通过params方法来获取点击事件了

static navigationOptions = ({ navigation, screenProps }) => ({ 
title: navigation.state.params?navigation.state.params.title:null, 
headerRight:( 
<Text onPress={navigation.state.params?navigation.state.params.navigatePress:null}> 返回 </Text> ) });

```

+ 让安卓实现Push动画

```
// 先引入这个方法 import CardStackStyleInterpolator from 'react-navigation/src/views/CardStackStyleInterpolator'; 
// 在StackNavigator配置headerMode的地方，使用transitionConfig添加 
{ headerMode: 'screen', 
transitionConfig:()=>({ screenInterpolator:CardStackStyleInterpolator.forHorizontal, }) }

```
+ 关于goBack返回指定页面
> 
> react-navigation是提供了goBack()到指定页面的方法的，那就是在goBack()中添加一个参数，但当你使用goBack('Main')的时候，你会发现并没有跳转，原因是react-navigation默认goBack()中的参数是系统随机分配的key，而不是手动设置的routeName，而方法内部又没有提供可以获得key的方法，所以这里只能通过修改源码将key换成routeName了。

```
把项目
/node_modules/react-navigation/src/routers/StackRouter.js
文件里的 
const backRoute = state.routes.find((route: *) => route.key === action.key); 
改成 
const backRoute = state.routes.find(route => route.routeName === action.key); 
但不是很完美, 这里的component要填想返回的组件的前一个组件的routeName, 比如你的栈里顺序是home1, home2, home3, home4, 在home4里要返回home2, 使用this.props.navigation.goBack('home3');; 并且又会带出一个问题: goBack()方法没反应了, 必须加个null进去, 写成goBack(null)...

关于goBack返回指定页面的修改完善版

if (action.type === NavigationActions.BACK) { 
let backRouteIndex = null; 
if (action.key) { const backRoute = state.routes.find(
 /* $FlowFixMe */ /* 修改源码 */ 
 route => route.routeName === action.key
  /* (route: *) => route.key === action.key */ ); 
  /* $FlowFixMe */ 
console.log('backRoute =====',backRoute); backRouteIndex = state.routes.indexOf(backRoute); console.log('backRoute =====',backRouteIndex); } if (backRouteIndex == null) { 
return StateUtils.pop(state); } if (backRouteIndex >= 0) { 
return { ...state, routes: state.routes.slice(0, backRouteIndex+1), index: backRouteIndex - 1 + 1, }; } }

将源码改成上面的样子，就可以使用goBack()返回指定页面了，这样的优点不言而喻，但缺点就是每次调用goBack()，如果只是简单的返回上一页需要加上null参数，类似这样goBack(null)，

```
+ 关于快速点击会导致多次跳转的问题解决
[链接](https://github.com/react-navigation/react-navigation/pull/1348/files)

```
修改react-navigation目录下，
scr文件夹中的addNavigationHelpers.js文件，
可以直接替换成下面的文本，也可以查看原版链接

export default function<S: *>(navigation: NavigationProp<S, NavigationAction>) { 
// 添加点击判断 let debounce = true; 
return { 
...navigation, 
goBack: (key?: ?string): boolean => navigation.dispatch( NavigationActions.back({ key: key === undefined ? navigation.state.key : key, }), ), 
navigate: (routeName: string, params?: NavigationParams, action?: NavigationAction,): boolean => { if (debounce) { debounce = false; navigation.dispatch( NavigationActions.navigate({ routeName, params, action, }), ); setTimeout( () => { debounce = true; }, 500, ); return true; } return false; }, /** * For updating current route params. For example the nav bar title and * buttons are based on the route params. * This means `setParams` can be used to update nav bar for example. */ setParams: (params: NavigationParams): boolean => navigation.dispatch( NavigationActions.setParams({ params, key: navigation.state.key, }), ), }; }

```
+ 安卓与iOSTabbar 保持统一在底部的方法：

```
import TabBarBottom from 'react-navigation/src/views/TabView/TabBarBottom';

TabNavigator ===》加

tabBarComponent: TabBarBottom
```

+ 对于外层是一个StackNavigation，StackNavigation里面有一个TabNavigation和组件2，TabNavigation里面有一个TabNavigator和组件2，TabNavigator里面有a，b，c，d四个tab，从组件2执行navigate进入TabNavigator，然后点击tab-d跳转到d，在d页面执行goBack（）会返回到a，当需要从d直接返回到组件2时，可以定义TabNavigator时，添加backBehavior:{}

+ headerTitleStyle：安卓上如果要设置文字居中，只要添加alignSelf:'center'。如果header有headerRight或headerLeft，headerTitle只是在header剩余部分居中，不是在header居中，解决办法是在左边或者右边也添加一个按钮。

+ 顶部的navigation下面那条线隐藏:iOS下隐藏navigation底部线条用shadowOpacity: 0，安卓上用elevation: 0，放到headerStyle里面即可

+ 安卓上，使用TextInput的时候会让TabBar顶起来的解决办法：
> 最简单的解决办法就是在android目录中，添加一句话
> 目录：android/app/src/main/AndroidManifest.xml中，添加

```
android:windowSoftInputMode="stateAlwaysHidden|adjustPan|adjustResize"
```

+ 修改页面的跳转动画

> 
> react-navigation一共提供了4种跳转动画：
> 	
   	1、从右向左： forHorizontal；
	2、从下向上： forVertical；
	3、安卓那种的从下向上： forFadeFromBottomAndroid；
	4、无动画： forInitial。
	
+ 安卓返回键在react-navigation中的正常监听

```
componentWillMount() { 
if (Platform.OS === 'android') { 
BackHandler.addEventListener('handwareBackPress',this.onBackAndroid) } } 

componentWillUnmount() { 
if (Platform.OS === 'android') { BackHandler.addEventListener('handwareBackPress',this.onBackAndroid) } } onBackAndroid = () => { const routers = nav.getCurrentRoutes(); if (routers.length > 1) { return true; } return false; }; …… } // 在跳转之后的页面中 onBackAndroid = ()=> { const {routes} = this.props; console.log(routes); // alert(routes) if (routes.length > 1) { // 因为其他页面获得不到this.props，所以只能每个页面都写这个方法。 this.props.navigation.goBack(); return true; } }

```

+ react-native 图标组件

> [react-native-chart ](https://github.com/tomauty/react-native-chart)
> 

+ react-native 倒计时组件

> [react-native-CountDowntimer](https://github.com/jackuhan/react-native-CountDowntimer)
> 


+ 文件上传

> [ReactNative-FileUpload](https://github.com/HAPENLY/ReactNative-FileUpload)
> 



