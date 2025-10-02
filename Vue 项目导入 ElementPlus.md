# Vue 项目导入 ElementPlus

## 安装必要的包
```shell
npm install element-plus
npm install -D unplugin-vue-components unplugin-auto-import  # 优化大小必备
```

## 修改配置文件 vite.config.ts，实现自动导入
```typescript
// 添加以下 3 个 import
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
    // 以上省略 ...
    plugins: [
        // ... 添加以下 AutoImport 和 Components
        AutoImport({
            resolvers: [ElementPlusResolver()],
        }),
        Components({
            resolvers: [ElementPlusResolver()],
        })
        // 结束 ... 
    ]
    // 以下省略 ...
})
```

## 开始在组件中灵活使用
上述步骤完成后，即可在项目的任何地方，无需 import 特定 element-plus 组件
也可以任意地使用 <el-button> 等组件标签！
