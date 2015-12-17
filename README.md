# 快速入门

## 第一步 创建一个账号

首先，注册并登录 Wilddog 账号，进入[控制面板](https://www.wilddog.com/dashboard)。然后，在控制面板中，添加一个新的应用。你将会获得一个以 `wilddogio.com` 结尾的应用URL。以后，我们要用这个URL去存储和同步数据。 （注意：‘wilddogio.com’ 中有个 ‘io’）

## 第二步 SDK导入

要将 Wilddog SDK 导入到你的工程中，推荐使用 [CocoaPods](https://cocoapods.org/)，如果没用过 CocoaPods，请先访问 [CocoaPods getting started](https://guides.cocoapods.org/using/getting-started.html)。 


打开工程目录，新建一个 Podfile 文件

	$ cd your-project-directory
	$ pod init
	$ open -a Xcode Podfile # opens your Podfile in XCode

然后在 Podfile 文件中添加以下语句

	pod 'Wilddog', '~> 0.9.3'
	
最后安装 SDK

	$ pod install
	$ open your-project.xcworkspace
	
#####手动集成 

1、下载 SDK。[下载地址](https://www.wilddog.com/download#ios)         
2、把 Wilddog.Framework 拖到工程目录中。  
3、选中 Copy items if needed 、Create Groups，点击 Finish。  
4、点击工程文件 -> TARGETS -> General，在 Linked Frameworks and Libraries 选项中点击 '+'，将 JavaScriptCore.framework、 libsqlite3 加入列表中。

## 第三步 开始使用

在所需要的类中，引入头文件

Objective-C 

```
1、#import <Wilddog/Wilddog.h>

```

Swift

```
1、import Wilddog

```

## 第四步 读写数据

### 1、写数据

数据写到Wilddog数据库是比较简单的。首先，我们通过url创建一个Wilddog的引用，这样我们就可以使用setValue写入任何合法的JSON数据。


Objective-C 

```
// 创建一个引用到我们的数据库
Wilddog *myRootRef = [[Wilddog alloc] initWithUrl:@"https://<appId>.wilddogio.com"];
// 写数据
[myRootRef setValue:@"Do you have data? You'll love Wilddog."];

```

Swift
```
// 创建一个引用到我们的数据库
var myRootRef = Wilddog(url:"https://<appId>.wilddogio.com")
// 写数据
myRootRef.setValue("Do you have data? You'll love Wilddog.")

```


### 2、读数据

Wilddog服务器把数据实时同步给每一个正在监听的客户端。我们用 observeEventType 方法监听数据变化，然后会有一个block回调一个WDataSnapshot对象，它里面包含我们所需数据。

Objective-C 

```
// 读数据并监听数据变化
[myRootRef observeEventType:WEventTypeValue withBlock:^(WDataSnapshot *snapshot) {
    NSLog(@"%@ -> %@", snapshot.key, snapshot.value);
}];

```

Swift
```
// 读数据并监听数据变化
myRootRef.observeEventType(.Value, withBlock: {
  snapshot in
  println("\(snapshot.key) -> \(snapshot.value)")
})

```

上述例子中，value这个事件会在初次获取到数据的时候被触发一次，此后每当数据发生改变，都会被触发。了解关于更多的事件类型和如何处理事件数据，请参见 [查询数据](guide/4) 。

## 第五步 认证用户

Wilddog给用户提供邮箱和密码认证。

要用邮箱和密码认证，需要设置Wilddog的控制面板：  
1、选择“终端用户认证”栏；  
2、在“野狗默认用户数据库”中，勾选“开启野狗默认用户数据库服务”。  

设置完成后，用户验证已启用，您可以创建一个新用户：  

Objective-C 

```
[myRootRef createUser:@"bobtony@example.com" password:@"correcthorsebatterystaple"
    withValueCompletionBlock:^(NSError *error, NSDictionary *result) {
    if (error) {
        // 创建帐户时出错
    } else {
        NSString *uid = [result objectForKey:@"uid"];
        NSLog(@"Successfully created user account with uid: %@", uid);
    }
}];

```

Swift
```
myRootRef.createUser("bobtony@example.com", password: "correcthorsebatterystaple",
    withValueCompletionBlock: { error, result in
    if error != nil {
        // 创建帐户时出错
    } else {
        let uid = result["uid"] as? String
        println("Successfully created user account with uid: \(uid)")
    }
})

```

当创建你的第一个用户后，你可以登录去使用authUser方法。

## 第六步 数据保护

你可以使用强大的”规则表达式”来控制数据的访问权限，并且可以实现输入数据的有效性校验。

```
{
  ".read": true,
  ".write": "auth.uid == 'admin'",
  ".validate": "newData.isString() && newData.val().length() < 500"
}

```
强烈建议你使用规则表达式来限制你的数据访问权限。规则表达式的语法非常强大和灵活，你可以通过它来实现数据访问的细粒度控制。