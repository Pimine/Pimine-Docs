# Библиотеки

### [EasySwiftLayout](https://github.com/pimine/EasySwiftLayout)

> Используется во всех проектах.

Позволяет писать UI код быстро, удобно и читаемо. Содержит подробную документацию с примером.

### [PimineSDK](https://github.com/pimine/PimineSDK)
> Используется во всех проектах.

Это наша собственная внутренняя большая библиотека, которая позволяет избежать повторений кода от проекта к проекту. Подробной документации по всем компонентам к сожалению пока нет.

Содержит в себе несколько подмодулей, которые можно подключать отдельно:
- Utilities
- HandyExtensions
- Firestore
- LocalStore
- RevenueCatStore
- SwiftyStore
- Math
- Database
- Concurrency

##### Utilities
Часто используемые utility-компоненты. В основном это небольшие классы с базовым функционалом, вспомогательные протоколы, property wrappers и другое. Является частью `Pimine/Core` (default submodule).

##### HandyExtensions
Удобные екстеншены для проекта. Подмодуль является расширением библиотеки `SwifterSwift`. В будущем планируется задокументировать, написать тесты и перенести в базовую библиотеку `SwifterSwift`. Является частью `Pimine/Core` (default submodule).

##### Firestore
Дополнительный функционал для удобной работы с `Firestore`. Позволяет рекурсивно получить все  `DocumentReference` в документе.

##### LocalStore / RevenueCatStore / SwiftyStore
3 модуля для удобной работы с in-app purchases.
- LocalStore. Локальная валидация рецепта.
- RevenueCatStore. Обертка для решения трекинга подписок RevenueCat.
- SwiftyStore. Валидация рецепта через сервера Apple, но все еще с shared secret на девайсе.

Во всех 3 взята за основу идея и архитектура [MerchantKit](https://github.com/benjaminmayo/merchantkit).

##### Math
Математические функции.

##### Database
DSL для Realm. 

##### Concurrency
Поддержка асинхронных операций.

### [ScuffedUI](https://github.com/pimine/ScuffedUI)
> Используется во всех проектах.

Небольшая библиотека, которая содержит часто используемые UI компоненты.
