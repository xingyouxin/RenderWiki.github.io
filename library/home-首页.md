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

1. **统一公式编写方法**
   - 由于清华ac.wiki网站上的公式仅支持`<math>公式</math>`方式渲染公式，而github.io托管平台不支持使用$$语法或math标签来渲染公式，所以我们临时采用 codecogs 的外链来进行公式渲染：
     - 格式：`![](http://latex.codecogs.com/svg.latex?公式代码)`
     - 如：L=k_dImax(0, n·l)
     - ![](http://latex.codecogs.com/svg.latex?L=k_dImax(0, nl))
     - 这种方式的弊端在于1. 没法对向量进行加粗表示；2.无法表示·等特殊符号。
   - **此外还需要辛苦编写一份**<math></math>标签修饰的公式，以支持清华网站ac.wiki显示（您可以登录该网站进行测试），如：
     - `<math>L=k_dImax(0, \pmb{n·l})</math>`

<div align=center>![Logo](https://renderwiki.github.io/ImageResources/清华acwiki网站支持的公式格式.png=800x430)</div>

2. **添加图片并使其居中**
   - `<div align=center>![Lambertian着色示意图](https://renderwiki.github.io/ImageResources/shading model/Lambertian着色示意图.png)</div>`
     - 效果预览：

<div align=center>![Lambertian着色示意图](https://renderwiki.github.io/ImageResources/shading model/Lambertian着色示意图.png)</div>

3. 文字居中
   - `<center>居中</center>`
     - 效果预览

<center>居中</center>

## 注意事项

1. 避免出现**本文**，**我们**等词语
2. **图片统一存放**到*\RenderWiki.github.io\ImageResources\您的文件夹名称\目录下，并在.md文件内设置好路径，例如：https://renderwiki.github.io/library/ImageResources/shading model/Lambertian着色示意图.png
3. 为您的图片指定好编号和并备注名称，例如：图1.1 xxx，名称放在图片下方，居中显示
4. **备注好参考文献**
5. 每个文件夹**必须按照** **id-名称** 来命名，每个.md文件**必须使用** **id-名称.md** 来命名
   id 不可删除，**删除后将wiki网站无法正常工作**
