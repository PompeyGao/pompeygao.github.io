---
layout:     post
title:      "vue-cli3 使用 antd-vue"
subtitle:   ""
date:       2020-06-26
author:     "pompeygao"
header-img: "img/in-post/other/antd.jpg"
tags:
    - vue
    - antd
---

### vue-cli3 使用 antd-vue

#### 方案一

1. 安装流程 [官方文档](https://www.antdv.com/docs/vue/use-with-vue-cli-cn/)

2. 参照文档有可能会遇到的问题

   - 提示无法加载less

     > ```
     > error  in ./node_modules/ant-design-vue/es/style/index.less 
     >Module build failed (from ./node_modules/less-loader/dist/cjs.js):
     > ```
     
     - 安装 `yarn add less less-loader -D`

    - 上述方法尝试后，仍提示错误

      > ```
      in ./node_modules/ant-design-vue/es/button/style/index.less
       >
       > Module build failed (from ./node_modules/less-loader/dist/cjs.js):
       >
       > // https://github.com/ant-design/ant-motion/issues/44
       > .bezierEasingMixin();
       >
       > Inline JavaScript is not enabled. Is it set in your options?
       > ```

      - 新建`vue.config.js`

        `vue.config.js`

        ```js
        module.exports = {
          // 配置less
            css: {
                loaderOptions: {
                    less: {
                        lessOptions: {
        	                javascriptEnabled: true
                        }
                    }
                }
            }
        };
        ```

        


#### 方案二

采用 [vue-cli-plugin-ant-design](https://github.com/vueComponent/vue-cli-plugin-ant-design) 插件，快速地搭建一个基于 Ant Design Vue 的项目。

采用此方案不会出现方案一中遇到的问题，所以推荐本方案。