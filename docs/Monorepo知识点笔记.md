# Monorepo 工程化知识点笔记

## 什么是 Monorepo

Monorepo（单仓库）是一种将多个相关项目放在同一个代码仓库中的管理方式。与传统的 Mutirepo（多仓库）相比，它便于统一管理和跨项目协作。

主流的 Monorepo 管理工具包括：Lerna、Nx、Turborepo 和 pnpm。

## pnpm Monorepo

pnpm 通过 `workspace` 实现 Monorepo 管理。

- **配置文件**: `pnpm-workspace.yaml`
  ```yaml
  packages:
    - "packages/*"
    - "apps/*"
  ```
- **执行命令**:
  - 工程级: `pnpm -w [command]` 或 `pnpm --workspace-root [command]`
  - 子包级: `pnpm -C 子包路径 [command]`

## 环境版本锁定

确保团队成员使用相同的环境版本，避免“在我机器上能跑”的问题。

- **配置**:
  - `package.json` 中的 `engines` 字段
  - `.npmrc` 中的 `engine-strict=true`

## TypeScript

实现类型安全，提升代码质量和开发体验。

- **配置**: 通过 `tsconfig.json` 统一管理，各子项目继承根配置。

## 代码风格与质量

### Prettier (代码格式化)

- 统一代码风格，自动格式化。
- 配置文件: `prettier.config.js`，通过脚本 `pnpm lint:prettier` 执行。

### ESLint (代码检查)

- 检查代码逻辑错误和风格问题。
- 配置文件: `eslint.config.js`，通过脚本 `pnpm lint:eslint` 执行。

### 拼写检查 (CSpell)

- 检查代码中的单词拼写错误。
- 配置文件: `cspell.json`，通过脚本 `pnpm lint:spellcheck` 执行。

## Git 提交规范

### Commitizen (交互式提交)

- 提供友好的交互界面，规范 Git 提交信息。
- 配置文件: `commitlint.config.js`，通过 `pnpm commit` 命令启动。

### Husky & Lint-Staged

- **Husky**: 用于管理 Git Hooks，在提交前触发检查。
- **Lint-Staged**: 只对暂存区（即将提交）的文件执行检查，不同文件设置不同检查，精细化检查。
- 配置文件: `.lintstagedrc.js`

## 打包

### 包依赖

components 子项目需要依赖 utils 项目，然后可以在 components 的package.json 里面的 dependencies 字段

```
"dependencies": {
    "vue": "^3.5.21",
    "@monorepo-demo/utils": "workspace:*" // *可以换成对应的版本号
  },
```

用到的 utils 不是从 npm 上去安装了，而是直接把 utils 文件夹，给他生成一个快捷方式（软链接），这样的话版本永远都是最新的

### node 的模块查找策略

当 components 导入 import { sum } from "@monorepo-demo/utils";utils 包的时候，搞不清楚导入的是哪一个

在 package.json 里面只找到一个 main “main“: "index.js" 找不到入口文件

ES Module，只需要在用到 utils 这个库的时候，实际用到的是打包结果，在配置文件上添加
"module": "./dist/index.esm.js" // 模块
"types": "./dist/index.d.ts" // 类型声明

依赖关系就建立起来了

### 统一测试，写单元测试（写测试脚本）

其中一个包改了，对应的用到了该包的地方都要重新测试
测试框架：Jest，Mocha，Vitest

配置文件: `vitest.config.js`，写测试脚本：对应子项目下建立**test**文件`@utils/__test__`

### 发布

根目录

```json
"publish:utils": "pnpm --filter \"monorepo/utils\" publish"
```

utils package.json 指定哪些是要发布到 npm 上面

```json
"files": ["dist"]

"publishConfig": {"access": "public"} // 公共包
```
