# Vue3新特点（部分）

## 组合式Api与setup

Vue2的全部变量是定义在 `data`的 `return`中的，方法是放在 `methods`中的，生命周期有`mounted`  等等。这是因为Vue2的vue2的全部变量以及方法都靠 `export default`来抛出。

Vue3采用的组合式Api去掉`export default`改用`setup` `import`引入也不需要在 `components`中申明，引用后直接使用即可。

同样也不用在后面 `return`把方法`return`出去。



## 父子传值

### 父传子

用 `defineProps`创建一个变量，里面用对象的形式来定义子组件接收父组件的值

```js
const props = defineProps({
  distribution: {
​    type: Array,
​    default: [],
  },
  userCount: {
​    type: Object,
​    default: {},
  },
});
```



### 子传父

用 `defineEmits`创建一个变量，里面用数组的形式来定义子对象emit的函数，并在后面用 `emit`来向父组件发送事件

```js
const emit = defineEmits(['finish','strat'])
function test(){
    emit('finish')
}
```



## 全局变量

vue2.x挂载全局是使用`Vue.prototype.$xxxx=xxx`的形式来挂载，然后通过`this.$xxx`来获取挂载到全局的变量或者方法
在vue3.x这种方法显然是不行了，vue3中在setup里面我们都获取不到this，官方提供了另一种方法来让我们使用

```js
const app = createApp(App) app.config.globalProperties.$httpUrl = 'https://www.baidu.com' //挂载
```



```js
import { defineComponent, getCurrentInstance, onMounted } from "vue"
onMounted(() => {
    const { appContext : { config: { globalProperties } = getCurrentInstance()
    console.log(globalProperties.$httpUrl) //使用
          })
```



# axios

## axios.create

创建一个axios的实例，参数为配置对象如：

```js
const instance = axios.create({
  timeout: 30000,	//超时时间
  headers: {		//请求头信息
    'Content-Type': 'application/json',
    "accept": 'application/json'
  }
})
```

## interceptors 拦截器

### 请求拦截器

```js
instance.interceptors.request.use(
​    (config) => {		//所有实例请求成功
    
​        // 每次请求之前先从本地缓存获取token并更新
​        config.headers.authorization = window.localStorage.getItem("token")
​        return config;
​    },
​    (error) => {
​        return new Promise.reject(error);
​    }
);
```

### 响应拦截器

```js
instance.interceptors.response.use(response => {//响应成功

  return response
}, async error => {//响应失败
    //error.response.statusa失败的状态码
    
    //做一些逻辑操作处理一些失败的状态码
  if (error.response.status === 422 || error.response.status === 403) {
​    message.error(error.response.data.message)
  } 
  return Promise.reject(error)
})
```

## 封装请求

```js
const http = {
        get(url, data) {
            //返回一个Promise
                return new Promise(function (resolve, reject) {
                        instance
                                .get(url, { params: data })
                                .then((res) => {
                                        resolve(res && res.data);
                                })
                                .catch((e) => {
                                        reject(e);
                                });
                });
        },
        post(url, data) {
                return new Promise(function (resolve, reject) {
                        // console.log(instance);
                        instance
                                .post(url, data)
                                .then((res) => {
                                        resolve(res.data);
                                })
                                .catch((e) => {
                                        reject(e);
                                });
                });
        },
```

## FormData类型

```js
//创建一个formData实例，把数据都通过append加入进去然后作为参数上传
let formData = new FormData();
formData.append("SID", eeo.SID);
formData.append("safeKey", eeo.safeKey);
formData.append("timeStamp", eeo.timeStamp);
http.eeo("cloud.api.php?action=uploadFile", formData).then(res=>{
})
```





# 路由

## 加载方式

### 动态加载

跳转一个路由加载一个路由提升网页响应速度

```js
{
	path: '/',
	component: () => import('../pages/home/index'),//动态加载路由提高页面加载速度
	meta: {
        title: '劳慧云总后台',
        breadcrumb: []
	}
}
```

### import加载

进入网站时一次加载所有路由，首次进入网站速度慢

```js
import { home } from "@/router/home";
{
	path: '/',
	component:home,
	meta: {
        title: '劳慧云总后台',
        breadcrumb: []
	}
}
```

## 路由缓存

针对与Vue3，缓存的路由来回切换并不会重新调用页面的相关渲染dom的生命周期，没有缓存的则会调用页面的setup，onMouted等生命周期

```js
{
	path: '/',
	component: () => import('../pages/home/index'),
	meta: {
        title: '劳慧云总后台',
        breadcrumb: [],
        keepAlive:true   //通过keepAlive字段控制那些路由缓存
	}
}
```

```html
<a-layout-content class="lay_content">
    <router-view v-slot="{ Component }">
        <keep-alive>
            <!-- 如果keepAlive为真放入keep-alive中，否则放入keep-alive外 -->
            <component
                       :is="Component"
                       :key="$route.name"
                       v-if="$route.meta.keepAlive && $route.meta"
                       />
        </keep-alive>
        <component :is="Component" v-if="!$route.meta.keepAlive" />
    </router-view>
</a-layout-content>
```



## 路由模式/守卫

Vue3中先在路由的js文件中把相关的内容引入进去

```js
import { createRouter, createWebHashHistory, createWebHistory } from 'vue-router'
```

### 路由模式

`createRouter`创造一个路由实例，参数为对象用来配置路由信息，如路由模式

```js
const router = createRouter({
        history: createWebHashHistory(),
        routes,
})
```

这里的history可以选择`createWebHashHistory`**哈希路由模式**或者`createWebHistory`**历史路由模式**两个模式的区别主要在于**哈希路由**地址栏上有“***#***”号而**历史路由**没有。

前端选择的这两模式的区别要和后端约定好，否则会出现打包之后资源加载了但是页面空白的情况。



### 路由守卫

路由跳转前后的拦截，主要用于用户跳入非法权限的拦截

```js
router.beforeEach((to, from, next) => {
    //如果路由需要跳转
    // 如果跳转登录界面，判断有无token，有直接进入首页，无放行。
    if (to.path == "/" || to.path == "/forget") {
        if (window.localStorage.getItem("token")) {
            next("/home");
        } else {
            next();
        }
    } else if (to.path == "/home") {
        window.localStorage.setItem("selectedKeys", "/home")
        next();
    } else {
        // 如果跳转非登录界面，判断有无token，有直接进入首页，无放行。
        if (window.localStorage.getItem("token")) {
            next();
        } else {
            next("/");
        }
    }
});
```

`from.path`为当前页面的路由路径，`to.path`为要跳转的页面的路由路径，`next()`为放行，`()`可填想要跳转的路径，也可以不填。



# TypeScript

## TS是什么

TypeScriptJavaScriptJavaScript 的超集用于解决大型项目的代码复杂性一种脚本语言，用于创建动态网页。可以在编译期间发现并纠正错误作为一种解释型语言，只能在运行时发现错误强类型，支持静态和动态类型弱类型，没有静态类型选项最终被编译成 JavaScript 代码，使浏览器可以理解可以直接在浏览器中使用支持模块、泛型和接口不支持模块，泛型或接口支持 ES3，ES4，ES5 和 ES6 等不支持编译其他 ES3，ES4，ES5 或 ES6 功能社区的支持仍在增长，而且还不是很大大量的社区支持以及大量文档和解决问题的支持



## TS的基础类型

### 1. Boolean 类型

```js
let isDone: boolean = false;
// ES5：var isDone = false;
```

### 2. Number 类型

```js
let count: number = 10;
// ES5：var count = 10;
```

### 3. String 类型

```js
let name: string = "Semliker";
// ES5：var name = 'Semlinker';
```

### 4. Array 类型

```js
let list: number[] = [1, 2, 3];
// ES5：var list = [1,2,3];

let list: Array<number> = [1, 2, 3]; // Array<number>泛型语法
// ES5：var list = [1,2,3];
```

### 5.  Enum 类型

使用枚举我们可以定义一些带名字的常量。 使用枚举可以清晰地表达意图或创建一组有区别的用例。 TypeScript 支持数字的和基于字符串的枚举。

**1.数字枚举**

```js
enum Direction {
  NORTH,
  SOUTH,
  EAST,
  WEST,
}

let dir: Direction = Direction.NORTH;
```

默认情况下，NORTH 的初始值为 0，其余的成员会从 1 开始自动增长。换句话说，Direction.SOUTH 的值为 1，Direction.EAST 的值为 2，Direction.WEST 的值为 3。上面的枚举示例代码经过编译后会生成以下代码：

```js
"use strict";
var Direction;
(function (Direction) {
  Direction[(Direction["NORTH"] = 0)] = "NORTH";
  Direction[(Direction["SOUTH"] = 1)] = "SOUTH";
  Direction[(Direction["EAST"] = 2)] = "EAST";
  Direction[(Direction["WEST"] = 3)] = "WEST";
})(Direction || (Direction = {}));
var dir = Direction.NORTH;
```

当然我们也可以设置 NORTH 的初始值，比如：

```js
enum Direction {
  NORTH = 3,
  SOUTH,
  EAST,
  WEST,
}
```

**2.字符串枚举**

在 TypeScript 2.4 版本，允许我们使用字符串枚举。在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化。

```js
enum Direction {
  NORTH = "NORTH",
  SOUTH = "SOUTH",
  EAST = "EAST",
  WEST = "WEST",
}
```

**3.异构枚举**

异构枚举的成员值是数字和字符串的混合：

```js
enum Enum {
  A,
  B,
  C = "C",
  D = "D",
  E = 8,
  F,
}
```

### 6. Any 类型

在 TypeScript 中，任何类型都可以被归为 any 类型。这让 any 类型成为了类型系统的顶级类型（也被称作全局超级类型）。

```js
let notSure: any = 666;
notSure = "Semlinker";
notSure = false;
```

any 类型本质上是类型系统的一个逃逸舱。作为开发者，这给了我们很大的自由：TypeScript 允许我们对 any 类型的值执行任何操作，而无需事先执行任何形式的检查。比如：

```js
let value: any;

value.foo.bar; // OK
value.trim(); // OK
value(); // OK
new value(); // OK
value[0][1]; // OK
```

在许多场景下，这太宽松了。使用 any 类型，可以很容易地编写类型正确但在运行时有问题的代码。如果我们使用 any 类型，就无法使用 TypeScript 提供的大量的保护机制。为了解决 any 带来的问题，TypeScript 3.0 引入了 unknown 类型。

### 7.  Unknown 类型

就像所有类型都可以赋值给 any，所有类型也都可以赋值给 unknown。这使得 unknown 成为 TypeScript 类型系统的另一种顶级类型（另一种是any）。下面我们来看一下 unknown 类型的使用示例：

```js
let value: unknown;

value = true; // OK
value = 42; // OK
value = "Hello World"; // OK
value = []; // OK
value = {}; // OK
value = Math.random; // OK
value = null; // OK
value = undefined; // OK
value = new TypeError(); // OK
value = Symbol("type"); // OK
```

对 value 变量的所有赋值都被认为是类型正确的。但是，当我们尝试将类型为 unknown 的值赋值给其他类型的变量时会发生什么？

```js
let value: unknown;

let value1: unknown = value; // OK
let value2: any = value; // OK
let value3: boolean = value; // Error
let value4: number = value; // Error
let value5: string = value; // Error
let value6: object = value; // Error
let value7: any[] = value; // Error
let value8: Function = value; // Error
```

unknown 类型只能被赋值给 any 类型和 unknown 类型本身。直观地说，这是有道理的：只有能够保存任意类型值的容器才能保存 unknown类型的值。毕竟我们不知道变量 value 中存储了什么类型的值。

现在让我们看看当我们尝试对类型为 unknown 的值执行操作时会发生什么。以下是我们在之前 any 章节看过的相同操作：

```js
let value: unknown;

value.foo.bar; // Error
value.trim(); // Error
value(); // Error
new value(); // Error
value[0][1]; // Error
```

将 value 变量类型设置为 unknown 后，这些操作都不再被认为是类型正确的。通过将 any 类型改变为 unknown 类型，我们已将允许所有更改的默认设置，更改为禁止任何更改。

### 8.  Tuple 类型

众所周知，数组一般由同种类型的值组成，但有时我们需要在单个变量中存储不同类型的值，这时候我们就可以使用元组。在 JavaScript 中是没有元组的，元组是 TypeScript 中特有的类型，其工作方式类似于数组。

元组可用于定义具有有限数量的未命名属性的类型。每个属性都有一个关联的类型。使用元组时，必须提供每个属性的值。为了更直观地理解元组的概念，我们来看一个具体的例子：

```js
let tupleType: [string, boolean];
tupleType = ["Semlinker", true];
```

在上面代码中，我们定义了一个名为 tupleType 的变量，它的类型是一个类型数组 [string, boolean]，然后我们按照正确的类型依次初始化 tupleType 变量。与数组一样，我们可以通过下标来访问元组中的元素：

```js
console.log(tupleType[0]); // Semlinker
console.log(tupleType[1]); // true
```

在元组初始化的时候，如果出现类型不匹配的话，比如：

```js
tupleType = [true, "Semlinker"];
```

此时，TypeScript 编译器会提示以下错误信息：

```js
[0]: Type 'true' is not assignable to type 'string'.
[1]: Type 'string' is not assignable to type 'boolean'.
```

很明显是因为类型不匹配导致的。在元组初始化的时候，我们还必须提供每个属性的值，不然也会出现错误，比如：

```js
tupleType = ["Semlinker"];
```

此时，TypeScript 编译器会提示以下错误信息：

```js
Property '1' is missing in type '[string]' but required in type '[string, boolean]'.
```

### 9.  Void 类型

某种程度上来说，void 类型像是与 any 类型相反，它表示没有任何类型。当一个函数没有返回值时，你通常会见到其返回值类型是 void：

```js
// 声明函数返回值为void
function warnUser(): void {
  console.log("This is my warning message");
}
```

以上代码编译生成的 ES5 代码如下：

```js
"use strict";
function warnUser() {
  console.log("This is my warning message");
}
```

需要注意的是，声明一个 void 类型的变量没有什么作用，因为它的值只能为 undefined 或 null：

```js
let unusable: void = undefined;
```

### 10.  Null 和 Undefined 类型

TypeScript 里，undefined 和 null 两者有各自的类型分别为 undefined 和 null。

```js
let u: undefined = undefined;
let n: null = null;
```

默认情况下 null 和 undefined 是所有类型的子类型。 就是说你可以把null 和 undefined 赋值给 number 类型的变量。**然而，如果你指定了--strictNullChecks 标记，null 和 undefined 只能赋值给 void 和它们各自的类型。**

### 11.  Never 类型

never 类型表示的是那些永不存在的值的类型。 例如，never 类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型。

```js
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

在 TypeScript 中，可以利用 never 类型的特性来实现全面性检查，具体示例如下：

```js
type Foo = string | number;

function controlFlowAnalysisWithNever(foo: Foo) {
  if (typeof foo === "string") {
    // 这里 foo 被收窄为 string 类型
  } else if (typeof foo === "number") {
    // 这里 foo 被收窄为 number 类型
  } else {
    // foo 在这里是 never
    const check: never = foo;
  }
}
```

## TypeScript 断言

有时候你会遇到这样的情况，你会比 TypeScript 更了解某个值的详细信息。通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。类型断言好比其他语言里的类型转换，但是不进行特殊的数据检查和结构。它没有运行时的影响，只是在编译阶段起作用。

类型断言有两种形式：

### “尖括号” 语法

```js
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

### as 语法

```js
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

