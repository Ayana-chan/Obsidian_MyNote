# 环境变量
配置环境变量后，需要在`package.json`里面手动设置dev模式，否则获取不到值：[VUE3+VITE 关于环境变量的设置 BUG\_vite3 import.meta.env 获取不到值\_万伏小太阳的博客-CSDN博客](https://blog.csdn.net/weixin_51009975/article/details/127350324)。

实际上，这个`--mode`后面的参数就是对应的环境名，因此可以自定义环境名。

在`defineConfig`里面加上`envPrefix: ["APP_","VITE_"]`即可读取以这两个为前缀的环境变量。

在`vite.config.js`中使用环境变量需要一定代价，代码示例如下：

```js
import { loadEnv } from 'vite';

export default ({ mode }) => {
  //参数mode为开放模式或生产模式
  //console.log(mode);  // development or product
  const env=loadEnv(mode, process.cwd());   // 获取.env文件里定义的环境变量
  //console.log(env);   //变量在命令行里打印出来
  
  return defineConfig({
    plugins: [vue()],
    //项目部署在主域名的子文件使用,例如http://localhost:3000/myvite/。不填默认就是/
    base: env.VITE_APP_BASE_URL || '/',
	......
  })
}
```

在`loadEnv`时无法加载`defineConfig`内的设置，也就无法改变配置文件路径等。