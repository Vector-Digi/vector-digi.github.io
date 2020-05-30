---
title: hexo自动化建站维护技巧
date: 2019-09-27 23:09:49
tags: [hexo,gitpage]
categories: [计算机民科]
description: 为你的hexo+gitpage建站方案引入持续集成、平滑升级等特性
---
&emsp;&emsp;hexo作为开发活跃且的静态框架受到广大博主和站长们的喜爱和二次开发，如果手上有Freenom的域名并且通过gitpage方式部署的话，长期运营网站甚至不会产生任何费用。尤其是今年Freenom暗改了免费域名的申请政策之后，薅羊毛变得十分困难的情况下，这种方式可以规避服务器宕机或欠费导致域名被闲置时刚好被Freenom逮到并收回的风险。本文的服务对象为使用hexo框架+gitpage方案建站保域名的小型站长，利用git自身特性进一步提高网站部署的自动化程度与可靠性。故不再赘述网络上广泛存在的搭建部分教程。  

## 平滑升级  
&emsp;&emsp;个性化定制作为网页设计的重要环节，在hexo框架上被“主题”功能大幅简化。本节不会探讨如何深度定制主题，但如果直接将找来的主题文件拷贝至theme路径使用或修改，而你的hexo开发环境本身又是一个git项目，那么你无法在保留主题的版本控制功能的前提下将其集成在自己的项目里。所以现在不少主题开发者在自己的主题中引入了[data files](https://hexo.io/docs/data-files.html)特性，将其作为网站git项目的submodule集成，允许站长用主题路径以外的配置文件对主题进行一定程度的客制，同时非常方便的拉取主题开发者的功能性或修复性更新。  
&emsp;&emsp;本例程默认读者已经创建已经在github上创建了一个托管hexo开发环境的git项目仓库，这个时候只要在开发环境的根目录下执行  

```
git submodule add <submodule_url> themes/
```

&emsp;&emsp;其中`<submodule_url>`填入你采用的主题git仓库地址即可。你可以在`.gitmodule`文件中找到所有引用的子模块URL与已经拉取的本地目录之间的映射。而如何确保每次编译静态网页时引用的主题为最新版本，在下一节会给出解决方案。

## 持续集成  
&emsp;&emsp;hexo框架特性使然允许的开发环境是唯一的。这意味着网站的发布和维护只能在同一台本地设备上进行，几乎没有容灾能力。而通过持续集成(Continuous Integration)来管理网站git项目的发布则可以做到将生成的网页文件和hexo开发环境分两条分支进行版本管理，仅需人为维护开发环境分支(Source Branch)，再由持续集成服务自动将改动重新生成网页发布到gitpage所在的分支(Content Branch)上，达到接近动态站一键发布的快感。  
&emsp;&emsp;抛开自建CI宿主不谈，现成的CI服务提供商当中同时支持Windows和Linux环境下开发项目的就只有[AppVeyor](https://www.appveyor.com/)了。为适应更多应用场景，建议实际建站时与本例程方案保持一致。  
1. 创建hexo网站git代码仓库  
在上一节中创建的GitHub仓库中创建一条新的分支并设置为用于存放gitpage静态网页(在本例程中的名称为master)，默认读者已解决主题git项目冲突问题。

2. 登录AppVeyor并添加Project  
在[登录页面](https://ci.appveyor.com/login)使用github账号注册后，添加步骤1中创建的hexo网站git仓库到[CI项目](https://ci.appveyor.com/projects/)中。

3. 添加CI脚本到Source分支  
AppVeyor的CI脚本文件为`appveyor.yml`，在Source分支的根目录下新建此文件并添加下面的内容：  

```yaml
clone_depth: 5

# 限制只编译source branch
branches:
  only:
  - <source_branch_name>

environment:
  my_variable:
    secure: <github_access_token>

install:
  - git submodule update --init --recursive # 每一次编译之前拉取最新的主题submodule
  - node --version
  - npm --version
  - npm install
  - npm install hexo-cli -g

build_script:
  - hexo generate

artifacts:
  - path: public

on_success:
  - git config --global credential.helper store
  - ps: Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:my_variable):x-oauth-basic@github.com`n"
  - git config --global user.email "%GIT_USER_EMAIL%"
  - git config --global user.name "%GIT_USER_NAME%"
  - git clone --depth 5 -q --branch=%TARGET_BRANCH% %STATIC_SITE_REPO% %TEMP%\static-site
  - cd %TEMP%\static-site
  - del * /f /q
  - for /d %%p IN (*) do rmdir "%%p" /s /q
  - SETLOCAL EnableDelayedExpansion & robocopy "%APPVEYOR_BUILD_FOLDER%\public" "%TEMP%\static-site" /e & IF !ERRORLEVEL! EQU 1 (exit 0) ELSE (IF !ERRORLEVEL! EQU 3 (exit 0) ELSE (exit 1))
  - git add -A
  - git commit -m "Update Static Site"
  - git push origin %TARGET_BRANCH%
  - appveyor AddMessage "Static Site Updated"
```

&emsp;&emsp;其中涉及到的参数`<github_access_token>`需要自行在github设置中生成，遵循[官方步骤](https://ci.appveyor.com/tools/encrypt)将生成后的明文token[加密](https://ci.appveyor.com/tools/encrypt)后粘贴到脚本中保存。  

4. 在AppVeyor Portal中设置CI脚本中的变量  
