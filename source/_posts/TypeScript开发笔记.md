---
title: TypeScript开发笔记
toc: true
date: 2018-08-21 22:11:36
tags:
    - React
    - TypeScript
---

在上一篇的文章中，学习了 TypeScript 的基本知识，以及 TS 在 React中的基本使用方法。在本文中，我们深入了解 TS在 React 中的实践。本文将采用 ant-design 作为基础的 UI 框架。
<!-- more -->

## 一、用 TS 创建 React 的 SFC(无状态组件)

在本次实践中，我们基于 antd 创建一个统一的 Input Form组件。该组件可以同时支持基本 Input、InputNumber和 Input.Text,我们将定义 ``elementTypeEnum``来判断该使用哪个输入组件。

1、引入 antd 的 ``Form, Input, InputNumber``组件（HXInputItem）

```js

import { Form, Input, InputNumber } from "antd";


```

2、定义输入类型的枚举

```js

enum elementTypeEnum {
    normal = "normal",//普通的字符串输入
    number = "number",//数字输入，对应antd的 InputNumber
    text = "text"//文本输入，对应 antd 的 Input.Text
}
```

3、创建 HXInputItem 的属性声明

```js

interface HXInputProps {
    label: string;//输入框的标签
    name: string;//输入框的变量名
    getFieldDecorator: (name, options) => any;//antd 的 Form 装饰器
    fieldDecoratorOptions: object;//antd Form 的表单域配置
    formLayout?: object;//antd 的 formLayout
    inputProps?: object;//对应 antd 的输入框的属性配置
    formItemProps?: object;//antd的 Form.Item的属性配置
    elementType?: elementTypeEnum;//表示使用哪种输入组件，默认normal
}
```

4、创建 HXInputItem的 SFC

这一步非常重要，跟普通 React创建 SFC 有点区别。具体用法如下：

```js

const HXInputItem: React.SFC<HXInputProps> = props => {}

```

可以看到在 TS 创建 SFC 需要使用 ``React.SFC``属性，否则无法创建成功。

5、填充我们的 HXInputItem SFC

```js
//把当前传入的属性通过解构获取。
const {
        label,
        name,
        getFieldDecorator,
        formLayout,
        fieldDecoratorOptions,
        inputProps,
        formItemProps,
        elementType
    } = props;
   return (
        <FormItem
            {...formLayout}
            label={<span className="form-label-text">{label}</span>}
            colon={false}
            hasFeedback
            {...formItemProps}
        >
            {getFieldDecorator(name, fieldDecoratorOptions)(
                elementType === "normal" ? (
                    <Input {...inputProps} />
                ) : elementType === "number" ? (
                    <InputNumber {...inputProps} />
                ) : (
                    <Input.TextArea rows={5} key={name} {...inputProps} />
                )
            )}
        </FormItem>
    );

```

通过``elementType``的值，调用不同的antd 中的输入组件，在项目中只引入当前SFC即可，让开发更加便利，快速和统一。若在以后的开发过程，存在特殊的需要，稍微兼容改造一下该组件即可，减少代码量。

6、声明该 SFC 的默认属性

```js

HXInputItem.defaultProps = {
    elementType: elementTypeEnum.normal
};

```

7、导出该组件

```
export default HXInputItem;

```

# 二、用TS 加载远程数据(fetch Api)

1、定义 http Response 的 Interface

```js

export default interface IRestRes {
    message?: string | null;//可以是 undefined、null、string类型，
    code?: number;
    filename?: string | null;//请求文件时的文件名
    data?: any;//普通 JSON 数据、file 二进制数据
    loginAccount?: string;
    name?: string;
    roles?: Array<string>;// 可以是undefined、字符串的数组
    token?: string;
    type?: string | null;
    total?: number;
}
```

2、定义统一的获取 JSON 数据的 fetch 方法

```js

/* *
* @desc 所有fetch请求的json 数据的基础；
  @author qimajiang  
* @param options object
* @param url stirng
* @return IRestRes   Promise
*/
export const initialFetch = (url:string, options:Object): Promise<IRestRes> => {
    return new Promise((resolve, reject) => {
        const finalOpts = {
            headers: new Headers({
                Authorization: myToken,//登录后需要传入该 header
            }),
            ...options
        };
        fetch(url, finalOpts)
            .then(res => {
                console.log(res);
                return res.text();
            })
            .then(text => {
                // TODO:拦截所有响应， 处理 token 失效等情况 responseMiddleware
                const resJson = text ? JSON.parse(text) : {};
                resolve(resJson as IRestRes);//将响应数据转换为IRestRes
            })
            .catch(e => {
                reject(e);//请求失败的错误捕捉
            });
    });
};

```

3、使用 fetch 的 get方法，请求JSON 数据

```js

/* *
通用 fetchGetJson
*/
export const fetchGetRequest = (api:string, params?:Object): Promise<IRestRes> => {
    return new Promise((resolve, reject) => {
        let url = new URL(api);//构造url
        url.search = new URLSearchParams(params);//构造 search
        initialFetch(url, {
            method: "GET"
        })
            .then(response => {
                console.log(response, 766);
                resolve(response);
            })
            .catch(e => {
                reject(e);
            });
    });
};


```

3、使用 fetch 的 post，提交表单(FormData)

```js
/**
通用 提交表单
*/
export const fetchPostForm = (url:string, values:object={}): Promise<IRestRes> => {
    return new Promise((resolve, reject) => {
        const formData = new FormData();
        Object.keys(values).forEach(key => {//手动构造 FormData
            const vl = values[key];
            if (vl) {
                if (vl.constructor !== Array) {
                    formData.append(key, vl);
                } else {
                    // 数组的数据
                    //  console.log(' 是数组，多个文件')
                    const fileList = vl;
                    for (let i = 0; i < fileList.length; i++) {
                        const item = fileList[i];
                        // console.log('每个文件内容',item,i, '数组长度',fileList.length)
                        formData.append(key, item);
                    }
                }
            }
        });
        initialFetch( url, {
            method: "POST",
            body: formData // data can be `string` or {object}!
        })
            .then(response => {
                console.log(response, 766);
                resolve(response);
            })
            .catch(e => {
                reject(e);
            });
    });
};

```
4、使用 fetch 获取文件流

```js
/* *
通用 fetchGetFile 获取文件
*/
export const fetchGetFile = (api:string, params?:obejct): Promise<IRestRes> => {
    let url = new URL(api);
    return new Promise((resolve, reject) => {
        const finalOps = {
            headers: new Headers({
                Authorization: myToken,
                "Access-Control-Allow-Origin": BASE_URL,//服务端必须响应 该header，否则无法获取到 Content-Disposition
            }),
            ...params
        };
        fetch(url, finalOps)
            .then(async res => {
                // TODO:拦截所有响应， 处理 token 失效等情况 responseMiddleware
                const contentDisposition = res.headers.get("Content-Disposition");
                const myBlob = await res.blob();
                const Ires: IRestRes = {
                    data: myBlob,
                    type: res.headers.get("Content-Type"),
                    filename: decodeURI(
                        String(contentDisposition).replace("attachment;filename=", "") || ""
                    )
                };
                resolve(Ires);
            })
            .catch(e => {
                reject(e);
            });
    });
};


```

# 三、 使用 TS创建普通的 React 组件

先上完整的组件代码

```js

import * as React from "react";
import { observer, inject } from "mobx-react";
import IStore from "../../interface/IStore";
import SearchType from "../../enum/SearchType";
// const styles = require("./styles/index.less");
interface SearchBoxProps {
    store?: IStore;
    placeholder?: string;
}
interface SearchBoxState {
    focus: boolean;
    focusLock: boolean;
    searchField: SearchType;
    words: string;
    searching: boolean;
}
@inject("store")
@observer
export default class SearchBox extends React.Component<SearchBoxProps, SearchBoxState> {
    constructor(props) {
        super(props);
        this.state = {
            focus: false,
            focusLock: false,
            searchField: SearchType.SEARCH_FIELD_ALL,
            words: "",
            searching: false
        };
    }

    static defaultProps : {
        placeholder: '输入关键词并搜索'
    }
    onFocus = () => {
        this.setState({
            focus: true
        });
    };
    onBlur = () => {
        if (this.state.focusLock) {
            return;
        }
        this.setState({
            focus: false
        });
    };
    onMouseEnter = () => {
        this.setState({
            focusLock: true
        });
    };
    onMouseLeave = () => {
        this.setState({
            focusLock: false
        });
    };
    render() {
        return (
            <div
                onFocus={this.onFocus}
                onBlur={this.onBlur}
                onMouseEnter={this.onMouseEnter}
                onMouseLeave={this.onMouseLeave}
            />
        );
    }
}
```

有看出与普通 JS 创建React 组件的区别了吗？

（1）、使用泛型定义组件的 props 与 state；

 (2)、创建默认的 state 值，必须在``SearchBoxState``已经定义，否则出错。

 (3)、   ``defaultProps`` 需要使用 ``static`` 修饰


 # 四、创建 d.ts 文件

 在项目根目录创建 typings 的文件夹，在 typings 中创建 ``index.d.ts``

 文件中输入以下内容


 ```js
interface Promise<T> {
    finally: (callback) => Promise<T>;
}

 ```

 由于我们在使用 fetch 请求数据时，返回的数据使用 ``Promise<IRestRes>``返回。如果不创建上门的 ``d.ts``，编辑器在编译时可能无法``Promise<IRestRes>``。为什么呢？

因为：

typings的存在是为了方便TypeScript识别、编译、智能提示TypeScript无法识别的JS库的特性和语法。

 

# 五、总结

TS在 React 的使用远不止这些，本文只是抛砖引玉，让我们对 TS 的了解更加深入，而且更是感叹 TS 的强大。



