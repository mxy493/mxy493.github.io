# 一、基本操作

1. 新建博文

   ```
   1. hexo new "文章名"
   ```

2. 清理缓存

   ```
   hexo clean
   ```

3. 构建静态网站

   ```
   hexo generate
   hexo g
   ```

4. 本地测试

   ```
   hexo s
   ```

5. 发布到远程仓库

   ```
   hexo deploy
   hexo d
   ```

# 二、相关说明

## 1、简述

目前配置了多个静态网站托管服务（包括GitHub、Gitee、Coding、Gitlab、腾讯云开发）。

只维护GitHub和腾讯云开发，原因：

- GitHub 使用较多，访问速度相对较快，可配置未备案的域名；
- 腾讯云开发静态网站托管访问速度最快，但配置个人域名需要备案，当前我的个人域名已IP备案能正常使用；
- Gitee 访问速度快，但是不开通会员只能使用默认域名，并且必须手动部署；
- Coding 的静态网页服务器地址为新加坡，访问速度慢；
- Gitlab 访问速度慢，需要将源文件推送到远程仓库，在远程自动构建并部署。

其中GitHub、Gitee、Coding配置方式相同，可在 `_config.yml` 文件内修改：

```
deploy:
- type: git
  repo: https://github.com/mxy493/mxy493.github.io.git
  branch: master
  # 包含根目录下的.github文件夹，以部署GitHub Action自动同步到腾讯云开发静态网页托管
  extend_dirs:
    - .github
    - README.md
# 取消注释即可同步到gitee和coding
# - type: git
#   repo: https://gitee.com/mxy493/mxy493.git
#   branch: master
# - type: git
#   repo: https://e.coding.net/mxy493/mxy493.git
#   branch: master
```

Gitlab 需要将源文件推送到远程仓库，然后 Gitlab 会自动构建静态网站并发布：

```
//查看远程仓库
git remote

//推送到某一个远程仓库
git push -u gitlab master
```

## 2、GitHub & 腾讯云开发

### （1）自动部署

当前使用 GitHub Action 自动部署到腾讯云开发，目前来说最方便的方案~

**腾讯官方说明：https://github.com/marketplace/actions/tencent-cloudbase-github-action**

配置 GitHub Action 需要在博客仓库的根目录下创建 `.github/workflows/main.yml` 文件，但本地的 `public` 文件夹是每次发布之前重新生成的，所以这个文件不能放到 `public` 文件夹内部。

基础操作中的 `hexo d` 是使用插件 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) 实现的，可以在博客的配置文件 `_config.yml` 中配置需要额外包含的要发布的文件，路径也是以博客的根目录，配置如下：

```
# 包含根目录下的.github文件夹，以部署GitHub Action自动同步到腾讯云开发静态网页托管
extend_dirs:
  - .github # 包含根目录下的.github文件夹，.github/workflows/main.yml已经配置好
  - README.md # 说明文档
```

配置 `.github/workflows/main.yml` 文件：

```
name: Tencent Cloudbase # name字段可以自行更改
on: [push] # push 代码时触发

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Tencent Cloudbase Github Action
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        
       # 使用云开发 Github Action 部署
      - name: Deploy static to Tencent CloudBase
        id: deployStatic
        uses: TencentCloudBase/cloudbase-action@v1.1.1
        with:
        # 云开发的访问密钥 secretId 和 secretKey
          secretId: ${{ secrets.SECRET_ID }}
          secretKey: ${{ secrets.SECRET_KEY }}
           # 云开发的环境id
          envId: ${{ secrets.ENV_ID }}
           # Github 项目静态文件的路径，这里设置为根目录"./"
          staticSrcPath: ./
```

### （2）手动部署

手动部署需要使用 `cloudbase/cli` （已安装并初始化）：

```
//安装cloudbase/cli
npm i -g @cloudbase/cli

//初始化云开发cli
tcb login

//进入到博客主目录下的public文件夹，上传静态网站代码
cd public
tcb hosting:deploy ./ -e mxy-3914fc
```