#IOS-自定义UIViewController转场动画

## 一：Modal Segue：
###页面打开动画：

```swift
class MainViewController: UIViewController, UIViewControllerTransitioningDelegate {

    // 1. 让ViewController实现UIViewControllerTransitioningDelegate协议;
    func animationControllerForPresentedController(presented: UIViewController, presentingController presenting: UIViewController, sourceController source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        
        // 2. 返回自定义的UIViewControllerAnimatedTransitioning;
        return CustomAnimatedTransitioning()
    }
    
    //3. 在Main.storyboard中配置segue的Identifier;
    //4. prepareForSegue方法中根据Identifier决定跳转某个页面的动画.
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if "showlogin" == segue.identifier{
            let toViewController = segue.destinationViewController as! UIViewController
            println("打开登录页面===\(toViewController)")

            toViewController.transitioningDelegate = self
        }
       
    }
}
```
###页面关闭动画：

```swift
// 1. 让ViewController实现UIViewControllerTransitioningDelegate协议;
// 2. 页面关闭后动画效果
func animationControllerForDismissedController(dismissed: UIViewController)
        -> UIViewControllerAnimatedTransitioning? {
    return DismissAnimatedTransitioning()
}    
```
###定义动画

```swift
import Foundation

/**
自定义转场动画，必须实现UIViewControllerAnimatedTransitioning协议.
**/
public class CustomAnimatedTransitioning : NSObject, UIViewControllerAnimatedTransitioning{
    
    // 单位:second
    public func transitionDuration(transitionContext: UIViewControllerContextTransitioning) -> NSTimeInterval {
        return 1.0;
    }
    
    public func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
        let fromViewController = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
        let toViewController = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
        let finalFrameForVC = transitionContext.finalFrameForViewController(toViewController)
        let containerView = transitionContext.containerView()
        let bounds = UIScreen.mainScreen().bounds
        
        toViewController.view.frame = CGRectOffset(finalFrameForVC, 0, -bounds.size.height)
        containerView.addSubview(toViewController.view)
        
        UIView.animateWithDuration(transitionDuration(transitionContext), delay: 0.0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0.0, options: .CurveLinear, animations: {

                fromViewController.view.alpha = 0.5
                toViewController.view.frame = finalFrameForVC
            }, completion: {
                finished in
                
                transitionContext.completeTransition(true)
                fromViewController.view.alpha = 1.0
        })
        
    }

}
```
## 二：UINavigationController中子ViewController转场动画
###页面打开和关闭动画：

```swift
import UIKit

// 1. 在navigationController的root view controller实现UINavigationControllerDelegate协议.
class MainViewController: UIViewController, UINavigationControllerDelegate{

    override func viewDidLoad() {
        super.viewDidLoad()
        // 2. 设置navigationController的代理
        navigationController?.delegate = self
    }
    
    // 3. 根据operation类型决定使用何种动画.
    func navigationController(navigationController: UINavigationController, animationControllerForOperation operation: UINavigationControllerOperation, fromViewController fromVC: UIViewController, toViewController toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        
        if operation == UINavigationControllerOperation.Pop{
            // 页面关闭动画
            return DismissAnimatedTransitioning()
        }else{
            // 页面打开动画
            return CustomAnimatedTransitioning()
        }
    }
}
```

## 三：UITabBarController中子ViewController转场动画
###页面打开和关闭动画：
类似UINavigationController实现方式，更改代理为UITabBarControllerDelegate即可。

参考：
* [自定义ViewController切换效果与动画](https://github.com/bboyfeiyu/iOS-tech-frontier/blob/master/issue-2/%E8%87%AA%E5%AE%9A%E4%B9%89ViewController%E5%88%87%E6%8D%A2%E6%95%88%E6%9E%9C%E4%B8%8E%E5%8A%A8%E7%94%BB.md)


