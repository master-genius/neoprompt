# System Context: Topbit.npargv

## 1. 核心定义
**Topbit.npargv** 模块是 Node.js 命令行参数解析扩展。它基于 **Schema（定义）** + **Options（配置）** 将 `process.argv` 转换为结构化对象。

## 2. Schema 推断逻辑 (优先级最高)
当解析 Schema 中的每一个 Key 时，必须遵循以下**Type 推断优先级**（从上到下）：

1.  **显式定义**：若对象中包含 `type` 字段，直接使用。
2.  **赋值符号推断**：若 Key 包含 `=` (如 `'--port='`) -> 推断为 `'string'`。
3.  **正则/回调推断**：若对象包含 `match` 或 `callback` -> 推断为 `'string'`。
4.  **范围推断**：若对象包含 `min` 或 `max` -> 推断为 `'int'`。
5.  **默认值推断**：若对象包含 `default`，且值为 `number/string/boolean` -> 推断为对应类型。
6.  **兜底推断**：以上皆不满足 -> 推断为 `'boolean'`。

**简写规范化**：
*   `{ '-v': 'verbose' }` → `{ name: 'verbose', type: 'boolean', default: false }`
*   `{ '--flag': null }` → `{ name: '--flag', type: 'boolean' }`

## 3. 解析算法与流程
AI 在模拟解析时必须严格遵守以下流程：

### A. 配置初始化
*   **strict**: 默认为 `true`。若为 `true`，遇到类型错误/范围错误立即返回 `{ok: false}`。若为 `false`，记录错误并保留默认值继续。
*   **autoDefault**: 默认为 `true`。自动补全默认值：
    *   `int/float` → `min` 值或 `0`
    *   `string` → `''`
    *   `bool` → `false`

### B. 命令与位置参数 (重要逻辑)
1.  **子命令判定**：检查 `options.commands` 列表。
    *   若 argv 头部匹配列表中的值 → 提取为 `command`，索引位置 `+1`。
    *   若不匹配且有 `defaultCommand` → 使用默认命令，索引位置**不**变。
2.  **位置参数 ($N)**：
    *   `$1`, `$2`... 对应**非 Flag** 参数的顺序。
    *   **Offset**：若命中了子命令，`$1` 指向子命令**之后**的第一个参数。

### C. 键值对解析
*   **Key 匹配**：支持 `--key` 或 `--key=value`。
*   **Value 获取**：
    *   `bool` 类型：不需要后续参数，值为 `true`。
    *   非 `bool` 类型：
        *   若格式为 `--key=val`，取 `val`。
        *   否则取 `argv[index + 1]`。若无下一个参数，报错。
*   **未知参数处理**：
    *   以 `\` 开头（转义）：去头后放入 `list`。
    *   不以 `-` 开头（普通值）：放入 `list`。
    *   以 `-` 开头但未定义：**直接丢弃/忽略**，既不报错也不放入 `list`。

## 4. 数据结构规范

### 输入 (Function Signature)
```typescript
Topbit.npargv(
  schema: Object, 
  options?: {
    strict?: boolean,       // Default: true
    autoDefault?: boolean,  // Default: true
    commands?: string[],    // Sub-commands list
    defaultCommand?: string // Fallback command
  }
)
```

### 输出 (Return Object)
```typescript
{
  ok: boolean,        // true if no errors (or strict=false)
  message: string,    // First error message (empty if ok)
  errors: string[],   // All error messages
  command: string,    // Detected subcommand
  args: {             // Parsed key-value pairs
    [key: string]: any 
  },
  list: string[]      // Unmatched positional args (non-flag)
}
```

## 5. Few-Shot Examples (推理参照)

**场景 1: 类型推断与简写**
*   **Schema**: `{ '--port=': { min: 80 }, '-v': 'verbose' }`
*   **Input**: `node app.js --port=8080 -v`
*   **Thinking**:
    *   `--port=` 含有 `=` → type: string (但 `min` 存在，优先检查源码逻辑... *Correction*: 源码逻辑中 `if (key.includes('=')) rule.type = 'string'` 优先级高于 min/max。因此 `--port=` 会被视为 string，但在转换时 `validateAndConvert` 检查的是 rule.type。源码中 normalizeSchema 步骤：如果 key 含 `=`，type 设为 string。**注意：这是源码的一个潜在冲突点，AI 应严格按源码执行：含等于号即视为 String**)。
    *   *Self-Correction based on provided code*: 源码逻辑：`if (key.includes('=')) { rule.type = 'string' }`. 
    *   `-v` 是简写 → type: boolean, name: verbose。
*   **Result**: `{ args: { port: "8080", verbose: true } }`

**场景 2: 子命令与位置参数**
*   **Schema**: `{ '$1': { name: 'file' } }`
*   **Options**: `{ commands: ['start'] }`
*   **Input**: `node app.js start config.json`
*   **Thinking**:
    *   `start` 匹配 commands → `command: 'start'`, index 移位。
    *   `config.json` 变为 `$1`。
*   **Result**: `{ command: 'start', args: { file: 'config.json' } }`

**场景 3: 严格模式与错误**
*   **Schema**: `{ '--num': { type: 'int' } }`
*   **Options**: `{ strict: true }`
*   **Input**: `node app.js --num=abc`
*   **Thinking**:
    *   `abc` 不是 int。Strict 为 true。
*   **Result**: `{ ok: false, message: "...必须是数字类型...", args: {...} }`

---
