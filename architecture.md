# Архитектура

![](https://i.imgur.com/LOhPlE6.png)
Pimine использует архитектуру [MVVM-C](https://medium.com/sudo-by-icalia-labs/ios-architecture-mvvm-c-introduction-1-6-815204248518) с некоторыми изменениями. _Рекомендуется_ ознакомиться с основной статьей на Medium перед прочтением. Эта документация служит как дополнения. Здесь будут описаны некоторые отличия и нюансы.

Рекомендованная литература:
1. [Introduction to iOS App Architectures by Daniel Lozano Valdés](https://medium.com/sudo-by-icalia-labs/introduction-to-ios-app-architectures-59f86801a2ad);
2. [iOS Architecture: MVVM-C by Daniel Lozano Valdés](https://medium.com/sudo-by-icalia-labs/ios-architecture-mvvm-c-introduction-1-6-815204248518).


##### AppDelegate
Это начальная точка входа в приложения для большинства случаев (за исключениям extensions). Здесь происходит создание `UIWindow`, а так же начало флоу для главного координатора `AppCoordinator`.

```swift
// Главный координатор создаеться глобально. 
// Хоть это и являеться анти-патерном, это одно из исключений для удобства.
var coordinator: AppCoordinator!

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    
    func application(
        _ application: UIApplication, 
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // Создаем окно приложения и делаем его главным.
        window = UIWindow()
        window!.makeKeyAndVisible()
        
        // Передаем созданное окно в качестве инициализации и запускаем флоу с помощью метода start.
        coordinator = AppCoordinator(window: window!)
        coordinator.start()
        
        return true
    }
}
```

##### Coordinator

Задача координатора выполнять навигацию в приложении и перемещение между экранами. Обычно это push или present нового контролера после действий пользователя. 

В [PimineSDK](https://github.com/Pimine/PimineSDK/blob/master/PimineUtilities/PMCoordinator.swift) реализован базовый функционал координатора в `PMCoordinator`. Он содержит методы `start` и `finish`, а так же методы для добавления дочерних координаторов. Удаление со списка `childCoordinators` происходит автоматически после вызова `finish`.

Пример реализации главного координатора с 2 разделами в таб баре:
```swift
// Подключаем библиотеку PimineUtilities для PMCoordinator
import PimineUtilities
import UIKit

final class AppCoordinator: PMCoordinator {
    let window: UIWindow
    
    // Абстрактная фабрика для создания и настройки экранов.
    private lazy var factory = AppFactory(coordinator: self)
    
    // `UITabBarController` с двумя разделами.
    private(set) lazy var tabBarController = factory.makeTabBarController()
    
    // Два разделама (сцены).
    let tabCoordinator1 = TabCoordinator1()
    let tabCoordinator2 = TabCoordinator2()
    
    var tabBarCoordinators: [PMCoordinator] {
        [childCoordinator1, childCoordinator2]
    }
    
    init(window: UIWindow) {
        self.window = window
        
        let rootViewController = configure(UINavigationController()) {
            $0.isNavigationBarHidden = true
        }
        super.init(rootViewController: rootViewController, autoStartFlow: false)
    }
    
    override func start() {
        // Устанавливаем в качестве главного контролера UIWindow рут контролер AppCoordinator-а.
        window.rootViewController = rootViewController
        openTabBar()
    }
    
    override func finish() { 
        // Оставляем finish пустым, так как AppCoordinator никогда не завершает свой флоу.
    }
    
    private func openTabBar() {
        tabBarCoordinators.forEach {
            addChildCoordinator($0)
            $0.start()
        }
        rootViewController.pushViewController(tabBarController, animated: false)
    }
}
```

##### Scene factory
Абстрактная фабрика была создана, чтобы убрать создания и биндинг экранов в самом координаторе, позволив координатору следовать принципам SOLID и быть Single Responsibility. Каждая сцена имеет один координатор и одну фабрику. Фабрика имеет `unowned` ссылку на координатор.

Пример реализации фабрики для главного координатора с 2 разделами в таб баре:
```swift
import UIKit

final class AppFactory {
    // Хотя и в данном случае unowned смысла не имеет (AppCoordinator живет всегда),
    // но всегда делает ссылку на coordinator unowned, чтобы избежать retain cycle.
    private unowned var coordinator: AppCoordinator
    
    init(coordinator: AppCoordinator) {
        self.coordinator = coordinator
    }
    
    func makeTabBarController() -> UITabBarController {
        let tab1 = coordinator.tabBarCoordinators[0].rootViewController
        tab1.tabBarItem = UITabBarItem(title: "Example1", image: nil,  selectedImage: nil)
        
        let tab2 = coordinator.tabBarCoordinators[1].rootViewController
        tab2.tabBarItem = UITabBarItem(title: "Example2", image: nil,  selectedImage: nil)
        
        let tabBar = UITabBarController()
        tabBar.viewControllers = [charger, themes]
        return tabBar
    }
}
```

##### ViewModel
Одна из проблем MVVM архитектуры заключаеться в том, что ViewModel начинает нарушать SRP, выполняя слишком много обязанностей. Чтобы этого избежать, есть service слой и ViewData. ViewModel имеет one-to-one отношения с `UIViewController`, причем `UIViewController` не создает ViewModel сам. С помощью dependency injection это делает абстрактная фабрика.

##### Service Layer
Сервисы позволяют нам снять нагрузку с ViewModel. Здесь обычно происходит общение с API, базой данных, какая-то специфичная логика. ViewModel может имееть несколько сервисов.

##### ViewData
Архитектура MVVM подразумевает, что view-cлой не должен работать с моделью, но как нам отобразить модель в интерфейсе? Для этого существует ViewData. Она хранить модель и выполняет подготовку и преобразования модели для отображения в пользовательском интерфейсе.
