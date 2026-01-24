# GoHttp 扩展开发提示词

## 项目概述
GoHttp 是一个高性能 Node.js HTTP/1.1 和 HTTP/2 客户端库，专注于低内存占用、流式 I/O 和高并发性能。它支持 HTTP/1.1 (Agent 复用) 和 HTTP/2 (Session 复用)，采用全链路流式处理。

## 核心特性
- 双协议支持：HTTP/1.1 和 HTTP/2
- 零内存积压：文件上传下载采用流式处理
- 连接池管理：内置智能 Agent 管理
- 安全性：支持 HTTPS 证书配置

## 扩展开发指导

### 1. HTTP/1.1 扩展 (基于 GoHttp 类)
- 继承 GoHttp 类以添加新功能
- 实现自定义中间件或拦截器
- 添加新的请求方法或便捷函数
- 扩展错误处理机制

### 2. HTTP/2 扩展 (基于 GoHttp2 类)
- 扩展现有 GoHttp2 类的功能
- 实现 HTTP/2 特定的优化策略
- 添加多路复用相关的高级功能
- 改进连接管理和重连机制

### 3. 通用扩展点
- 添加新的数据格式支持 (如 Protobuf, MessagePack)
- 实现缓存机制
- 添加请求/响应压缩功能
- 集成监控和日志系统
- 实现请求重试和熔断机制

## TypeScript 接口定义

```typescript
// 基础配置选项接口
interface BaseOptions {
  /** 请求超时时间 (毫秒) */
  timeout?: number;
  /** 自定义请求头 */
  headers?: Record<string, string>;
  /** 是否验证 HTTPS 证书 */
  verifyCert?: boolean;
  /** 客户端证书路径 */
  cert?: string;
  /** 客户端私钥路径 */
  key?: string;
  /** 查询参数 */
  query?: Record<string, any>;
  /** 请求体 */
  body?: any;
  /** 原始请求体 */
  rawBody?: Buffer | string;
  /** 请求方法 */
  method?: string;
  /** 编码 */
  encoding?: string;
  /** 是否显示进度条 */
  progress?: boolean;
  /** 目录路径 (用于下载) */
  dir?: string;
  /** 是否启用 SSE */
  sse?: boolean;
  /** SSE 回调函数 */
  sseCallback?: (chunk: Buffer | null, response: any) => void;
}

// HTTP/1.1 请求选项接口
interface GoHttpOptions extends BaseOptions {
  /** 文件路径 (用于上传) */
  file?: string;
  /** 表单字段名 (用于上传) */
  name?: string;
  /** 是否为下载请求 */
  isDownload?: boolean;
}

// HTTP/2 请求选项接口
interface GoHttp2Options extends BaseOptions {
  /** 路径 (HTTP/2) */
  path?: string;
  /** 路径名 (HTTP/2) */
  pathname?: string;
  /** 是否保持连接 */
  keepalive?: boolean;
  /** 重新连接延迟 */
  reconnDelay?: number;
  /** 调试模式 */
  debug?: boolean;
  /** 最大响应体大小 */
  maxBody?: number;
  /** 是否跳过前缀 */
  withoutPrefix?: boolean;
  /** 多部分表单数据 */
  multipart?: boolean;
  /** 文件列表 (用于上传) */
  files?: Record<string, string>;
  /** 表单数据 (用于上传) */
  form?: Record<string, any>;
  /** HTTP/2 会话选项 */
  options?: http2.ClientSessionOptions;
}

// 响应接口
interface HttpResponse {
  /** HTTP 状态码 */
  status: number;
  /** 响应头 */
  headers: Record<string, string>;
  /** 响应数据 */
  data?: Buffer;
  /** 数据长度 */
  length?: number;
  /** 是否成功 */
  ok: boolean;
  /** 错误信息 */
  error?: Error;
  /** 是否超时 */
  timeout: boolean;
  /** 返回文本内容 */
  text: (encoding?: string) => string;
  /** 返回 JSON 对象 */
  json: (encoding?: string) => any;
  /** 返回 Blob 对象 */
  blob: () => Buffer;
}

// GoHttp 类接口
interface GoHttp {
  /** 构造函数 */
  new(options?: GoHttpOptions): GoHttpInterface;
  
  /** 发起请求 */
  request(url: string | Record<string, any>, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** GET 请求 */
  get(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** POST 请求 */
  post(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** PUT 请求 */
  put(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** PATCH 请求 */
  patch(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** DELETE 请求 */
  delete(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** OPTIONS 请求 */
  options(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** 上传文件 */
  upload(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** 上传文件 (简写) */
  up(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** 下载文件 */
  download(url: string, options?: GoHttpOptions): Promise<boolean>;
  
  /** 传输数据 */
  transmit(url: string, options?: GoHttpOptions): Promise<HttpResponse>;
  
  /** 创建兼容实例 */
  connect(url: string, options?: GoHttpOptions): any;
}

// GoHttp2 类接口
interface GoHttp2 {
  /** 构造函数 */
  new(url: string, options?: GoHttp2Options): GoHttp2Interface;
  
  /** 发起请求 */
  request(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** GET 请求 */
  get(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** POST 请求 */
  post(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** PUT 请求 */
  put(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** PATCH 请求 */
  patch(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** DELETE 请求 */
  delete(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 上传文件 */
  upload(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 上传文件 (简写) */
  up(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 下载文件 */
  download(reqobj: GoHttp2Options): Promise<{ok: boolean, path: string}>;
  
  /** 关闭连接 */
  close(): void;
}

// HTTP/2 会话池接口
interface SessionPool {
  /** 构造函数 */
  new(url: string, options?: GoHttp2Options): SessionPoolInterface;
  
  /** 发起请求 */
  request(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** GET 请求 */
  get(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** POST 请求 */
  post(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** PUT 请求 */
  put(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 上传文件 */
  upload(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 上传文件 (简写) */
  up(reqobj: GoHttp2Options): Promise<HttpResponse>;
  
  /** 下载文件 */
  download(reqobj: GoHttp2Options): Promise<any>;
  
  /** 关闭连接池 */
  close(): void;
}

// 导出类型
module.exports = {
  GoHttp,
  GoHttp2,
  hcli: new GoHttp(),
  http2Connect: (url: string, options?: GoHttp2Options) => new GoHttp2(url, options),
  h2cli: {
    connect: (url: string, options?: GoHttp2Options) => new GoHttp2(url, options)
  }
}
```

## 扩展开发最佳实践

### 1. 性能优化
- 使用流式处理避免内存积压
- 合理管理连接池
- 实现适当的缓存策略

### 2. 错误处理
- 提供清晰的错误信息
- 实现重试机制
- 记录错误日志

### 3. 安全性
- 验证输入数据
- 支持 HTTPS 证书验证
- 防止请求伪造

### 4. 可维护性
- 遵循代码规范
- 提供充分的文档
- 编写单元测试

## 扩展示例

### 示例1: 添加请求拦截器
```javascript
class ExtendedGoHttp extends GoHttp {
  constructor(options = {}) {
    super(options);
    this.interceptors = [];
  }
  
  addInterceptor(interceptor) {
    this.interceptors.push(interceptor);
  }
  
  async request(url, options = null) {
    // 在请求前执行拦截器
    for (const interceptor of this.interceptors) {
      if (interceptor.request) {
        options = await interceptor.request(options);
      }
    }
    
    const response = await super.request(url, options);
    
    // 在响应后执行拦截器
    for (const interceptor of this.interceptors) {
      if (interceptor.response) {
        response = await interceptor.response(response);
      }
    }
    
    return response;
  }
}
```

### 示例2: 添加缓存功能
```javascript
class CachedGoHttp extends GoHttp {
  constructor(options = {}) {
    super(options);
    this.cache = new Map();
    this.cacheTTL = options.cacheTTL || 300000; // 5分钟
  }
  
  async request(url, options = null) {
    // 如果是 GET 请求且未禁用缓存
    if ((typeof url === 'string' && options?.method !== 'POST') || 
        (typeof url === 'object' && url.method !== 'POST')) {
      
      const cacheKey = typeof url === 'string' ? url : JSON.stringify(url);
      
      if (this.cache.has(cacheKey)) {
        const cached = this.cache.get(cacheKey);
        if (Date.now() - cached.timestamp < this.cacheTTL) {
          return cached.response;
        } else {
          this.cache.delete(cacheKey);
        }
      }
    }
    
    const response = await super.request(url, options);
    
    // 将 GET 请求结果缓存
    if (response.status === 200 && 
        (typeof url === 'string' || url.method === 'GET')) {
      const cacheKey = typeof url === 'string' ? url : JSON.stringify(url);
      this.cache.set(cacheKey, {
        response,
        timestamp: Date.now()
      });
    }
    
    return response;
  }
}
```

## 注意事项
- 保持向后兼容性
- 遵循 HTTP/1.1 和 HTTP/2 协议规范
- 优化内存使用，避免内存泄漏
- 确保线程安全（在 Node.js 的单线程环境中）
- 提供良好的错误处理和日志记录