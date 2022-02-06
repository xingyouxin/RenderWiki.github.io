# RenderWiki渲染知识点整理

<br>

![Logo](https://renderwiki.github.io/ImageResources/Logo.png)

# 使用方法

## 工具要求

- 熟悉git基本指令
- 熟悉markdown文件的基本格式及写法

## 操作流程（待完善）

1. **准备工作：**
   1. 下载安装git工具
   2. 下载安装Atom工具——用于本地预览wiki网站（可选）

2. **拉取并修改项目：**

   1. 在本地目录下右键->git bash here，打开git bash：

      输入`git pull https://github.com/RenderWiki/RenderWiki.github.io.git `拉取项目

   2. 修改*\RenderWiki.github.io\library\xxx\目录下的.md文件

3. **上传项目**

   1. 在项目根目录右键->git bash here，打开git bash：

      - 输入`git add .`，回车；
      - 输入`git commit -m "添加shading model部分。"`，回车；
      - 输入`git push`，回车。

      结束。

**请务必对.md本地文件进行额外的备份，请务必对.md本地文件进行额外的备份，请务必对.md本地文件进行额外的备份！！！**

当前项目库仅采用单个main分支，所以每次上传项目前先执行git pull命令以拉取更新项目，然后修改内容并进行上传，避免产生版本冲突。

### 常见markdown语法

1. 公式
   - 行内公式`$L=k_dImax(0, \pmb{n·l}) \tag{1}$`
     - 效果预览：$L=k_dImax(0, \pmb{n·l}) $
   - 行间公式`$$L=k_dImax(0, \pmb{n·l}) \tag{1}$$`
     - 效果预览：


$$L=k_dImax(0, \pmb{n·l}) \tag{1}$$

2. 添加图片并使其居中
   - `<div align=center>![Lambertian着色示意图](https://renderwiki.github.io/ImageResources/shading model/Lambertian着色示意图.png)</div>`
     - 效果预览：

<div align=center>![Lambertian着色示意图](https://renderwiki.github.io/ImageResources/shading model/Lambertian着色示意图.png)</div>

3. 文字居中
   - `<center>居中</center>`
     - 效果预览

<center>居中</center>

## 注意事项

1. 避免出现**本文**，**我们**等词语
2. 图片统一存放到*\RenderWiki.github.io\ImageResources\您的文件夹名称\目录下，并在.md文件内设置好路径，例如：https://renderwiki.github.io/library/ImageResources/shading model/Lambertian着色示意图.png
3. 为您的图片指定好编号和并备注名称，例如：图1.1 xxx
4. 对公式采用markdown语法编辑
5. 备注好参考文献
6. 每个文件夹**必须按照** **id-名称** 来命名，每个.md文件**必须使用** **id-名称.md** 来命名
   id 不可删除，删除后将无法正常工作

网站中的公式调用了在线库中的渲染工具，所以有时无法显示完全，请手动刷新页面以显示。