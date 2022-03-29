# Лекция 10. Наследование, виртуальный полиморфизм, паттерны проектирования  — 15.02.2022

## Что такое наследование и зачем оно нужно

Наследование помогает объединять классы с одинаковыми свойствами. Класс наследник получает все свойства класса родителя, может их переопределять, расширять и так далее.

```cpp

class Person {
public:
    Person(const std::string& name) : name_(name) {}

    std::string GetInfo() const {
        return "Name: " + name_;
    }
public:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string& name, int group) : Person(name), group_(group) {}

    std::sting GetInfo() const {
        return "Name: " + name_ + ", Group: " + std::to_string(group_);
    }

private:
    int group_ = 0;
};

class Teacher : public Person {
public:
    Teacher(const std::string& const std::sring& dicipline) : Person(name), dicipline_(dicipline) {}

    std::sting GetInfo() const {
        return "Name: " + name_ + ", Dicipline: " + dicipline_;
    }

private:
    std::string dicipline_ = 0;
}

```

Если бы мы писали каждый класс по отдельности, нам бы приходилось каждый раз переписывать метод GetInfo, и классы были бы не связаны между собой. Теперь классы имеют общую логику, а также общее поле `name`.

**Важно!** Поле `name` было отмечено как `protected`: его видно в текущем классе и всех его наследниках.

```cpp
Student student("Vasily", 123);
Teacher teacher("Dmitry", "OIMP");

std::cout << student.GetInfo(); // Name: Vasily, Group: 123

std::cout << teacher.GetInfo(); // Name: Dmitry, Discipline: OIMP
```

## Виртуальные методы

Но есть одна проблема: функция такого вида будет работать не совсем так, как мы наверное ожидаем

```cpp
void PrintInfo(const Person& person) {
    std::cout << person.GetInfo << std::endl;
}
```
```cpp
PrintInfo(student); // Name: Vasily
```

Чтобы функция вызывалась от реального типа, а не от базового класса, его нужно сделать **виртуальным**. Такой метод будет виртуальным и в других классах. В наследниках принято писать `override`, чтобы потом избежать багов.

```cpp
class Person {
public:
    Person(const std::string& name) : name_(name) {}

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }
public:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string& name, int group) : Person(name), group_(group) {}

    std::sting GetInfo() const override {
        return "Name: " + name_ + ", Group: " + std::to_string(group_);
    }

private:
    int group_ = 0;
};

class Teacher : public Person {
public:
    Teacher(const std::string& const std::sring& dicipline) : Person(name), dicipline_(dicipline) {}

    std::sting GetInfo() const override {
        return "Name: " + name_ + ", Dicipline: " + dicipline_;
    }

private:
    std::string dicipline_ = 0;
}
```
```cpp
PrintInfo(student); // Name: Vasily, Group: 123
```

**Важно!** Это работает только если мы передаем в функцию `PrintInfo` объект по ссылке или указателю. Если передать по значению, то вызовется метод базового класса.

Механизм, когда в зависимости от того, какой объект передается в качестве реального типа называется **виртуальным полиморфизмом**.

Полиморфизм - механизм, который позволяет придать коду контекстно зависимую ситуацию.

Виртуальный полиморфизм - то же, только касаемо наследования. Опять же, работает **только если есть ссылка или указатель на базовый тип**.

### Как в runtime понимать, какой объект был вызван?

Реализация может варьироваться, но как правило это работает так: в базовый класс добавляется указательна таблицу виртуальных функций

```cpp
class Person {
public:
    Person(const std::string& name) : name_(name) {}

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }
public:
    __vf_ptr // неявный указатель та таблицу
    std::string name_;
};
```

Вызов виртуальной функции происходит в три этапа:

1) Достаем из указателя на таблицу виртуальных функций адрес таблицы
2) Переходим в таблицу, находим там соответствующий метод. Достаем адрес, где лежит метод
3) Переходим по адресу, выполняем

Можно сделать вывод, что вызов виртуальных методов медленнее, чем обычных

### Как звать методы из базового класса?

```cpp
class Person {
public:
    Person(const std::string& name) : name_(name) {}

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }
public:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string& name, int group) : Person(name), group_(group) {}

    std::sting GetInfo() const override {
        return Person::GetInfo() + ", Group: " + std::to_string(group_);
    }

private:
    int group_ = 0;
};
```

Также из базового класса можно вызывать методы наследника

## Конструкторы и деструкторы при наследовании

**Сначала конструируется базовый класс, только потом наследник. С деструкторами наоборот: сначала деструктор наследника, потом базового класса**

Поэтому виртуальные методы в конструкторах не вызываем.

Также есть правило:

Если от класса кто-то наследуется, либо уже есть виртуальный метод: делаем виртуальный деструктор. Иначе не будут удалены поля в классе наследнике. Пример:

```cpp
class Person {
public:
    Person(const std::string& name) : name_(name) {}

    virtual std::string GetInfo() const {
        return "Name: " + name_;
    }

    ~Person() {
        std::cout << "~Person" << std::endl;
    }

public:
    std::string name_;
};

class Student : public Person {
public:
    Student(const std::string& name, int group) : Person(name), group_(group) {}

    std::sting GetInfo() const override {
        return Person::GetInfo() + ", Group: " + std::to_string(group_);
    }

     ~Student() {
        std::cout << "~Student" << std::endl;
    }

private:
    int group_ = 0;
};

int main() {
    Person* person = new Student("Petr", 1000);
    delete person;
} 

// вывод после завершения программы:
// ~Person
```

 А это не то что мы хотим. Если написать так, то все будет в порядке и утечки памти не произойдет:

 ```cpp
virtual ~Person() {
     std::cout << "~Person" << std::endl;
}
```

## Public, private и proected наследование

Все как в полях в классе:

1) public - знают все
2) private - внутри все знают, а снаружи нет
3) protected - очень редкое, его никто не использует

У классов по умолчанию private, у структур public

## Pure virtual методы

Такие методы не имеют реализации в базовом классе и требуют реализации в наследовании. Классы, где есть хотя бы один pure virtual метод называются **абстрактными**.

Эти методы выглядят так:

```cpp
virtual void Print() const = 0; // сигнатура произвольная, главное virtual ... = 0;
```

**Создать объект абстрактного класса нельзя. Если в наследнике такой метод не определили, то наследник тоже будет абстрактным классом**

При этом ссылки такого класса делать можно. Но реальный тип будет другой.

## Интерфейсы

Интерфейс - класс, где все методы чисто виртуальные

Замечание, которое не понятно, где сказать: модификатор доступа к методу определяется в классе, с которым мы сейчас работаем (пусть и абстрактным). Если в каком-то абстрактном классе Person метод Print был публичным, и мы работаем с объектом типа Person, который на самом деле Student, то даже если в Student метод Print был переопределен как приватный, мы все равно сможем им воспользоваться.

## Принцип подстановки Лисков

**Любой код который корректно работает с базовым классом должен корректно работать с классом наследника**

Попробуем отнаследовать квадрат от прямоугольника

```cpp
class Rectancle {
public:
    int GetWidth() const {
        return width_;
    }

    void SetWidth(int width) {
        width_ = width;
    }

    int GetHeight() const {
        return height_;
    }

    void SetHeight(int heigh) {
        height_ = height;
    }
}


class Square : public Rectangle {
public:
    void SetWidth(int width) {
        width_ = width;
        height_ = width;
    }

    void SetHeight(int width) {
        width_ = width;
        height_ = width;
    }
}
```

Но если в следующую функцию передать квадрат, а не прямоугольник, то мы получим на выходе 20 * 20 = 400, а не 200, так как при вызове `SetHeight` у квадрата также меняется и `width`

```cpp
bool TestRectangle(Rectangle& rectangle) {
    rectangle.SetWidth(10);
    rectangle.SetHeight(20);
    int area = rectangle.GetWidth() * rectangle.GetHeight();
    return area == 200; // по формуле площади прямоугольника
}
```

С точки зрения наследования, такой класс квадрата не наследник прямоугольника. А значит принцип подстановки Лисков не выполняется.

## Шаблоны проектирования

Стандартные решения стандартной проблемы

### Логгирование
```cpp
class Executor {
public:
    void Run() {
        std::cout << "Start" << std::endl;
        Do()
        std::cout << "End" << std::endl;
    }
    virtual ~Executor() {};

private:
    virtual void Do() = 0;
};

class MyExecutor {
    void Do() override {
        std::cout << "DO WORK" << std::endl;
    }
};

int main() {
    Executor* executor = new MyExecutor();
    executor->Run();
    delete executor;
} 

// вывод:
// Start
// DO WORK
// End
```

Таким образом, разработчику не прилось оформлять логику вывода. Надо оформлять только логику Do.

### Factory

Позволяет организовать логику создания объектов

```cpp
class PersonFactory {
    static Person* CreatePerson(const std::string& type) {
        if (type == "student") {
            return new Student("", 0);
        } else {
            return new Teacher("", "");
        }
    }
}

int main() {
    Person* person = PersonFactory::CreatePerson("teacher");
}
```
