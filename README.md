## Electron项目模板（自用）
> 整合了Electron、React、TypeScript、Vite,使用了pnpm包管理器，以及Electron的打包工具，electron-builder

```shell
// 安装依赖
pnpm i
// 运行项目
pnpm start 
```
> 可能会遇到的问题：
1. 执行`pnpm add electron -D`的时候，安装electron会下载失败，这是因为下载源在 github.com ，国内访问会很慢，导致连接超时。
**解决办法**：进入到 `/node_modules/electron/install.js`里面，找到`downloadArtifact({...})`方法，往里面添加:
    ``` js
    downloadArtifact({
      // .... 以上省略
      arch,
      mirrorOptions:{
        // 这个就是下载源，换成了淘宝镜像
        mirror: 'https://npm.taobao.org/mirrors/electron/',
        // 在版本号前面添加一个字符串v
        customDir: 'v{{ version }}'
      }
    }) 
    ```
    然后终端里面输入：`cd /node_modules/electron/`，执行 `node install.js`，再次进行安装，就会成功了。
2. 使用`electron-builder`进行打包的时候，也会遇到下载问题。跟上面的差不多。这个时候看报的错，如果哪个东西下载失败（比如-electron-v17.0.0-win32-x64），就去`https://npm.taobao.org/mirrors/electron/`这个里面下载对应的electron版本，其它下载失败的东西也一样的做法（报错里面一般会给出下载链接），然后将下载包移动到`C:\Users\***\AppData\Local\electron\Cache`下面，还有 `C:\Users\***\AppData\Local\electron-builder\Cache`
> 开发时的几个依赖(devDependencies)：
* `kill-port`:清理端口
* `cross-env`:设置环境变量
* `npm-run-all` 顺序执行script脚本
* `concurrently` 并行执行script脚本
* `tsc-watch`:编译ts文件,并在文件修改后重新执行编译
* `wait-on` 等待文件/端口等变化后执行script脚本
> 脚本说明：
```json
"scripts": {
    "dev": "vite",
    "start": "kill-port 3920 && concurrently -k \"vite\" \"wait-on tcp:3920 && npm-run-all watch\"",
    "watch": "tsc-watch --noClear -p tsconfig.e.json --onSuccess \"npm-run-all start:ect\"",
    "start:ect": "cross-env NODE_ENV=development electron ./output/build/main.js",
    "build": "npm-run-all build:react build:tsc build:win",
    "build:react": "tsc --noEmit && vite build",
    "build:tsc": "tsc -p tsconfig.e.json",
    "build:mac": "electron-builder --mac",
    "build:win": "electron-builder --win --x64",
    "preview": "cross-env NODE_ENV=production electron ./output/build/main.js"
  }
```
* 终端运行 `yarn start` / `pnpm start` 开始启动项目
* `start`:
清理端口3920(kill-port 3920)  然后(&&) 并行执行(concurrently -k)命令vite 和 wait-on tcp:3920 && npm-run-all watch
vite:启动vite开发服务器 指定端口3920
wait-on tcp:3920 && npm-run-all watch:监听端口3920,连通时执行npm-run-all watch
* `watch`:
使用tsc-watch 将ts文件编译成js文件;  通过配置文件tsconfig.e.json 指定只编译electron文件夹下的文件,并将js文件输出到output/build 文件夹当ts文件编译完成后(--onSuccess),执行命令npm-run-all start:ect
启动后tsc-watch将持续监听electron文件夹,每次文件改动都会重新执行编译并运行(--onSuccess)后的命令
* `start:ect`:
设置环境变量cross-env NODE_ENV=development,electron主进程启动时可以拿到此变量(见electron/utils/createWindow.ts),并据此判断是加载外部链接还是加载打包后文件
启动electron app,指定入口文件./output/build/main.js
* `pnpm build` 开始执行打包脚本
* `build:react` 打包vue文件,目录为output/dist
* `build:tsc`将ts的electron文件编译为js,目录为output/build
* `build:all`打包两个平台 也可以分别运行build:vue,build:tsc,build:mac来打包指定平台
* `"main": "output/build/main.js"`,是在tsconfig.e.json中指定的,编译ts文件后输出到build目录 electron-builder通过它找到打包入口
* `preview`: 使用electron预览打包后的vue文件
