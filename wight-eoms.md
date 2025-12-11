# Wight 框架 EOMS 项目案例指南

## 1. 项目总体结构

### 1.1 标准目录结构
```
project/
|-- config.json        # 核心配置文件 (路由、扩展、组件、模板配置)
|-- app.js             # 应用入口 (Main Function)
|-- app.css            # 全局样式
|-- pages/             # 页面目录
|   |-- home/
|       |-- home.js    # 页面逻辑
|       |-- home.html  # 结构模板
|       |-- home.css   # 页面样式
|-- components/        # 组件目录
|   |-- x-image/       # 组件命名必须带连字符 (-)
|       |-- x-image.js
|       |-- template.html
|       |-- explain.json
|       |-- static/    # 组件私有资源
|-- extends/           # 扩展库 (内置与自定义)
|-- templatesPages/    # 页面外壳模板
|-- static/            # 全局静态资源
```

### 1.2 Pages 页面结构
- 每个页面为一个子目录，包含 `.js`、`.html`、`.css` 三个文件
- 页面数量：约 30+ 个页面，涵盖首页、商品列表、商品详情、订单、用户中心等
- 页面注册在 `config.json` 中的 `pages` 数组中
- 页面生命周期：`onload`、`onshow`、`onhide`、`onunload`、`onresize`、`onscroll`、`onbottom`

### 1.3 Components 组件结构
- 高质量组件包括：image-slide、x-image、x-icon、org-info、org-goodslist、org-goods
- 组件命名遵循 kebab-case（短横线分隔）规范
- 组件继承 `w.Component` 基类，使用 Shadow DOM 实现样式隔离
- 支持属性定义、数据绑定、事件绑定和通信通道

### 1.4 Extends 扩展结构
- 扩展库包含 HTTP 请求、数据处理、时间处理、验证、工具函数等
- 通过 `<-('NAME')` 导入扩展功能，比如 let ejson = <-('ejson')
- 扩展采用模块化设计，按需加载

## 2. 高质量组件实现模式

### 2.1 ImageSlide 组件 (image-slide/)
基于触摸事件的图片轮播组件，支持多种交互方式（点击、滑动、定时）

**核心特性：**
- 属性定义：`images` (uri-json)、`height` (float)、`slidetime` (number) 等
- 触摸事件处理：通过 `touchevent` 扩展实现滑动交互
- 自动播放：使用 `setInterval` 实现定时切换
- 图片适配：根据容器尺寸和图片原始比例进行自适应缩放

```javascript
// 属性定义示例
this.properties = {
  images: { type: 'uri-json', default: [] },
  height: { type: 'float', default: 60, min: 2, max: 5000 },
  slidetime: { type: 'number', default: 5500, min: 1000, max: 90000 }
}

// 轮播实现
nextSlide(set_ratio=false) {
  // ...
  this.query(`[data-name=image_${i}]`, nd => {
    nd.className = 'image image-pre-hide'
  })
  this.cur_index = next_index
  this.showCurImage()
}
```

### 2.2 XImage 组件 (x-image/)
功能完整的图片加载组件，支持延迟加载、重试、占位符等

**核心特性：**
- 多种加载策略：支持正常加载、重试加载、备用图片
- 延迟加载：支持随机延迟或固定延迟加载
- 属性控制：`src`、`retrysrc`、`retrycount`、`delay` 等
- 加载状态：支持加载中提示、加载失败提示

```javascript
// 图片加载示例
loadImage(imgsrc, rcount=0) {
  let img = new Image()
  img.src = imgsrc
  return new Promise((rv, rj) => {
    img.onload = () => { /* 成功处理 */ }
    img.onerror = (err) => { /* 失败处理 */ }
  })
}
```

### 2.3 XIcon 组件 (x-icon/)
基于 XImage 的图标专用组件，提供多种尺寸

**核心特性：**
- 尺寸映射：`size` 属性映射到 CSS 类名
- 图标加载：复用 XImage 组件功能
- 简洁接口：针对图标场景优化后的封装

### 2.4 OrgInfo 组件 (org-info/)
企业信息展示组件，支持多种主题模板

**核心特性：**
- 多模板支持：`common`、`think`、`think-dark`、`deepdrink` 等
- 内容丰富：展示轮播图、商品列表、行业、经营项目、详细信息等
- 响应式适配：根据屏幕尺寸调整布局
- 渲染函数：采用 `renderTemplate` 模式实现内容展示

```javascript
renderTopImage(images) {
  if (['think', 'think-dark', 'deepdrink'].indexOf(this.attrs.template) >= 0) {
    return this.renderThinkImages(images)
  }
  return `<image-slide images="${ejson(images)}" radius=3></image-slide>`
}
```

### 2.5 OrgGoodslist 组件 (org-goodslist/)
商品列表组件，支持分页加载和搜索

**核心特性：**
- 分页加载：支持滚动加载更多
- 搜索功能：通过通信通道接收搜索参数
- 列表缓存：本地缓存已加载列表数据
- 加载状态：集成 loading 提示

### 2.6 OrgGoods 组件 (org-goods/)
商品详情组件，展示详细信息和规格

**核心特性：**
- 商品信息展示：名称、图片、详情、规格等
- SKU 选择：支持规格切换与价格计算
- 内容渲染：支持 Markdown 渲染、外链处理等

## 3. Extends 扩展功能详解

### 3.1 网络请求 (apicall.js)
通用网络请求封装，支持多种 HTTP 方法、请求重试、Token 管理等

**API 定义:**
```javascript
interface ApiOptions {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
  query?: Record<string, any>;   // URL 参数
  body?: Record<string, any> | string | FormData;
  headers?: Record<string, string>;
  dataType?: 'json' | 'blob' | 'text'; // 默认 json
  retry?: number;      // 重试次数
  retryDelay?: number; // 重试延迟(ms)
  coverText?: string;  // 请求时显示的全屏遮罩文字
  success?: (res: ApiResponse) => void;  // 成功回调
  fail?: (res: ApiResponse) => void;     // 失败回调
  onError?: (res: ApiResponse) => void;  // 错误回调
}

interface ApiResponse {
  ok: boolean;
  status: number;
  data: any;
  error?: Error;
  headers: Headers;
  exec: (handlers: { success?: Function, fail?: Function, onError?: Function }) => void;
}

// 使用示例
const apicall = <-('apicall');
const response = await apicall.get('/api/data', { query: { page: 1 } });
```

**快捷方法:**
- `apicall.get(url, options)` - GET 请求
- `apicall.post(url, options)` - POST 请求
- `apicall.put(url, options)` - PUT 请求
- `apicall.delete(url, options)` - DELETE 请求

### 3.2 认证管理 (token.js)
Auth Token 管理器，支持自动刷新

**API 定义:**
```javascript
const token = <-('token');

const TokenManager = {
  get(): TokenObject | null,           // 获取当前 Token 对象
  verify(k?: string): boolean,         // 验证 Token 是否有效
  refresh(refresh_api?: string, quiet?: boolean): Promise<boolean>, // 刷新 Token
  remove(): void                       // 清除 Token
};

interface TokenObject {
  token: string;
  refresh_token: string;
  expires: number;    // 过期时间
  time: number;       // 获取时间
}
```

### 3.3 数据模型 (model.js)
基于 API 的数据操作封装

**API 定义:**
```javascript
const model = <-('model');

class ApiModel extends w.Model {
  // 商品相关
  getPublicGoodsList(query={}): Promise<ApiResponse>     // 获取公开商品列表
  getGoodsInfo(id, query={}): Promise<ApiResponse>       // 获取商品详情
  getGoodsRecommand(id, query={}): Promise<ApiResponse>  // 获取商品推荐
  getGoodsHot(query={}): Promise<ApiResponse>            // 获取热门商品
  getOrgInfo(id, query={}): Promise<ApiResponse>         // 获取企业信息
}
```

### 3.4 模板渲染 (renders.js)
Array 和 Object 原型扩展，用于快速生成 HTML 字符串

**API 定义:**
```javascript
// Array.prototype.renders
arr.renders(callback: (item: T, index: number) => string, firstText?: string, endText?: string): string

// Object.prototype.renders  
obj.renders(callback: (value: any, key: string) => string, firstText?: string, endText?: string): string
```

### 3.5 HTML 标签 (htmltag.js)
HTML 标签模板函数，防止 XSS 攻击

**API 定义:**
```javascript
const htmltag = <-('htmltag');

// 防止 XSS 的模板函数
htmltag`<div>${value}</div>`           // 不转义 HTML
htmltag.ehtml`<div>${value}</div>`     // 转义 HTML

// 内部实现
function htmltag(strings, ...keys) {
  // 安全处理模板参数
  return _html(strings, keys, false)
}
```

### 3.6 DOM 查询 (querybind.js)
Element 原型扩展，提供 jQuery 风格的查询方法

**API 定义:**
```javascript
// Element.prototype.query
element.query(selector: string, callback?: (el: Element) => void): Element | null

// Element.prototype.queryAll
element.queryAll(selector: string, callback?: (el: Element) => void): NodeListOf<Element>
```

### 3.7 确认对话框 (confirm.js)
自定义确认对话框，替代原生 confirm

**API 定义:**
```javascript
const confirm = <-('confirm');

interface ConfirmOptions {
  text: string;           // 提示内容 (支持 HTML)
  callback: (args: any) => void; // 点击"确定"后的回调
  args?: any;             // 传递给 callback 的参数
  cancel?: () => void;    // 点击"取消"后的回调
  dark?: boolean;         // 深色模式
  transparent?: boolean;  // 透明背景
  buttonStyle?: [string, string]; // [确定按钮样式, 取消按钮样式]
  autoHide?: boolean;     // 点击后是否自动关闭弹窗
}

// 使用示例
confirm({
  text: '确定要删除该条目吗？',
  callback: (id) => { /* 删除逻辑 */ },
  args: item.id
});
```

### 3.8 URL 参数解析 (parseUrlParams.js)
解析 URL 参数

**API 定义:**
```javascript
const parseUrlParams = <-('parseUrlParams');
const params = parseUrlParams(); // 返回 URL 参数对象
```

### 3.9 加载状态管理 (loading.js)
全局加载状态管理器

**API 定义:**
```javascript
const loading = <-('loading');

loading.show();        // 显示加载遮罩
loading.hide();        // 隐藏加载遮罩
loading.delayHide();   // 延迟隐藏加载遮罩
```

### 3.10 时间字符串格式化 (timestr.js)
时间格式化工具

**API 定义:**
```javascript
const timestr = <-('timestr');

// 使用示例
timestr({ time: new Date(), mode: 'short' });  // "2023-12-01"
timestr({ time: new Date(), mode: 'middle' }); // "2023-12-01 14:30"
timestr({ time: new Date(), mode: 'long' });   // "2023-12-01 14:30:45"
```

### 3.11 触摸事件处理 (touchevent.js)
触摸事件处理工具类

**API 定义:**
```javascript
const TouchEvent = <-('touchevent');

class TouchEventHandle {
  constructor(options?: { key: 'page' | 'screen' | 'client' });
  onstart(event): void;   // 触摸开始
  onmove(event): object;   // 触摸移动 (返回方向信息)
  onend(event): object;    // 触摸结束 (返回方向信息)
}
```

### 3.12 Markdown 渲染 (renderMarkdown.js)
Markdown 转 HTML 渲染器

**API 定义:**
```javascript
const renderMarkdown = <-('renderMarkdown');

// 异步渲染
const result = await renderMarkdown('# 标题\n文本内容', options);
// result.ok: boolean, result.data: string
```

### 3.13 SKU 价格计算 (skuprice.js)
SKU 价格计算工具

**API 定义:**
```javascript
const skuprice = <-('skuprice');

// 价格计算
const priceInfo = skuprice(sku, { renderDiscount: true, fmt: true });
// 如果 renderDiscount 为 true，返回对象包含价格和折扣信息
// 否则返回数字价格或格式化价格字符串
```

### 3.14 数据加密/解密扩展
- `ejson.js`: JSON 数据 URL 编码
- `djson.js`: JSON 数据 URL 解码

**API 定义:**
```javascript
const ejson = <-('ejson');
const djson = <-('djson');

const encoded = ejson({ data: 'example' });  // URL 编码
const decoded = djson(encoded);              // URL 解码
```

### 3.15 图像压缩 (imageCompress.js)
图像压缩工具

**API 定义:**
```javascript
const imageCompress = <-('imageCompress');

// 压缩图像
const result = await imageCompress(file, {
  quality: 0.9,           // 压缩质量
  maxWidth: 1200,         // 最大宽度
  maxHeight: 1920,        // 最大高度
  toData: 'blob',         // 返回数据类型
  watermarkText: 'text',  // 水印文字
  watermarkImagePath: ''  // 水印图片路径
});
```

### 3.16 订单状态管理 (orderstate.js)
订单状态管理工具

**API 定义:**
```javascript
const orderstate = <-('orderstate');

// 状态定义
const states = orderstate.states;        // 订单状态
const payStates = orderstate.payStates;  // 支付状态
const stateName = orderstate.stateName;  // 状态名称映射
```

### 3.17 密码验证 (passwdcheck.js)
密码强度验证工具

**API 定义:**
```javascript
const passwdcheck = <-('passwdcheck');

const result = passwdcheck('password123');
// result.ok: boolean, result.message: string
```

### 3.18 时间选择器 (timeselect.js)
时间范围选择工具

**API 定义:**
```javascript
const timeselect = <-('timeselect');

// 选择不同时间范围
timeselect.month();       // 本月
timeselect.week();        // 本周
timeselect.season();      // 本季度
timeselect.year();        // 本年
timeselect.last_month();  // 上月
timeselect.last_week();   // 上周
// 返回包含 start_time 和 end_time 的对象
```

### 3.19 AI 聊天 (aichat.js)
AI 聊天对话工具

**API 定义:**
```javascript
const AIChat = <-('aichat');

// 创建聊天实例
const chat = new AIChat({ baseURL: 'https://api.example.com' });

// 事件监听
chat.on('message', (data) => { /* 接收到消息 */ });
chat.on('error', (err) => { /* 发生错误 */ });

// 开始聊天
await chat.startChat(query, options);
```

## 4. TemplatePages 模板与配置

### 4.1 模板文件列表
- `shopbox.html`: 商店主页面模板，包含顶部菜单、页面内容和底部空间
- `fullbox.html`: 完整页面模板，直接插入页面内容
- `commonbox.html`: 通用页面模板，带滚动条
- `orgspace.html`: 企业空间模板，包含顶部菜单
- `orderbox.html`, `spacebox.html`: 其他特定页面模板

### 4.2 模板配置示例 (config.json)
```json
{
  "templatePages": {
    "commonbox": ["login", "register", "service-protocol"],
    "fullbox": ["org-ginfo", "cart", "address", "agent", "wxuser", "order"],
    "shopbox": "*"
  }
}
```

**配置说明:**
- `commonbox`: 应用于登录、注册、协议等页面
- `fullbox`: 应用于商品详情、购物车、个人中心等页面
- `shopbox`: 应用于大部分商店页面（通配符 `*`）
- 模板通过 `__WIGHT_PAGE__` 替换位注入页面内容

### 4.3 模板特点
- 轻量级：模板设计简洁，仅包含必要的布局结构
- 响应式：使用 CSS 响应式设计适配不同设备
- 可扩展：支持通过模板变量传递参数

## 5. 开发最佳实践

### 5.1 组件开发规范
1. 属性定义：明确声明组件支持的属性类型、默认值和限制
2. 生命周期：合理使用 `init()`、`render()`、`afterRender()` 等方法
3. 事件绑定：使用 `data-on*` 属性绑定事件，避免直接在 JS 中绑定
4. 通信机制：合理使用 `channelInput` 和 `channelOutput` 进行组件间通信
5. 样式隔离：利用 Shadow DOM 实现样式隔离

### 5.2 页面开发规范
1. 数据获取：在 `onload` 或 `onshow` 中获取数据
2. 状态管理：合理使用 `w.share` 进行全局状态管理
3. 事件处理：使用 `data-on*` 绑定页面事件处理函数
4. 数据渲染：使用 `this.view()` 方法进行数据绑定和渲染

### 5.3 扩展使用规范
1. 按需导入：使用 `<-('extensionName')` 导入所需扩展
2. 工具封装：将常用功能封装到扩展中，提高复用性
3. 错误处理：在扩展中实现统一的错误处理机制
4. 类型安全：在扩展中验证参数类型，防止运行时错误