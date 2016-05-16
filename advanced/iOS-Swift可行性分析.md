##swift优势：
#####兼容性:
* 对oc无缝兼容:
通过建立桥接文件实现使用swift语法调用原oc代码
例如：新建zaozuo-ios-Bridging-Header.h文件，导入oc需要暴露给swift的类

```objective-c
//FMDB
#import "FMDB.h"

// shareSdk
#import <ShareSDK/ShareSDK.h>
#import <ShareSDKUI/ShareSDK+SSUI.h>
#import <ShareSDKConnector/ShareSDKConnector.h>
//腾讯SDK头文件
#import <TencentOpenAPI/TencentOAuth.h>
#import <TencentOpenAPI/QQApiInterface.h>
//微信SDK头文件
#import "WXApi.h"
//新浪微博SDK头文件
#import "WeiboSDK.h"
```
* Foundation、UiKit等系统框架保持oc调用方式

#####新特性：
* 新增泛型;
* 支持闭包（类似block），可用于回调;
* 新增元组；
* 数据类型更加简洁，常用基本数据类型：String、Int、Double、Array<T>、Dictionary<K, V>、Set<T> ，Array、Dictionary都具有可变性 ；
* 函数支持嵌套、多返回值（借助元组），函数作为参数传递;
* 新增Optionals，借助Optionals nil检查减少空指针异常；
* 新增as语法，可用于类型转换, 例如：String as NSString；
* 无所不能的switch。

##swift劣势：
#####兼容性问题
* swift调用原有oc导致的类型转换问题:
例如：原oc中NSArray会被转换为Array<AnyObject>，如果原oc NSArray中存放的对象非继承自AnyObject（例如：枚举类型）带来的转换问题（枚举类型可调用rawValue传入原始值）；
* 枚举等一些特殊类型调用方式略变。

##### CocoaPods支持问题:
* **Embedded frameworks require a minimum deployment target of iOS 8**
deployment target:7.0+无法使用CocoaPods引入swift开源库;
* 7.0+ CocoaPods替代方案：
采用swift library 可采用git subModule管理（当项目target:8.0+只需移除subModule，加入到CocoaPods），oc library 可保持CocoaPods依赖方式;

##### nil 处理方式改变，可能引入的新问题:
* swift采用Optionals处理变量值为nil的情况，正常情况下增强了调用的安全性，对于Optionals类型如果采用!强制取值可能引发无法避免的空指针；
例如：

```swift
public static func dictionaryToEntityList<T:BaseModel>(set:FMResultSet)
        -> Array<T>{
        var arr:Array<T> = []
        while set.next(){
            var entity:AnyObject = T.classForCoder().alloc()
            if entity is T{
                dictionaryToEntity(set.resultDictionary(), object: entity as! T)
                arr.append(entity as! T)
            }
        }
        return arr
    }
```
上面函数，传入的参数set是非Optionals，当采用以下方式调用时，可能引发空指针异常，而在dictionaryToEntityList函数中却无法避免此问题

```swift
var set:FMResultSet? = nil
dictionaryToEntityList(set!)
```

* 解决方案:
将set更改为Optionals或在dictionaryToEntityList函数中传入set前做非nil校验。

##UI方案：
采用storyborad:
* 多人协作不可同时修改storyboard中同一个ViewController


