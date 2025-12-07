# 🚀 Topbit + NeoPG 项目架构集成规范

**这份文档定义了 Topbit Web 框架与 NeoPG 数据库扩展在本项目中的标准集成方式。请严格遵守以下架构约定：**

## 1. 标准目录结构 (Directory Structure)

项目必须遵循 **TopbitLoader** 和 **NeoPG** 的最佳组合结构：

```text
project/
├── app.js                 # 入口文件 (Entry)
├── config/
│   ├── database.js        # 数据库配置
│   └── config.js          # 服务选项配置
├── lib/                   # 通用模块
├── middleware/            # 中间件定义 (Class-based recommended)
│   ├── auth.js            # 中间件模块
│   └── parse.js           # 中间件模块
├── controller/            # 路由与控制器 (Controllers)
│   ├── __mid.js           # 全局或分组中间件定义
│   ├── user.js            # 映射为 /user/ 路由，根据文件内部的this.param拼接按照RESTful规则映射路由
│   └── api/               # 自动处理子目录路由 /api/*
│       ├── __mid.js       # /api分组下的中间件配置，仅对/api下的所有路由起作用
│       └── post.js        # 根据文件内部的this.param拼接按照RESTful规则映射路由
└── model/                 # 业务逻辑与数据模型
```

## 2. 模型定义规范 (Model Location)
*   **存放位置**：所有数据库模型文件必须存放在根目录下的 `./model` 文件夹中。
*   **加载方式**：**禁止**在代码中手动 `require` 单个模型文件。必须使用 NeoPG 的 `db.loadModels('./model')` 进行批量扫描和注册。

## 3. 服务绑定与初始化 (Service Binding - The Critical Step)

这是将数据库能力注入 Web 框架的关键步骤。必须在 `app.js` 中利用 `TopbitLoader` 提供的 `modelLoader` 钩子完成。

**标准 `app.js` 模板（请以此为准）：**

```javascript
'use strict'

process.chdir(__dirname)

const Topbit = require('topbit')
const NeoPG = require('neopg')
const { Loader } = Topbit
const config = require('./config/config.js')
/**
 * 数据库配置示例
   {
        host: 'localhost',
        port: 5432,
        database: 'my_db',
        user: 'postgres',
        password: 'password',
        max: 50,             // Connection pool size
        idle_timeout: 30,    // Idle connection timeout in seconds
        debug: false,        // Enable query logging
        schema: 'public'     // Default schema
    }
 * */
const dbconfig = require('./config/database.js')

// 1. 初始化 Web 框架
const app = new Topbit({
    debug: true,
    // 其他 Topbit 配置...
})

// 2. 仅在 Worker 进程中加载业务逻辑
if (app.isWorker) {
    new Loader({
        // Topbit 路由前缀配置
        prePath: '/api', 
        // 开启文件即路由分组
        fileAsGroup: true, 
        
        // --- 关键集成点：数据库加载与绑定 ---
        modelLoader: async (service) => {
            // A. 初始化 NeoPG 连接
            const db = new NeoPG(dbconfig)

            // B. 自动加载 ./model 目录下的所有模型
            console.log('>> Loading Database Models...')
            await db.loadModels('./model')
            
            // C. 同步表结构 (仅在开发环境或明确需要时开启 force)
            await db.sync({ force: true })

            // D. 【核心】将 db 实例挂载到 Topbit 的 service 容器上
            // 这样在 Controller 中就可以通过 ctx.service.db 访问了
            service.db = db
            
            console.log('>> Database Ready.')
        }
    }).init(app)
}

// 3. 启动服务
app.run(3003)
```

## 4. 在业务代码中调用 (Usage in Controller)

在 Controller 中，通过 `ctx.service.db` 获取数据库实例。

**示例代码：**

```javascript
// controller/api/user.js
class UserController {

    constructor() {
        //最后拼接的路由参数
        this.param = '/:id'
    }

    // GET /api/user/:id
    async get(ctx) {
        const db = ctx.service.db

        let u = await db.model('User').where('id', ctx.param.id).select('id,name,create_time').get()

        if (!u) return ctx.status(404).to('user not found')

        ctx.to(u)
    }
    
    // GET /api/user
    async list(ctx) {
        // 1. 从上下文获取 db 实例
        const db = ctx.service.db
        
        // 2. 使用 db.model('模型类名') 进行查询
        // 注意：这里是 'User'，对应 model/User.js 中的类名
        const users = await db.model('User')
                                .select('id, username, create_time')
                                .limit(20)
                                .find()
            
        ctx.to({ 
            code: 0, 
            data: users 
        })
    }

    // POST /api/user
    async post(ctx) {
        const db = ctx.service.db
        
        // 3. 混合查询示例：使用 db.sql 模板
        const { sql } = db
        const { username } = ctx.body
        
        // 插入并返回 ID
        const newUser = await db.model('User')
            .returning('id')
            .insert({ 
                username, 
                meta: { source: 'api' } 
            })
            
        ctx.to(newUser)
    }
}

module.exports = UserController
```
