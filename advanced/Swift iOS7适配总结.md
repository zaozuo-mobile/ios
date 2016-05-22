* SDWebImage默认开启图片解压缓存造成内存增长不可控；
2. 使用复杂泛型（where语句）在iOS7产生EXC_BAD_ACCESS错误;
3. 使用autolayout时constain to margin默认勾选，与vc.view产生的顶部约束在iOS7有间隙;
4. 系统默认启动页设置方式无法适配5c问题，通过LaunchImage针对每个机型设置启动图片；
5. 关闭页面时，页面未执行pop就push新的页面造成错误， Unbalanced calls to begin/end appearance transition，
    如果有关闭页面后需要使用通知的方式告知需要push时，建议在deinit中发送通知；
6. 筛选在7.0快速滑动时报NSGenericException，建议从网络获取到数据后立即调用reload刷新数据；
7. UIButton 在7.0中tint默认为蓝色，影响效果；
8. UIButton的图片总是会受到tintColor的影响，自动改变图片的颜色，
    解决方案：在button的type里把System改为Custom
9. viewcontroller 重新执行viewWillAppear时，会重新调用该view的layoutSubviews，导致晃动
10. UIButton设置text时，如果const修改，必须在更改const之前，否则在ios7.x会显示不全。
11. Product Name为中文造成在ios7编译问题，解决办法：
          Bundle Name：Framework或Application的名字；
          Bundle Display Name: 显示给用户的Application的名字；
          只需要修改Bundle Display Name为“造作”，Bundle Name对应的Product Name保持“zaozuo”

12. xcode 7.3 swift 2.2 编译的app,如代码中使用泛型，在iOS7会直接crash,解决方法,使用7.2.1版本编译运行：
     12.1. 安装xcode 7.2.1
     12.2. 把 xcode 7.3 “/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport” 9.3
              文件夹 拷贝到 xcode 7.2.1的目录下
     https://bugs.swift.org/browse/SR-1096
     https://bugs.swift.org/browse/SR-815
     http://stackoverflow.com/questions/36235369/issue-with-swift-2-2-generics-xcode-7-3