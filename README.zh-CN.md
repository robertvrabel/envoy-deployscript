# envoy-deployscript
Laravel Envoy 部署脚本

基于 [papertank/envoy-deploy](https://github.com/papertank/envoy-deploy) 开发

由以下文章或代码启发而来:

* [papertank/envoy-deploy](https://github.com/papertank/envoy-deploy)
* [Deploying with Envoy (Cast)](https://serversforhackers.com/video/deploying-with-envoy-cast)
* [Enhancing Envoy Deployment](https://serversforhackers.com/video/enhancing-envoy-deployment)
* [An Envoyer-like deployment script using Envoy](https://iatstuti.net/blog/an-envoyer-like-deployment-script-using-envoy)
* [Rocketeer](http://rocketeer.autopergamene.eu/)
* [Deploy your app to DigitialOcean from Codeship using Envoy](http://laravelista.com/deploy-your-app-to-digitialocean-from-codeship-using-envoy/)


这个源代码仓库包含了一个Envoy.blade.php脚本模板文件 设计目的是提供一个基本的"不间断"部署功能 基于laravel的开源附属ssh远程工具程序 [Laravel Envoy](http://laravel.com/docs/5.1/envoy)

## 需求

这个脚本是设计用于laravel 5的项目,但是你也可以修改后用于其他类型的项目.

注意你的本地部署服务器/工作站/笔记本 需要和你的远程目标服务器之间配置基于ssh公钥模式的认证设置,参考: [SSH Key-Based Authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server).

## 安装

你的本地部署服务器/工作站/笔记本 必须通过composer全局安装过Laravel的Envoy工具 [Envoy](http://laravel.com/docs/5.1/envoy)
>composer global require "laravel/envoy=~1.0"

## 用法

### 设置

1. 下载或在克隆当前代码仓库

2. 拷贝 envoy.config.php 和 Envoy.blade.php 到你的本地部署服务器/工作站的 laravel 项目根路径(例如: ~/code/mysite)

3. 然后编辑 envoy.config.php 文件修改其中的ssh登录用户, Git仓库地址,服务器部署路径.
`$deploybasepath` (服务基础路径) 的目录 必须预先创建好,并且有正确的权限,或在说ssh用户有权限创建这对应路径(例如 用户:www-data,对目录$deploybasepath有读写权限).
你需要把你的web容器的网站根路径地址指向**`$deploybasepath/$appname`/current/public**
(例如: /var/www/mysite/current/public)

4. 把 `.envoydeploy/` 本地临时部署路径添加到你的laravel项目根路径的 .gitignore 文件中去,这样在管理代码的时候就能略过本地临时部署路径.

5. 你需要创建一个 .env.production 的环境变量文件在你的laravel项目的根路径中(参考.env.example) 这个文件将被当做环境文件部署到远程服务器上去.
又或者,你在执行命令时指定了环境变量(例如: development/testing) ,你也需要创建/软链接对应名称的环境变量文件(比如: .env.development/.env.testing)
另外如果你不想你线上生产环境或测试环境的环境变量文件被提交到源代码管理里你可以把这些对应的文件添加到.gitignore文件的忽略文件列表中去.

6. 你需要创建一个 `extra/custom/` 目录在你的laravel项目根路径, 部署脚本会拷贝覆盖这个子路径中的所有文件到目标部署路径中

>例如:

>你创建了的文件:

>`extra/custom/node_modules/laravel-elixir/Config.js`

>部署后,部署脚本将会将其覆盖拷贝到

>`($deployed_project_root)/node_modules/laravel-elixir/Config.js`
这通常用来做一些源码管理之外的配置文件自定义工作

### 初始化

当你做好了基础的环境配置,和配置文件的设定, 在你的本地laravel项目根路径中执行以下命令来完成远程和本地部署环境的初始化工作:

> envoy run deploy_init

你也可以在命令中指定环境变量 (类似于 artisan:migrate 指令) 和git的分支作为命令行参数

> envoy run deploy_init --branch=develop --env=development

通常情况下你只需要执行一次这个任务.

初始化任务会创建一个`.env` 文件在你的远程服务器的应用部署路径里 (e.g /var/www/mysite/.env )- 请保证你的这个环境变量文件能被你及时正确的更新.

### 部署

每次你想部署代码的时候,你只需要在你本地服务器/工作站的当前项目根路径中执行:
> envoy run deploy

你也可以在命令中指定环境变量 (类似于 artisan:migrate 指令) 和git的分支作为命令行参数

>envoy run deploy --branch=develop --env=development

#### 本地工作拷贝部署

如果你的远程服务器内存太小或者composer太慢无法在远程服务器执行`composer install`成功 (例如: 你在使用某些廉价的VPS服务器实例)
你可以使用 `deploy_mix_pack` 指令来替代 `deploy` 指令.
> envoy run deploy_mix_pack

**deploy_mix_pack** 任务 假设你当前的本地工作拷贝使用了和目标部署环境一样的git分支(如dev/master)一样的环境变量(development/production)
并且你在本地的工作拷贝中成功执行过 `composer install` 和可选的 `npm install` 或 `bower install` 任务指令(换句话说你的当前工作拷贝是你的调试上线环境).
( 你也可以注释/屏蔽掉Envoy.blade.php文件中的 `npm install` 或 `bower install` 对应的行,如果你的项目不需要用到npm/bower前端管理工具)
然后这个任务会将当前工作拷贝中的 `vendor` 和 `node_modules` 目录打包成 deps.tgz文件并scp拷贝到远程服务器的对应临时路径上去
再在远程服务器上克隆你的代码仓库,并完成后续的解压文件和软链接部署等相关工作.

#### 发布版本和依赖环境的本地构建和部署

如果你的远程服务器没有访问你git仓库的权限又或者无法顺利的执行`composer install` (例如:一些内部web服务器网络状况不好导致无法顺利的执行composer install/update)

> envoy run deploy_localrepo_install --branch=master --env=production

**deploy_localrepo_install** 任务会在你本地临时部署路径中克隆和创建对应的git分支env环境变量的版本并执行构建好完整的依赖环境
然后打包成release.tgz传输到远程服务器后并完成后续的解压文件和软链接部署等相关工作.

### 回滚

如果你发现你最近一次部署的版本有问题,你可以执行`rollback`回滚任务来将软链接回滚到上一次部署的路径中去:
>envoy run rollback

注意回滚任务只会将*current*路径的软链接指向指向上一个发布版本,但**并不**会将本次的数据库迁移回滚,如果你需要回滚数据库迁移,

请**先**执行`database_migrate_public_rollback`任务:
>envoy run database_migrate_public_rollback --branch=master --env=production

再去执行*rollback*任务,来回滚软链接.

如果你连续执行两次*rollback*任务,相当于*current*路径的软链接又将指向最后一次部署的路径.


## 原理/设计规划

你的 `$deploybasepath` 远程部署基础路径将会看起来像如下这样(如果你有多个子应用在同一个部署基路径里).
```
	mysite/
	mysite2/
	mysite3/
```
你的 `$deploybasepath/$appname` 远程部署应用路径将会看起来如下.

```
	releases/release_20150717032737/
	releases/release_20150717034646/
	current -> ./releases/release_20150717034646
	shared/storage/
	tmp/
	.env
```

正如你所见的, *current* 目录是软链到最近一次的部署目录的

在你的部署目录中文件列表看起来像下面这样(只列举了部分文件做例子):

```
	app/
	artisan
	boostrap/
	public/index.php
	composer.json
	.env -> ../../.env
	storage -> ../../shared/storage
	vendor/
```

部署目录中的.env文件和storage文件夹都软链接到了上级应用目录中的公共文件/文件夹了,这样部署本身只部署源代码和vendor等依赖环境,
storage等公共数据在shared文件夹中并不随代码部署,节省了空间也保留延续了日志/应用cache等相关基础数据在应用中的使用

## 特性

* 你可以部署多个项目到相同的远程服务器使用不同的$appname和配置设定即可.
* 因为在 *Laravel Envoy* 中 **暂时还无法在任务宏中调用其他的任务宏**,
你可以通过拷贝粘贴来自于(`deploy_mix_pack` | `deploy_mix_update` | `deploy_localrepo_install` | `deploy_remote_install`) 任务宏中的代码块到
`deploy` 任务宏的代码区块中来改变默认的`deploy`任务行为.
你也可以通过屏蔽部分如 `copy_custom_extra_localrepo` 任务 或 `npm install` 和 `bower install` 指令来定制你所需要的部署设定.

* 想要了解更多特性? RTFC, 自己修改代码来适配你自己的项目.

## 提示
* http/https 协议的git仓库地址有可能需要你交互式输入密码,这会打断git的克隆流程所以在git的仓库配置项中使用git协议,
在你的scm管理服务中设定好deploy key来解决这一问题.
(例如 github 仓库 ->settings->Deploy keys 添加部署公钥,然后在*envoy.config.php*中设定使用git协议的地址: `$repo = 'git@github.com:user/mysite.git'`)

* 清除老发布版本的任务`cleanup_oldreleases` 有时候无法确定完成清除任务如果你的代码项目中的文件太多(xargs无法接受太多文件项).
你也许需要自己手工到服务器上清理或尝试在每次部署完成后手工执行这一任务指令.
你可以调整服务器上保留的release发布版本份数,通过修改对应的指令行代码`tail -n +4` 4表示保留最近3次的发布版本.



## 使用样例

你可以 [从Codeship上使用Envoy部署你的应用到DigitialOcean](http://laravelista.com/deploy-your-app-to-digitialocean-from-codeship-using-envoy/)
使用当前的这个Envoy.blade.php 脚本和 envoy.config.php 配置文件来替代其中的Envoy.blade.php模板，如果需要也可以替换最后一个步奏的部署命令.

你可能需要把部署命令从:

>~/.composer/vendor/bin/envoy run deploy

改变成:

>~/.composer/vendor/bin/envoy run deploy_localrepo_install --branch=master --env=production

如果你的laravel项目运行在一个小内存的(比如 512MB)的实例上.

部署指令 `deploy_localrepo_install ` 会在部署服务器(这里是Codeship的虚拟机)上运行 `composer install`和压缩等其他相关任务,这样你的小内存服务器就可以不必运行`composer install`和`git clone`就可以直接运行应用了。

## 待改进

* 把deploy任务改进的更灵活

## 贡献代码

欢迎大家提交改进和修正 :)

## 作者

[Nick Fan(阿熊)](http://axiong.me)

