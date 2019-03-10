

package.json

方便分享、方便控制版本号



```json
{
    "name":"my-package",		//必要
    "version":"0.0.1",		//必要
    "main":"./index",		//入口
    "private":"true",		//避免发布
    "dependencies":{		//相关依赖
        "colors":"0.5.0"
    }
}
```



如何处理高并发？？



错误的处理、，若不及时处理抛出的错误，那么进程的状态就不确定了



流

process中有三个标准流

* stdin 标准输入
* stdout 标准输出
* stderr 标准错误