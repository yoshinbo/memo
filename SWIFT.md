# Swift memo

## 32bit端末と64bit端末(iphon5s以降)のromdom関数について
The debugger is misleading you. The real problem is arc4random,
which will return an UInt32 on both iPhone 5 and 5s. But as iPhone 5 is a 32-bit device,
the Int(arc4random()) will cause an overflow if the random number is big enough.
Instead of using Int(arc4random()), you can try to replace it by using arc4random_uniform.
```swift
    // 32bit端末だと動かない
    var randomIndex: Int = Int(arc4random()) % (panels.count)
    // これだと両方動く
    var randomIndex: Int = Int(arc4random_uniform(UInt32(panels.count)))
```

## initとinitWithCoderの違い
StoryBoadで追加されたViewControllerはイニシャライズの時
initWithCoderが呼ばれる

## UIViewControllerのDelegateが呼ばれる順番
1. viewDidLoad                    // ViewController読み込み時
2. viewWillAppear                // View表示前
3. viewWillLayoutSubviews   // Viewの位置調整前
4. viewDidLayoutSubviews   // Viewの位置調整後 (StoryBoardのレイアウト調整はココで)
5. viewDidAppear                // View表示後
6. viewDidDisappear           // Viewが閉じられた後

## UIViewControllerで個別のstoryboadを使う場合
* Hoge StoryBoadを作成
* 読み込む側のViewController

```swift
import UIKit
class HomeViewController: UIViewController {
    // 略
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews();

        let size: CGSize = CGSize(width: self.contentScrollView.frame.width, height: self.contentScrollView.frame.height);

        //HogeViewController
        var hogeVc = HogeViewController.connectionViewController();
        self.addChildViewController(hogeVc);
        hogeVc.view.frame = CGRect(origin: CGPoint(x: 0, y: 0), size: size);
        self.contentScrollView.addSubview(hogeVc.view);
    }
}
```

* 読み込まれる側のViewController

```swift
import UIKit
class HogeViewController: UIViewController {
    class func connectionViewController() -> (HogeViewController) {
        var storyboard : UIStoryboard = UIStoryboard(name: "Hoge", bundle: nil);
        var viewController = storyboard.instantiateInitialViewController() as UIViewController;
        return (viewController as HogeViewController);
    }
}
```

## UIViewControllerで他のUIViewControllerを使う場合
* StoryBoadに読み込むUIViewControllerを定義
```
StoryBoadで定義したUIViewControllerのIdentity:StoryBoadIDに名前を定義
ここでは仮にhogeVcとする
```
* 読み込む側のUIViewController
```swift
let vc = self.storyboard?.instantiateViewControllerWithIdentifier("HogeVc") as UIViewController;
```
* 読み込んだUIViewControllerのviewを自分のCustomUIViewを使う
```
StoryBoadで定義したUIViewControllerの中のUIViewに自分でつくったCustomUIViewとする
コードでは
let domainPicker = vc.view as MyCustomUIView;
とすればそのまま使える
StoryBoad上からIBOutletをCutomViewに貼付けることもできる
```

## イベント
NSNotificationCenter
http://dev.classmethod.jp/references/ios-8-swift-nsnotification-userinfo/

ちゃんと消しましょう。
```
override func viewWillDisappear(animated: Bool) {
    super.viewWillDisappear(animated);
    NSNotificationCenter.defaultCenter().removeObserver(self);
}
```

## アニメーション
CAKeyframeAnimation

## UNIXタイム
* エポックからミリ行を取得
時刻取得関数 CFAbsoluteTimeGetCurrent() は2001年1月1日を起点としていて
1970年1月1日から2001年1月1日までの経過時間の値が定数kCFAbsoluteTimeIntervalSince1970
として用意されているので二つを足すと現在時間のミリ秒がでる
```
var now = CFAbsoluteTimeGetCurrent() + kCFAbsoluteTimeIntervalSince1970;
```
## UIViewController
* 最前面のViewControllerを取得する
```
// 最前面のViewContorllerを返す。最前面がNavigationViewControllerの場合はそのtopViewContollerを返す。
class func topMostController() -> UIViewController {
    var topController = UIApplication.sharedApplication().keyWindow!.rootViewController!;
    while((topController.presentedViewController) != nil) {
        topController = topController.presentedViewController!
    }
    if topController.isKindOfClass(UINavigationController) {
        var naviVc = topController as UINavigationController
        topController = naviVc.topViewController
    }
    return topController
}
```

## UIView
* UIViewのキャプチャをとる
```
func convertViewToImage()->UIImage{
    UIGraphicsBeginImageContext(self.bounds.size); // self is UIView
    self.drawViewHierarchyInRect(self.bounds, afterScreenUpdates: true);
    var image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```
http://qiita.com/nasu_st/items/561d8946966015abd448

* ブラーをバックぐらんどに設定
```
func setBackgroundBlur(){
    var blurredImage = self.view.convertViewToImage();
    blurredImage = blurredImage.applyBlurWithRadius(10, tintColor: UIColor(white: 1.0, alpha: 0.2), saturationDeltaFactor: 1.3, maskImage: nil);
    self.view.backgroundColor = UIColor(patternImage: blurredImage);
}
```

* タップした時のアクションを追加
```
override func viewDidLoad() {
    self.view.addGestureRecognizer(UITapGestureRecognizer(target: self, action: "hoge:"))
}

func hoge(sender: UITapGestureRecognizer) {
}
```
## AlertController
アラート出したいときさくっとでるので便利
```
var alertViewController = UIAlertController(title: "title", message: "message", preferredStyle: UIAlertControllerStyle.Alert)
alertViewController.addAction(UIAlertAction(title: "いいえ", style: .Cancel, handler: nil))
alertViewController.addAction(UIAlertAction(title: "はい", style: .Default, handler: {
    (action:UIAlertAction!) -> Void in
        println("OK")
}))
self.presentViewController(alertViewController, animated: true, completion: nil)
```

## UITextView
TextViewの文字列を上そろえにする方法
```
self.textView.textContainer.lineFragmentPadding = 0; // 左のpaddingを0に
self.textView.textContainerInset = UIEdgeInsetsZero; // 上のpaddingを0に
```
## Storyboad
* storyboadからUIViewだけ取り出してセット
```swift
var storyboad: UIStoryboard = UIStoryboard(name: "HogeViewController", bundle: nil)
var hogeUIViewController = storyboad.instantiateViewControllerWithIdentifier("HogeUIView") as UIViewController
self.view.addSubview(hogeUIViewController.view)
```

## Mail送信
http://www.mikamiz.jp/dev/iphone/a0005.html
http://stackoverflow.com/questions/24311073/mfmailcomposeviewcontroller-in-swift

## ローカライズ
https://akira-watson.com/iphone/localization.html

## cocoapods
ライブラリを管理してくれる便利な人
* インストール
なんか0.40系だとうまく動かないよう(2014/10/04現在)
yosemiteだと動かないらしい(2014/10/20現在)

```bash
sudo gem install pods -v 0.34.1
pod setup
```
* Podfileお作成
ここにいろいろとライブラリの依存関係を記載していく
プロジェクトルートにPodfileを用意して下記のように記述

```bash
platform : ios, "8.0"
pod 'AFNetworking', '~> 2.0'
```

ライブラリのインストール
Podfileのあるディレクトリで
```bash
pod install
```

## GLDTween アニメーション
```swift
        GLDTween.addTween(heartImageView, withParams: [
            "duration"      : 0.5,
            "delay"         : 1.5,
            "easing"        : GLDEasingInSine,
            "alpha"         : 0.0
            "completionBLock" : GLDTweenBlock({
                heartImageView.removeFromSuperview()
            })
        ])
```
