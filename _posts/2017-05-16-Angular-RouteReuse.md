---
layout:     post
title:      "Angular之RouteReuse路由复用策略"
subtitle:   ""
date:       2017-05-16 21:51:19
author:     "Kai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - ng2
    - RouteReuse
    - 路由复用策略
    - 返回无刷新
---



Angular 利用 RouteReuseStrategy 贯穿路由状态并决定构建组件的方式。官方有提供教科书式的路由复用功能，只要重写下面几个基本方法：

### 基础
```js
export class SimpleRouteReuseStrategy implements RouteReuseStrategy {
    cacheRouters: any = [];
    shouldDetach(route: ActivatedRouteSnapshot): boolean {
        return true;
    }
    store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle): void {
        // 按path作为key存储路由快照&组件当前实例对象
        // path等同RouterModule.forRoot中的配置
        this.cacheRouters[route.routeConfig.path] = handle;
    }
    shouldAttach(route: ActivatedRouteSnapshot): boolean {
        // 在缓存中有的都认为允许还原路由
        return !!route.routeConfig && !!this.cacheRouters[route.routeConfig.path];
    }
    retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle {
        // 从缓存中获取快照，若无则返回null
        if (!route.routeConfig || !this.cacheRouters[route.routeConfig.path]) return null;
        return this.cacheRouters[route.routeConfig.path];
    }
    shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
        // 同一路由时复用路由
        return future.routeConfig === curr.routeConfig;
    }
```

1. `shouldDetach` 是否允许复用路由
2. `store` 当路由离开时会触发，存储路由
3. `shouldAttach` 是否允许还原路由
4. `retrieve` 获取存储路由
5. `shouldReuseRoute` 进入路由触发，是否同一路由时复用路由

### 白名单

上面代码中使用了一个自定义变量`cacheRouters`来存储缓存，里面的键值用的路由路径`route.routeConfig.path`。<br>
这样一来整个Angular项目默认都会应用缓存策略。<br>
如果我们只希望某几个组件页面需要用到。就需要更改`shouldDetach`方法。<br>
```js
    shouldDetach(route: ActivatedRouteSnapshot): boolean {
        const detach = ['someCompontent'];
        return detach.indexOf(route.routeConfig.path) > -1;
    }
```
需要注意的是，如果你项目中使用了懒加载组件，这里的route取到的是child里的Compontent名称。此时我们可以自定义我们需要缓存的组件名。<br>
首先在router文件中加入拦截器`blockIn`
```js
...
import { BlockIn } from './services/blockIn.service';
...
    {
        canActivate: [BlockIn],
        path: '',
        loadChildren: './pages/home/home.module#HomeModule'
    },
...
```
在`blockIn.service`中我们就可以开心的自定义路由中的很多东西了，不止自定义组件名称还可以用来鉴权等。<br>
此处我们在路由里增加一个变量`cname`
```js
    canActivate(nextRoute: ActivatedRouteSnapshot) {

        if (nextRoute.component) {
            nextRoute['cname'] = nextRoute.parent.routeConfig.path + nextRoute.routeConfig.path;
        }

    }
```
然后在`shouldDetach`方法里使用这个`cname`
```js
    shouldDetach(route: ActivatedRouteSnapshot): boolean {
        const detach = ['someCompontent', ...];
        return detach.indexOf(route['cname']) > -1;
    }
```

### 更新部分缓存

有时候回到缓存的页面，我们希望能自动更新部分变量。比如从购物车页面去列表页增加了物品，返回后不刷新页面只更新列表。
这个时候我们需要在组件的`constructor`重写`shouldReuseRoute`方法
```js
constructor(
    public route: ActivatedRoute,
    public router: Router,
    ){
        this.router.routeReuseStrategy.shouldReuseRoute = (future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot) => {
            // 在这里 你可以为所欲为了
    }
}
```
需要注意的是`shouldReuseRoute`会被执行四次，分别为四种状态，要注意区分。

### 清除缓存

最后我们要清除缓存，还是在一开始的路由复用策略的服务中声明。然后在需要的组件中调用就行。

```js
    clearCacheRouters() {
        for (const key in this.cacheRouters) {// 清除所有缓存 当然 你也可以自定义清除部分缓存
            this.cacheRouters[key] = null;
        }
    }

```
惯例，肯定要有注意的地方，放心`坑`我都替你踩过了，如果是同一组件只是链接不同比如list?page=1 、 list?page=2 会出现这个错误[https://github.com/angular/angular/issues/13831](https://github.com/angular/angular/issues/13831)
目前有效解决方案是使用`componentRef`的`destroy`方法

```js
    clearCacheRouters() {
        for (let key in this.cacheRouters) {
            if (this.cacheRouters[key]){
                this.deactivateOutlet(this.cacheRouters[key])
            }
        }
    }

    private deactivateOutlet(handle: DetachedRouteHandle): void {
        let componentRef: ComponentRef<any> = handle ? handle['componentRef'] : null;
        if (componentRef) {
            componentRef.destroy();
        }
    }


```
还有什么坑，欢迎留言