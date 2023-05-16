# Vue3新特点（部分）

## 组合式Api与setup

Vue2的全部变量是定义在 `data`的 `return`中的，方法是放在 `methods`中的，生命周期有`mounted`  等等。这是因为Vue2的vue2的全部变量以及方法都靠 `export default`来抛出。

Vue3采用的组合式Api去掉`export default`改用`setup` `import`引入也不需要在 `components`中申明，引用后直接使用即可。

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