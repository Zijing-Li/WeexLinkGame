# WeexLinkGame
纯用Weex内置模块做的一个简单的连连看游戏

+ `/LinkGame.vue`是主代码文件，把此文件连通图片资源复制到weex项目中。
+ `/image`里面的是代码中要用到的图片资源，当然，图片资源的引用方式视个人项目配置不同而不同，请把代码中`require(...)`这部分代码改成适合自己项目的，否则代码有可能报错。
+ 游戏规则：使用最多转两次弯的线连接消除两个一样的图片，所有图片消除为赢，无下一步为输。
+ 已实现功能：连线消除、下一步提示、错误提示、计分
+ 缺少功能：得分榜、音效、倒计时等
