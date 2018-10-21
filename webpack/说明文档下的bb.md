要摆脱加载lodash依赖，即npm安装
> npm install --save lodash

配置好出口以后

npx webpack执行

然而用cli安装的方式始终不高效，因此现在我们使用配置文件
> webpack.config.js

当有配置文件的时候执行npx webpack自动通过配置方式
注意：
当在 windows 中通过调用路径去调用 webpack 时，必须使用反斜线()。例如 node_modules\.bin\webpack --config webpack.config.js

在配置文件里面增加
> "build":"webpack"

此时就不需要cli方式执行了，现在只需要
> npm run build

## 管理资源

### 管理CSS

动态打包，每个资源都有它自身的依赖‘

webpack通过正则表达式寻找文件，并为其提供指定的loader’

如果遇到报错lodash没有定义，那么
> npm cache clean
> npm install(cnpm install)
> 

### 管理图片

## 管理输出

如果有多个输入，也就是多个entry
那么，此时的输出就不仅仅一个出口，通过设置[name].bundle.js，这样就可以根据入口中的命名自动命名构建

> npm install --save-dev html-webpack-plugin

安装插件使得index.html动态更新html引用新的js

> npm install clean-webpack-plugin --save-dev

安装插件清理disk，也就是disk中只保留最后发布的文件
以上追踪的实现原因是通过manifest保持追踪

## 代码分离

这个特性能把代码分离到不同的bundle中并可以控制其加载顺序或者并行加载
也就是把其中的共有部分进行管理和分离

方法：使用CommonsChunkPlugin插件