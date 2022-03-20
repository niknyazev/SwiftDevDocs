**SearchController может автоматически убираться при скролинге**. Для этого необходимо установить свойство

```swift

navigationItem.hidesSearchBarWhenScrolling = true

```
работает это для TableViewController. Как сделать для обычного VC с TableView, я не понял

В SearchController можно из коробки встроить SegmentedController. Для этого используем конструкцию

```swift

searchController.searchBar.scopeButtonTitles = ["All", "New", "Learned"]

```

В CoreData для того чтобы сделать вычисляемое поле, можно использовать Derivation выражения. 
Пример подсчета количества карточек:

```swift

flashcards.@count

```