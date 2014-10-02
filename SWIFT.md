# initとinitWithCoderの違い
StoryBoadで追加されたViewControllerはイニシャライズの時
initWithCoderが呼ばれる

# UIViewControllerのDelegateが呼ばれる順番
1. viewDidLoad                    // ViewController読み込み時
2. viewWillAppear                // View表示前
3. viewWillLayoutSubviews   // Viewの位置調整前
4. viewDidLayoutSubviews   // Viewの位置調整後 (StoryBoardのレイアウト調整はココで)
5. viewDidAppear                // View表示後
6. viewDidDisappear           // Viewが閉じられた後

# UIViewControllerで個別のstoryboadを使う場合
* Hoge StoryBoadを作成
* 読み込む側のViewController

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

* 読み込まれる側のViewController

import UIKit
class HogeViewController: UIViewController {
    class func connectionViewController() -> (HogeViewController) {
        var storyboard : UIStoryboard = UIStoryboard(name: "Hoge", bundle: nil);
        var viewController = storyboard.instantiateInitialViewController() as UIViewController;
        return (viewController as HogeViewController);
    }
}
