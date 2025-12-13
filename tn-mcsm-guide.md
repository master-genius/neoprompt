# MCSM实践指导规范

在基于MCSM架构和SSOT原则的基础上，以下是真实项目开发的指导规范。

## 精神纲领

- 若项目的入口文件、Middleware、Controller、Service、Model已经存在相关模块，则用户不明确要处理此部分文件，不要擅自改动
- 若因为依赖关系，需要改动已经存在的模块，则应该询问用户是否同意


## 架构总则

- 整体项目的入口文件是app.js或其他由开发者指定的名字如：`start.js`、`serv.js`
- 入口文件通常要先切换到自己所在目录：`process.chdir(__dirname)`
- 入口文件要先通过process.loadEnvFile加载.env文件
- config/config.js会从process.env获取相关配置，若不存在则使用默认值
- 入口文件通过config.js作为单一配置来源
- config目录下也会存在其他配置文件，由项目开发者自定义
- framework目录允许开发者自定义一些其他核心模块文件
- tmp目录是缓存目录，服务运行创建的日志文件等会存放到此目录


## 路由控制器规范

- 严格按照小写命名文件，多个单词采用连字符，示例：`user-permission`
- 简单的操作，允许控制器直接调用Model层完成

---

## 模型（Model）扩展规范

- 允许模型文件存在syncinit方法，此方法用于在同步数据结构之后运行一些初始化工作

## lib目录

- lib目录是通用模块目录
- 此目录下存放可以被其他各个部分引用的通用模块

## storage目录

- 存放上传的文件


## Token验证

- 不需安装其他扩展，Topbit.Token即可完成验证
- 示例代码：
```javascript
const Topbit = require('topbit')
const TopbitToken = Topbit.Token

const token = new TopbitToken({
  key       : 'your-32-byte-secret-key-here!!',   // 必须 32 字节（AES-256）
  expires   : 60 * 60 * 24,                       // 24 小时（单位：秒）
  refresh   : true                                // 开启自动刷新（最后 1/5 时间刷新）
})

module.exports = token   // 直接导出实例，TopbitLoader 自动识别
```

```javascript
// controller/user.js
class User {
  async post(c) {                     // POST /user/login
    // 登录验证成功后
    let userinfo = {
      uid   : 10010,
      name  : 'Alice',
      role  : 'admin',
      // expires 可单独设置更长时间
    }

    //签发token并返回
    c.to({token: token.makeToken(userinfo)})
  }
}

```
