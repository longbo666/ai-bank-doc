

# AI手机银行原生对接文档

## 修订记录

| 版本 | 日期         | 作者 | 说明                                       |
| --- |------------| --- |------------------------------------------|
| v0.1 | 2025-10-17 | 龙波 | init                                     |
| v0.2 | 2025-10-20 | 龙波 | 新增智能体对话部分；<br />卡片发消息的jsapi改为sendUserMsg |
| v0.3 | 2025-10-23 | 龙波 | 新增关闭卡片插件hideView; 首次自动发送的标识；新增picker     |

## 目录

- [AI手机银行原生对接文档](#ai手机银行原生对接文档)
  - [修订记录](#修订记录)
  - [卡片的jsapi](#卡片的jsapi)
    - [1、showView 展示新界面](#1showview-展示新界面)
      - [参数](#参数)
      - [回调结果](#回调结果)
      - [1.1、卡片唤起卡片](#11卡片唤起卡片)
      - [1.2、卡片唤起离线包](#12卡片唤起离线包)
      - [1.3、卡片跳转离线包](#13卡片跳转离线包)
    - [2、callbackParent卡片将数据发给自己的父卡片](#2callbackparent卡片将数据发给自己的父卡片)
      - [参数](#参数-1)
      - [回调结果](#回调结果-1)
    - [3、login唤起登录](#3login唤起登录)
      - [参数（同CCUser.login）](#参数同ccuserlogin)
      - [回调结果](#回调结果-2)
    - [4、sendUserMsg发送聊天信息给智能体](#4sendusermsg发送聊天信息给智能体)
      - [参数](#参数-2)
    - [5、rpc 网关请求](#5rpc-网关请求)
    - [6、getUser 获取用户信息](#6getuser-获取用户信息)
      - [参数](#参数-3)
      - [回调结果](#回调结果-3)
    - [7、CC_MASK全局遮罩](#7cc_mask全局遮罩)
    - [8、hideView关闭界面](#8hideview关闭界面)
    - [9、showPicker弹出选择器](#9showpicker弹出选择器)
      - [参数](#参数-4)
      - [dataSource 数据结构](#datasource-数据结构)
      - [返回值](#返回值)
      - [回调结果](#回调结果-4)
      - [Value.models 数据结构](#valuemodels-数据结构)
  - [离线包的jsapi](#离线包的jsapi)
    - [1、AIBank.callbackParent 离线包将结果数据发给自己的父卡片](#1aibankcallbackparent-离线包将结果数据发给自己的父卡片)
  - [智能体对话相关](#智能体对话相关)
    - [对话扩展参数](#对话扩展参数)
    - [禁止打断标识](#禁止打断标识)
    - [初次进入对话后的自动消息](#初次进入对话后的自动消息)

# 卡片的jsapi

## 1、showView 展示新界面

### 参数

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| showView | String | API 名称 | 是 | - | 固定值 `showView` |
| params | Object | 目标视图的打开参数 | 是 | - | 结构见下表 |
| callback | Function | 处理完成后的回调函数 | 否 | - | 触发时会带上执行结果 `res` |

**params.startParam 字段**

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| showType | String | 展示方式 | 否 | `bottom` | `bottom` 底部弹出支持卡片和离线包、`push`仅限跳离线包 |
| canceledOnTouchOutside | Boolean/String | 点击蒙层是否关闭 | 否 | `true` | 兼容字符串布尔值，建议传布尔型 |
| height | String | 高度 | 否 | auto | `auto`自动高度、0~1的数字（屏幕高度的比例） |
| width | String | 宽度 | 否 | 屏幕宽度 |  |

**params 其他字段** 

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| templateType | String | 目标模板类型 | 是 | `CUBE` | `CUBE` 表示卡片，`NEBULA` 表示离线包 |
| nebula | Object | 离线包信息 | `templateType=NEBULA` 时必选 | - | 离线包信息 |
| extData | Object | 扩展参数 | 否 | - | 可用于调试、灰度标记等扩展场景 |

**params.cube 字段**

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| templateId | String | 被唤起卡片的唯一标识 | `templateType=CUBE` 时必选 | - | 卡片 ID |
| templateVersion | String | 被唤起卡片的版本号 | 否 | - | 卡片版本号 |
| templateData | Object | 传给目标卡片的业务参数 | 否 | `{}` | 结构由业务自定义，会原样传给被唤起侧 |

**params.nebula 字段**

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| appId | String | 离线包 ID | 是 | - | 对应  离线包的唯一标识 |
| url | String | 离线包内入口路径 | 是 | - | 离线包页面 |
| data | Object | 离线包的启动参数 | 否 |  | 结构由业务自定义，会原样传给被唤起的离线包 |

### 回调结果

`navigator.callAsync("showView", params, callback)` 的 `callback(res)` 中包含以下字段：

| 字段 | 类型 | 描述 | 备注 |
| ---- | ---- | ---- | ---- |
| ErrorCode | Number | 执行状态码 | `0` 表示成功，非 `0` 表示失败 |
| ErrorMessage | String | 错误描述信息 | 成功时通常为空 |
| Value | Object | 业务返回内容 | 成功 (`ErrorCode=0`) 时返回业务数据；无需返回数据时Value为空 |

### 1.1、卡片唤起卡片

- 描述：**A卡片**唤起**B卡片**，期待**B卡片**处理完成后的结果**回调给A**

- 场景：**转账录入卡片**唤起**账户选择器卡片**，用户选择账户后将账户信息回调给**录入卡片**

- **A卡片的**代码示例

    ```javascript
    <script>
      const navigator = requireModule("srcbCube"); // 约定的自定义Module标识
    
      const params = {
        startParam: {
          showType: "bottom",
          canceledOnTouchOutside: true,
          height: "0.6", // 可根据需求设置为0~1之间的比例，或留空采用自适应
        },
        templateType: "CUBE",
        cube: {
          templateId: "B卡片的id",
          templateVersion: "B卡片version",
          templateData: data, // B卡片所需数据
        },
        extData: {}, // 扩展参数，可按需透传
      };
    
      export default {
        methods: {
          onClick() {
            navigator.callAsync("showView", params, (res) => {
              // res 是B卡片处理完成的结果
              // 【B卡片传递结果的方式，详见下一个jsapi2】
            });
          },
        },
      };
    </script>
    ```


### 1.2、卡片唤起离线包

- 描述：**卡片**唤起**离线包**，期待**离线包**处理完成后的结果**回调给卡片**

- 场景：**转账确认卡片**唤起**安全工具离线包**，离线包处理完成后将结果回调给**转账确认卡片**

- **卡片的**代码示例

    ```javascript
    <script>
        const navigator = requireModule("srcbCube");//约定的自定义Module标识
      let startParam={
        "showType":"bottom",
        "canceledOnTouchOutside":true,
      }
      let json={
        "startParam":startParam,
        "templateType":"NEBULA",
        "nebula": { // 离线包信息
        	appId: "21100045",
        	url: "/www/index.html" ,// 默认的安全工具页面
          data: this.data//离线包所需数据
      	},
        "extData":""
      }; 
        export default {
            methods: {
                onClick() {
                    navigator.callAsync("showView" , json, (result)=>{
                      // result是离线包处理完成的结果
                      // 【离线包传递结果的方式，详见离线包的jsapi2.1】
                    });
                }
            }
        }
    </script>
    ```

### 1.3、卡片跳转离线包

- 描述：**卡片**跳转**离线包**，离开智能体对话界面，期待**离线包**处理完成后的结果**回调给卡片**
- **卡片的**代码示例

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
  let startParam={
    "showType":"push",// 把showType改为push就是跳转
  }
  let json={
    "startParam":startParam,
    "templateType":"NEBULA",
    "nebula": { // 离线包信息
    	appId: "21100045",
    	url: "/www/index.html" ,
        data: {aaa:111}
  	},
    "extData":""
  }; 
    export default {
        methods: {
            onClick() {
                navigator.callAsync("showView" , json, (result)=>{
                  // result是离线包处理完成的结果
                  // 【离线包传递结果的方式，详见离线包的jsapi2.1】
                });
            }
        }
    }
```

## 2、callbackParent卡片将数据发给自己的父卡片

- 描述：**B卡片**被**A卡片**唤起，**B卡片**在处理完成后，将处**理结果回调给A**。调用完成后**B**自动被**关闭**
- 场景：**账户选择器卡片**弹出，用户操作选择后，将账户信息传给唤起自己的**父卡片-转账卡片**
- **调用方式**：`navigator.callAsync("callbackParent", params, callback)`

### 参数

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| callbackParent | String | API 名称 | 是 | - | 固定值 `callbackParent` |
| params | Object | 回传给父卡片的数据载体 | 是 | - | 结构见下表 |
| callback | Function | 回调通知执行结果 | 否 | - | 原生在处理完成后触发 |

**params 字段**

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| callbackData | Object | 要传递给父卡片的业务结果 | 否 | `{}` | 为空时仅表示通知，无附加数据 |
| cubeCallbackId | String | 原生分配的回调标识 | 是 | - | 直接透传 `this._cubeCallbackId`，用于找到父卡片 |

### 回调结果

`callback(res)` 中返回原生执行状态，字段含义如下：

| 字段 | 类型 | 描述 | 备注 |
| ---- | ---- | ---- | ---- |
| ErrorCode | Number | 执行状态码 | `0` 表示成功通知父卡片，非 `0` 表示失败 |
| ErrorMessage | String | 错误描述信息 | 失败时返回具体原因 |
| Value | Object | 补充信息 | 预留字段，通常为空 |

- **B卡片的**代码示例

```javascript
    const navigator = requireModule("srcbCube");//约定的自定义Module标识
		const data = {AcctNo: "123"} // 这是最终处理完成的数据
    export default {
        methods: {
            onClick() {
              	// 原生会根据找到当前卡片的唯一创建者，将data回调
                navigator.callAsync("callbackParent" ,
                                    {
                  									 callbackData: data,
                  									 // 原生自动会将_cubeCallbackId字段塞进卡片
                                     cubeCallbackId :this._cubeCallbackId
                                    }, 
                                    (result)=>{});
            }
        }
    }
```



## 3、login唤起登录

### 参数（同CCUser.login）

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| login | String | API 名称 | 是 | - | 固定值 `login` |
| params | Object | 登录配置参数 | 否 | `{}` | 保留字段，如无额外参数可省略 |
| callback | Function | 登录完成后的回调函数 | 否 | - |  |

### 回调结果

`navigator.callAsync("login", params, callback)`（或省略 `params`）的 `callback(res)` 中包含以下字段：

| 字段 | 类型 | 描述 | 备注 |
| ---- | ---- | ---- | ---- |
| ErrorCode | Number | 执行状态码 | `0` 表示登录成功，非 `0` 表示用户取消或失败 |
| ErrorMessage | String | 错误描述信息 | 根据失败原因返回提示文字 |
| Value | Object | 登录成功返回的用户信息 | 包含用户标识、会话信息；失败或取消时Value为空 |

- 描述：**卡片**唤起登录，登录完成后回调。**原生会自动将cookie添加在所有的对话请求内并且维护会话；**
- **卡片的**代码示例

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
    export default {
        methods: {
            onClick() {
                navigator.callAsync("login" ,
                                    {},
                                    (result)=>{
                  // 同CCUser.login 登录成功后有用户数据；登录取消或失败也有回调
                });
            }
        }
    }
```

## 4、sendUserMsg发送聊天信息给智能体

sendUserMsg代码示例

### 参数

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| sendUserMsg | String | API 名称 | 是 | - | 固定值 `sendUserMsg` |
| query | String | query消息 | 是 | - |  |
| extInfo | Object | 扩展参数 | 否 | `{}` | 想透传给智能体的信息，原生会将其合并到**extParams**；deviceId/apptp/athenaToken之类的公共参数原生会自动上送，前端无需关心 |


```javascript
const navigator = requireModule("srcbCube");// 约定的自定义Module标识
export default {
    methods: {
        onClick() {
            navigator.callAsync('sendUserMsg', {
                    "query": "你好",
                    "extInfo": {xxx: 888} //想透传给智能体的信息，会与原生的extInfo合并
                },
                (res) => {}
            )
        }
    }
}
```



## 5、rpc 网关请求

rpc为已实现功能，目前需要集成小程序sdk

## 6、getUser 获取用户信息

### 参数

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| getUser | String | API 名称 | 是 | - | 固定值 `getUser` |
| params | Object | 预留参数 | 是 | `{}` | 如无额外参数可省传空对象 |
| callback | Function | 获取完成后的回调函数 | 否 | - | 原生完成处理后触发 |

#### 回调结果

| 字段 | 类型 | 描述 | 备注                   |
| ---- | ---- | ---- |----------------------|
| ErrorCode | Number | 执行状态码 | `0` 表示获取成功，非 `0` 表示失败 |
| ErrorMessage | String | 错误描述信息 | -                    |
| Value | Object | 用户信息，未登录情况下为空 | 服务端返回的用户信息 |

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
    export default {
        methods: {
            onClick() {
                navigator.callAsync("getUser" ,
                                    {},
                                    (result)=>{
                  if(result.ErrorCode==0){
                    console.log(result.Value)
                  }
                });
            }
        }
    }
```

## 7、CC_MASK全局遮罩

在`rpc`请求头里增加CC_MASK: "YES"的键值对，可自动实现原生全局遮罩。由原生控制遮罩的生命周期



## 8、hideView关闭界面
> [!IMPORTANT]
>
> hideView暂未实现，卡片可先用callbackParent替代

与showView对应， hideView用来关闭已弹出的界面

同时只会有1个view被弹出，无需指定ID

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
    export default {
        methods: {
            onClick() {
                navigator.callAsync("hideView" ,
                                    {},
                                    (result)=>{
                });
            }
        }
    }
```

## 9、showPicker弹出选择器

showPicker代码示例

### 参数

| 名称       | 类型          | 描述       | 必选 | 默认值 | 备注                          |
| ---------- | ------------- | ---------- | ---- | ------ | ----------------------------- |
| showPicker | String        | API 名称   | 是   | -      | 固定值 showPicker             |
| title      | String        | 弹窗标题   | 否   | ""     | 将展示在选择器顶部            |
| dataSource | Array<Object> | 选项数据源 | 是   | -      | 数组元素用于定义多列/多级结构 |

#### dataSource 数据结构

| 字段     | 类型          | 描述       | 必选 | 备注                                         |
| -------- | ------------- | ---------- | ---- | -------------------------------------------- |
| text     | String        | 展示文案   | 是   | 例如业务名称、省市名称等                     |
| code     | String        | 业务标识   | 是   | 用于回传所选项                               |
| children | Array<Object> | 下一级选项 | 否   | 存在时形成多列/多级联动                      |
| extras   | Object        | 扩展信息   | 否   | 将在回调结果中一并返回，便于携带自定义元数据 |

### 返回值

#### 回调结果

| 字段 | 类型 | 描述 | 备注                   |
| ---- | ---- | ---- |----------------------|
| ErrorCode | Number | 执行状态码 | `0` 表示获取成功，-1表示取消 |
| ErrorMessage | String | 错误描述信息 | -                    |
| Value | Object | 插件返回结果 |  |
| Value.indexs | Array<Number> | 用户选择后的序号 |  |
| Value.models | Array<Object> | 用户选择后的数据 |  |

#### Value.models 数据结构

| 字段   | 类型   | 描述     | 必选 | 备注                   |
| ------ | ------ | -------- | ---- | ---------------------- |
| text   | String | 展示文案 | 是   | 来源于入参的dataSource |
| code   | String | 业务标识 | 是   | 来源于入参的dataSource |
| extras | Object | 扩展信息 | 否   | 来源于入参的dataSource |

- 回调入参 `res` 为原生侧返回的结果对象。常见字段约定：
  - `indexs`：Array，记录各列选中的索引。
  - `models`：Array，包含选中项的 code、text 以及 extras。

```javascript
const navigator = requireModule("srcbCube"); // 约定的自定义Module标识

export default {
    data: {
        pickerData1: [
            { code: "A01", text: "理财产品" },
            { code: "A02", text: "信用卡申请" },
            { code: "A03", text: "生活缴费" }
        ],
        regionData: [
            {
                text: "北京市",
                code: "11",
                children: [{ text: "北京市", code: "1000" }]
            },
            {
                text: "重庆市",
                code: "50",
                children: [
                    { text: "重庆市石柱", code: "6871" },
                    { text: "重庆市酉阳", code: "6874" }
                ]
            }
        ]
    },
    methods: {
        onShowPickerSingle() {
            navigator.callAsync(
                "showPicker",
                {
                    title: "业务类型",
                    dataSource: this.pickerData1
                },
                (res) => {
                    // 处理单列选择结果
                    console.log(res.Value);
                    // { indexs: [1], models: [{ code: "A02", text: "信用卡申请" }] }
                }
            );
        },
        onShowPickerCascade() {
            navigator.callAsync(
                "showPicker",
                {
                    title: "省市选择",
                    dataSource: this.regionData
                },
                (res) => {
                    // 处理多级联动结果
                    console.log(res.Value);
                    // { indexs: [0, 0], models: [{ code: "11", text: "北京市" }, { code: "1000", text: "北京市" }] }
                }
            );
        }
    }
};
```



# 离线包的jsapi

## 1、AIBank.callbackParent 离线包将结果数据发给自己的父卡片

- 描述：**离线包**被**卡片**唤起，离线包在处理完成后，将处**理结果回调给卡片**
- 场景：**安全工具离线包**在输入密码后发起交易，将交易最终结果传给唤起自己的**父卡片-转账确认卡片**
- **离线包的**代码示例

```javascript
let res = {msg: "转账成功", jnLNo: "12345"}
ap.call('AIBank', {
                  'method': 'callbackParent',
                  'args': {
                    "callbackData": res,
                    // 原生会将_cubeCallbackId放在离线包的启动参数
                    "_cubeCallbackId": this._cubeCallbackId
                  }
                });
```

# 智能体对话相关

## 对话扩展参数
- 客户端主动发起的消息、卡片通过sendUserMsg发的消息，**每次都会自动包含以下扩展参数**
- 卡片透传的**extInfo**会被合并进来
- **extInfo**

| 名称 | 类型 | 描述 | 必选 | 默认值 | 备注 |
| ---- | ---- | ---- | ---- | ------ | ---- |
| _AthenaToken | String | 登录token | 否 | - | cookie里的登录token；**没传就是没登录** |
| appid | String |  | 是 | "9999" | 深圳版 9999 |
| bankid | String | - | 是 | "9999" | 深圳版 9999 |
| apptp | String |  | 是 | "A" | A、深圳版 <br />B、广西支行版<br /> C、广西村镇版 |
| applvl | String |  | 是 | "1" | 1、大众版<br /> 2、尊享版 <br />4、尊爱版 <br />5、英文版 |
| CC-Device-Id | String | 设备ID | 是 | - | 同原生deviceId |
| welcome | String | 首次自动发送的标识 | 否 | - | 如有，值也固定为"welcome" |
| hasUsedToday | Bool | 今日使用过的标识 | 否 | - | 今日内使用过为true，**该字段只会和welcome一起出现** |
| ...extInfo   |        | 前端的扩展参数 | 否 | - | 前端的extInfo会结构合并进来                      |

## 禁止打断标识

- 一些不希望被打断的关键步骤，**智能体会下发禁止打断标识**。客户端在解析到此标识后，会将消息发送按钮置灰为不可用；
- 直到下次智能体的响应里不再有此标识，消息发送按钮才能再次可用；
- // todo: 具体字段名待定

## 初次进入对话后的自动消息
- 用户打开对话界面后，客户端会自动发消息给智能体，此条消息会在extInfo里携带特定字段。
- 本消息不会显示在用户发送侧

```javascript
{
  "query": "welcome"
  "extInfo": {
    "welcome": "welcome", // 有此扩展字段表示此条消息是自动发送的
    "hasUsedToday": true, // 有此扩展字段表示该用户今天已经用过AI对话
  	... // 其他扩展参数规则不变
  }
}
```
