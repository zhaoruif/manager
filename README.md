
## 前言
VUE+elementUI结构的后台管理系统模板
## 功能
1 Element UI
2 登录/注销
3 首页
4 表格数据{导入Excel/导出Excel}
5 Tab 选项卡
6 表单
7 富文本编辑器  markdown 编辑器
8 支持切换主题色 :sparkles:
9 列表拖拽排序
10 权限测试  404 / 403
11 三级菜单
12 自定义图标
13 可拖拽弹窗
14 国际化{中英文语言切换}
## 安装步骤
```
1>克隆项目到本地  
git clone https://github.com/zhaoruif/manager.git 
2>安装npm依赖 
 npm install   
 ```   
## 启动服务
```
npm run dev

// 执行构建命令，生成的dist文件夹放在服务器下即可访问
npm run build
```
## 导出Excel/导入插件
1>npm install -S file-saver xlsx
2>npm install -D script-loader