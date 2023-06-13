---
slug: Vue
title:  Vue 
authors: [bkbk]
tags: [Vue]
---
 
 
:::tip 目录
**Vue框架技术** 

**Vue基础**  

**Vue整合**  
::: -->
<!--truncate-->
##  Vue 

[vue文档](https://cn.vuejs.org) 

### Vue框架技术	
 
#### 1.ES6基础  
#### 2.Vue模板语法  
#### 3.Vue指令  
#### 4.Vue综合案例  
#### 5.Vue动态样式绑定  
#### 6.Vue计算属性和监听属性  

###  Vue基础	 
#### 1.Vue生命周期及对应钩子函数  
#### 2.Vue脚手架  
 [Vue-cli](https://cli.vuejs.org/zh/)
选择自定义安装 Router、Vuex、Css-preprocessing、Babel  



#### 3.Vue组件  
#### 4.Vue路由  

###  Vue整合

#### 1.Vue整合axios  
``` bash
  npm install axios
```
``` js
 import axios from "axios";
 created(){
       axios.get("http://localhost:8888/student/all2").then(res=>{
        console.log(res.data);

        if(res.status == 200){
          this.page = res.data.data;
        }
      });
    }

 ```
#### 2.Vue整合ElementUI  
Vue3可以使用
 [ElementUI-PLUS](https://element-plus.gitee.io/zh-CN/)
``` bash
 # NPM
npm install element-plus --save

# Yarn
yarn add element-plus
``` 
``` js
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)
app.mount('#app')
```


#### 3.Vue整合Echarts  
 [Echarts](https://echarts.apache.org/zh/index.html)  



#### 4.Vue整合常用文本编辑器  
 [ueditor](http://fex.baidu.com/ueditor/) 重量   
[wangeditor](https://www.wangeditor.com/) 轻量   

#### 5.Vue+SpringBoot综合案例  

[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/) vue模板项目  
**vue-element-admin**是基于element-ui 的一套后台管理系统集成方案。

#### 6.Vuex基本使用  
在vue-cli构建时选择 vuex 项目中生成store文件夹  
最适合用vuex的场景，多个组件共同使用一份数据，而且这些组件之间关系乱套  


