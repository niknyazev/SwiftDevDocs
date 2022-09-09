![](blob:https://dvp1c.atlassian.net/3a01d2f5-04b0-4876-b7da-6398a95806e9)

Swift ARC.pdf

15 дек. 2021, 07:52 PM

**Ссылки**
https://medium.com/@almalehdev/you-dont-always-need-weak-self-a778bec505ef

Резюме статьи:
- `[unowned self]` is almost always a bad idea
- Non-escaping closures do not require `[weak self]` **unless** you care about delayed deallocation
- Escaping closures require `[weak self]` if they get stored somewhere or get passed to another closure **and** an object inside them keeps a reference to the closure
- `guard let self = self` can lead to delayed deallocation in some cases, which can be good or bad depending on your intentions
- GCD and animation calls generally do not require `[weak self]` **unless** you store them in a property for later use
- Be careful with timers!
- If you aren’t sure, [deinit](https://docs.swift.org/swift-book/LanguageGuide/Deinitialization.html) and Instruments are your friends

**ARC - механизм подсчета количества ссылок** на объект

Объекты отпускаются, если количество сильных ссылок равно нулю

Чтобы не происходило взаимное пересечение сильных ссылок (т.н. цикл сильных ссылок или цикличные сслыки), используется ключевое слово weak или unowned. Тем самым мы помогаем ARC разрывать цикл сильных ссылок

**По умолчанию все ссылки сильные**

**Отличие weak от unowned** - во втором случае ссылка должна быть инициализирована. unowned никогда не может указывать на nil и на ссылки с опциональным типом. На практике это значит, что если self это ViewController, который может быть закрыт, таким образом превратившись в nil, тогда использование unowned может привести к runtime ошибке.

**Для разрешения конфликта нужно определить какой объект является главным, какой второстепенным** и назначить в нем ссылку на главный объект как weak

**По умолчанию клоужеры захватывают ссылку на объект**, т.е. если мы меняем объект, каждый вызов клоужера будет отображать значение объекта с учетом изменений. Чтобы этого избежать, надо указывать _список захвата_

**Возможные проблемы памяти:**

-   Нехватка памяти - приложение отъедает память, система пытается высвободить еще памяти закрывая другие приложение или даже отключая собственные службы
-   Зависшие объекты - они же цикличные ссылки (утечка памяти), ранее созданные объекты зависают в памяти, при этом ни одной ссылки на объект нет
-   Несуществующие объекты - обращение к несуществующим объектам, например к опциональным со значением nil
    
**Типы объектов:**

-   Типы значений
-   Ссылочные типы
    
**Типы значений:**

-   Строки, числа, булево
-   Структуры, массивы, словари, перечисления
    
**Ссылочные типы:**

-   Классы
-   Клоужеры (именно поэтому в клоужерах требуется указывать слабые ссылки)
    
**В панеле дебагинга если значение отображается** - это тип значения, если не отображается - это ссылочный тип

**Чтобы увидеть значения ссылочного типа,** надо нажать ПКМ в поле дебагинга и выбрать **Print description** или нажав кнопку i

**Для выявления утечки памяти можно использовать инструмент Leaks**, который входит в состав Xcode

**При создании UI классов, UI свойства нужно делать lazy**, поскольку при инициализации класса у нас есть нет доступа к view. Соотвественно свойства станут доступны, когда доступ к UI появится

**В клоужерах пишется self** чтобы мы не забывали, что это место потенциальной утечки памяти. Если замыкание не убегающее - то self писать не нужно

**В оутлетах ставится weak** если к нему будет обращение через self

Таким образом в клоужере, если идет обращение к свойству класса или self или свойство должно быть слабой ссылкой

**При использовании делегатов, надо следить за сильными ссылками**. В данном примере если не использовать weak в строке 24, деинициализаторы не вызовуться:

```swift

protocol ProtocolMethod: class {
    func anyMethod()
}

class Launcher: ProtocolMethod {
    
    let secondClass = SecondClass()
    
    init() {
        secondClass.delegate = self
    }
    
    func anyMethod() {
        print("\(self)")
    }
    
    deinit {
        print("\(self) is deinited")
    }
}

class SecondClass {
    
    weak var delegate: ProtocolMethod?
    
    func executeMethod() {
        delegate?.anyMethod()
    }
    
    deinit {
        print("\(self) is deinited")
    }
}

var launcher: Launcher? = Launcher()
launcher = nil

```

**Чтобы в свойстве класса можно было обратиться к view** надо использовать слово lazy. Поскольку класс еще не инициализирован, и мы не можем обращаться к контексту

**Аутлеты можно и не создавать как weak параметры**, если нет к ним оращения через self

**Когда мы обращаемся через self в клоужерах**, у нас создается еще одина ссылка (?) - self

Swift требует написание self в клоужерах, чтобы напоминать, что создается еще одна ссылка на экземпляр класса

По поводу использования листов захвата клоужеров в теле методов:

> The key question to ask yourself is if your alert object is "owned" by self. In this case, it is not (because you declared `let alert = ...` in the function body). So you do not need to create this as a weak or unowned reference.

в клоужерах weak или onown зависит от опционаьности самого клоужера (?)