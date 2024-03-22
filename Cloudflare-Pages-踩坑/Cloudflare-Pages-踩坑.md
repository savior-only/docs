

# Cloudflare Pages 踩坑

在搭建 Cloudflare Pages 过程中的遇到多处坑点，分享一下解决办法和思路

warning

注意，本文写于 2021.3.4, 遇到的问题在未来有可能已经解决

最近 Cloudflare Pages 开放使用，第一时间来尝尝鲜，结果遇到多个问题，下面进行下总结

- - -

# [](#%E4%B8%8D%E6%94%AF%E6%8C%81%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93)不支持私有仓库

已经给 cloudflare pages 应用配置存储库访问权限，都能选择私有仓库了，结果无法 clone

但关键问题是，其无法 clone 的表现是显示一直在排队，在第二天早上我发现已经排了将近 10 个小时，改为公开仓库才得以解决

这个问题在去年年底就存在

[![](assets/1710207494-a9e9fedf5b5abb23519491e6bd6627dc.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/10.png)

info

截至我写完这个文章，该问题好像已经被修复，私有仓库也可以 clone

- - -

# [](#%E4%B8%8D%E6%94%AF%E6%8C%81%E5%AD%90%E7%9B%AE%E5%BD%95%E4%B8%AD%E5%B8%A6-git-%E7%9B%AE%E5%BD%95)不支持子目录中带 .git 目录

以 hugo 为例，在 themes 目录下 clone 一个主题，该主题文件夹是肯定会存在 .git 目录的，但 cloudflare pages 在 clone 你的仓库时，会 clone 失败

需要手动删除 themes 目录下主题的 .git 目录

- - -

# [](#%E9%BB%98%E8%AE%A4%E7%89%88%E6%9C%AC%E8%BF%87%E4%BD%8E)默认版本过低

开幕雷击，我都怀疑这开发团队是从 2019 年穿越过来的

[![](assets/1710207494-9f229bf84e86044f69aec4c49e0b155f.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/8.png)

不过官方文档也说明了可以自己改变量使用新版本

[![](assets/1710207494-1ef83456a27d975155adc3dbb54855d4.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/6.png)

question

官方你既然知道，那为啥不把 hugo 0.54 换一换呢？

改下变量，可恶，少个 .0

[![](assets/1710207494-f5652ed8483fc9b891677f9de943405d.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/1.png)

改用较新的 0.81.0

[![](assets/1710207494-2eebeee68c7c8a75046753c2a6b68c8e.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/3.png)

一番折腾后终于成功部署

[![](assets/1710207494-19b061b95c0a8151dea97e6e4ac6d404.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/2.png)

- - -

# [](#%E5%9F%9F%E5%90%8D%E5%B7%B2%E5%85%B3%E8%81%94)域名已关联

在测试第一个项目时绑定了 r0fus0d.ffffffff0x.com 域名，但删除第一个项目后，看上去绑定关系还是存在，dns 记录中也并没有记录，这种情况下 pages 中域名就不能被使用了

[![](assets/1710207494-ec0991b250d7b6ad72712e63c30c2e03.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/4.png)

搜了下，同样有人也遇到了这个问题

[![](assets/1710207494-efab078e19ac9ea9928fae8d909c0f22.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/9.png)

没有办法，换个子域名，接下来又是个小坑

- - -

# [](#%E4%B8%8D%E6%94%AF%E6%8C%81%E4%B8%AD%E6%96%87%E8%B7%AF%E5%BE%84)不支持中文路径

这…

[![](assets/1710207494-7ac3c8e102c230f2d2ad92fde1ba374c.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/5.png)

没办法只好改为英文了

[![](assets/1710207494-c2e9a55b82ac230861d11e3786b35eb8.png)](https://r0fus0d.blog.ffffffff0x.com/img/Cloudflare_Pages_Test/7.png)
