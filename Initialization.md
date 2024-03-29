[[Temp Swift]]

Инициализация собсвенных свойств класса должно быть определено перед вызовом конструктора суперкласса (Фаза 1), а переопределение свойств супер класса - после (Фаза 2)

```swift

class Test01 {
    var i: Int
    
    init() {
        i = 10
    }    
}

class Test02: Test01 {
    var j: Int
    
    override init() {
        j = 20 // j можем инициализировать только до super.init()
        super.init()
        i = 30 // i можем переопределить только после super.init()
    }
}

```

Свойства класса надо инициализировать непосредственно в самом кострукторе. Компилятор не "увидит" инициализацию свойств, если их вынести в отдельные методы - обертки.

## Рефакторинг конструктора
Конструктор не позволяет вызывать методы экземпляра, пока не инициализированы все свойства. Из-за этого нельзя провести рефакторинг коструктора, развиб его на несколько отдельных методов. С этим можно бороться следующим образом:
- Сделать свойства принудительно извлекаемыми опционалами. Но это не безопасно.
- Сделать методы инициализации приватными и статическими.