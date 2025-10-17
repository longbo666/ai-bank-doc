# 待实现的 JSAPI 需求整理

## 修订记录

| 版本 | 日期 | 作者 | 说明 |
| --- | --- | --- | --- |
| v0.1 | 2025-10-16 | 龙波 | init |

## 目录

- [待实现的 JSAPI 需求整理](#待实现的-jsapi-需求整理)
  - [修订记录](#修订记录)
  - [卡片的jsapi](#卡片的jsapi)
    - [1、showView 展示新界面](#1showview-展示新界面)
      - [参数](#参数)
      - [回调结果](#回调结果)
      - [1.1、卡片唤起卡片](#11卡片唤起卡片)
      - [1.2、被唤起的卡片将结果数据发给自己的父卡片](#12被唤起的卡片将结果数据发给自己的父卡片)
      - [1.3、卡片唤起离线包](#13卡片唤起离线包)
      - [1.4、卡片跳转离线包](#14卡片跳转离线包)
    - [2、login唤起登录](#2login唤起登录)
      - [参数](#参数-1)
      - [回调结果](#回调结果-1)
  - [离线包的jsapi](#离线包的jsapi)
    - [被唤起的离线包将结果数据发给自己的父卡片](#被唤起的离线包将结果数据发给自己的父卡片)

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
          canceledOnTouchOutside: false,
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
              // 【B卡片传递结果的方式，详见下一个jsapi】
            });
          },
        },
      };
    </script>
    ```

### 1.2、被唤起的卡片将结果数据发给自己的父卡片

- 描述：**B卡片**被**A卡片**唤起，**B卡片**在处理完成后，将处**理结果回调给A**
- 场景：**账户选择器卡片**弹出，用户操作选择后，将账户信息传给唤起自己的**父卡片-转账卡片**
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

### 1.3、卡片唤起离线包

- 描述：**卡片**唤起**离线包**，期待**离线包**处理完成后的结果**回调给卡片**

- 场景：**转账确认卡片**唤起**安全工具离线包**，离线包处理完成后将结果回调给**转账确认卡片**

- **卡片的**代码示例

    ```javascript
    <script>
        const navigator = requireModule("srcbCube");//约定的自定义Module标识
      let startParam={
        "showType":"bottom",
        "canceledOnTouchOutside":"false",
        "layoutStyle":"default" //default(居中)/half(半屏)/
      }
      let json={
        "startParam":startParam,
        "templateType":"NEBULA”,
        "nebula": { // 离线包信息
        	appId: "21100045",
        	url: "/www/unified-security-popup.html" 
      	},
        "templateData":data,//离线包所需数据
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

### 1.4、卡片跳转离线包

- 描述：**卡片**跳转**离线包**，离开智能体对话界面，期待**离线包**处理完成后的结果**回调给卡片**
- **卡片的**代码示例

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
  let startParam={
    "showType":"push",// 把showType改为push就是跳转
  }
  let json={
    "startParam":startParam,
    "templateType":"NEBULA”,
    "nebula": { // 离线包信息
    	appId: "21100045",
    	url: "/www/index.html" 
  	},
    "templateData":data,//离线包所需数据
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

## 2、login唤起登录

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

## 3、sendUserMessage发送聊天信息给智能体

sendUserMessage为已实现功能；





# 离线包的jsapi

### 被唤起的离线包将结果数据发给自己的父卡片

- 描述：**离线包**被**卡片**唤起，离线包在处理完成后，将处**理结果回调给卡片**
- 场景：**安全工具离线包**在输入密码后发起交易，将交易最终结果传给唤起自己的**父卡片-转账确认卡片**
- **离线包的**代码示例

```javascript
let res = {msg: "转账成功", jnLNo: "12345"}
ap.call('AIBank', {
                  'method': 'callbackParent',
                  'args': {
                    "callbackData": res，
                    “_cubeCallbackId”: this._cubeCallbackId // 原生会将_cubeCallbackId放在离线包的启动参数
                  }
                });
```
