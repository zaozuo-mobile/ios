#IOS-AutoLayout Constraint及动画使用

###Content Hugging Priority 和 Content Compression Resistance Priority：
* Content Hugging Priority，阻止超过frame默认长宽约束优先级，标准值为251；
当新添加的长宽约束优先级小于标准值251时此约束生效，使view按照frame默认长宽或小于默认长宽显示（以view实际内容为准）；
* Content Compression Resistance Priority，阻止小于frame默认长宽约束优先级，标准值为750；
当新添加的长宽约束优先级小于标准值750时此约束生效，使view按照frame默认长宽或大于默认长宽显示（以view实际内容为准）；
* 注，新建的Constraint默认优先级为1000，故正常情况下不会触发以上约束；
* 当多个平级的view的约束产生相互依赖时，例如，TableViewCell中viewA和viewB都未设置高度约束
viewA.top = superView.top， viewA.bottom = viewB.top，viewB.bottom = superView.bottom，
viewA高依赖viewB，viewB高度依赖viewA和superView，viewA、viewB高依赖对方，此时需要设置以上优先级来优先确定某一个view的高度<br>
1> 优先viewA：设置viewA Content Hugging Priority = 250, Content Compression Resistance Priority = 751
   将会按照viewA的内容不低于viewA默认frame的高来优先确定viewA的高;

###对已有Constraint更新、删除: 
* UIView的宽高约束属于UIView本身；
* center，edge（left/top/right/bottom）等约束属于所在UIView的父View;

```swift
@IBOutlet weak var closeBtn: UIButton!
@IBOutlet weak var closebtnWidthCon: NSLayoutConstraint!
@IBOutlet weak var closebtnHeightCon: NSLayoutConstraint!
@IBOutlet weak var closebtnTopCon: NSLayoutConstraint!

private func testUpdateAndDelete(){
    self.closeBtn.setTranslatesAutoresizingMaskIntoConstraints(false)

    // 删除原有的Constraint
    if closebtnWidthCon != nil{
        self.closeBtn.removeConstraint(closebtnWidthCon)
    }
    if closebtnHeightCon != nil{
        self.closeBtn.removeConstraint(closebtnHeightCon)
    }
    // 更改closeBtn宽高
    closeBtn.snp_makeConstraints { (make) -> Void in
        make.width.equalTo(150)
        make.height.equalTo(150)
    }
    
    // 从父View中移除closeBtn top约束
    self.view.removeConstraint(closebtnTopCon)
    // 更改closeBtn距离顶部距离top
    self.closeBtn.snp_makeConstraints { (make) -> Void in
        make.top.equalTo(self.view).offset(150)
    }
}
```

###采用Constraint的View动画:
####更改UIView的Frame:
* 更改UIView的Frame只会影响当前UIView，不会影响跟该UIView有约束关系的其它UIView
```swift
private func testAnim(){
        UIView.animateWithDuration(2.0, delay: 0.2, options: .CurveEaseInOut, animations: { () -> Void in
            
            // 更改位置坐标
            self.centerLabel.center.y = 100
            self.centerLabel.center.x = 100
            
            // 旋转动画
            let rotation = CGAffineTransformMakeRotation(CGFloat(M_PI))
            self.centerLabel.transform = rotation
            
            // 缩放动画
            let scale = CGAffineTransformMakeScale(0.5, 0.5)
            self.centerLabel.transform = scale
            
            
            }) { (isFinished:Bool) -> Void in
                
        }
}
```
###更改UIView的Constraints
* 修改已有约束，会引起其它相关UIView约束同步变化
```swift
private func modifyConstraints(){
    // 修改已有约束，会引起其它相关UIView约束同步变化
    self.closebtnWidthCon.constant = 200
    self.closebtnHeightCon.constant = 200
    
    UIView.animateWithDuration(2.0, animations: { () -> Void in
        // 刷新所有相关约束的UIView
        self.closeBtn.layoutIfNeeded()
        self.testSwitch.layoutIfNeeded()
        
        }) { (isFinished) -> Void in
    
    }
}
```    


