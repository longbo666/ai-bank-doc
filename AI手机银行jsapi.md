# 待实现的 JSAPI 需求整理

## 修订记录

| 版本 | 日期 | 作者 | 说明 |
| --- | --- | --- | --- |
| v0.1 | 2025-10-16 | 龙波 | init |

## 目录

- [1、卡片的jsapi](#1卡片的jsapi)
  - [1.1、卡片唤起卡片/离线包](#11卡片唤起卡片离线包)
  - [1.2、被唤起的卡片将结果数据发给自己的父卡片](#12被唤起的卡片将结果数据发给自己的父卡片)
  - [1.3、卡片唤起离线包](#13卡片唤起离线包)
  - [1.4、卡片唤起登录](#14卡片唤起登录)
- [2、离线包的jsapi](#2离线包的jsapi)
  - [2.1、被唤起的离线包将结果数据发给自己的父卡片](#21被唤起的离线包将结果数据发给自己的父卡片)

## 1、卡片的jsapi

### 1.1、卡片唤起卡片

- 描述：**A卡片**唤起**B卡片**，期待**B卡片**处理完成后的结果**回调给A**

- 场景：**转账录入卡片**唤起**账户选择器卡片**，用户选择账户后将账户信息回调给**录入卡片**

- **A卡片的**代码示例

    ```javascript
    <script>
        const navigator = requireModule("srcbCube");//约定的自定义Module标识
      let startParam={
        "showType":"center",
        "canceledOnTouchOutside":"false",
        "layoutStyle":"half" //default(居中)/half(半屏)/
      }
      let json={
        "startParam":startParam,
        "templateType":"CUBE",
        "templateId":"B卡⽚的id",
        "templateVersion":"B卡⽚version",
        "templateData":data,//B卡⽚所需数据
        "extData":""
      }; 
        export default {
            methods: {
                onClick() {
                    navigator.callAsync("showView" , json, (result)=>{
                      // result是B卡片处理完成的结果
                      // 【B卡片传递结果的方式，详见下一个jsapi】
                    });
                }
            }
        }
    </script>
    ```

### 1.2、被唤起的卡片将结果数据发给自己的父卡片

- 描述：**B卡片**被**A卡片**唤起，**B卡片**在处理完成后，将处**理结果回调给A**
- 场景：**账户选择器卡片**弹出，用户操作选择后，将账户信息传给唤起自己的**父卡片-转账卡片**
- **B卡片的**代码示例

```javascript
<script>
    const navigator = requireModule("srcbCube");//约定的自定义Module标识
		const data = {AcctNo: "123"} // 这是最终处理完成的数据
    export default {
        methods: {
            onClick() {
                navigator.callAsync("callbackMyParent" ,
                                    {"callbackData": data}, 
                                    (result)=>{
                  // 原生会自动找到当前卡片的唯一创建者，将json塞给对应的回调
                });
            }
        }
    }
</script>
```

### 1.3、卡片唤起离线包

- 描述：**卡片**唤起**离线包**，期待**离线包**处理完成后的结果**回调给卡片**

- 场景：**转账确认卡片**唤起**安全工具离线包**，离线包处理完成后将结果回调给**转账确认卡片**

- **卡片的**代码示例

    ```javascript
    <script>
        const navigator = requireModule("srcbCube");//约定的自定义Module标识
      let startParam={
        "showType":"center",
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



### 1.5、卡片唤起登录

- 描述：**卡片**唤起登录，登录完成后回调。**原生会自动将cookie添加在所有的对话请求内并且维护会话；**
- **卡片的**代码示例

```javascript
const navigator = requireModule("srcbCube");//约定的自定义Module标识
    export default {
        methods: {
            onClick() {
                navigator.callAsync("login" ,
                                    (result)=>{
                  // 同CCUser.login 登录成功后有用户数据；登录取消或失败也有回调
                });
            }
        }
    }
```



## 2、离线包的jsapi

### 2.1、被唤起的离线包将结果数据发给自己的父卡片

- 描述：**离线包**被**卡片**唤起，离线包在处理完成后，将处**理结果回调给卡片**
- 场景：**安全工具离线包**在输入密码后发起交易，将交易最终结果传给唤起自己的**父卡片-转账确认卡片**
- **离线包的**代码示例

```javascript
let res = {msg: "转账成功", jnLNo: "12345"}
ap.call('AIBank', {
                  'method': 'callbackMyParent',
                  'args': {
                    "callbackData": res
                  }
                });
// 原生会自动找到当前卡片的唯一创建者，将json塞给对应的回调
```
