# 博客站

地址：https://striving-dev.github.io/blog/

## 如何写文章

如果熟悉的话，完全可以直接在source/_posts 添加写好的MD文件，然后直接`push`到main分支即可。但是一般情况下建议先本地查看效果再进行推送。

``` bash
# install
git clone && npm install

# start server
npm run server
```



## 介绍

博客站使用[hexo](https://hexo.io/zh-cn/)构建，其中主题使用的[Fluid](https://github.com/fluid-dev/hexo-theme-fluid)。一般情况下我们只需要修改 `/source/_posts`下的文件即可。这个文件最终会渲染成文章。



和其他MD文件稍有不同，hexo的md文件需要在文章开头增加一些元信息，官方说法是：[front-matter](https://hexo.io/zh-cn/docs/front-matter)，在实际渲染中，front-matter会发挥重要作用，并且不同的主题也会自定义不同的字段，所以之后可以研究下hexo和主题所提供的能力。



目前需要写的front-matter：

- title: test    标题test
- date: 2020-12-05 16:48:05   时间
- author     你的昵称

