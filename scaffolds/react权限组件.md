# react 权限组件

之前的博客讲到了该如何控制权限，

当存在一些问题，包括不能针对特定角色，如博客详情页，如果当前属于博客创建者，有编辑、删除按钮。

之前的问题是，在控制页面的权限时，如果页面层级较多，需要配置每一级的权限。具体来说，有如下路由：

![多层菜单](http://oyy3cbpm3.bkt.clouddn.com/15352897424980.jpg)

如果希望控制「科技图书管理员」负责维护「科技」页面，「文学图书管理员」维护「文学」页面，那么就有如下代码：

```js
// 角色的权限配置
const pagePermissionConfig = {
    // 科技图书管理员对应角色 id
    0: [
        'page__cate_root',
        'page__books_root',
        'page__books_type_root',
        'page__books_type_0'
    ],
    // 文学图书管理员对应角色 id
    1: [
        'page__cate_root',
        'page__books_root',
        'page__books_type_root',
        'page__books_type_1'
    ],
};
// 该配置方式见《前端路由与菜单》
const routes = [
    {
        path: '/cate',
        name: '类别',
        resource: 'page__cate_root',
        children: [
            {
                path: '/books',
                name: '图书',
                resource: 'page__books_root',
                children: [
                    {
                        path: '/:type',
                        resource: 'page__books_type_root',
                        component: Category
                    },
                    {
                        path: '/0',
                        resource: 'page__books_type_0',
                        name: '科技'
                    },
                    {
                        path: '/1',
                        resource: 'page__books_type_1',
                        name: '文学'
                    },
                ]
            },
        ]
    }
];
```


