# Iconify



Iconify支持大量的icon集，工程里只需要引入一个类库iconify，就可以直接使用所有的icon



**这个过程具体是怎么实现的？**



### 一. 工程依赖

| 依赖                    | 版本   | 作用                                                        |
| ----------------------- | ------ | ----------------------------------------------------------- |
| @iconify/iconify        | ^3.0.0 | Iconify 核心库                                              |
| @iconify/json           | ^2.1.7 | Iconify 所有图标集                                          |
| @purge-icons/generated  | ^0.8.0 | 提取使用的图标名称，然后将图标的数据 (SVG) 捆绑到您的代码中 |
| vite-plugin-purge-icons | ^0.8.1 | PurgeIcons 的 vite 插件                                     |
| vite-plugin-svg-icons   | ^2.0.1 | 用于本地生成 svg 雪碧图                                     |



我们的代码中直接用封装好的Icon组件，类似

```vue
<Icon icon="raphael:home" @click="homeClick" class="jtk-btn" />
```

经过purge-icons和vite-plugin-purge-icons插件，就会收集到工程中引用到的所有icon，并把相关的svg信息打包到工程的js文件里

这个过程看似非常棒，只会按需引入图标，不会造成大量无需引入的图标，大大减少了工程文件大小



**但是！我们的低代码平台支持用户配置图标，而配置的图标是动态的，不会被插件收集到工程打包出来的文件中，这时候为什么icon依旧可以看到呢?**

  

### 二. Iconify API 

当icon的svg在本地找不到时候，iconify会自动请求api，api会返回相应图标的svg信息，从而就可以展示出对应的图标了
默认api地址：https://api.iconify.design

也有2个backup server

- https://api.simplesvg.com
- https://api.unisvg.com

示例：https://api.unisvg.com/mdi.json?icons=apple-keyboard-command%2Cbell-ring-outline%2Cchat-processing%2Cdatabase-check-outline%2Cdatabase-sync-outline%2Cdesktop-tower-monitor%2Cdog%2Cfire-circle%2Cmap-plus%2Cmessage-text-outline%2Cset-right%2Cshield-check-outline



**但这时候，问题又来了，很多项目环境是无外网的，访问不到公网的api....**



### 三. 手动预先引入图标库

```typescript
import Iconify from '@iconify/iconify';

const antdJson = (await import('@iconify/json/json/ant-design.json')).default;
const mdiJson = (await import('@iconify/json/json/mdi.json')).default;
Iconify.addCollection(antdJson);
Iconify.addCollection(mdiJson);
```

这样写后，ant-design和mdi的所有图标，就都会强制引用，然后打包时候就会都打到工程内去了。

这样可以决绝绝大多数情况的问题，但也限制了工程师只能使用这两种icon集，如果有看到更喜欢的，就要提前说了，并在代码里加进去，不然就又访问不到了。



### 四. 自建Iconfiy API Server

https://iconify.design/docs/api/

https://github.com/iconify/api

npm install

npm run build

npm start



```typescript
import { addAPIProvider } from '@iconify/iconify';

// Override default API provider with Iconify API hosted at 'https://iconify.my-project.com'
addAPIProvider('', {
   resources: ['http://localhost:3000'],
});
```



![image-20230724202227742](C:\Users\Lori\AppData\Roaming\Typora\typora-user-images\image-20230724202227742.png)