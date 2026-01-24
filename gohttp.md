# GoHttp 扩展提示词

## 概述
GoHttp 是一个高性能的 Node.js HTTP/1.1 和 HTTP/2 客户端库，专注于低内存占用、流式 I/O 和高并发性能。提供简洁的 API 来处理各种网络请求，包括文件上传下载、流式数据处理等。

## 核心特性
- 支持 HTTP/1.1 和 HTTP/2 协议
- 连接池管理和复用
- 流式文件上传下载（内存友好）
- HTTPS 证书配置和验证控制
- 内置命令行工具（httpcmd, httpbench, httpab）

## TypeScript 接口定义

```typescript
// HTTP/1.1 客户端接口
interface GoHttp {
  // 基础请求方法
  request(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  
  // HTTP 方法快捷方式
  get(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  post(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  put(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  patch(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  delete(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  options(url: string | RequestConfig, options?: RequestOptions): Promise<HttpResponse>;
  
  // 文件上传
  upload(url: string, options: UploadOptions): Promise<HttpResponse>;
  up(url: string, options: UploadFileOptions): Promise<HttpResponse>;
  
  // 文件下载
  download(url: string, options: DownloadOptions): Promise<boolean>;
  
  // 兼容性连接方法
  connect(url: string, options?: RequestOptions): HiiCompat;
}

// HTTP/2 客户端接口
interface GoHttp2 {
  // 构造函数: new GoHttp2(url: string, options?: Http2Options)
  
  // 基础请求方法
  request(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  
  // HTTP 方法快捷方式
  get(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  post(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  put(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  patch(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  delete(reqobj: Http2RequestOptions): Promise<HttpResponse>;
  
  // 文件操作
  upload(reqobj: Http2UploadOptions): Promise<HttpResponse>;
  up(reqobj: Http2UploadFileOptions): Promise<HttpResponse>;
  download(reqobj: Http2DownloadOptions): Promise<any>;
  
  // 连接管理
  close(): void;
}

// 通用请求配置
interface RequestConfig {
  protocol?: string;
  hostname?: string;
  port?: number | string;
  path?: string;
  pathname?: string;
  search?: string;
  hash?: string;
  method?: string;
  headers?: Record<string, string>;
  socketPath?: string; // Unix domain sockets
}

// 通用请求选项
interface RequestOptions {
  method?: string;
  headers?: Record<string, string>;
  query?: Record<string, any> | string;
  body?: any;
  rawBody?: Buffer | string;
  timeout?: number; // 默认 35000ms
  verifyCert?: boolean; // 默认 true
  cert?: string; // 证书路径
  key?: string; // 私钥路径
  agent?: any; // 自定义 agent
  encoding?: string;
  isDownload?: boolean;
  /** 是否启用 SSE */
  sse?: boolean;
  /** SSE 回调函数 */
  sseCallback?: (chunk: Buffer | null, response: any) => void;
}

// 上传选项
interface UploadOptions extends RequestOptions {
  files?: Record<string, string>; // 文件路径映射
  form?: Record<string, any>; // 表单数据
  multipart?: boolean;
}

// 上传文件选项
interface UploadFileOptions extends RequestOptions {
  file: string; // 文件路径
  name?: string; // 表单字段名，默认 'file'
}

// 下载选项
interface DownloadOptions extends RequestOptions {
  dir?: string; // 下载目录，默认 './'
  progress?: boolean; // 是否显示进度条
}

// HTTP/2 特定选项
interface Http2Options {
  verifyCert?: boolean; // 默认 true
  timeout?: number; // 默认 15000ms
  keepalive?: boolean; // 默认 true
  reconnDelay?: number; // 重连延迟，默认 1000ms
  debug?: boolean; // 调试模式
  maxBody?: number; // 最大响应体大小，默认 200MB
  checkServerIdentity?: Function;
}

// HTTP/2 请求选项
interface Http2RequestOptions {
  method?: string;
  path?: string;
  pathname?: string;
  headers?: Record<string, string>;
  query?: Record<string, any> | string;
  body?: any;
  timeout?: number;
  options?: any; // http2 stream options
  isDownload?: boolean;
  withoutPrefix?: boolean;
  multipart?: boolean;
  files?: Record<string, string>;
  form?: Record<string, any>;
}

// HTTP/2 上传选项
interface Http2UploadOptions extends Http2RequestOptions {
  multipart: true;
}

// HTTP/2 上传文件选项
interface Http2UploadFileOptions extends Http2UploadOptions {
  file: string;
  name?: string;
}

// HTTP/2 下载选项
interface Http2DownloadOptions extends Http2RequestOptions {
  dir?: string;
  progress?: boolean;
}

// 响应对象
interface HttpResponse {
  status: number;
  headers: Record<string, string | string[]>;
  data: Buffer;
  length: number;
  ok: boolean;
  error: Error | null;
  timeout: boolean;
  
  // 响应数据转换方法
  text(encoding?: string): string;
  json<T = any>(encoding?: string): T;
  blob(): Buffer;
}

// 兼容层接口，用于HTTP/1.1兼容HTTP/2
interface HiiCompat {
  // 属性
  host: string;
  port: string | number;
  headers: Record<string, string>;
  prefix: string;
  
  // 方法
  setHeader(key: string, val: string): HiiCompat;
  setHeader(headers: Record<string, string>): HiiCompat;
  
  // HTTP 方法
  get(opts?: RequestOptions): Promise<HttpResponse>;
  post(opts?: RequestOptions): Promise<HttpResponse>;
  put(opts?: RequestOptions): Promise<HttpResponse>;
  patch(opts?: RequestOptions): Promise<HttpResponse>;
  delete(opts?: RequestOptions): Promise<HttpResponse>;
  options(opts?: RequestOptions): Promise<HttpResponse>;
  
  // 文件操作
  upload(opts: UploadOptions): Promise<HttpResponse>;
  up(opts: UploadFileOptions): Promise<HttpResponse>;
  download(opts: DownloadOptions): Promise<any>;
}

// 导出对象
interface GoHttpExports {
  GoHttp: GoHttp;
  GoHttp2: GoHttp2;
  hcli: new GoHttp();           // HTTP/1.1 默认实例
  http2Connect: (url: string, options?: Http2Options) => new GoHttp2(url, options);
  h2cli: {
    connect: (url: string, options?: Http2Options) => new GoHttp2(url, options);
  };
}
```

## 使用示例

### HTTP/1.1 请求
```javascript
const { hcli } = require('gohttp');

// GET 请求
const res = await hcli.get('https://api.example.com/users?id=1');
console.log(res.status, res.json());

// POST JSON
const res = await hcli.post({
  url: 'https://api.example.com/login',
  body: { user: 'admin', pass: '123' }
});

// 文件上传
await hcli.up({
  url: 'https://api.example.com/upload',
  file: './video.mp4',
  name: 'file'
});

// 文件下载
await hcli.download({
  url: 'https://cdn.example.com/image.png',
  dir: './downloads',
  progress: true
});
```

### HTTP/2 请求
```javascript
const { http2Connect } = require('gohttp');

const client = http2Connect('https://http2.golang.org', {
  keepalive: true,
  verifyCert: false
});

try {
  const res1 = await client.get({ path: '/reqinfo' });
  console.log('Response 1:', res1.text());

  const res2 = await client.post({
    path: '/echo',
    body: 'Hello H2'
  });
  console.log('Response 2:', res2.text());
} finally {
  client.close();
}
```

## 开发提示
1. 使用流式处理来处理大文件，避免内存溢出
2. 合理配置超时时间和连接池参数以优化性能
3. 在生产环境中始终验证 HTTPS 证书
4. 利用内置的命令行工具进行调试和压力测试
5. 对于长时间运行的应用，注意正确关闭连接以释放资源
