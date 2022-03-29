# Классы
## Модификаторы доступа
```c++

#include <iostream>
#include <compare>
#incluse <cmath>

public;
protected;
private;


class Vector {
public:
	size_t Size(){
		return values_.size();
	}
  public:
	private:
	std::vector<double> values_;

	friend void PrintSize(const Vector& vector) 
};

void PrintSize(const Vector& vector){
	std::cout << vector.values_size() << std::endl;
}

int main(){
	Vector v;
	v.values.push_back(10);
	std::cout << v.values.size() << std::endl;
	
}

```
- Классы от структуры отличается тем, что имеют модификаторы доступа

- Если мы в классе не пишем private, члены класса все равно будут private, в структуре по умолчанию члены паблик

- Приватные члены доступны только изнутри нашего класса

- protected делает члены класса доступными в текущем классе и его наследниках

- В структуре по умолчанию члены public


```c++

#include <iostream>
#include <compare>
#incluse <cmath>

public;
private;
protected;


class Vector {

private:
	 
	 std::vector<double> values_;

	friend void PrintSize(const Vector& vector) //говорит о том, что является другом нашего класса vector и имеет доступ ко всем его приватным и публичным членам, friend используется в крайнем случае
};

void PrintSize(const Vector& vector){
	std::cout << vector.values_size() << std::endl;
}

int main(){
	Vector v;
	PrintSize(v);
	
}

```

- Friend - это плохой механизм, который используется в случай крайней необходимости; скорее всего нужно перепроектировать публичные члены класса

## Методы

Как изнутри метода обратиться к полям текущего объекта

- Если мы хотим обратиться ни к самому объекту, а к-то его полю, то мы может просто написать имя этого поля.

- Метод - это функция член класса

```c++

public;
private;
protected;


class Vector {
public:
	size_t Size(){
		return values.size();
	}
private:
	std::vector<double> values_;

void PrintSize(Vector& vector){
	std::cout << vector.Size() << std::endl;
}

int main(){
	Vector v;
	PrintSize(v);
	
}

```
- Если мы хотим обратиться к самому объекту, то у нас есть специльное ключевое слово this

```c++
	class Vector {
public:
	size_t Size(){
		int values_ = 0;
		return this->values_.size();
private:
	std::vector<double> values_;
	}

```

Зачем нужен this? Мы можем создать переменную с таким же именем, как наше поле и тогда переменная перекроет его


### Константные методы

```c++

#include <iostream>
#include <compare>
#incluse <cmath>




class Vector {
public:
	size_t Size() const{
		return values.size();
	}
	size_t Size() {
		return values.size();
	}
private:
	std::vector<double> values_;

void PrintSize(const Vector& vector){
	std::cout << vector.Size() << std::endl;
}

int main(){
	Vector v;
	std::cout << v.Size() << std::endl;
	PrintSize(v);
	
}

```

- Если вызываем метод от константоного объекта, тогда вызовется константный метод
- Константность метода позволяет на уровне интерфейса говорить какие методы модифицируют наш объект, а какие нет

Иногда нам нужно изнутри константных методов состояние менять ( по дефолту поля константных методов тоже константные)

```c++

class Vector {
public:
	
	size_t Size() const{
		++calls; //тут будет ошибка без mitable
		return values.size();
	}
	size_t Size() {
		++calls;
		return values.size();
	}
private:
	std::vector<double> values_;
	mutable size_t calls_ = 0; //счетчик вызова методов
}

int main(){
	Vector v;
	std::cout << v.Size() << std::endl;
	PrintSize(v);
	
}
```

- mutable позволяет менять поля в константных методах

Какой тип имеет this (зависит от контекста)

```c++
	size_t Size() {
	// this -> Vector* const здесь указатель константный
	// this = nullptr; так сделать не можем
		++calls;
		return values.size();
	}

// в константном методе мы еще не можем менять наш текущий объект
	size_t Size() const{
		// this -> const Vector* const здесь указатель константный
	// this = nullptr; так сделать не можем
		++calls; //тут будет ошибка без mitable
		return values.size();
	}

```
### Static метод

```c++

#include <iostream>
#include <compare>
#incluse <cmath>

public;
private;
protected;


class Vector {
public:
	size_t Size(){
		return values.size();
	}
private:
	std::vector<double> values_;

static void PrintSize(Vector& vector){
	std::cout << vector.Size() << std::endl;
}

int main(){
	Vector v;
	Vector::PrintSize(v);  
	
}

```
- Static метод это по сути не метод, а функция, которая находится внутри класса, такая функция имеет доступ к приватным полям класса, но внутри нее не доступен this


## Как создавать наш класс

Специальный вид векторов - конструкторы

- Перед вызовом конструктора вызовутся конструкторы от всех полей нашего класса

```c++
class Vector {
public:

## конструктор класса 
		Vector(){
			std::cout << "Create Vector" << std:endl;
		}  
}  

int main() {
	Vector v; - //создаем объект вектор, в этот момент выызвается его конструктор
}
``````

```c++
 
#include <iostream>
#include <compare>
#incluse <cmath>

public;
private;
protected;


class Vector {
public:
	Vector(size _t size){
		values_.resize(size)
		size_ = size;
		
	}
	void SetValues(const std::vector<double>& values){
		values_ = values;
	
	}

	size_t Size() const {
		return this->size;
	}
	
	size_t Size()  {
		return this->size;
	}

private:
	std::vector<double> values_;
	size_t size_ = 0


int main(){
	Vector v;
	Vector::PrintSize(v);  
	
}

```
```c++
Vector(size _t size): values_(size), size_(size){

}

```

- Для инициализации полей есть следующий синтаксис

```c++ 
class Vector {
public:
```
## Инициализиурем поле ссылочного типа

Единственный способ проинициализировать поле ссылочного типа

```c++
		Vector(size_t& size): values_(size), size_(size){
		}  
}  

```
```c++
class Vector {
public:

//Когда пишем конструктор с одним параметром, пишем explicit, если не нужно неявное приведение типов
		expicit Vector(size_t& size): values_(size), size_(size){
		}  
}  

```
- По умолчанию делаем все методы константными
```c++
size_t Size() const{
		
}

```
## Делегирующий конструктор

```c++
class Vector{
	public:
			Vector(): values_(0), size_(0)
		 explicit Vector(size_t size): values_s(size), size_(size){
		}
		//конструкторы выше одинаковые, поэтому делаем так
		
		 explicit Vector(size_t size): values_s(size), size_(size){
		 }
		 Vector(): Vector(0);
		
```

```c++
class Vector {
public:

//Когда пишем конструктор с одним параметром, пишем explicit, если не нужно неявное приведение типов
		expicit Vector(size_t& size): values_(size), size_(size){
		}  

			Vector() = default; //если мы хотим объявить конструктор с пустым телом
			Vector() = delete //удалим конструктор
}  

```

## Деструктор


```c++
class Vector {
public:

//Когда пишем конструктор с одним параметром, пишем explicit, если не нужно неявное приведение типов
		expicit Vector(size_t& size): values_(size), size_(size){
		}  
//синтаксис деструктора
		 ~Vector(){

		 }
}  

```

## Конструктор копирования

Если мы хотим создать два вектора и проинициализировать один конструктор другим, надо создать конструктор копирования

```c++

int i = 0;
Vector v1(1);
Vector v2 = v1;

```

```c++
class Vector {
public:

//Когда пишем конструктор с одним параметром, пишем explicit, если не нужно неявное приведение типов
		expicit Vector(size_t& size): values_(size), size_(size){
			std::cout << "Vector" << std::endl;
		}  
// конструктор копирования
		 Vector(const Vector& other){
				values_ = other.values_;
				size_ = other.size_;
		 }

int main(){
		int i = 0;
		Vector v1(1); //вызов обычного конструктора
		Vector v2 = v1; //чтобы такой синтаксис работал, нужен конструктор копирования
	}
}  

```
## Оператор присваивания
```c++

v2 = v1 // чтобы такое работало нужно написанть оператор:

Vector& operator=(const Vector& other){
	if (this == &other){
					return *this;
	}

  values_ = other.values_;
	size_ = other.size_;


	return *this;
}

```