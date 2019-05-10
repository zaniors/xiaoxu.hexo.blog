---
title: react-use-redux
date: 2019-05-08 22:46:42
tags: [react, redux]
categories: [前端]
thumbnail: http://cdn.compelcode.com/image/fe/redux-flow.jpg
---

Redux在React中基本的使用
> Redux = Reducer + Flux，Redux相当于Flux的升级版，除了借鉴了Flux的理念，还增加了Reducer

## Redux的三大块

### Store
> 只负责存储所有数据的总仓库
### Action
> 只负责描述数据的类型和内容
### Reducer
> 只负责记录数据和数据在总仓库的关联关系（修改数据）

## Chrome调试Redux
- 安装Chrome插件，Redux DevTools
- 打开开发者工具，找到Redux，这是还不能使用，调试项目Redux还需配合官方文档配置下![redux-devtools](images/redux-devtools.png)
- ```
    export const store = createStore(
        reducer,
        (window as any).__REDUX_DEVTOOLS_EXTENSION__ && (window as any).__REDUX_DEVTOOLS_EXTENSION__()
    );
  ```

## 举例：场景商品列表组件和购物车，当商品列表中商品被加入购物车后，购物车的商品应更新数据
### 商品数量变化
新建新的react项目(我这里使用typescript)
``` bash
npx create-react-app react-simple-cart --typescript
```
组件文件结构
> ┌┈src
> ├┈┼store
> ├┈┼┈┼store
> ├┈┼┈┼reducers
> ├┈┼┈┼┈┴shop.reducer.tsx
> ├┈┼┈┴index.tsx
> ├┈┼Shop.component.tsx
> ├┈┼CartShopList.component.tsx
> └┈┴CartShopItem.component.tsx

Redux文件
> store/index.tsx

``` jsx 
import { createStore } from 'redux';
import { shopReducer } from './reducers/shop';

export const store = createStore(shopReducer);
```

> store/reducers/shop.tsx

``` jsx reducer是一个纯函数，state就是shop放在store的数据(一般会初始化一个state给store，请求后台的数据)，action是记录要修改什么数据，数据是什么
const initState: ShopState = {
    list: [
        {
            id: 1,
            name: '纯牛奶 100ml',
            quantity: 2,
            unit_price: 5
        },
        {
            id: 2,
            name: '花生油 500ml',
            quantity: 1,
            unit_price: 47
        }
    ]
};

export const shopReducer = (state = initState, action: any) => {
    return state;
}
```

接下来渲染shopReducer的默认数据
``` jsx Shop.component.tsx
...
readonly state = store.getState();   // initState的值
...
render() {
    return (
        <div>
            {
                this.state.list.map((item, index) => {
                    return (
                        <p key={item.id}>
                            <span>{item.name}</span>
                            <span> {item.unit_price}元</span>
                            <button onClick={this.handleSubtract.bind(this, item.id)}>-</button>
                            {item.quantity}
                            <button onClick={this.handleAdd.bind(this, item.id)}>+</button>
                        </p>
                    )
                })
            }
        </div>
    )
}
```

转发action，dispatch会将action通知reducer
``` jsx Shop.component.tsx
handleAdd(id: number) {
    const action = {
        type: 'ADD_GOOD_ITEM',
        id
    };
    store.dispatch(action);
}

handleSubtract(id: number) {
    const action = {
      type: 'SUB_GOOD_ITEM',
      id
    };
    store.dispatch(action);
}
```

reducer接收到通知，reducer里的action就是store.dispatch过来的action，告诉reducer应该处理谁的数据
``` jsx shop.reducer.tsx
export const shopReducer = (state = initState, action: any) => {
    const newState: ShopState = JSON.parse(JSON.stringify(state));
    switch (action.type) {
        case 'ADD_GOOD_ITEM':
            newState.list.forEach(element => {
                if (element.id === action.id) {
                    element.quantity++;
                }
            });
            return newState;
        case 'SUB_GOOD_ITEM':
            newState.list.forEach(element => {
                if (element.id === action.id) {
                    element.quantity--;
                }
            });
            return newState;
        default:
            return state;
    }
}
```
> 提示：到这一步store仓库数据已经更新了，但是组件的数据还未更新，还需要在组件内去订阅store的变化

``` jsx Shop.component.tsx
constructor() {
    ...
    store.subscribe(this.updateState);
}
...
private updateState() {
    this.setState(store.getState());
}
```

![redux-devtools-post](images/redux-devtools-post.png)

### 查看完整代码[https://gitee.com/Astraycat/react-cart-simple-use-redux)