# Лекция 9. Наследование и виртуальный полиморфизм (15.02.2022)

# Наследование

Пусть у нас есть код:

```cpp
#include <iostream>

class Student {
public:
    Student(const std::string &name, int group) : name_(name), group_(group) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Group: " + std::to_string(group_);
    }

private:
    std::string name_;
    int group_;
};

class Teacher {
public:
    Teacher(const std::string &name, const std::string &discipline) : name_(name), discipline_(discipline) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

private:
    std::string name_;
    std::string discipline_;
};

void PrintInfo(const Student &student) {
    std::cout << student.GetInfo() << std::endl;
}

int main() {
    Student student("Vasily", 123);
    Teacher teacher("Dmitriy", "OiMP");

    PrintInfo(student);
    return 0;
}
```

Однако у него есть проблема.

Мы не можем передать в функцию объект `teacher` в функцию.`PrintInfo` вот так: `PrintInfo(teacher)`.

Единственный выход  — сделать копию функции `PrintInfo` с аргументом `const Teacher &teacher`:

```cpp
#include <iostream>

class Student {
public:
    Student(const std::string &name, int group) : name_(name), group_(group) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Group: " + std::to_string(group_);
    }

private:
    std::string name_;
    int group_;
};

class Teacher {
public:
    Teacher(const std::string &name, const std::string &discipline) : name_(name), discipline_(discipline) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

private:
    std::string name_;
    std::string discipline_;
};

void PrintInfo(const Student &student) {
    std::cout << student.GetInfo() << std::endl;
}

void PrintInfo(const Teacher &teacher) { // Копия предыдущей функции
    std::cout << teacher.GetInfo() << std::endl;
}

int main() {
    Student student("Vasily", 123);
    Teacher teacher("Dmitriy", "OiMP");

    PrintInfo(student);
    PrintInfo(teacher);
    return 0;
}
```

Но такой способ не очень хороший.

Нас может спасти наследование.

Наследование — механизм, который позволяет связать два класса друг с другом так, что класс наследник получает **все** свойства класса родителя при этом может их доопределять, расширять или изменять.

Посмотрим, как это выглядит на практике:

```cpp
#include <iostream>

class Person { // Базовый класс
public:
    Person(const std::string &name) : name_(name) {
    }

    std::string GetInfo() const {
        return "Name: " + name_;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Group: " + std::to_string(group_);
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Student student("Vasily", 123);
    Teacher teacher("Dmitriy", "OiMP");

    std::cout << student.GetInfo() << std::endl; // Name: Vasily Group: 123
    std::cout << teacher.GetInfo() << std::endl; // Name: Dmitriy Discipline: OiMP
    return 0;
}
```

Но теперь у нас есть другая проблема.

Если мы вызовем метод `PrintInfo` от `student` и `teacher`, то будет вызван метод из базового класса `Person`.

```cpp
PrintInfo(student); // Name: Vasily
PrintInfo(teacher); // Name: Dmitriy
```

## Виртуальные методы

Чтобы исправить эту проблему, надо сделать метод `PrintInfo` виртуальным. Это можно реализовать с помощью ключевого слова `virtual`. Тогда во всех наследниках класса метод с ключевым словом `virtual` будут виртуальным, даже если мы не укажем ключевое слово `virtual` в методе класса наследника.

Однако принято либо всё равно писать `virtual` в наследниках, либо использовать ключевое слово `override` (второй способ более правильный).

Ключевое слово `override` защищает от того, что можно поменять сигнатуру метода в базовом классе, а в классе наследнике ничего не менять, тогда получится, что это два разных метода (а это нехорошо, потому что тогда будет вызываться только метод из базового класса).

```cpp
#include <iostream>

class Person {
public:
    Person(const std::string &name) : name_(name) {

    }

    virtual std::string GetInfo() const { // Виртуальный метод, который перегружен в наследниках
        return "Name: " + name_;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override { // Виртуальный метод
        return "Name: " + name_ + " Group: " + std::to_string(group_);
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override { // Виртуальный метод
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Student student("Vasily", 123);
    Teacher teacher("Dmitriy", "OiMP");

    std::cout << student.GetInfo() << std::endl; // Name: Vasily Group: 123
    std::cout << teacher.GetInfo() << std::endl; // Name: Dmitriy Discipline: OiMP

    PrintInfo(student); // Name: Vasily Group: 123
    PrintInfo(teacher); // Name: Dmitriy Discipline: OiMP
    return 0;
}
```

Такой код будет работать, если мы работаем с объектом базового типа по ссылки или указателю.

- По ссылке:
    
    ```cpp
    const Person &person = student;
    std::cout << person.GetInfo() << std::endl; // Name: Vasily Group: 123 — вызвался метод из Student
    ```
    
- По указателю:
    
    ```cpp
    const Person *person = &student;
    std::cout << person->GetInfo() << std::endl; // Name: Vasily Group: 123 — вызвался метод из Student
    ```
    
- Но если мы будем работать с объектом по значению, то вызовется метод из базового класса:
    
    ```cpp
    const Person person = student;
    std::cout << person.GetInfo() << std::endl; // Name: Vasily — вызвался метод из Person
    ```
    

Полиморфизм — механизм, который позволяет придать коду контекстно-зависимую семантику. Например, если в функцию `PrintInfo` передавать объекты разного типа, то у нас будет разное поведение. Это и называется полиморфизмом.

Виртуальный полиморфизм — механизм, поведение которого зависит от того, с объектом какого типа мы работаем через ссылку или указатель на базовый класс.

Виртуальный полиморфизм работает, если у нас есть ссылка или указатель на базовый класс. А если мы работам с ним по значению, то виртуальный полиморфизм не работает.

Важно понимать, что полиморфизм работает не на этапе компиляции, а в момент исполнения программы (то есть в runtime).

Как это реализовано? В базовом классе неявно добавляется `protected` поле, которое называется `__vfptr` — это указатель на таблицу виртуальных функций.

Таблица виртуальных функций — это таблица, в которой перечислены все виртуальные методы и написан адреса, где лежит код каждого из этих методов.

Для каждого класса создаётся своя таблица виртуальных функций, если мы создаём объект этого класса. В объекте класса в специальном указателе лежит адрес, по которому в runtime мы можем определить тип объекта по тому, на какую таблицу виртуальных функций он указывает.

Вызов виртуального метода работает так:

1. Достаём из указателя на таблицу виртуальных функций адрес на эту таблицу.
2. Идём в эту таблицу, находим там соответствующий метод, который мы хотим вызывать.
3. Достаём адрес, по которому лежит код этого метода.
4. Делаем переход по этому адресу и продолжаем выполнение по этому адресу.

Можно сказать, что вызов виртуального метода в общем случае работает медленнее, чем вызов обычного метода.

## Вызов метода базового класса из класса наследника

Если написать так, то вызовется виртуальный метод `GetInfo` из класса `Student`:

```cpp
class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return this->GetInfo() + " Group: " + std::to_string(group_); // Тут запись return GetInfo() аналогична записи return this->GetInfo()
    }

private:
    int group_;
};
```

Если бы у класса `Student` были какие-то ещё наследники, то вызвался бы `GetInfo` из самого последнего наследника, который определяет этот метод.

Если написать так, то вызов `GetInfo` уже не будет виртуальным, а будет вызван метод `GetInfo` из базового класса `Person`:

```cpp
class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

private:
    int group_;
};
```

## Вызов виртуальных методов из конструктора и деструктора

Сначала вызываются конструкторы от базовых частей, а только потом от наследников.

Это означает, что в момент, когда мы находимся в конструкторе, наши объекты ещё сконструированы не полностью.

Например, если мы находимся в конструкторе класса `Person` и вызываем метод `GetInfo`, то у нас ещё не будет готова часть, которая относится к классам наследникам $\Longrightarrow$ мы никак не сможем вызвать `GetInfo`, который определён в классе наследнике. Таким образом, если мы находимся в конструкторе, то иерархия наследования, которую видит компилятор, ограничена тем классом, который мы сейчас конструируем.

💡 Рекомендация: лучше не вызывать виртуальные методы из конструктора, потому что это часто приводит не к тому поведению, которое хотелось бы получить.


Если надо позвать виртуальный метод из конструктора, то можно сделать отдельный метод, в котором будет инициализация объекта и вызов виртуальных методов.

Для деструкторов аналогично: иерархия наследования, которая видна в данный момент, обрезается на том классе, в деструкторе которого мы сейчас находимся.

## Важное про деструкторы

Напишем код с деструкторами в каждом из классов.

```cpp
#include <iostream>

class Person {
public:
    Person(const std::string &name) : name_(name) {

    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Person *person = new Student("Petr", 1000);
    delete person; // Вызовется только деструктор Person
    return 0;
}
```

Однако в нём есть проблема: при удалении `person` вызовется **только** деструктор `Person`, но не его наследников.

Чтобы это исправить, нужно сделать деструктор виртуальным:

```cpp
#include <iostream>

class Person {
public:
    Person(const std::string &name) : name_(name) {

    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    virtual ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Person *person = new Student("Petr", 1000);
    delete person; // Будут вызваны деструкторы: сначала ~Student, потом ~Person
    return 0;
}
```

Так как у деструкторов одинаковая сигнатура, то можно не писать `virtual` или `override` в классах наследниках.

💡 Рекомендация: если в классе есть виртуальный метод, то нужно делать виртуальный конструктор, а если от класса будет кто-то наследоваться, то нужно делать виртуальный деструктор.


## Секции `public`, `protected` и `private` при наследовании

Рассмотрим на примере. При наследовании эти ключевые слова говорят, кто будет знать, что `Student` является наследником `Person`.

- `public` — доступно везде.
- `private` — доступно только в `Student`.
    
    Например, в `Studet` можно написать строчку: `const Person &person = *this;`
    
    А если написать то же самое в `main`, то такой код не сработает.
    
- `protected` — доступно в `Student` и в его наследниках.

`protected` наследование применяется очень редко.

Обычно применяется `public` или `private` наследование.

### Модификаторы доступа по умолчанию

- У классов по умолчанию `private`.
- У структур по умолчанию `public`.

## Абстрактный класс. Методы без реализации в классе

Рассмотрим класс `Person`.

Если мы хотим объявить метод `Print` в классе `Person`, но не хотим его реализовывать в этом классе, то можно написать такой код в классе `Person`:

```cpp
virtual void Print() const = 0;
```

Метод `Print` метод будет чисто виртуальным. Это означает, этот метод не будет иметь реализации в классе `Person`, поэтому его обязательно нужно реализовать в каком-то из классов наследников.

Абстрактный класс — класс, в котором есть хотя бы один чисто виртуальный метод.

То есть теперь `Person` — абстрактный класс. Заметим, что наследники `Person` — это тоже виртуальные методы, потому что все поля и методы родителей есть в и в наследниках.

Это означает, что `Person` — это в некотором смысле неполноценный класс, потому что один из его методов в этом классе не реализован, из-за этого мы не можем создать объект такого типа.

Чтобы улучшить ситуацию, надо в каком-то из наследников реализовать метод `Print`. Сделаем это в классе наследнике `Teacher`, чтобы потом в `main` мы имели возможность создать объект класса `Teacher` (то есть теперь `Teacher` больше не будет являться абстрактным классом).

```cpp
#include <iostream>

class Person {
public:
    Person(const std::string &name) : name_(name) {

    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    virtual void Print() const = 0;

    virtual ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    void Print() const override {
        std::cout << GetInfo() << std::endl;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Teacher teacher("Vasa", "C++");
    return 0;
}
```

## Интерфейс

Интерфейс — это класс, котором все методы чисто виртуальные.

Сделаем интерфейс:

```cpp
class Printable {
    virtual void Print() const = 0;
};
```

Уберём метод `Print` из `Person` и отнаследуем класс `Person` от только что созданного `Printable`. Заметим, что в самом `Person` больше нет чисто виртуальных методов, но так как он наследуется от класса `Printable`, в котором есть чисто виртуальный метод, то получается, что этот чисто виртуальный метод есть и в классе `Person`, значит, `Person` — абстрактный класс. Создать объект типа `Person` всё равно нельзя, пока мы не определим этот метод. Поэтому не забудем добавить реализацию метода `Print` в `Person`, чтобы можно было создать объект класса `Person` и объекты его ****наследников. Кстати, если мы напишем реализацию `Print` только в каком-то из наследников класса `Person`, то мы сможем создать только объект типа этого наследника, а не объект типа `Person`, что достаточно логично.

```cpp
#include <iostream>

class Printable {
    virtual void Print() const = 0;
};

class Person : public Printable {
public:
    Person(const std::string &name) : name_(name) {

    }

    void Print() const override {
        std::cout << GetInfo() << std::endl;
    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    virtual ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Person &person) {
    std::cout << person.GetInfo() << std::endl;
}

int main() {
    Person person("Petya");
    Teacher teacher("Vasa", "C++");
    return 0;
}
```

## Транзитивность наследования

Что это значит? Это значит, что наследники `Person` будут наследниками `Printable` тоже. Проверим это:

```cpp
#include <iostream>

class Printable {
public: // Изменим с дефолтного private на public
    virtual void Print() const = 0;
};

class Person : public Printable {
public:
    Person(const std::string &name) : name_(name) {

    }

    void Print() const override {
        std::cout << GetInfo() << std::endl;
    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    virtual ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Printable &printable) { // Передаём константную ссылку на Printable
    printable.Print(); // Вызываем метод Print() класса Printable. Name: Vasa Discipline: C++
}

int main() {
    PrintInfo(Teacher("Vasa", "C++"));
    return 0;
}
```

Обратим внимание на то, для доступа к методу `Print` важен только модификатор доступа в классе `Printable`, а все остальные модификаторы доступа для метода `Print` значения не имеют.

---

## Ещё один пример кода с наследованием и чисто виртуальными методами

```cpp
#include <iostream>

class Executor {
public:
    void Run() {
        std::cout << "Start" << std::endl;
        Do();
        std::cout << "End" << std::endl;
    }

    virtual ~Executor() {}

    virtual void Do() = 0;
};

class MyExecutor : public Executor {
    void Do() override {
        std::cout << "DO WORK" << std::endl;
    }
};

int main() {
    Executor *executor = new MyExecutor();
    executor->Run();
    return 0;
}
```

Мы вызвали не виртуальный метод `Run()` у `Executor`.

Метод `Run()` в себе содержит вызов виртуального приватного метода `Do()`.

Рассмотрим метод `Run()` подробнее:

1. Выполняем вывод на экран строчки *“Start”*.
2. Выполняем виртуальный вызов метода `Do()` . Так как объект у нас типа `MyExecutor`, то в момент вызова мы обращаемся к таблице виртуальных функций для класса `MyExecutor`, там понимаем, что нужно пойти в метод `Do()` в классе `MyExecutor`, вызываем его, он выводит на экран *"DO WORK"*. Далее возвращаемся в точку, откуда уходили, то есть точку вызова метода `Do()` в методе `Run()`.
3. Выполняем вывод на экран строчки *“End”*.

## Принцип хорошего наследования

Принцип подстановки Лисков: любой код, который работает с типом базового класса, должен корректно работать, если подставить вместо базового класса объект какого-то из его наследников.

Этот принцип был назван в честь [Барбары Лисков](https://ru.wikipedia.org/wiki/%D0%9B%D0%B8%D1%81%D0%BA%D0%BE%D0%B2,_%D0%91%D0%B0%D1%80%D0%B1%D0%B0%D1%80%D0%B0).

## Паттерн фабрика

```cpp
#include <iostream>

class Printable {
public:
    virtual void Print() const = 0;
};

class Person : public Printable {
public:
    Person(const std::string &name) : name_(name) {

    }

    void Print() const override {
        std::cout << GetInfo() << std::endl;
    }

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    virtual ~Person() {
        std::cout << "~Person" << std::endl;
    }

protected:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string &name, int group) : Person(name), group_(group) {
    }

    std::string GetInfo() const override {
        return Person::GetInfo() + " Group: " + std::to_string(group_);
    }

    ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_;
};

class Teacher : public Person {
public:
    Teacher(const std::string &name, const std::string &discipline) : Person(name), discipline_(discipline) {
    }

    std::string GetInfo() const override {
        return "Name: " + name_ + " Discipline: " + discipline_;
    }

    ~Teacher() {
        std::cout << "~Teacher" << std::endl;
    }

protected:
    std::string discipline_;
};

void PrintInfo(const Printable &printable) {
    printable.Print();
}

// Паттерн фабрика:
class PersonFactory {
public:
    static Person *CreatePerson(const std::string &type) {
        if (type == "student") {
            return new Student("", 0);
        } else {
            return new Teacher("", "");
        }
    }
};

int main() {
    Person *person = PersonFactory::CreatePerson("student");
    return 0;
}
```

Теперь все зависимости сосредоточены в `PersonFactory`. В тех местах, где нам нужно создавать объект типа `Person`, мы можем вызывать метод `CreatePerson` класса `PersonFactory`.

## Паттерн абстрактная фабрика

```cpp
#include <iostream>

class Tank {

};

class Ship {

};

// Паттерн абстрактная фабрика:
class UnitFactory {
public:
    virtual Tank *CreateTank() = 0;

    virtual Ship *CreateShip() = 0;
};

class ProtosUnitFactory : public UnitFactory {
    Tank *CreateTank() override {
        std::cout << "CreateTank";
        return nullptr;
    }

    Ship *CreateShip() override {
        std::cout << "CreateShip";
        return nullptr;
    }

};

int main() {
    
    return 0;
}
```

Этот паттерн позволяет создавать группы каких-то классов наследников разных базовых классов, связанных друг с другом.