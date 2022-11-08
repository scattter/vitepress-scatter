# arco design 与 vite + vue3 的使用

## 1. 按需加载
需要配合`unplugin-vue-components` 和 `unplugin-auto-import` 和 `vite-plugin-style-import`)

### 1.1 `unplugin-vue-components`
该插件主要是用来解决每新建一个vue文件都需要手动导入一些相同组件(包括外部和内部组件)的问题,
如ui组件库这些(单独的如Message这种组件还是需要配合下面的插件完成), 具体使用如下所示, 按需导入`arco`组件
```javascript
Components({
	// 指定组件位置，默认是src/components
  dirs: ['src/components'],
	resolvers: [
  	ArcoResolver({
  	  sideEffect: true
  	})
  ]
})
```
插件会生成一个ui库组件以及指令路径`components.d.ts`文件, 在这个文件里面的就是一些自动导入的组件

### 1.2 `unplugin-auto-import`
该插件主要是用来自动导入vue3的hooks, 当然在里面也可以添加自己的想要自动导入的组件,
这里以Message为例, 想要自动导入的话需要下面的配置
```javascript
 AutoImport({
  // 这里的按需加载和官网上的一样, 和arco-pro默认配置的有出入
  // resolvers: [ArcoResolver()],
  // targets to transform
  include: [
    /\.[tj]sx?$/, // .ts, .tsx, .js, .jsx
    /\.vue$/,
    /\.vue\?vue/, // .vue
    /\.md$/, // .md
  ],
  // 配置后会在根目录生成./auto-imports.d.ts文件
  dts: './auto-imports.d.ts', // or a custom path
  imports: [
    // presets
    'vue',
    'vue-router',
    {
      '@arco-design/web-vue': ['Message'],
    }
  ],
  eslintrc: {
    // eslintrc-auto-import.json文件
    enabled: true,
    filepath: './.eslintrc-auto-import.json',
    globalsPropValue: true,
  },
})
 ```

同时为了防止eslint报错, 也还会生成一个eslint的自动引入文件, `.eslintrc-auto-import.json`,
最后在eslint的extends里面配置该文件, 进而使其生效, 不会造成eslint报错
```javascript
module.exports = {
  extends: [
    // ...
    './.eslintrc-auto-import.json'
  ],
}
```

### 1.3 `vite-plugin-style-import`
该插件主要是用来自动引入相关组件的样式, 如果不使用该组件的话,
如Message这种组件根本不能显示, 当然使用全局导入样式也能展示出来, 但是这样的话打包后的文件就会多出来一些无用的内容
这里需要注意的是: arco里面一些组件的样式是在父组件样式文件里面, 所以还需要进行处理.

具体的插件配置如下所示:
```javascript
createStyleImportPlugin({
  libs: [
    {
      libraryName: '@arco-design/web-vue',
      esModule: true,
      resolveStyle: name => {
        // The use of this part of the component must depend on the parent, so it can be ignored directly.
        // 这部分组件的使用必须依赖父级，所以直接忽略即可。
        const ignoreList = [
          'config-provider',
          'anchor-link',
          'sub-menu',
          'menu-item',
          'menu-item-group',
          'breadcrumb-item',
          'form-item',
          'step',
          'card-grid',
          'card-meta',
          'collapse-panel',
          'collapse-item',
          'descriptions-item',
          'list-item',
          'list-item-meta',
          'table-column',
          'table-column-group',
          'tab-pane',
          'tab-content',
          'timeline-item',
          'tree-node',
          'skeleton-line',
          'skeleton-shape',
          'grid-item',
          'carousel-item',
          'doption',
          'option',
          'Optgroup',
          'icon',
        ]
        // List of components that need to map imported styles
        // 需要映射引入样式的组件列表
        const replaceList: any = {
          'typography-text': 'typography',
          'typography-title': 'typography',
          'typography-paragraph': 'typography',
          'typography-link': 'typography',
          'dropdown-button': 'dropdown',
          'input-password': 'input',
          'input-search': 'input',
          'input-group': 'input',
          'radio-group': 'radio',
          'checkbox-group': 'checkbox',
          'layout-sider': 'layout',
          'layout-content': 'layout',
          'layout-footer': 'layout',
          'layout-header': 'layout',
          'month-picker': 'date-picker',
          'range-picker': 'date-picker',
          row: 'grid', // 'grid/row.less'
          col: 'grid', // 'grid/col.less'
          'avatar-group': 'avatar',
          'image-preview': 'image',
          'image-preview-group': 'image',
        }
        if (ignoreList.includes(name)) return ''
        // eslint-disable-next-line no-prototype-builtins
        return replaceList.hasOwnProperty(name)
          ? `@arco-design/web-vue/es/${replaceList[name]}/style/css.js`
          : `@arco-design/web-vue/es/${name}/style/css.js`
        // less
        // return `@arco-design/web-vue/es/${name}/style/index.js`;
      },
    },
  ],
})
```

### 1.4 arco icon模板引入问题
::: warning
在项目中使用模板方法创建menu菜单, 出现了arco icon不能自动导入的问题, 目前的处理方案就是全局导入, 还没找到更好的方案
:::


## 2. 项目vite config配置示例
> 有待后面再整理一次, 添加一些备注
### 2.1 prod配置
`vite.prod.config.js`
```javascript
import { resolve } from 'path'
import { mergeConfig } from 'vite'

// 分析打包后文件大小
// import configVisualizerPlugin from '../plugins/visualizer'
import BaseConfig from './vite.base.config'

export default mergeConfig(
  {
    mode: 'production',
    // plugins: [configVisualizerPlugin()],
    build: {
      outDir: './dist',
      rollupOptions: {
        output: {
          // 拆分打包文件目录
          assetFileNames: (assetInfo: { name: string }) => {
            const info = assetInfo.name.split('.')
            let extType = info[info.length - 1]
            if (/\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/i.test(assetInfo.name)) {
              extType = 'media'
            } else if (/\.(png|jpe?g|gif|svg)(\?.*)?$/.test(assetInfo.name)) {
              extType = 'img'
            } else if (/\.(woff2?|eot|ttf|otf)(\?.*)?$/i.test(assetInfo.name)) {
              extType = 'fonts'
            }
            return `static/${extType}/[name]-[hash][extname]`
          },
          chunkFileNames: 'static/js/[name]-[hash].js',
          entryFileNames: 'static/js/[name]-[hash].js',
          manualChunks: {
            arco: ['@arco-design/web-vue'],
            vue: ['vue', 'vue-router', 'pinia'],
            store_vendor: [`${resolve(__dirname, '/src/store/index.ts')}`],
          },
        },
      },
      chunkSizeWarningLimit: 1024,
    },
  },
  BaseConfig
)
```

### 2.2 base配置
`vite.base.config.js`
```javascript
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import { resolve } from 'path'
import AutoImport from 'unplugin-auto-import/vite'
// import { ArcoResolver } from 'unplugin-vue-components/resolvers'
import { defineConfig } from 'vite'

import { arcoResolver } from '../plugins/arcoResolver'
import configStyleImportPlugin from '../plugins/styleImport'

// https://vitejs.dev/config/
export default defineConfig({
  resolve: {
    alias: [
      {
        find: '@',
        // 这里可以根据自己的根目录进行修改
        replacement: resolve(__dirname, '../../src'),
      },
      // 这里是为了解决menu render的编译问题, 使用运行时编译
      {
        find: 'vue',
        replacement: 'vue/dist/vue.esm-bundler.js', // compile template
      },
    ],
    extensions: ['.js', '.ts', '.jsx', '.tsx', '.json'],
  },
  plugins: [
    vue(),
    vueJsx(),
    AutoImport({
      // 这里的按需加载和官网上的一样, 和arco-pro默认配置的有出入
      // resolvers: [ArcoResolver()],
      // targets to transform
      include: [
        /\.[tj]sx?$/, // .ts, .tsx, .js, .jsx
        /\.vue$/,
        /\.vue\?vue/, // .vue
        /\.md$/, // .md
      ],
      dts: './auto-imports.d.ts', // or a custom path
      imports: [
        // presets
        'vue',
        'vue-router',
        {
          '@arco-design/web-vue': ['Message'],
        },
      ],
      eslintrc: {
        // eslintrc-auto-import.json文件
        enabled: true,
        filepath: './.eslintrc-auto-import.json',
        globalsPropValue: true,
      },
    }),
    // 自动引入arco design 这个文件在1.1章节中有, 主要内容一样, 函数包装一下就行
    arcoResolver(),
    // 按需导入arco design样式 这个文件在1.3章节中有, 主要内容一样, 函数包装一下就行
    configStyleImportPlugin(),
  ],
  css: {
    preprocessorOptions: {
      less: {
        modifyVars: {
          hack: `true; @import (reference) "${resolve('src/assets/style/breakpoint.less')}";`,
        },
        javascriptEnabled: true,
      },
    },
  },
})
```

### 2.3 dev配置
`vite.dev.config.js`
```javascript
import { mergeConfig } from 'vite'
import eslint from 'vite-plugin-eslint'

import BaseConfig from './vite.base.config'

// https://vitejs.dev/config/
export default mergeConfig(
  {
    mode: 'development',
    server: {
      port: 3001,
      hmr: true,
      proxy: {
        // 选项写法
        '/api/v1': {
          target: 'http://localhost:8001', // 所要代理的目标地址
          ws: true, // 支持websocket
          // rewrite: path => path.replace(/^\/search/, ''), // 重写传过来的path路径，比如 `/api/index/1?id=10&name=zs`（注意:path路径最前面有斜杠（/），因此，正则匹配的时候不要忘了是斜杠（/）开头的；选项的 key 也是斜杠（/）开头的）
          changeOrigin: true, // true/false, Default: false - changes the origin of the host header to the target URL
        },
      },
    },
    plugins: [
      // 用来配置eslint
      eslint({
        cache: false,
        include: ['src/**/*.ts', 'src/**/*.tsx', 'src/**/*.vue'],
        exclude: ['node_modules'],
      }),
    ],
  },
  BaseConfig
)
```



















