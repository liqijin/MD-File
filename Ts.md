## 一、基础概念（必问）

### 1. TypeScript 是什么？和 JavaScript 的核心区别？

**核心答案**：

- TypeScript 是微软开发的 **JavaScript 超集（Superset）**，在 JS 基础上增加了**静态类型系统**，最终会编译为纯 JS 运行；

- 核心区别：

  | 维度     | TypeScript                             | JavaScript                       |
  | :------- | :------------------------------------- | :------------------------------- |
  | 类型检查 | 静态类型（编译时检查类型错误）         | 动态类型（运行时才暴露类型错误） |
  | 语言特性 | 支持接口、枚举、泛型、装饰器等扩展特性 | 无原生类型相关特性               |
  | 执行方式 | 需编译为 JS 才能运行                   | 直接在引擎中执行                 |
  | 工程化   | 强类型提升可维护性，适合大型项目       | 灵活但类型模糊，易出隐性 bug     |

### 2. TypeScript 的编译流程？

**核心答案**：

TS 编译器（tsc）的核心流程：

1. **词法分析**：将代码拆分为最小单元（如关键字、变量名、运算符）；
2. **语法分析**：生成抽象语法树（AST）；
3. **类型检查**：核心环节，验证类型合法性（如变量赋值、函数参数），不通过则抛出编译错误；
4. **AST 转换**：将 TS AST 转为 JS AST（移除类型注解、转换枚举 / 接口等）；
5. **代码生成**：输出纯 JS 代码，可选生成 `.d.ts` 类型声明文件。

### 3. TypeScript 的类型系统是 “结构类型” 还是 “名义类型”？举例说明

**核心答案**：

- TS 采用 **结构类型系统**（Structural Typing）：判断类型兼容的核心是 “结构是否匹配”，而非 “名称是否一致”；

- 对比：Java/C# 是名义类型（需显式继承 / 实现才兼容）；

- 示例：

  ```tsx
  interface User { name: string }
  class Person { name: string }
  
  const user: User = new Person(); // 合法！结构一致（都有 name: string）
  ```

### 4. TypeScript 的严格模式（strict）包含哪些子选项？开启 strict 的意义？

**核心答案**：

`strict: true` 是 tsconfig.json 的核心配置，会开启以下关键子选项：

- `strictNullChecks`：严格空检查（`null`/`undefined` 不能赋值给非空类型）；
- `noImplicitAny`：禁止隐式 `any` 类型（未注解的变量 / 参数不能推断为 any）；
- `strictFunctionTypes`：严格函数类型检查（函数参数逆变、返回值协变）；
- `strictPropertyInitialization`：类属性必须初始化或设为可选；
- 意义：强制开发者显式处理类型，避免 “any 脚本”，从源头减少类型相关 bug。

## 二、核心语法（高频）

### 1. 基本类型有哪些？和 JS 类型的区别？

**核心答案**：

- TS 基本类型：`string`/`number`/`boolean`/`null`/`undefined`/`symbol`/`bigint`/`void`/`any`/`never`；
- 新增特有的类型：
  - `void`：表示函数无返回值（`undefined` 是其子类型）；
  - `any`：关闭类型检查，任意类型可赋值 / 被赋值；
  - `never`：表示永不存在的值（如抛出错误的函数、死循环函数的返回值）；
- 区别：JS 只有运行时类型，TS 增加了编译时的类型约束，且补充了 `void`/`never` 等语义化类型。

### 2. 接口（interface）和类型别名（type）的区别？

**核心答案**：

|   维度   |               interface                |                 type                 |
| :------: | :------------------------------------: | :----------------------------------: |
| 扩展方式 |           用 `extends` 扩展            |        用 `&`（交叉类型）扩展        |
| 合并特性 | 支持同名接口自动合并（如库的类型扩展） |        不支持同名合并，会报错        |
| 适用场景 |   定义对象 / 类的形状（语义更清晰）    | 定义联合类型、交叉类型、基本类型别名 |
| 映射类型 |          支持，但需配合 type           |     天然支持复杂映射 / 条件类型      |

- 示例：

  ```tsx
  // interface 扩展
  interface User { name: string }
  interface User extends { age: number } // { name: string; age: number }
  
  // type 扩展
  type User = { name: string }
  type UserWithAge = User & { age: number }
  
  // interface 合并
  interface A { a: string }
  interface A { b: number } // 合并为 { a: string; b: number }
  ```

### 3. 泛型的作用？举例说明泛型的基本使用

**核心答案**：

- 泛型是 “类型层面的函数”，核心作用：**复用类型逻辑**，让类型具备 “参数化” 能力，避免重复定义类型；

- 基本使用：

  ```tsx
  // 1. 泛型函数（复用数组反转逻辑）
  function reverse<T>(arr: T[]): T[] {
    return arr.reverse();
  }
  const nums = reverse<number>([1,2,3]); // number[]
  const strs = reverse<string>(['a','b']); // string[]
  
  // 2. 泛型接口（通用响应数据）
  interface ApiResponse<T> {
    code: number;
    data: T;
    msg: string;
  }
  // 复用为用户响应、商品响应
  type UserResponse = ApiResponse<{ name: string; age: number }>;
  type GoodsResponse = ApiResponse<{ id: number; price: number }>;
  ```

### 4. 联合类型（|）和交叉类型（&）的区别？

**核心答案**：

- **联合类型（`|`）**：“或” 关系，类型是多个类型中的**任意一个**；

- **交叉类型（`&`）**：“且” 关系，类型是多个类型的**组合**；

- 示例：

  ```tsx
  // 联合类型
  type NumOrStr = number | string;
  let a: NumOrStr = 1; // 合法
  a = 'hello'; // 合法
  
  // 交叉类型
  type A = { a: string };
  type B = { b: number };
  type C = A & B; // { a: string; b: number }
  let c: C = { a: '1', b: 2 }; // 必须同时包含 a 和 b
  ```

### 5. 可选链（?.）、空值合并（??）、非空断言（!）的作用？

**核心答案**：

- **可选链（`?.`）**：安全访问嵌套对象属性，避免 `Cannot read property 'x' of undefined`；

- **空值合并（`??`）**：仅当左侧为 `null/undefined` 时返回右侧，区别于 `||`（会把 0/''/false 视为空）；

- **非空断言（`!`）**：告诉 TS 编译器 “该值一定不是 null/undefined”，跳过严格空检查；

- 示例：

  ```tsx
  const obj = { a: { b: 1 } };
  obj?.a?.b; // 安全访问，若 a 不存在返回 undefined
  obj.a?.c ?? '默认值'; // c 不存在，返回 '默认值'
  const num: number | undefined = 1;
  num!.toString(); // 断言 num 非空，编译通过
  ```

## 三、高级特性（区分度考点）

### 1. 类型守卫的作用？常见的类型守卫方式？

**核心答案**：

- 类型守卫：在代码块中**缩小类型范围**，让 TS 精准推断类型，避免类型断言；

- 常见方式：

  1. typeof类型守卫（判断基本类型）：

     ```tsx
     function fn(x: number | string) {
       if (typeof x === 'string') {
         x.length; // TS 推断 x 是 string
       } else {
         x.toFixed(); // TS 推断 x 是 number
       }
     }
     ```

  2. instanceof 类型守卫（判断类实例）：

     ```tsx
     class A {}
     class B {}
     function fn(x: A | B) {
       if (x instanceof A) {
         // x 是 A 实例
       }
     }
     ```

  3. 自定义类型守卫（is关键字）：

     ```tsx
     interface User { type: 'user'; name: string }
     interface Admin { type: 'admin'; role: string }
     function isUser(x: User | Admin): x is User {
       return x.type === 'user';
     }
     function fn(x: User | Admin) {
       if (isUser(x)) {
         x.name; // x 是 User
       }
     }
     ```

### 2. 内置工具类型（Partial/Required/Pick/Omit/ReturnType）的作用？

**核心答案**：

TS 内置了常用的类型工具，基于泛型实现，核心用法：

|   工具类型   |          作用           |                示例                 |                           |
| :----------: | :---------------------: | :---------------------------------: | :-----------------------: |
|  `Partial`   | 将 T 的所有属性变为可选 |      `Partial` → 所有属性可选       |                           |
|  `Required`  | 将 T 的所有属性变为必选 |      `Required` → 所有属性必选      |                           |
|    `Pick`    |  从 T 中选取 K 个属性   |         `Pick<User, 'name'          | 'age'>` → 仅保留 name/age |
|    `Omit`    |  从 T 中排除 K 个属性   |        `Omit` → 排除 address        |                           |
| `ReturnType` | 获取函数 T 的返回值类型 | `ReturnType<() => number>` → number |                           |

- 示例：

  ```tsx
  interface User {
    name: string;
    age: number;
    address?: string;
  }
  type UserPartial = Partial<User>; // 所有属性可选
  type UserPick = Pick<User, 'name'>; // { name: string }
  type UserOmit = Omit<User, 'age'>; // { name: string; address?: string }
  ```

### 3. 条件类型的语法和作用？举例说明

**核心答案**：

- 条件类型：类似 JS 的 `三元表达式`，实现**类型层面的条件判断**，语法：`T extends U ? X : Y`；

- 作用：实现复杂的类型逻辑（如类型过滤、类型映射）；

- 示例：

  ```tsx
  // 1. 基础条件类型（判断类型是否为数组）
  type IsArray<T> = T extends Array<any> ? true : false;
  type A = IsArray<number[]>; // true
  type B = IsArray<string>; // false
  
  // 2. 分布式条件类型（过滤类型）
  type FilterNumber<T> = T extends number ? T : never;
  type C = FilterNumber<string | number | boolean>; // number
  ```

### 4. 装饰器（Decorator）的作用？常用装饰器类型？

**核心答案**：

- 装饰器是 TS 的高级特性（需开启 `experimentalDecorators`），本质是**特殊函数**，用于修饰类、方法、属性、参数；

- 常用装饰器类型：

  1. 类装饰器：修饰整个类，参数是类的构造函数；

     ```tsx
     function LogClass(target: Function) {
       console.log('类被装饰：', target.name);
     }
     @LogClass
     class User {} // 输出：类被装饰：User
     ```

     

  2. 方法装饰器：修饰类的方法，参数是「类原型、方法名、方法描述符」；

     ```tsx
     function LogMethod(target: any, key: string, descriptor: PropertyDescriptor) {
       const original = descriptor.value;
       descriptor.value = function(...args: any[]) {
         console.log('方法调用：', key);
         return original.apply(this, args);
       };
     }
     class User {
       @LogMethod
       say() { console.log('hello'); }
     }
     new User().say(); // 输出：方法调用：say → hello
     ```

## 四、工程实践（落地考点）

### 1. tsconfig.json 核心配置项？

**核心答案**：

```json
{
  "compilerOptions": {
    "target": "ES2020", // 编译目标 JS 版本
    "module": "ESNext", // 模块系统（ESModule/CommonJS）
    "moduleResolution": "NodeNext", // 模块解析策略（兼容 Node.js）
    "strict": true, // 开启严格模式（核心！）
    "jsx": "react-jsx", // React 项目的 JSX 编译模式
    "sourceMap": true, // 生成 sourceMap 方便调试
    "resolveJsonModule": true, // 允许导入 JSON 文件
    "esModuleInterop": true, // 兼容 CommonJS 模块（如 require 导入）
    "baseUrl": "./", // 基础路径
    "paths": { "@/*": ["src/*"] }, // 路径别名（需配合构建工具）
    "outDir": "./dist", // 编译输出目录
    "declaration": true // 生成 .d.ts 类型声明文件（库开发必开）
  },
  "include": ["src/**/*"], // 要编译的文件
  "exclude": ["node_modules"] // 排除的文件
}
```

### 2. 如何处理第三方库无类型声明（找不到 .d.ts）？

**核心答案**：

1. 安装社区类型声明：`npm i @types/xxx -D`（如 `@types/lodash`）；

2. 自定义类型声明：在项目根目录创建 typings/xxx.d.ts：

   ```tsx
   // 声明无类型的模块
   declare module 'xxx'; // 简单声明，模块类型为 any
   // 或精准声明
   declare module 'xxx' {
     export function fn(): void;
   }
   ```

3. 临时关闭检查：用 `// @ts-ignore` 忽略单行错误（不推荐大规模使用）。

### 3. TypeScript 项目中如何处理动态导入 / 异步函数的类型？

**核心答案**：

1. 动态导入：用 import() 结合泛型指定模块类型；

   ```tsx
   type ModuleType = { fn: () => number };
   const module = await import<ModuleType>('./module');
   ```

2. 异步函数：用 Promise<T> 指定返回值类型；

   ```tsx
   async function fetchUser(): Promise<{ name: string; age: number }> {
     const res = await fetch('/api/user');
     return res.json();
   }
   ```

## 五、常见坑点（避坑考点）

### 1. 为什么 `const arr = [1, '2']` 推断为 `(number | string)[]`，而非 `[number, string]`？如何解决？

**核心答案**：

- TS 默认将数组字面量推断为 “联合类型数组”，而非 “元组”；
- 解决方式：
  1. 显式注解为元组：`const arr: [number, string] = [1, '2'];`；
  2. 用 `as const` 断言为只读元组：`const arr = [1, '2'] as const;`。

### 2. 为什么 `null`/`undefined` 赋值给普通类型会报错？如何处理？

**核心答案**：

- 开启 `strictNullChecks` 后，`null`/`undefined` 是独立类型，不能赋值给非空类型；
- 处理方式：
  1. 显式允许空类型：`let num: number | null = null;`；
  2. 非空断言：`num!.toFixed();`（确保运行时非空）；
  3. 可选链：`num?.toFixed();`（安全访问）。

### 3. 泛型中为什么不能直接使用 `T.length`？如何解决？

**核心答案**：

- 泛型 `T` 是任意类型，TS 无法确定 `T` 有 `length` 属性，因此报错；

- 解决方式：用泛型约束限定 T 的范围：

  ```tsx
  function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
  }
  getLength('hello'); // 合法（string 有 length）
  getLength([1,2]); // 合法（数组有 length）
  ```

## 总结

1. **核心基础**：TS 核心是静态类型系统，`strict` 模式是保障类型安全的关键，需掌握基础类型、接口 / 类型别名、泛型的核心用法；
2. **高级特性**：类型守卫、内置工具类型、条件类型是区分高级开发者的考点，重点是 “用类型逻辑解决实际问题”；
3. **工程实践**：`tsconfig.json` 配置、第三方库类型处理、装饰器使用是落地关键，需结合项目理解；
4. **避坑要点**：严格空检查、元组推断、泛型约束是高频踩坑点，掌握对应的解决方式即可应对。

# 一、TypeScript 深入原理（高级底层，面试核心区分点）

## 1.1 类型擦除（Type Erasure）完整机制

### 核心原理

TS 是**编译时类型系统**，所有类型信息（泛型、interface、type、联合类型）在编译为 JS 时**完全被擦除**，仅保留可执行 JS 逻辑；唯一例外是通过 `reflect-metadata` 手动注入的运行时元数据。

### 关键结论

1. 泛型在运行时不存在：`function identity(arg:T):T` 编译后就是 `function identity(arg){return arg}`
2. 无法通过 `typeof`/`instanceof` 判断泛型类型
3. 接口 / 类型别名不会生成任何运行时代码
4. 装饰器元数据需依赖 `reflect-metadata` 垫片（NestJS/Angular 核心依赖）

### 面试追问

> Q：类型擦除导致运行时无法获取类型，如何解决？
>
> A：结合**运行时校验库（Zod/Joi）**+ 类型守卫，或传入构造函数 / 类型标识。

## 1.2 协变、逆变、不变、双向协变（类型变体核心原理）

TS 函数类型兼容遵循**类型变体规则**，是高级类型设计的底层基础：

- **协变（Covariance）**：子类型可赋值给父类型（TS 默认：对象、返回值）
- **逆变（Contravariance）**：父类型可赋值给子类型（TS 仅函数参数，`strictFunctionTypes:true` 时强制）
- **不变（Invariant）**：只有自身可赋值（最安全）
- **双向协变**：非严格模式下函数参数同时支持协变 + 逆变（不安全）

### 核心示例

```tsx
type Animal = { name: string }
type Dog = { name: string; bark: () => void }

// 函数返回值：协变（Dog → Animal 合法）
type GetAnimal = () => Animal
const getDog: () => Dog = () => ({ name: '旺财', bark: () => {} })
const fn1: GetAnimal = getDog // ✅ 合法

// 函数参数：逆变（Animal → Dog 合法，strictFunctionTypes 开启）
type FeedDog = (dog: Dog) => void
const feedAnimal: (animal: Animal) => void = (a) => console.log(a.name)
const fn2: FeedDog = feedAnimal // ✅ 合法
```

### 关键配置

`strictFunctionTypes: true` 强制函数参数**仅支持逆变**，修复历史类型漏洞，是生产环境必开配置。

## 1.3 分布式条件类型（Distributive Conditional Types）

### 底层原理

当**裸类型参数**（T 而非 `T[]`/`readonly T`）作为条件类型左侧时，TS 会自动**将联合类型拆开分发执行**，最后合并结果。

### 触发条件

- 条件类型：`T extends U ? X : Y`
- T 是**裸类型参数**（无包装）
- T 是联合类型

### 关闭分布式

用数组 / 元组包装 T：`[T] extends [U] ? X : Y`

### 经典应用：联合类型转交叉类型（面试高频手写题）

```tsx
// 利用逆变特性实现 UnionToIntersection
type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends ((k: infer I) => void) ? I : never

type Res = UnionToIntersection<{a:number} | {b:string}> // {a:number} & {b:string}
```

## 1.4 结构类型系统 vs 名义类型系统

### 核心原理

TS 是**结构类型系统（鸭式辨型）**：**类型兼容只看结构，不看名称 / 声明关系**；

Java/C# 是名义类型：必须显式继承 / 实现才兼容。

### 坑点：多余属性检查

```tsx
interface User { name: string }
const u: User = { name: '张三', age: 20 } // ❌ 报错：多余属性age
// 原因：对象字面量赋值时会触发严格多余属性检查，避免笔误
```

## 1.5 声明合并底层规则

TS 支持多种声明自动合并，是库类型设计的核心：

1. **接口合并**：同名接口属性合并，方法重载，后定义优先级高
2. **命名空间合并**：合并导出成员
3. **函数 + 函数**：重载
4. **类 + 接口**：类实现接口结构
5. **禁止合并**：类与类、type 与 type 不支持合并

# 二、TypeScript 高级运用（工程落地 + 类型体操）

## 2.1 高级泛型编程（类型体操，手写高频）

### 2.1.1 递归工具类型（深度操作）

```tsx
// 深度 Partial
type DeepPartial<T> = T extends object ? {
  [K in keyof T]?: DeepPartial<T[K]>
} : T

// 深度 Readonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: DeepReadonly<T[K]>
}
```

### 2.1.2 联合类型过滤 / 转换

```tsx
// 提取函数类型
type ExtractFunction<T> = T extends (...args: any[]) => any ? T : never
// 排除对象类型
type ExcludeObject<T> = T extends object ? never : T
```

## 2.2 类型守卫高级：断言函数（Asserts）

解决**函数内窄化后，外部仍无法识别类型**的问题，比普通类型谓词更强：

```tsx
// 断言 value 是 string，否则抛出错误
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') throw new Error('不是字符串')
}

function fn(input: unknown) {
  assertIsString(input)
  input.length // ✅ TS 自动推断为 string
}
```

## 2.3 装饰器与元编程（reflect-metadata）

### 核心原理

1. 装饰器执行顺序：**参数装饰器 → 方法 / 属性装饰器 → 类装饰器**
2. 元数据存储：`Reflect.defineMetadata`/`getMetadata` 突破类型擦除，实现运行时类型信息

### 实战场景（NestJS 核心原理）

```tsx
import 'reflect-metadata'
const CONTROLLER_KEY = 'controller'

// 类装饰器
function Controller(path: string) {
  return (target: any) => {
    Reflect.defineMetadata(CONTROLLER_KEY, path, target)
  }
}

@Controller('/user')
class UserController {}

// 运行时获取元数据
console.log(Reflect.getMetadata(CONTROLLER_KEY, UserController)) // /user
```

## 2.4 TS 工程化高级：Project References（Monorepo 必备）

### 核心作用

解决大型项目 / Monorepo 中**增量编译、模块隔离、跨包类型共享**问题，大幅提升编译速度。

### 配置示例



```json
// tsconfig.base.json 基础配置
// packages/a/tsconfig.json
{
  "composite": true, // 开启项目引用
  "outDir": "../../dist/a",
  "references": [{ "path": "../b" }] // 依赖 packages/b
}
```

## 2.5 框架高级类型实践

### Vue3：defineProps 泛型组件（高级用法）

```tsx
<script setup lang="ts" generic="T">
const props = defineProps<{
  data: T[]
  renderItem: (item: T) => VNode
}>()
</script>
```

### React：泛型 HOC 与 Props 类型推导

```tsx
type WithLoadingProps<P> = P & { loading: boolean }
function withLoading<P extends object>(Component: React.FC<P>) {
  return (props: WithLoadingProps<P>) => {
    if (props.loading) return <div>加载中</div>
    return <Component {...props as P} />
  }
}
```

## 2.6 unknown/never/any 高级边界使用

1. **any**：关闭所有类型检查（高危，禁止滥用）
2. **unknown**：安全的未知类型，必须窄化后使用（推荐）
3. **never**：穷尽性检查，防止遗漏联合类型分支

```tsx
type Tab = 'home' | 'user'
function handleTab(tab: Tab) {
  switch (tab) {
    case 'home': break
    case 'user': break
    default:
      const _exhaustive: never = tab // ✅ 新增tab类型会在此报错，穷尽性检查
  }
}
```

# 三、TS 高级常见问题与生产解决方案

## 3.1 泛型推断失败 / 约束失效

### 问题 1：泛型无法访问 `length`

```tsx
function getLength<T>(arg: T) {
  return arg.length // ❌ 报错：T 无length属性
}
```

**解决方案**：泛型约束 `extends`

```tsx
function getLength<T extends { length: number }>(arg: T): number {
  return arg.length
}
```

### 问题 2：分布式条件类型意外触发

**现象**：联合类型被自动分发，不符合预期

**解决方案**：关闭分布式（用数组包装）

```tsx
// 错误：分布式触发
type IsString<T> = T extends string ? true : false
type R = IsString<string | number> // boolean

// 正确：关闭分布式
type IsString<T> = [T] extends [string] ? true : false
type R = IsString<string | number> // false
```

## 3.2 strictNullChecks 严格空报错（高频）

### 问题 1：变量可能为 null/undefined

**解决方案**：

1. 可选链 `?.` + 空值合并 `??`
2. 非空断言 `!`（确认运行时非空）
3. 类型守卫窄化

### 问题 2：类属性未初始化（strictPropertyInitialization）

```tsx
class User {
  name: string // ❌ 报错：未初始化
}
```

**解决方案**：

1. 构造函数初始化
2. 明确可选 `name?: string`
3. 明确赋值断言 `name!: string`

## 3.3 交叉类型产生 `never`

### 问题原因

同一属性存在冲突类型，交叉后为 never：

```tsx
type A = { id: string }
type B = { id: number }
type C = A & B // id: never
```

**解决方案**：

1. 修复业务类型冲突
2. 用 `Omit` 剔除冲突属性再交叉

## 3.4 类型擦除导致运行时类型判断失效

### 问题

泛型 T 编译后消失，无法运行时判断 `T` 类型

**解决方案**：

1. 传入构造函数：`function fn(cls: new () => T)`
2. 运行时校验库：Zod/Joi/ClassValidator

```tsx
import { z } from 'zod'
const UserSchema = z.object({ name: z.string() })
type User = z.infer<typeof UserSchema>

// 运行时+编译时双安全
function parseUser(data: unknown): User {
  return UserSchema.parse(data)
}
```

## 3.5 第三方库无类型声明 / 类型错误

### 解决方案

1. 优先安装 `@types/xxx`
2. 项目根目录创建 `typings/index.d.ts` 自定义声明：

```tsx
// 无类型模块声明
declare module 'untyped-lib' {
  export function hello(): string
}
// 全局类型扩展
declare global {
  interface Window { $$track: (msg: string) => void }
}
```

1. 用 `skipLibCheck: true` 跳过第三方库类型检查（编译提速）

## 3.6 Vite/Webpack 中 TS 路径别名不生效

### 问题

tsconfig paths 配置后，构建工具识别不到

**解决方案**：

1. TS 仅负责类型检查，构建工具需单独配置别名
2. Vite: `vite.config.ts` 配置 `resolve.alias`
3. Webpack: `resolve.alias` 配合 `tsconfig-paths-webpack-plugin`

## 3.7 递归类型栈溢出

### 问题

深度递归类型（如 DeepPartial）处理超大对象时报栈溢出

**解决方案**：

1. 限制递归深度
2. 用惰性计算 / 条件中断递归

# 四、高级面试高频追问（必背）

1. 协变逆变在实际项目中解决了什么问题？
2. 如何实现联合类型转交叉类型？原理是什么？
3. 类型擦除对前端框架设计有什么影响？
4. 为什么 strictFunctionTypes 是必开配置？
5. 大型 Monorepo 中如何做 TS 类型隔离与增量编译？
6. unknown 比 any 安全在哪里？never 的实际业务用途？



## 一、深入解析：类型擦除（Type Erasure）

### 1.1 类型擦除核心原理

TS 是**编译时类型系统**，编译器仅在编译阶段校验类型，生成 JS 时会**完全移除所有类型相关代码**（泛型、interface、type、类型注解、联合类型等），最终运行的代码无任何类型信息。

#### 编译前后对比（直观理解）

```tsx
// TS 源码（含泛型、类型注解、接口）
interface User {
  name: string;
  age: number;
}

function getUser<T extends User>(user: T): T {
  return user;
}

const user: User = getUser({ name: "张三", age: 20 });
```

```js
// 编译后 JS 代码（所有类型信息被擦除）
function getUser(user) {
  return user;
}

const user = getUser({ name: "张三", age: 20 });
```

#### 关键结论

1. 泛型仅存在于编译阶段：运行时无法通过 `typeof`/`instanceof` 判断泛型参数 `T` 的类型；
2. 接口 / 类型别名不生成任何运行时代码：仅作为编译时的 “语法约束”；
3. 类型注解（如 `: User`）、类型断言（如 `as User`）完全消失；
4. 唯一例外：通过 `reflect-metadata` 手动注入的元数据（可保留部分类型信息到运行时）。

### 1.2 类型擦除的生产坑点 & 解决方案

#### 坑点 1：运行时无法判断泛型类型

**场景**：封装通用请求函数，希望根据泛型 `T` 自动校验返回值类型，但运行时无泛型信息。

```tsx
// 问题代码：泛型 T 仅编译时有效，运行时无法获取
async function request<T>(url: string): Promise<T> {
  const res = await fetch(url);
  const data = await res.json();
  // 无法基于 T 校验 data 类型（T 已被擦除）
  return data as T; // 仅编译时断言，运行时无校验
}

// 调用：返回值类型错误（如接口返回 { name: 123 }），编译时无法发现
const user = await request<User>("/api/user");
```

**解决方案**：结合「编译时类型 + 运行时校验」（Zod/Valibot 库）

```tsx
import { z } from "zod";

// 1. 定义运行时校验schema（同时推导编译时类型）
const UserSchema = z.object({
  name: z.string(),
  age: z.number(),
});
// 2. 从schema推导TS类型（无需重复定义）
type User = z.infer<typeof UserSchema>;

// 3. 通用请求函数：编译时类型约束 + 运行时校验
async function request<T>(url: string, schema: z.ZodSchema<T>): Promise<T> {
  const res = await fetch(url);
  const data = await res.json();
  // 运行时校验：不符合schema直接抛出错误
  return schema.parse(data);
}

// 调用：编译时提示类型，运行时校验数据
const user = await request("/api/user", UserSchema);
```

#### 坑点 2：装饰器无法直接获取参数类型

**场景**：用装饰器实现接口参数校验，但类型擦除导致无法获取参数的 TS 类型。

**解决方案**：`reflect-metadata` 注入元数据（突破类型擦除）

```tsx
import "reflect-metadata";

// 定义元数据key
const PARAM_TYPE_KEY = "design:paramtypes";

// 装饰器：记录参数类型
function Validate() {
  return (target: any, methodName: string) => {
    // 获取参数类型（通过reflect-metadata注入的元数据）
    const paramTypes = Reflect.getMetadata(PARAM_TYPE_KEY, target, methodName);
    console.log("参数类型：", paramTypes); // [String, Number]
  };
}

class UserService {
  @Validate()
  addUser(name: string, age: number) {}
}

// 编译时：TS 注入参数类型元数据；运行时：可获取类型构造函数
new UserService().addUser("张三", 20);
```

### 1.3 类型擦除的设计取舍

TS 选择类型擦除的核心原因：

1. **兼容 JS 运行时**：不引入任何 TS 专属运行时，编译后的 JS 可在任意环境运行；
2. **性能开销为 0**：无类型信息的运行时代码体积更小、执行更快；
3. **灵活性**：不限制运行时逻辑，仅强化编译时约束。

## 二、深入解析：结构类型系统

### 2.1 核心规则：鸭式辨型（Duck Typing）

TS 判断类型兼容的唯一标准：**只要两个类型的 “结构” 完全匹配（属性 / 方法一致），即使名称 / 声明方式不同，也视为兼容**（区别于 Java/C# 的 “名义类型系统”）。

#### 基础示例

```tsx
// 两个完全独立的类型
interface Cat {
  name: string;
  meow(): void;
}

class Dog {
  name: string;
  meow() {
    console.log("假猫叫");
  }
}

// 结构匹配 → 兼容
const cat: Cat = new Dog(); // ✅ 编译通过
```

### 2.2 结构类型的核心 “陷阱”（生产高频坑点）

#### 陷阱 1：多余属性检查的 “例外逻辑”

结构类型允许 “超集赋值给子集”，但**对象字面量直接赋值**时会触发 “严格多余属性检查”（防止笔误），导致看似矛盾的结果：

```tsx
interface User {
  name: string;
}

// 场景1：变量赋值（超集 → 子集）→ 兼容
const userObj = { name: "张三", age: 20 };
const u1: User = userObj; // ✅ 编译通过

// 场景2：字面量直接赋值 → 严格检查（多余属性报错）
const u2: User = { name: "张三", age: 20 }; // ❌ 报错：对象字面量只能指定已知属性

// 场景3：绕过严格检查（类型断言）
const u3: User = { name: "张三", age: 20 } as User; // ✅ 编译通过
```

**坑点解析**：严格多余属性检查仅针对 “对象字面量直接赋值”，是 TS 为减少笔误的 “语法糖”，并非结构类型的核心规则。

#### 陷阱 2：隐式兼容导致的逻辑错误

**场景**：两个业务含义完全不同的类型，因结构巧合一致导致隐式兼容，引发 bug。

```tsx
// 订单ID：业务含义是“订单编号”
interface OrderId {
  value: string;
}

// 用户ID：业务含义是“用户编号”
interface UserId {
  value: string;
}

// 函数：接收订单ID
function getOrder(orderId: OrderId) {
  console.log(`查询订单：${orderId.value}`);
}

// 问题：用户ID结构与订单ID一致 → 隐式兼容
const userId: UserId = { value: "u123" };
getOrder(userId); // ✅ 编译通过，但业务逻辑错误（用用户ID查订单）
```

**解决方案：名义化类型（Nominal Typing）改造**

通过添加 “私有属性 / 唯一符号” 破坏结构兼容性，实现类似名义类型的效果：

```tsx
// 方案1：唯一符号（推荐，无运行时开销）
declare const __brand: unique symbol;

type OrderId = {
  [__brand]: "order"; // 唯一标识，破坏结构兼容
  value: string;
};

type UserId = {
  [__brand]: "user";
  value: string;
};

// 编译报错：结构不兼容（__brand 不同）
getOrder(userId); // ❌ 类型 "{ [__brand]: "user"; value: string; }" 不能赋值给类型 "{ [__brand]: "order"; value: string; }"
```

#### 陷阱 3：函数参数的双向协变（非严格模式）

结构类型下，函数参数的兼容性默认是 “双向协变”（既协变又逆变），容易导致类型不安全：

```tsx
interface Animal {
  name: string;
}

interface Dog extends Animal {
  bark(): void;
}

// 函数1：接收 Animal 类型参数
type F1 = (a: Animal) => void;
// 函数2：接收 Dog 类型参数（更具体）
type F2 = (d: Dog) => void;

// 非严格模式（strictFunctionTypes: false）→ 双向协变 → 兼容
const f1: F1 = (d: Dog) => d.bark(); // ✅ 编译通过
// 运行时风险：若调用 f1({ name: "猫" })，会执行 d.bark() 导致报错
```

**解决方案**：开启 `strictFunctionTypes: true`（强制函数参数 “逆变”）

```tsx
// 严格模式下：函数参数仅支持逆变 → 报错
const f1: F1 = (d: Dog) => d.bark(); // ❌ 报错：类型 "(d: Dog) => void" 不能赋值给类型 "(a: Animal) => void"
```

### 2.3 结构类型在框架中的核心应用

#### 案例：Vue3 组件 Props 的结构类型校验

Vue3 的 `defineProps` 基于结构类型系统实现，无需显式继承，只需结构匹配即可复用类型：

```vue
<script setup lang="ts">
// 通用按钮Props类型
type BaseButtonProps = {
  type: "primary" | "secondary";
  size: "small" | "large";
};

// 业务按钮：扩展BaseButtonProps（结构匹配即可）
const props = defineProps<BaseButtonProps & { loading: boolean }>();
</script>
```

#### 案例：React 高阶组件（HOC）的类型兼容

React HOC 利用结构类型系统，无需修改原组件即可扩展 Props：

```tsx
import React from "react";

// 原组件：结构为 { text: string }
const Button = ({ text }: { text: string }) => <button>{text}</button>;

// HOC：扩展Props（结构合并）
function withLoading<P>(Component: React.FC<P>) {
  return (props: P & { loading: boolean }) => {
    if (props.loading) return <div>加载中</div>;
    return <Component {...props as P} />;
  };
}

// 结构兼容 → 无缝扩展
const LoadingButton = withLoading(Button);
<LoadingButton text="提交" loading={false} />; // ✅ 编译通过
```

## 三、类型兼容性高级问题（协变 / 逆变 / 不变）

### 3.1 核心概念（结合生产场景）

|        类型变体        |    核心规则     |               适用场景                | 安全级别 |
| :--------------------: | :-------------: | :-----------------------------------: | :------: |
|   协变（Covariance）   | 子类型 → 父类型 |         对象属性、函数返回值          |   安全   |
| 逆变（Contravariance） | 父类型 → 子类型 | 函数参数（strictFunctionTypes: true） |   安全   |
| 双向协变（Bivariance） | 子类型 ↔ 父类型 |        函数参数（非严格模式）         |  不安全  |
|   不变（Invariant）    | 仅自身类型兼容  |      自定义类型（如名义化类型）       |  最安全  |

### 3.2 生产高频兼容性问题 & 解决方案

#### 问题 1：数组的协变风险

数组默认是协变的，可能导致 “存入子类型，取出父类型” 的运行时错误：

```tsx
interface Animal { name: string }
interface Dog extends Animal { bark(): void }
interface Cat extends Animal { meow(): void }

// 协变：Dog[] → Animal[] 兼容
const animals: Animal[] = [new Dog(), new Dog()];
// 运行时风险：向 Animal[] 中存入 Cat，破坏原数组类型
animals.push(new Cat());
// 取出时假设是 Dog，调用 bark() 报错
(animals[0] as Dog).bark(); // ❌ 运行时错误（实际是 Cat）
```

**解决方案**：使用 `ReadonlyArray` 或不变数组类型

```tsx
// ReadonlyArray 是不变的 → 禁止修改
const animals: ReadonlyArray<Dog> = [new Dog()];
animals.push(new Cat()); // ❌ 报错：ReadonlyArray 无 push 方法
```

#### 问题 2：函数返回值协变的合理使用

函数返回值协变是安全且常用的，可用于扩展函数返回值类型：

```tsx
// 父类型返回值
type GetAnimal = () => Animal;
// 子类型返回值（协变 → 兼容）
const getDog: () => Dog = () => ({ name: "旺财", bark: () => {} });
const fn: GetAnimal = getDog; // ✅ 安全：返回 Dog 符合 Animal 结构
```

#### 问题 3：函数参数逆变的工程价值

开启 `strictFunctionTypes` 后，函数参数强制逆变，可避免 “参数类型过窄” 导致的运行时错误：

```tsx
// 严格模式下：函数参数逆变
type FeedDog = (dog: Dog) => void;
// 父类型参数（Animal）→ 子类型参数（Dog）兼容
const feedAnimal: (animal: Animal) => void = (a) => console.log(a.name);
const fn: FeedDog = feedAnimal; // ✅ 安全：Animal 包含 Dog 的所有属性

// 反向赋值（子类型参数 → 父类型参数）→ 报错（不安全）
const feedDog: (dog: Dog) => void = (d) => d.bark();
const fn2: (a: Animal) => void = feedDog; // ❌ 报错：Animal 无 bark 方法
```

## 四、TS 高级运用实际案例（可直接落地）

### 案例 1：Monorepo 类型隔离与增量编译（Project References）

**场景**：大型项目拆分为多个包（如 `core`/`ui`/`api`），需实现：

1. 包之间类型隔离，避免循环依赖；
2. 增量编译，提升编译速度。

**实现步骤**：

1. 根目录 `tsconfig.base.json`（基础配置）：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

1. 子包 `packages/core/tsconfig.json`（开启 composite）：

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../../dist/core",
    "rootDir": "./src",
    "composite": true // 开启项目引用
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

1. 子包 `packages/ui/tsconfig.json`（依赖 core）：

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../../dist/ui",
    "rootDir": "./src",
    "composite": true
  },
  "references": [{ "path": "../core" }], // 声明依赖 core 包
  "include": ["src/**/*"]
}
```

1. 编译命令（增量编译）：

bash

```bash
tsc --build packages/ui --incremental
```

**核心价值**：仅编译修改的包，大型项目编译速度提升 50%+，且类型依赖清晰。

### 案例 2：类型体操落地 —— 业务通用类型工具

**场景**：后端返回的枚举值（数字），需转换为前端联合类型，并实现运行时校验。

**实现代码**：

```tsx
// 1. 后端枚举（数字）
const OrderStatus = {
  PENDING: 1,
  PAID: 2,
  SHIPPED: 3,
} as const;

// 2. 类型体操：枚举值转联合类型
type OrderStatusValue = (typeof OrderStatus)[keyof typeof OrderStatus]; // 1 | 2 | 3

// 3. 类型体操：过滤有效状态（排除废弃状态）
type ValidOrderStatus = Exclude<OrderStatusValue, 3>; // 1 | 2

// 4. 运行时校验函数（结合类型守卫）
function isValidOrderStatus(status: number): status is ValidOrderStatus {
  return [OrderStatus.PENDING, OrderStatus.PAID].includes(status);
}

// 5. 业务使用
function handleOrder(status: number) {
  if (
```