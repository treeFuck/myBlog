# npm 发包

## 一、版本号

`npm包` 中的模块版本都需要遵循 `SemVer`规范——由 `Github` 起草的一个具有指导意义的，统一的版本号表示规则。实际上就是 `Semantic Version`（语义化版本）的缩写。

> SemVer规范官网： https://semver.org/

版本号采用 `X.Y.Z` 的格式，其中 X、Y、Z 为非负整数：

+ X 主版本号(major)：大幅改动，与旧版本使用方式不一致
+ Y 次版本号(minor)：做了功能性新增，但不影响旧版本使用方式
+ Z 修订号(patch)：打了个补丁

>1.0.0 的版本号用于界定公共 API。当你的软件发布到了正式环境，或者有稳定的API时，就可以发布1.0.0版本了。所以，当你决定对外部发布一个正式版本的npm包时，把它的版本标为1.0.0。

## 二、发包

第一次发布

```bash
npm init    # 生成package.json文件
npm adduser # 第一次，创建账号
npm login   # 不是第一次，登录账号
npm publish # 发包
```

查看包信息

```bash
# 包当前状态
npm info @tencent/mp-weui 
npm view @tencent/mp-weui

# 历史版本
npm view @tencent/mp-weui versions

# 查看依赖其他库信息
npm ls
```

修改本地版本号

```bash
npm version <update_type>
# update_type 有三种取值
npm version patch # 补丁，如 1.0.0 → 1.0.1
npm version minor # 小改，如 1.0.0 → 1.1.0
npm version major # 大改，如 1.0.0 → 2.0.0

# 也可以直接指定版本号
npm version 1.0.0-beta 
```

发布

```bash
# 提交正式版本 latest
npm publish    
# 提交正式版本的候选版本 rc
npm publish --tag rc 
# 提交测试版本 beta
npm publish --tag beta
# 提交内部版本 alpha
npm publish --tag alpha

```

查看/修改 tag

```bash
# 查看 npm 包的 tag
npm dist-tag ls @tencent/mp-weui

# 修改 tag
npm dist-tag add @tencent/mp-weui@1.0.1-beta.0 beta # 指定 1.0.1-beta.0 版本为 beta 版本
```



## 三、packjson 版本管理

```json
 {
   "dependencies": {
    "signale": "1.4.0", 
    "figlet": "*", 
    "react": "16.x",
    "table": "~5.4.6",
    "yargs": "^14.0.0"
  }
 }
```

逻辑如下：

`"signale": "1.4.0"`  → 固定版本

`"signale": "*"`  → 任意版本（>=0.0.0）

`"signale": "16.x"`  → 匹配主要版本（\>=16.0.0 <17.0.0）

`"signale": "16.3.x"`  → 匹配主要版本和次要版本（\>=16.3.0 <16.4.0）

`"signale": "~1.4.0"`  → 安装到  `x.y.z` 里最新的 `z`

`"signale": "^1.4.0"`  → 安装到  `x.y.z `里最新的 `y` 和最新的 `z`





## 四、Changelog 

![image-20220303201423534](/Users/bolewang/Library/Application Support/typora-user-images/image-20220303201423534.png)