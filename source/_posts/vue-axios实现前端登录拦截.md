---
title: vue-axios实现前端登录拦截
date: 2021-03-16 9:30:03
tag: []
categories: [vue]
---

## 路由拦截

首先在定义路由的时候就需要多添加一个自定义字段 requireAuth，用于判断该路由的访问是否需要登录。如果用户已经登录，则顺利进入路由， 否则就进入登录页面

```js
const routes = [
  { path: "/", name: "/", component: Index },
  {
    path: "/repository",
    name: "repository",
    meta: {
      requireAuth: true, // 添加该字段，表示进入这个路由是需要登录的
    },
    component: Repository,
  },
  { path: "/login", name: "login", component: Login },
];
```

定义完路由后，我们主要是利用 vue-router 提供的钩子函数 beforeEach()对路由进行判断

```js
router.beforeEach((to, from, next) => {
  if (to.meta.requireAuth) {
    // 判断该路由是否需要登录权限
    if (store.state.token) {
      // 通过vuex state获取当前的token是否存在
      next();
    } else {
      next({
        path: "/login",
        query: { redirect: to.fullPath }, // 将跳转的路由path作为参数，登录成功后跳转到当前路由
      });
    }
  } else {
    next();
  }
});
```

to.meta 中是我们自定义的数据，其中就包括我们刚刚定义的 requireAuth 字段，通过这个字段来判断该路由是否需要登录权限。需要的话，同时当前应用不存在 token，则跳转到登录页面，进行登录。登录成功后跳转到目标路由这种方式只是简单的前端路由控制，并不能真正阻止用户访问需要登录权限的路由还有一种情况便是：当前 token 失效了，但是 token 依然保存在本地。这时候你去访问需要登录权限的路由时，实际上应该让用户重新登录， 这时候就需要结合 http 拦截器 + 后端接口返回的 http 状态码来判断

## Axios 拦截器

要想统一处理所有 http 请求和响应，就得用上 axios 的拦截器，通过配置 http response inteceptor，当后端接口返回 401 Unauthorized（未授权）， 让用户重新登录

```js
// http request 拦截器
axios.interceptors.request.use(
  (config) => {
    if (store.state.token) {
      // 判断是否存在token，如果存在的话，则每个http header都加上token
      config.headers.Authorization = `token ${store.state.token}`;
    }
    return config;
  },
  (err) => {
    return Promise.reject(err);
  }
);

// http response 拦截器
axios.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response) {
      switch (error.response.status) {
        case 401:
          // 返回 401 清除token信息并跳转到登录页面
          store.commit(types.LOGOUT);
          router.replace({
            path: "login",
            query: { redirect: router.currentRoute.fullPath },
          });
      }
    }
    return Promise.reject(error.response.data); // 返回接口返回的错误信息
  }
);
```

通过上面这两步，就可以在前端实现登录拦截了,登出功能也就很简单，只需要把当前 token 清除，再跳转到首页即可
