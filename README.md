# Swinjectを使用してみた
### SwinJectとは、Swiftの軽量なDIフレームワークです。

### 目的
* DIフレームワークを用いた実装を練習するため
* DIフレームワークを導入するとどのようなメリットがあるのか知りたかったから

### 使用技術
* Swift

### 機能の説明
> * 依存しているオブジェクトを中で作らずに外から渡してあげることで、オブジェクト同士を柔軟に組み合わせることが可能になる。
> * テストがしやすくなる。

### 使用するために必要な条件
* iOS 9.0+ / Mac OS X 10.10+ / watchOS 2.0+ / tvOS 9.0+
* Xcode 10.2+
* Swift 4.2+
* Carthage 0.18+ (if you use)
* CocoaPods 1.1.1+ (if you use)

### Installation
1. PodFileに記述を追加する
```html
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'DI-Example' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  pod 'Swinject'

end
```

2. pod installを行う
```html
pod install
```

##### 今回SwinjectStoryboardは使用していませんが、使用する場合は下記のようにPodfileを作成してください。

```html
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'DI-Example' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  pod 'Swinject'
  pod 'SwinjectStoryboard'
end
```

### 基本的な使い方
SwinjectのREADMEを参考にしました。

1. 依存関係を持つモデルを定義する
```html
protocol Animal {
    var name: String? { get }
}

class Cat: Animal {
    let name: String?

    init(name: String?) {
        self.name = name
    }
}
```

```html
protocol Person {
    func play()
}

class PetOwner: Person {
    let pet: Animal

    init(pet: Animal) {
        self.pet = pet
    }

    func play() {
        let name = pet.name ?? "someone"
        print("I'm playing with \(name).")
    }
}
```

2. Swinjectを使ってDIをする
```html
import Swinject

let container = Container()
container.register(Animal.self) { _ in Cat(name: "Mimi") }
container.register(Person.self) { r in
    PetOwner(pet: r.resolve(Animal.self)!)
}

let person = container.resolve(Person.self)!
person.play() // prints "I'm playing with Mimi."
```

3. SwinjectStoryboardを使わない場合のService登録処理
```html
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    let container: Container = {
        let container = Container()
        container.register(Animal.self) { _ in Cat(name: "Mimi") }
        container.register(Person.self) { r in
            PetOwner(pet: r.resolve(Animal.self)!)
        }
        container.register(PersonViewController.self) { r in
            let controller = PersonViewController()
            controller.person = r.resolve(Person.self)
            return controller
        }
        return container
    }()

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {

        // Instantiate a window.
        let window = UIWindow(frame: UIScreen.main.bounds)
        window.makeKeyAndVisible()
        self.window = window

        // Instantiate the root view controller with dependencies injected by the container.
        window.rootViewController = container.resolve(PersonViewController.self)

        return true
    }
}
```

SwinjectStoryboardの処理は
```html
import SwinjectStoryboard

extension SwinjectStoryboard {
    @objc class func setup() {
        defaultContainer.register(Animal.self) { _ in Cat(name: "Mimi") }
        defaultContainer.register(Person.self) { r in
            PetOwner(pet: r.resolve(Animal.self)!)
        }
        defaultContainer.register(PersonViewController.self) { r in
            let controller = PersonViewController()
            controller.person = r.resolve(Person.self)
            return controller
        }
    }
}
```

##### SwinjectのREADMEは下記
`https://github.com/Swinject/Swinject/blob/master/README.md`
