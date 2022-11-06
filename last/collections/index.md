# Перечисление

## IEnumerable и IEnumerator

IEnumerator - базовый интерфейс, с помощью которого перечисляются коллекции в однонаправленной манере.

```csharp
public interface IEnumerator
{
    bool MoveNext();
    object Current { get; }
    void Reset();
}
```

Описание:

- MoveNext() - передвигает текущий курсор на следущую позицию, возвращая false, если в коллекции больше не осталось элементов
- Current - возвращает элемент в текущей позиции
- Reset() - выполняет перемещение курсора к началу, делая возможным перечисление коллекции заново (он необязателен, потому что обычно создает новый экземпляр перечислителя)

Примечания:

- Перед вызовом Current, необходимо вызвать MoveNext(), для учета пустой коллекции (компилятор говорит об этом)

IEnumerable - интерфейс, который можно воспринимать как поставщика IEnumerator, также он является базовым интерфейсом, который реализуют классы коллекций

```csharp
public interface IEnumerable
{
    IEnumerator GetEnumerator();
}
```

Почему классы коллекций реализуют IEnumerable, а не IEnumerator:

- реализация логики итерации может быть возложена на другой класс
- несколько потребителей способны выполнять перечисление коллекции одновременно, не влияя друг на друга

Пример низкоуровневого использование интерфейсов

```csharp
var someString = "hello";
var enumerator = someString.GetEnumerator();

while (enumerator.MoveNext())
{
    Console.WriteLine(enumerator.Current);
}
```

Пример через синтаксическое сокращение

```csharp
var someString = "hello";

foreach (var item in someString)
{
    Console.WriteLine(item);
}
```

## IEnumerable`<T>` и IEnumerator`<T>`

- обобщенные версии
- обеспечивают статическую типизацию и избегают (упаковки/распаковки)
- массивы автоматически реализуют IEnumerable`<T>`

```csharp
public interface IEnumerator<T> : IEnumerator, IDisposable
{
    T Current { get; }
}

public interface IEnumerable<T> : IEnumerable
{
    IEnumerator<T> GetEnumerator ();
}
```

## IEnumerable`<T>` и IDisposable

- IEnumerator`<T>` унаследован от IDisposable

Это позволяет перечислителям хранить ссылки на такие ресурсы, как подключения к базам данных, и гарантировать, что ресурсы будут освобождены, когда перечисление завершится.

Оператор `foreach` распознает такие детали и транслирует код в таком формате

```csharp
foreach (var element in somethingEnumerable) { ... }

using (var rator = somethingEnumerable.GetEnumerator())
{
    while (rator.MoveNext())
    {
        var element = rator.Current;
    }
}
```

## Когда использовать необобщенные интерфейсы?

Бывает надо создать метод, который принимает параметры любого типа, к примеру (IEnumerable e). Но почему же не использовать обобщенную версию? Дело в том, что есть устаревшие коллекции, которые не поддерживают обобщения.

## Реализация интерфейсов перечисления

Когда потребуется реализация IEnumerable или IEnumerable`<T>`:

- для поддержки `foreach`
- для взаимодействия со всем, что ожидает стандартную коллекцию

Чтобы реализовать IEnumerable или IEnumerable`<T>` потребуется предоставить перечислитель. Сделать это можно тремя способами:

- реализовать IEnumerable
- через итератор с использованием оператора yield return
- построение класса, который реализует IEnumerator напрямую

Первый способ сводится к тому, чтобы реализовать GetEnumerator.

```csharp
public class MyCollection : IEnumerable
{
    int[] data = { 1, 2, 3 };

    public IEnumerator GetEnumerator()
    {
        foreach (int i in data)
            yield return i;
    }
```

Cтолкнувшись с оператором `yield return`, компилятор за кулисами генерирует скрытый вложенный класс перечислителя, проводит рефакторинг метода `GetEnumerator` для создания и возвращения экземпляра данного класса.

Второй способ.

```csharp
public static IEnumerable<int> GetSomelntegers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

Третий способ, это то, что компилятор делает за кулисами.

```csharp
public class MylntList : IEnumerable
{
    int[] data = { 1, 2, 3 };
    public IEnumerator GetEnumerator()
    {
        return new Enumerator(this);
    }
    class Enumerator : IEnumerator
    {
        MylntList collection;
        int currentlndex = -1;

        public Enumerator(MylntList collection)
        {
            this.collection = collection;
        }

        public object Current
        {
            get
            {
                if (currentlndex == -1)
                    throw new InvalidOperationException("Enumeration not started!");
                if (currentlndex == collection.data.Length)
                    throw new InvalidOperationException("Past end of list!");
                return collection.data[currentlndex];
            }
        }

        public bool MoveNext()
        {
            if (currentlndex >= collection.data.Length - 1) return false;
            return ++currentlndex < collection.data.Length;
        }

        public void Reset() { currentlndex = -1; }
    }
}
```

# ICollection и IList

Добавляют нофую функциональность:

- получить размер коллекции
- доступ к элементам коллекции по индексу
- модификация
- поиск

Обобщенные и необобщенные версии отличаются между собой, т.к. обобщения появились позже.

Поэтому ICollection`<T>` не расширяет ICollection, IList`<T>` не расширяет IList. Разумеется, что класс коллекции свободен в реализации обеих версий интерфейса.

## ICollection`<T>` и ICollection

- интерфейс - предоставляющий методы для модификации коллекций, поверки на содержание элемента в коллекции, подсчета количества элементов
- расширяет IEnumerable

```csharp
public interface ICollection<T> : IEnumerable<T>, IEnumerable
{
    int Count { get; }
    bool IsReadOnly { get; }

    bool Contains (T item);
    void CopyTo (T[] array, int arraylndex);

    void Add (T item);
    bool Remove (T item);
    void Clear ();
}
```

```csharp
public interface ICollection : IEnumerable
{
    int Count { get; }
    bool IsSynchronized { get; }
    object SyncRoot { get; }

    void CopyTo (Array array, int index);
}
```

## IList`<T>` и IList

- интерфейс для коллекций, поддерживающих индексацию по позиции
- расширяет IEnumerable, ICollection
- массивы реализуют IList`<T>` и IList, хотя методы модификации скрыты через явную реализацию интерфейса и в случае вызова генерируют исключение

```csharp
public interface IList<T> : ICollection<T>, IEnumerable<T>, IEnumerable
{
    T this [int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
}
```

```csharp
public interface IList : ICollection, IEnumerable
{
    object this [int index] { get; set; }
    bool IsFixedSize { get; }
    bool IsReadOnly { get; }
    int Add(object value);
    void Clear();
    bool Contains(object value);
    int IndexOf(object value);
    void Insert(int index, object value);
    void Remove(object value);
    void RemoveAt(int index);
}
```

# Класс Array

- неявный базовый класс для массивов
- является ссылочным типом
- реализует IEnumerable, IList

Как массив преобразуется в класс

- CLR неявно создает подтип класса Array
- CLR выделяет массиву непрерывный участок в памяти (делает индексацию высокоэффективной, но невозможно изменить размер в будущем)

Класс Array в действительности предлагает статический метод Resize, хотя он работает путем создания нового массива и копирования в него всех элементов. Вдобавок к такой неэффективности ссылки на массив в других местах программы будут по-прежнему указывать на его исходную версию.

Класс Array предлагает набор методов для поиска элементов внутри массива. Ни один из этих методов не генерирует исключение, если значение не найдено. Взамен этому, методы возвращают целочисленное значение -1. А если метод возвращает обобщенный тип, то значние по умолчанию.

# Списки, очереди, стеки и наборы

## Классы List`<T>` и ArrayList

- List`<T>` - обобщенный
- ArrayList - необобщенный

Оба класса представляют `массив объектов` с возможностью динамического изменения размера. При достижении предела емкости, массив заменяется на более крупный.

- List реализует IList, IList`<T>`
- ArrayList реализует IList

## Класс LinkedList`<T>`

- двусвязный список - цепочка узлов, в которой каждый узел ссылаяется на предыдущий и следующий и действительный узел
- класс реализует IEnumerable`<T>` и ICollection`<T>`

Преимущество:

- эффективно вставить элемент в любое место, т.к. надо создать новый узел и обновить пару ссылок. Однако поиск позиции для вставки может оказаться медленным, в виду того, что в LinkedList не встроена индексация (должен производиться обход каждого узла, и двоичный поиск невозможен)

Как выглядит узел

```csharp
public sealed class LinkedListNode<T>
{
    public LinkedList<T> List { get; }
    public LinkedListNode<T> Next { get; }
    public LinkedListNode<T> Previous { get; }
    public T Value { get; set; }
}
```

Класс предоставляет такие внутренние поля как:

- получить кол-во элементов в списке
- получить голову и хвост списка

## Queue`<T>` и Queue

- структура данные FIFO
- класс реализует IEnumerable`<T>` и ICollection`<T>`
- внутренне реализован с применением массива, который расширяется при необходимости
- очередь поддержвивает индексы, которые указывают на начальный и последний элементы

Отсутствет:

- доступ по индексу, т.к. не реализуют IList/IList`<T>` (можно обойти применив метод `ToArray()`)

## Stack`<T>` и Stack

- структура данных LIFO
- класс реализует IEnumerable`<T>` и ICollection`<T>`
- внутренне реализован с применением массива, который расширяется при необходимости

# Словари

- коллекция, где каждый элемент пара ключ/значение

## IDictionary`<TKey,TValue>`

- расширяет ICollection`<T>`, IEnumerable

Добавление элемента в словарь:

- метод Add - если ключ дублируется, то исключение
- по индексу - если ключ дублируется, то значение ключа изменяется

Получение элемента из словаря:

- метод TryGetValue - если ключ не существует, то false
- по индексу - если ключ не существует, то исключение

Особенности:

- дублирование ключей запрещено

## Dictionary`<TKey,TValue>` и Hashtable

- обобщенный класс
- для хранения ключей и значений использует структуру данных в форме хеш-таблицы

```csharp
var d = new Dictionary<string, int>();

d.Add("One", 1);
d["Two"] = 2;
d["Two"] = 22;
d["Three"] = 3;

Console.WriteLine (d["Two"]);
Console.WriteLine (d.ContainsKey ("One")); // (быстрая операция)
Console.WriteLine (d.ContainsValue (3)); // (медленная операция)

int val = 0;
if (Id.TryGetValue ("onE", out val))
Console.WriteLine ("No val"); // "No val" (чувствительно к регистру)

// Три разных способа перечисления словаря;
foreach (KeyValuePair<string, int> kv in d)
Console.WriteLine (kv.Key + "; " + kv.Value)
foreach (string s in d.Keys) Console.Write (s)
Console.WriteLine();
foreach (int i in d.Values) Console.Write (i);
```

Как работает хеш-таблица:

- преобразует ключ каждого элемента в хеш-код (псевдоуникальное значение)
- затем применяет алгоритм для преобразования хеш-кода в хеш-ключ
- хеш-ключ используется внутренне для определения того, к какому "сегменту" относится запись

Если сегмент содержит более одного значения, то тогда в нем производится линейный поиск

Недостаток Dictionary и HashTable:

- элементы не отсортированы
- нарушется первоначальный порядок добавленных элементов

## OrderedDictionary

- необобщенный словарь, хранит элементы в порядке их добавления
- можно получать элементы по индексам и ключам
- комбинация HashTable и ArrayList

Недостаток:

- не является отсортированным

## ListDictionary и HybridDictionary

- ListDictionary - в этом классе применяется односвязный список (медленно работает с большими списками)
- HybridDictionary - это ListDictionary, который автоматически преобразуется в HashTable при достижении определенного размера, решая проблемы с низкой производительностью класса ListDictionary

## Отсортированные словари

- .NET предлагает два класса словарей, которые внутренне устроены так, что их содержимое всегда сортируется по ключу
- SortedDictionary`<TKey, TValue>` - применяет красно-черное дерево
- SortedListc`<TKey, TValue>` - внутренне реализован с помощью пары упорядоченных массивов
