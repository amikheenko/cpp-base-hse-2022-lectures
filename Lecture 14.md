# Лекция 14. Итераторы и алгоритмы стандартной библиотеки. (01.03.2022)


## Итераторы

Итератор - объект, являющийся неким обобщением указателей и выполняющий по сути 2 функции:

- получение доступа к какому-то элементу физической или виртуальной коллекции
- обеспечение перемещения по элементам коллекции

У всех итераторов один интерфейс, за счет чего их можно использовать в шаблонных функциях.



### Виды итераторов

Итераторы различаются по методам прохода по коллекции:

#### Forward iterator
Определен только `operator++`    
Пример - `std::forward_list`    

#### Bidirectional iterator
Определены `operator++` и `operator--`    
Пример - `std::list`    

#### Random access iterator
Определены операторы `++`, `--`, `+=`, `-=`, `либо operator[]` в коллекции, который обычно работает так же, как и `итератор + число` - то есть имеем доступ к любому элементу за константу.
Пример - `std::vector`.

Также есть еще одна классификация итераторов:

#### Input iterator
Может считать значение, оператор разыменовывания * возвращает константную ссылку (либо копию) на элемент, на который он указывает.

#### Output iterator
Его оператор * возвращает неконстантную ссылку на элемент, на который он указывает, что позволяет записать по нему значение.

Что делать, если мы хотим:

- понимать, как далеко 2 итератора находятся друг от друга? 
- перемещать итератор на произвольное число элементов?

Если у нас есть `random access iterator`, то можно просто использовать `operator-=` и `operator+=`.    
Однако у `std::list`, к примеру, такой оператор не определен (так как итератор коллекции `std::list` - `bidirectional iterator`).    
Такой код не скомпилируется:

```c++
std::list<int> numbers = {1, 2, 3};
auto it1 = numbers.begin();
auto it2 = numbers.end();

std::cout << it2 - it1 << std::endl; // ошибка компиляции
```

Можно идти оператором ++ и считать количество шагов, а можно использовать уже готовую функцию `std::distance`.

```c++
std::list<int> numbers = {1, 2, 3};
auto it1 = numbers.begin();
auto it2 = numbers.end();

std::cout << std::distance(it1, it2) << std::endl; // 3
```

Это шаблонная функция, имеющая специализацию для разных итераторов - если она получит `random access iterator`, то она вычислит значение за константу.    
Если мы хотим сдвинуть итератор на произвольное число элементов, то можем использовать функцию `std::advance`, которая также имеет специализации для каждого вида итераторов.

```c++
std::list<int> numbers = {1, 2, 3};
auto it1 = numbers.begin();

std::advance(it1, 2);

std::cout << *it1 << std::endl; // 3
```



### begin и end


```c++
auto it2 = numbers.end(); // итератор на виртуальный элемент, существующий за концом коллекции
```

Этот элемент не всегда является итератором, он имеет тип `Sentinel` - специальный тип, который можно сравнить с итератором. `Iterator` и `Sentinel` взаимозаменяемы.


    
### Как обобщенно вызывать begin и end?

У массива нет методов `begin` и `end`, итератора как отдельного типа у него тоже нет, но такой код скомпилируется:
```c++
int n[] = {1, 2, 3};

for (auto num : n) {
    std::cout << num << std::endl;
}
```
В стандартной библиотеке есть функции для обобщенного получения итераторов на начало и конец коллекции - `std::begin` и `std::end`.
Они имеют специализации - если у коллекции есть методы `begin` и `end`, то вызываются они, если коллекция - массив, то возвращаются указатели на начало и (за) конец. У указателей и итераторов один и тот же интерфейс, по сути указатель подходит под требования `random access iterator`.

Не покаждой коллекции можно проитерироваться дважды. Пример - итератор, считывающий значения из консоли.


    

## Алгоритмы стандартной библиотеки

Почти у всех стандартных алгоритмов интерфейс начинается одинаково - сначала идет пара итераторов на начало - конец последовательности.

```c++
std::vector<int> numbers = {3, 2, 1};

std::sort(numbers.begin(), numbers.end());

for (auto num : numbers) {
    std::cout << num << ' '; // 1 2 3
}
```
    

### Почему в стандартной библиотеке последовательности описываются в виде пар итераторов, почему не написали функции, принимающие сами коллекции?

Тогда не получится, например, отсортировать часть последовательности.

```c++
std::vector<int> numbers = {3, 2, 1};

std::sort(numbers.begin(), numbers.begin() + 2); // сотрирует первые 2 элемента, так как вторая граница не включается

for (auto num : numbers) {
    std::cout << num << ' '; // 2 3 1
}
```
Такой подход позволяет работать с любыми подпоследовательностями одинаково. 

`std::sort` принимает только `random access iterator`.

    

### std::all_of

```c++
std::string str = "Hello world!";

std::cout << std::all_of(str.begin(), str.end(), [](char c) { return std::isalpha(c); }); // false
```

третий параметр `std::all_of` - предикат - функция, отображающая множество элементов в `{0, 1}`.

```c++
std::string str = "Helloworld";

std::cout << std::all_of(str.begin(), str.end(), [](char c) { return std::isalpha(c); }); // true
```

Предикат ожидает элемент того типа, который имеет элемент коллекции. Поэтому нельзя написать так:

```c++
std::string str = "Helloworld";

std::cout << std::all_of(str.begin(), str.end(), std::isalpha); // ошибка компиляции
```

`std::isalpha` принимает тип `int`, а `std::all_of` ожидает, что там будет функция (или объект, похожий на функцию), принимающая элемент от `std::string`, то есть `char`.

При выводе типов шаблонных параметров не может происходить неявное преобразование типов, поэтому явно приходится написать функцию, которая принимает `char` и возвращает `bool`.

Значит, можно написать так:


```c++
bool IsAlpha(char ch) {
    return std::isalpha(ch);
}

int main() {

    std::string str = "Helloworld";

    std::cout << std::all_of(str.begin(), str.end(), IsAlpha); // true
    
    return 0;
}
```

Также в стандартной библиотеке есть функции `std::any_of` и `std::none_of`.    
Первый возвращает `true`, если хотя бы один элемент последовательности удовлетворяет предикату, второй - если ни один не удовлетворяет.

    

### std::for_each

Выполняет функцию, которая передается третьим параметром, для каждого элемента коллекции.

Если итераторы, которые мы используем при вызове этой функции, не являются константными, то по ним можно изменять значения, на которые они указывают.

```c++
std::string str = "Hello world!";

std::for_each(str.begin(), str.end(), [](char& ch) { ch = std::toupper(ch); });

std::cout << str; // HELLO WORLD!
```

Если оператор константный, то его разыменование возвращает `const char&`, а значит поменять по нему значение не получится:

```c++
const std::string str = "Hello world!";

std::for_each(str.begin(), str.end(), [](char &ch) {
    ch = std::toupper(ch);
}); // ошибка компиляции, так как const char& не приводится к char&
```

```c++
const std::string str = "Hello world!";

std::for_each(str.begin(), str.end(), [](const char &ch) {
    ch = std::toupper(ch);
}); // ошибка компиляции, так как нельзя присвоить другое значение константной ссылке
```

Аналогично можно явно вызвать методы, возвращающие константные итераторы:

```c++
std::string str = "Hello world!";

std::for_each(str.cbegin(), str.cend(), [](char& ch) { ch = std::toupper(ch); }); // ошибка компиляции
```

В функции `std::all_of` можно также изменять значения по неконстантным итераторам:

```c++
std::string str = "Hello world!";

std::all_of(str.begin(), str.end(), [](char &ch) {
    ch = std::toupper(ch);
    return std::isalpha(ch);
});

std::cout << str; // HELLO world!
```

Однако стандартом не гарантируется, что функция `std::all_of` останавливается, если элемент не удовлетворяет предикату, поэтому на это не стоит полагаться.



### std::copy

Первые 2 аргумента стандартные, третий - итератор на то место коллекции, куда хочется скопировать элементы.

```c++
std::string str1 = "Hello world!";
std::string str2;

str2.resize(str1.size()); // необходимо, так как потом итератор уйдет за границы памяти,
                         // принадлежащей строке и получим UB

std::copy(str1.begin(), str1.end(), str2.begin());

std::cout << str2; // Hello world!
```

`resize` делать неудобно, так как в общем случае интервал, в который надо будет копировать, нельзя получить за `O(1)`.

```c++
std::string str1 = "Hello world!";
std::string str2;

std::copy(str1.begin(), str1.end(), std::back_inserter(str2));
// возвращает специальный итератор, который в момент присваивания значения внутри итератора
// делает insert (или push_back) в конец коллекции.

std::cout << str2; // Hello world!
```
Вектор из `std::unique_ptr` нельзя копировать, так как `std::copy` пытается скопировать каждый элемент, а у `std::unique_ptr` нет конструктора копирования.

Но можно сделать так:

```c++
std::vector<std::unique_ptr<int>> ptrs1;
std::vector<std::unique_ptr<int>> ptrs2;

std::move(ptrs1.begin(), ptrs1.end(), std::back_inserter(ptrs2));
```
Но тут важно понимать, что работать с элементами `ptrs1` больше нельзя, так как их состояние было передано в `ptrs2`.

Существует похожая на `std::copy` функция - `std::copy_if`, принимающая четвертым параметром предикат - если элемент удовлетворяет ему, то он копируется, нет иначе.

Также существует аналогичная функция `std::move_if`.
    
    
#### Также существует функция std::copy_backward

```c++
std::string str1 = "Hello world!";
std::string str2 = "aaaaabbbbbccccc";

std::copy_backward(str1.begin(), str1.end(), str2.end());

std::cout << str2; // aaaHello world!
```

Она копирует элементы `str1` в конец `str2`, не меняя размер `str2`. Чтобы это работало, итератор должен быть `bidirectional iterator`, чтобы вычислить длину последовательности, которую нужно вставить, сделать отступ назад и дальше скопировать последовательность.

Если размер `str2` меньше `str1`, то получим `UB`.



### std::transform

Первые  2 параметра стандартные, третий параметр - итератор, куда записывать результат, четвертый - функция, которая описывает преобразование последовательности.    
То есть для каждого элемента последовательности применяем функцию и получаем новую последовательность.

```c++
std::string str1 = "Hello world!";
std::string str2;

std::transform(str1.begin(), str1.end(), std::back_inserter(str2), [](char ch) { return std::toupper(ch); });

std::cout << str2; // HELLO WORLD!
```



### std::remove_if

Удаляет элементы последовательности, которые удовлетворяют предикату.    
На самом деле ничего не удаляет, а просто сдвигает элементы последовательности таким образом, что сначала идут те, что не удовлетворяют предикату, а потом повторяется конец строки. То есть функция не делает `resize` коллекции в конце, вместо этого возвращает итератор на конец получившейся последовательности:

```c++
std::string str = "Hello@ world!";

auto end_it = std::remove_if(str.begin(), str.end(), [](char ch) { return !std::isalpha(ch); });
// str.erase(end_it, str.end()); // - удалит 3 ненужные символа, если раскомментировать

std::cout << str2; // "Helloworldld!"
```

Есть аналогичная функция `std::remove`, но она принимает третьим параметром элемент, который надо удалить:

```c++
std::string str = "Hello@ world!";

auto end_it = std::remove(str.begin(), str.end(), ' ');
str.erase(end_it, str.end());

std::cout << str; // Hello@world!
```



### std::unique

2 стандартных параметра на вход, удаляет подряд идущие элементы последовательности. Работает аналогично `std::remove`, то есть на самом деле ничего не удаляет и возвращает итератор.    
Часто используется в связке с `std::sort`, чтобы удалить вообще все неуникальные элементы последовательности:

```c++
std::string str = "Hello@ world!";

std::sort(str.begin(), str.end());
auto end_it = std::unique(str.begin(), str.end());
str.erase(end_it, str.end());

std::cout << str; //  !@Hdelorw
```
    
____
    
        
С другими функциями можно ознакомиться [тут](https://en.cppreference.com/w/cpp/algorithm).
