# eslint和prettier的配置
四个依赖项: `eslint` 
, `prettier` 
, `eslint-config-prettier` 
, `eslint-plugin-prettier`

## 1. eslint和prettier搭配使用
- eslint主要用来进行代码校验, 不进行格式化(之前eslint不支持格式化), 一般使用prettier进行格式化

但是由于eslint和prettier各自有各自的规则, 有时候会有一些冲突, 所以`eslint-config-prettier`就出现了
`eslint-config-prettier`只是关闭了 ESLint 中一些不必要的规则以及可能与 Prettier冲突的规则。
(这个配置只能关闭规则，所以只有和其他配置一起使用才有意义。)
具体使用:
我们将` eslint-config-prettier` 添加到 .eslintrc.js 文件的 "extends" 数组中.
```javascript
module.exports = {
  extends: [
    // ...其他
    "prettier", // eslint-config-prettier 可简写成 prettier
  ]
}
```

## 2. 基于eslint使用prettier格式化
处理完eslint和prettier的规则冲突后, 就需要配置一些规则, 让eslint起作用, `eslint-plugin-prettier`就出现了
`eslint-plugin-prettier` 以 ESLint 规则的方式运行 Prettier，通过 Prettier 找出格式化前后的差异，并以 ESLint 问题的方式报告差异，同时针对不同类型的差异提供不同的 ESLint fixer
具体使用:
将prettier的规则放到eslint配置文件的rule里面
```js
// .eslintrc.js
module.exports = {
  // ...其他
  rules: {
    "prettier/prettier": "error"
  }
}
```

`eslint-plugin-prettier`还提供了这样一个配置: `plugin:prettier/recommended`
所以综合上面的内容得到下面一个简单的eslint配置文件
```javascript
module.exports = {
  root: true, // 是否开启 eslint。
  env: {
    commonjs: true,
    es2021: true,
    node: true,
  },
  extends: ['plugin:prettier/recommended'], // 把plugin里面prettier和extends里面prettier集成在一起
  overrides: [],
  parserOptions: {
    ecmaVersion: 2021,
  },
  rules: {
    'no-unused-vars': 'error', // 开启没有用过的变量检测
    'prettier/prettier': 'error', // 开启规则
  },
}
```

## 3. 项目中详细的配置示例
> 这里后续还需要继续整理下
```javascript
module.exports = {
  root: true, // 是否开启 eslint。
  env: {
    // 配置编译器宏环境
    browser: true,
    es2020: true,
    node: true,
  },
  extends: [
    // ESLint Definition for rule 'import/extensions' was not found
    "plugin:import/recommended",
    // 在此处添加更多通用规则集。
    'eslint:recommended',
    'plugin:vue/vue3-essential',
    'plugin:@typescript-eslint/recommended',
    'prettier',
    'plugin:prettier/recommended',
    './.eslintrc-auto-import.json'
  ],
  parser: 'vue-eslint-parser',
  parserOptions: {
    // 解析器选项
    ecmaVersion: 'latest', // 指定要使用的 ECMAScript 语法版本
    parser: '@typescript-eslint/parser', // 可选的。自定义解析器。
    sourceType: 'module', // 默认是 “script”。当代码在 ECMAScript 模块中时其值需设为 “module”。
    ecmaFeatures: { // eslint 解析jsx, tsx
      "jsx": true,
      "tsx": true
    }
  },
  plugins: [
    'vue',
    '@typescript-eslint',
    'simple-import-sort',
    'eslint-plugin-prettier',
  ],
  rules: {
    // 在此处 覆盖 或 添加 规则设置。
    // 项目导入顺序检测
    'simple-import-sort/imports': 'error',
    'prettier/prettier': 1,
    // Vue: Recommended rules to be closed or modify
    'vue/require-default-prop': 0,
    'vue/singleline-html-element-content-newline': 0,
    'vue/max-attributes-per-line': 0,
    // Vue: Add extra rules
    'vue/custom-event-name-casing': [2, 'camelCase'],
    'vue/no-v-text': 1,
    'vue/padding-line-between-blocks': 1,
    'vue/require-direct-export': 1,
    'vue/multi-word-component-names': 0,
    // Allow @ts-ignore comment
    '@typescript-eslint/ban-ts-comment': 0,
    '@typescript-eslint/no-unused-vars': 1,
    '@typescript-eslint/no-empty-function': 1,
    '@typescript-eslint/no-explicit-any': 0,
    'import/no-unresolved': 'off',
    'import/extensions': 'off',
    'no-console': process.env.NODE_ENV === 'production' ? 2 : 0,
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0,
    'no-param-reassign': 0,
    'prefer-regex-literals': 0,
    'import/no-extraneous-dependencies': 0,
  },
  settings: {
    // 设置别名
    'import/resolver': {
      alias: [['@', './src']],
    },
  },
}
```





