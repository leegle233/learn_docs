# 如何配置一个 Electron + VueJS 项目


## 前言
Electron 就是个单独的外壳，可以无痛套在原有的 Vue 项目之外。用 Electron 开发，其实本质是在：
用一个包装成 App 的 Chrome 浏览器（其实就是 Electron），去访问一个部署在本地的、已经编译好的 Vue 项目。
和浏览器直接打开 index.html 没有本质区别，只是浏览器被换壳成为了一个看起来是本地的 App 而已。

所以，我们将 Electron 集成到原有的 Vue 项目中，本质上就是在：
第一，启动原有的 Vue 项目（可通过 localhost 或 build 的方式）；
第二，用我们的 Electron “浏览器”去“访问”这个 Vue 项目，通过 loadFile（生产）或 loadURL（开发）的方式。

本着这些背景和思路，我们开始来配置 Electron + Vue 的项目。

## 创建 Vue 项目并安装必要的包

使用 WebStorm 新建一个 Vue 项目，注意自定义配置，使用 TypeScript。

然后，安装以下包：
```shell
npm install electron --save-dev
npm install electron-builder --save-dev
npm install concurrently --save-dev
```

## 关键文件的创建


创建 Electron 入口文件 ./src/electron_main.ts：
```typescript
import { app, BrowserWindow } from 'electron'
import { join, dirname } from 'path'
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const createWindow = () => {
    const win = new BrowserWindow({
        width: 800,
        height: 600
    })

    console.log(process.env.NODE_ENV)
    console.log(__filename, __dirname)

    const indexFile = join(dirname(__dirname), 'dist', 'index.html')
    console.log("Index", indexFile)
    win.loadFile(indexFile)
    // win.loadURL('http://localhost:5173')

    // 开发模式下加载 Vite 开发服务器
    if (process.env.NODE_ENV === 'development') {

        win.webContents.openDevTools()
    } else {
        // 生产模式下加载打包后的文件

    }
    // win.loadURL("http://localhost:5173")
}

app.whenReady().then(() => {
    createWindow()
})
```

创建 electron-builder.json 文件，该文件储存的 Electron 在 build 时需要访问的配置，
这些配置中关键的是 files，包含了已经打包好的 vue 项目文件，必须被包含，如果没有被包含打包的 App 中将不会有任何界面显示。 文件内容如下：
```json
{
  "directories": {
    "output": "dist_electron"
  },
  "files": [
    "dist/**/*",
    "src/**/*",
    "package.json"
  ]
}
```

## 关键文件的设置

设置 package.json，添加以下关键的属性：main 和 package, dev, build.

```json
{
  "main": "./src/electron_main.ts",
  "scripts": {
    "package": "electron-builder",
    "dev": "concurrently -k \"vite\" \"electron .\"",
    "build": "vue-tsc && vite build"
  }
}
```

设置 vite.config.ts 中的 baseURL 如下：

```javascript
export default defineConfig({
    base: "./", // 这是为了解决 build 之后，文件中的引用均为绝对路径的问题。
    plugins: [
        vue(),
        vueDevTools(),
    ],
    resolve: {
        alias: {
            '@': fileURLToPath(new URL('./src', import.meta.url))
        },
    },

})

```




## 运行起来
```shell
npm run dev   # 启动调试
npm run build  # 单独打包编译 vue 项目
npm run package  # 将 Electron+Vue 项目打包为安装包/程序包
```
