---
title: 当React遇到Typescript-01
date: 2019-04-25
thumbnail: https://cdn.compelcode.com/image/fe/react+typescript.png
tags: [react, typescript]
categories: [前端]
---

在React中如何使用TS来编写（基础篇）
> TS是JS的超集，一大语法糖，TS的好处不言而喻，让JS拥有强类型语言特性，类型检测，泛型，接口等等，让项目在构建时发现错误。现在诸多开源项目，比如：Ant-design、Vscode、Angular等等都是使用TS来编写底层代码

## 技术点
### 使用React脚手架工具构建
``` bash
$ npx create-react-app react-ts-app --typescript
```

### 无状态组件
1. 新建一个React组件Button，Button.tsx
2. ``` jsx
    // Button.tsx
    const Button = React.FC = () => {
        return <button>按钮</button>
    }
    export default Button;

    // App.tsx
    const App: React.FC = () => {
        return (
            <Button />
        );
    }
    export default App;
   ```
> 提示：React.SFC已被废弃，最新版本使用React.FC

### 无状态组件（接受父组件数据）
无状态的组件，是没有state的，可以看下React.FC申明`type FC<P = {}> = FunctionComponent<P>;`，是FunctionComponent的一个别名
``` jsx
    interface FunctionComponent<P = {}> {
        (props: PropsWithChildren<P>, context?: any): ReactElement | null;
        propTypes?: WeakValidationMap<P>;
        contextTypes?: ValidationMap<any>;
        defaultProps?: Partial<P>;
        displayName?: string;
    }
```
可以看到FunctionComponent接受一个泛型P，这里表示子组件要接受的Props接口数据

``` jsx
    // Button.tsx
    interface ButtonProps {
        data: string;
    }

    const Button: React.FC<ButtonProps> = (props: ButtonProps) => {
        return (
            <button onClick={handleBtnClick}>{props.data}</button>
        )
    }

    const handleBtnClick = (event: any) => {
        console.log(event.target.innerText);
    }
    export default Button;

    // App.tsx
    const App: React.FC = () => {
        return (
            <Button data={'hello world!'}/>
        );
    }
    export default App;
   ```

### 有状态组件
1. 新建一个组件Step，Step.tsx
2. 
``` jsx
    // Step.tsx
    class Step extends Component {
        render() {
            return (
                <div>
                    <button>-</button>
                    <span>1</span>
                    <button>+</button>
                </div>
            );
        }
    }
    export default Step;
```
可能会问这个和JS的React有什么不同吗？我们看下继承的Component源码(截取部分)
``` jsx
    class Component<P, S> {
        ...
        constructor(props: Readonly<P>);
        ...
        state: Readonly<S>;
        ...
    }
```
可以看到Component有两个泛型参数，P则是Component父类构造器初始化的Props，S则是组件的State状态
 
``` jsx 完整的代码 
    const initState = {
        count: 0
    }
    type State = Readonly<typeof initState>

    class Step extends Component<{}, State> {
        constructor(props: {}) {
            super(props);

            this.handlePrevClick = this.handlePrevClick.bind(this);
            this.handleNextClick = this.handleNextClick.bind(this);
        }

        readonly state = initState;

        render() {
            return (
                <div>
                    <button onClick={this.handlePrevClick}>-</button>
                    <span>{this.state.count}</span>
                    <button onClick={this.handleNextClick}>+</button>
                </div>
            );
        }

        handlePrevClick() {
            const count = this.state.count;
            this.setState({
                count: count - 1
            })
        }

        handleNextClick() {
            const count = this.state.count;
            this.setState({
                count: count + 1
            })
        }
    }

    export default Step;
```
``` jsx
    const initState = {
        count: 0
    }
    type State = Readonly<typeof initState>
    ...
    readonly state = initState;
```
1. 初始化值
2. 通过Readonly<T> 将所有属性都设置为<font color="red">**readonly**</font>
3. 将state设置为<font color="red">**readonly**</font>
> 目的是：借助TS避免开发者在编码时直接修改state和state的属性值