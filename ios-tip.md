
####好用的工具
xclog
 
#### colleciton view cell 里面有buttom, cell 重用无法清除上次信息的状态
	遇到了一次这个问题很是无解
 
####获取状态栏点击事件
 
 ```
 override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {
    super.touchesBegan(touches, withEvent: event)
    let events = event!.allTouches()
    let touch = events!.first
    let location = touch!.locationInView(self.window)
    let statusBarFrame = UIApplication.sharedApplication().statusBarFrame
    if CGRectContainsPoint(statusBarFrame, location) {
        NSNotificationCenter.defaultCenter().postNotificationName("statusBarTouched", object: nil)
    }
}
```

#### 竟可能少用lazy ,lazy 是得原本判断nil的逻辑无法正常实现

```
	private lazy var confirmAddressButton : UIButton? = {[weak self] in
        self?.getConfirmAddressButton()
    }()
   ```
* 父类调用子类，父类方法接受一个block,子类重写方法调用super传递这个block 

* 可测试性，书写uiview或其他业务逻辑时，业务逻辑要支持可配置，在测试或后续修改中，通过修改状态就可以复现不同状态下的业务展示，而不用去配置数据（真实重现业务不方便） 

*  collection view 设置了 contentInset 之后，位置滚动方法，scrollToCellAtIndexPath 不好用了 mCollectionView.setContentOffset

```
private func scrollToBottom(){
//        let scrollIndex = optionList.count - 1
//        mCollectionView.scrollToItemAtIndexPath(NSIndexPath(forItem: scrollIndex, inSection: 0), atScrollPosition: .Bottom, animated: false)
        let contentHeight = mCollectionView.contentSize.height
        let currentOffSet = contentHeight - mCollectionView.bounds.size.height
        mCollectionView.setContentOffset(CGPoint(x: 0, y: currentOffSet), animated: false)
    //}
```    

#### reload 刷新失效
 collection view  reloaddata 会和 reloadItemsAtIndexPaths,scrolltoItemAtIndexPath 方法冲突，如果执行二者之后再执行reload data 则不会生效，sku 选择弹层，sku 套餐选中之后没有调用binddata,因为之前有地方调用了reloadItemsAtIndexPaths





####多个optional 类型解包   

```
 if vc == nil || itemInfo == nil{
    return
 }
 guard let vcNonNil = vc, let itemInfoNonNil = itemInfo else {
    return
 }
```


####hittest 事件传递

way1:希望这样子但没成功，二者没有super 关系

```
override func hitTest(point: CGPoint, withEvent event: UIEvent?) -> UIView?{
//        UIView *hitView = [super hitTest:point withEvent:event
        let hitView = super.hitTest(point, withEvent: event)
        if let btnView = hitView?.superview as? UIButton {
            return btnView
        }
        return hitView
    }
```

way2:最后的解决方式如下：

```
 func promotionViewTap(tap:UITapGestureRecognizer){
        if let viewTag = tap.view?.tag {
            let btnTag = viewTag - 10
            if let senderButton = self.viewWithTag(btnTag) as? UIButton {
                btnClick(senderButton)
            } else if (btnTag == 0) {
                emptySenderWhenTagisZero.tag = 0
                btnClick(emptySenderWhenTagisZero)
            }
        }
    }
```

way3: pointInsite,touchbegin


#####动画
http://www.devtalking.com/articles/uiview-transition-animation/
https://zsisme.gitbooks.io/ios-/content/index.html

######平移动画
```
  func showView(){
        self.hidden = false
        let originY = self.frame.origin.y
        self.frame.origin.y = AppHeight * 2
        
        self.duringAnimate = true
        
        UIView.animateWithDuration(1.0,
            animations: { [weak self]() -> Void in
                self?.frame.origin.y = originY
            },
            completion: {[weak self](finish) -> Void in
                self?.duringAnimate = false
            }
        )
    }
```

##### 第三方库加载中间层
ipv6 第三方库修改方案图片库是直接使用的，导致修改成本很高

##### ios中唯一标示
使用Keychain解决iOS的UDID问题:
http://www.jianshu.com/p/f8f380cb06cb

####通话中加载 app lauchimage 变形的问题
启动图片那个问题解决了，需要同时设置，launch screen file 和 lauch image, ios7以上优先使用launch screen file 展示，ios7及其以下版本使用 lauch image

    

 
   
   
   