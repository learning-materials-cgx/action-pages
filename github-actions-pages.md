## github actions 构建

1. github 执行actions 构建， 首先需要在项目根目录下新建 `.github/workflows/ci.yml`(取文件名随意，后缀得是`.yml`)
2. 在`ci.yml` 中，添加执行action, 具体参考gitihub actions 说明，[actions 文档](https://docs.github.com/zh/actions)
   1.  `deploy_key: ${{ secrets.ACCESS_TOKEN }}` , 需要在本地 `git Bash` 创建密钥，
   2.   公钥添加在项目 `settings -> Deploy keys -> Add deploy key`, 需要选中 `Allow write access` 复选框。
   3.  私钥添加在项目得`settings -> secrets and Variables -> actions -> New repository secrets`, ps: 需要把私有`所有内容都复制`
   4.  也可以使用github token, settings -> Developer settings -> Persoonal access tokens, （未测试）
``` markdown
name: CI

on:
  push:
    branches: ['main']
  pull_request:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Run a one-line script
        run: echo start build the project!

      - name: node_modules cache
        id: node-modules-cache
        uses: actions/cache@v2
        env:
          cache-name: node-modules-yarn
          cache-fingerprint: ${{ env.node-version }}-${{ hashFiles('yarn.lock') }}
        with:
          path: node_modules
          key: ${{ runner.os }}-${{ env.cache-name }}-${{ env.cache-fingerprint }}
          restore-keys: ${{ runner.os }}-${{ env.cache-name }}
      - name: yarn install
        if: steps.node-modules-cache.outputs.cache-hit != 'true'
        run: yarn install --prefer-offline --frozen-lockfile

      - name: yarn build
        run: yarn build

      - name: Deploy to gh-pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACCESS_TOKEN }}
          publish_dir: ./build

```
参考文档：
1. [https://blog.csdn.net/xixihahalelehehe/article/details/120178280](https://blog.csdn.net/xixihahalelehehe/article/details/120178280)
2. [http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)



## 自动发布在pages
1. 配置自动执行命令发布到pages, 可以访问， 首先必须把项目设置为 `public`, 私有项目可以转换。
2. 在`settings -> pages`， 
   1. 选择 `source` 为 `Deploy from a bracnh`(这里选择branch， 需要把构建后得代码存到一个分支上，选择哪个分支，就把代码发布到哪个分支上去， 这里是`gh-pages`, 无需手动创建分支)
   2. save 保存后，刷新可以看见网站链接。
   3. 每次编译后，可以发布到`gh-pages`分支， 网站会自动更新


参考文档
1.[https://docs.github.com/zh/pages](https://docs.github.com/zh/pages)

### 需要注意得是 .yml 文件的配置。