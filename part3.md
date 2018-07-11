## Шаблоны функций

Шаблон функций позволяет задачть описание функции без привязки к конретным типам. В место них применяют типы-параметры.

```c++
template<class T>
T square(T param)
{
   return param*param;
}
```

Вызов шаблонной функции подразумевает автоматическое определение типа параметра:

```c++
int x=2;
cout<<square(x);
```

В простейших случаях вывод нужного типа не вызывает проблем. Однако, могут встречаться и более сложные случае, особенно при использовании ссылочных типов.   


### Случай 1.  lvalue-ссылка

Ссылочная часть игнорируется

```c++
template<class T>
void printSquare(T& param)
{
   cout<<param*param;
}

int a=10;
const int b=a;
constr int& c=a;
 
f(a); // T - int,       param - int&
f(b); // T - const int, param - const int& 
f(c); // T - const int, param - const int&
```

При добавление **const** в параметр:

```c++
template<class T>
void printSquare(const T& param)
{
   cout<<param*param;
}

int a=10;
const int b=a;
constr int& c=a;
 
f(a); // T - int, param - const int&
f(b); // T - int, param - const int& 
f(c); // T - int, param - const int&
```

### Случай 2.  rvalue-ссылка

В этом случае надо смотреть, к какому типу выражений (lvalue или rvalue) принадлежит фактический параметр.

```c++
template<class T>
void printSquare(T&& param)
{
   cout<<param*param;
}

int a=10;
const int b=a;
constr int& c=a;
 
f(a); // T - int&,       param - int&       (a - lvalue)
f(b); // T - const int&, param - const int& (b - lvalue)
f(c); // T - const int&, param - const int& (c - lvalue)
f(10);// T - int,        param - int&&      (27 - rvalue)
```


### Пример шаблона

Наиболее известный пример: функция обмена значений двух переменных

```c++
template<class T>
void swap(T& a, T& b)
{
   T tmp;
   tmp=a;
   a=b;
   b=tmp;
}
```

Для получения функции проведем *инстанцирование*:

```c++void fun() {  int a=1, b=2;  double c=1.1, d=2.2; 
  swap<int>(a,b);  // все в порядке
  swap(c,d);       // тоже все хорошо
  swap(a,d);       // ошибка, разные типы a и d!}
```

### Передача ссылки на массив

Отдельный интерес вызывает случай, когда в функцию по ссылке передается массив.

Рассмотрим случай, когда ссылки нет:

```c++
#include <iostream>
using namespace std;

template<class T>
void fun(T arg)
{
    cout<<typeid(arg).name()<<endl;
}
int main()
{
    int arr[]{1,2,3};
    fun(arr);
    return 0;
}
```

В качестве ответа, выводимого на экран мы увидим **Pi**, то есть указатель на int.  Так происходит потому что при передаче имени массива без ссылочного параметра в функции мы получим указатель на первый элемент.

Рассмотрим случай с ссылкой:

```c++
#include <iostream>
using namespace std;

template<class T>
void fun(T & arg)
{
    cout<<typeid(arg).name()<<endl;
}

int main()
{
    int arr[]{1,2,3};
    fun(arr);
    return 0;
}
```

Теперь ответ другой: **Ai3**, то есть массив из трех целых. 

### Пример шаблона с целочисленным параметром

```c++
template< int BufferSize > // целочисленный параметр char* read(){   char *buffer = new char[ BufferSize ]; 
   return buffer;}
...
char *ReadString = read< 20 >; 
delete [] ReadString; 
ReadString = read< 30 >;
...
```

### Параметры шаблона

В шаблонах допускается использование различных видов параметров

```c++template<class T1, // параметр-тип typename T2, // параметр-тип int I,   // параметр обычного типа T1 DefaultValue, // параметр обычного типа template< class > class T3,// параметр-шаблон class Character = char // параметр по умолчанию >
```

### Перегрузка шаблонов

```c++
template<class T> T sqrt(T);template<class T> complex<T> sqrt(complex<T>); 
double sqrt(double);
void fun(complex<double> z) {  sqrt(2);  sqrt(2.0);  sqrt(z);}
```

### Специализация шаблонов

По своему назначению, шаблоны подходят к использованию со множеством типов. Но иногда нужно определить версию шаблона для конкретного типа. Это называется **специализация**:

```c++
template<class T> bool less(T a, T b) 
{   return a < b; 
}template<> bool less<const char*>(const char* a, const char* b){   return strcmp(a, b) < 0;}
```

Для числовых типов можно использовать операцию "меньше", а для строк С - вызов библиотечной функции.