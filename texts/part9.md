## Многопоточность

### Простое создание потока

Рассмотрим простой пример программы, создающей дополнительный поток:

```c++
#include <iostream>
#include <thread>
using namespace std;

void fun()
{
   cout<<"Hello"<<endl;
}

int main()
{
   thread t(fun);
   cout<<"Main"<<endl;
   t.join();
   return 0;
}
```

Используется класс `thread`, конструктору которого при создании экземпляра передается функция, исполняющаяся в теле потока. Выполнение функции начинается сразу, после создания потока.

Вызов `join()` позволяет остановить главный поток до завершения работы созданного потока.

### Гонка за ресурс

В следующем примере создается 100 потоков, каждый из которых стремится вывести строку на экран.

```c++
#include <iostream>
#include <vector>
#include <thread>
using namespace std;

void func(int x) {
    cout << "Inside thread " << x << endl;
}

int main() 
{
    vector<thread*> ts;
    for(int i=0;i<100;i++)
      ts.push_back(new thread(&func, i));

    for(auto &th: ts)
       th->join();
    cout << "Outside thread" << endl;
    return 0;
}
```

Результаты программы ужасны, поскольку каждый поток борется за право использовать консоль. Необходима синхронизация.

![](img/threads1.png)

### Синхронизация

Самый простой способ синхронизации - использование **мьютексов**:

```c++

mutex mu;

void func(int x) {
    mu.lock();
    cout << "Inside thread " << x << endl;
    mu.unlock();
}
```

Другим, более надежным способом является использование объектов **atomic**, особенно если речь идет об объектах, разделяемых несколькими потоками.

В следующем примере задается аккумулятор, в который потоки записывают квадраты чисел:

```c++
#include <atomic>

atomic<int> accum(0);

void square(int x) {
    accum += x * x;
}
```



### Временные задержки в потоках

```c++
#include <chrono>
this_thread::sleep_for(chrono::milliseconds(10));
```

### Реализация модели "потребитель-поставщик"


Класс `condition_variable` является примитивом синхронизации, который может быть использован для блокирования потока или нескольких потоков одновременно, пока не произойдет любое из событий:

* будет получено извещение из другого потока
* выйдет тайм-аут
* произойдет ложное пробуждение

Любой поток, который намерен ждать на `std::condition_variable` должен сначала приобрести `std::unique_lock`. Операция ожидания атомарно освобождает мьютекс и приостанавливает выполнение потока. Когда переменная условия уведомляется, поток пробуждается, и мьютекс снова приобретается.



```c++
#include <thread>
#include <iostream>
#include <queue>

std::mutex mx;
std::condition_variable cv;
std::queue<int> q;

bool finished = false;

void producer(int n) {
    for(int i=0; i<n; ++i) {
        {
            std::lock_guard<std::mutex> lk(mx);
            q.push(i);
            std::cout << "pushing " << i << std::endl;
        }
        cv.notify_all();
    }
    {
        std::lock_guard<std::mutex> lk(mx);
        finished = true;
    }
    cv.notify_all();
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lk(mx);
        cv.wait(lk, [](){ return finished || !q.empty(); });
        while (!q.empty()) {
            std::cout << "consuming " << q.front() << std::endl;
            q.pop();
        }
        if (finished) break;
    }
}

int main() {
    std::thread *t1=new std::thread(producer, 10);
    std::thread *t2=new std::thread(consumer);
    t1->join();
    t2->join();
    std::cout << "finished!" << std::endl;
}
```