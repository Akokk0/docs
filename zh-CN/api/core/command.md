# 指令 (Command)

::: tip
参见：[开发 > 交互基础 > 指令开发](../../guide/basic/command.md)
:::

指令系统是 Koishi 的核心功能之一。通过 `ctx.command()` 方法获得的是指令的实例，它含有下面的方法：

## Argv 对象

Argv 对象会作为 `cmd.action()`, `cmd.userFields()` 等方法的回调函数中的第一个参数。它具有以下的属性：

- **args:** `any[]` 参数列表
- **options:** `{}` 选项列表
- **next:** `Next` 中间件的 next 回调函数
- **session:** [`Session`](./session.md) 所在的会话对象

## 实例方法

### cmd.option(name, desc?, config?)

- **name:** `string` 选项的名字
- **desc:** `string` 选项的描述
- **config:** `OptionConfig`
  - **config.fallback:** `any` 选项的[默认值](../../guide/basic/command.md#选项的默认值)
  - **config.value:** `any` 选项的[重载值](../../guide/basic/command.md#选项的重载)
  - **config.type:** `DomainType` 选项的[类型定义](../../guide/basic/command.md#选项的临时类型)
  - **config.hidden:** `boolean` 是否[隐藏选项](../../guide/basic/command.md#隐藏指令和选项)
  - **config.notUsage:** `boolean` 是否[计入调用](../../manual/usage/command.md#速率限制)
  - **config.authority:** `number` 选项的[权限等级](../../manual/usage/command.md#权限管理)
- 返回值: `this`

为指令添加一个选项。

```ts
type DomainType = string | RegExp | ((source: string) => any)
```

### cmd.removeOption(name)

- **name:** `string` 指令的名称
- 返回值: `this`

删除一个选项。注意：如果你为一个选项注册了多个别名，则删除任何一个别名都相当于删除整个选项。

### cmd.usage(text)

- **text:** `string` 使用方法说明
- 返回值: `this`

为指令添加使用方法。多次调用此方法只会保留最后一次的定义。

### cmd.example(example)

- **example:** `text` 使用示例
- 返回值: `this`

为指令添加使用示例。多次调用此方法会一并保留并显示在帮助的最后面。

### cmd.action(action, prepend?)

- **action:** `CommandAction` 执行函数
- **prepend:** `boolean` 是否前置
- 返回值: `this`

为指令添加执行函数。

```ts
type Awaitable<T> = [T] extends [Promise<unknown>] ? T : T | Promise<T>
type CommandAction = (argv: Argv, ...args: any[]) => Awaitable<string | void>
```

### cmd.before(action, append?)

- **action:** `CommandAction` 执行函数
- **append:** `boolean` 是否后置
- 返回值: `this`

为指令添加检测函数。

### cmd.userFields(fields)

- **fields:** `FieldCollector<UserField>` 要请求的用户字段
- 返回值: `this`

如果指令需要用到用户数据，你可以提前声明，这样有助于合并多次请求，从而提高性能。
参见[按需加载](../../guide/database/builtin.md#声明所需字段)章节。

```ts
type FieldCollector<K extends string> =
  | Iterable<K>
  | ((argv: Argv, fields: Set<K>) => void)
```

### cmd.channelFields(fields)

- **fields:** `FieldCollector<ChannelField>` 要请求的频道字段
- 返回值: `this`

如果指令需要用到频道数据，你可以提前声明，这样有助于合并多次请求，从而提高性能。
参见[按需加载](../../guide/database/builtin.md#声明所需字段)章节。

### cmd.alias(name, config?)

- **name:** `string` 要设置的别名
- **config:** `Command.Alias`
  - **config.args:** `any[]` 要带的参数列表，将与传入的参数合并
  - **config.options:** `Dict` 要带的选项列表，将与传入的选项合并
- 返回值: `this`

设置指令别名。

### cmd.subcommand(name, desc?, config?)

- **name:** `string` 指令名以及可能的参数
- **desc:** `string` 指令的描述
- **config:** [`Command.Config`](./context.md#ctx-command) 指令的配置
- 返回值：`Command` 注册或修改的指令

注册或修改子指令。子指令会继承当期指令的上下文。参见[指令的多级结构](../../guide/basic/command.md#指令的多级结构)章节。

### cmd.parse(input)

- **input:** `Argv` 令牌化的输入，通常是 `Argv.parse()` 的返回值
- 返回值: `Argv` 解析结果，包含了 `args` 和 `options` 等属性

解析一段指令调用文本。

### cmd.execute(argv, next?)

- **argv:** `Argv` 执行配置
  - **argv.args:** `any[]` 指令的参数列表
  - **argv.options:** `Record<string, any>` 指令的选项
  - **argv.session:** [`Session`](./session.md) 当前的会话对象
- **next:** [`Next`](../../guide/basic/middleware.md) 所处的中间件的 `next` 回调函数
- 返回值: `Promise<string>` 执行函数的返回结果，可用于指令插值

执行当前指令。

### cmd.dispose()

- 返回值: `void`

移除当前指令及其所有子指令。
