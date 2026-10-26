# Лабораторная работа №5  
## Стек, очередь, дек, приоритетная очередь


**Вариант 4**  
**Дисциплина:** Алгоритмы и системы данных  
**Выполнила:** Тришина О. Д., группа ПИЖ-б-о-25-1(2)  
**Преподаватель:** ассистент Белик А. И.

---

## Цель работы
Изучить принципы работы базовых структур данных: стек (LIFO), очередь (FIFO), дек (двусторонняя очередь) и приоритетная очередь. Реализовать указанные структуры данных с нуля без использования STL-контейнеров. Решить прикладные задачи с применением этих структур (поиск минимума за O(1) и слияние K списков). Закрепить навыки управления динамической памятью и обработки исключительных ситуаций.

## Задание (Вариант 4)
- Стек: Найти минимальный элемент в стеке за O(1).
- Очередь: Найти минимальный элемент в очереди за O(1).
- Дек: Найти минимальный элемент в деке за O(1).
- Приоритетная очередь: Объединить K отсортированных списков.

---

## Теоретическая часть

| Концепция | Описание |
|---|---|
| Стек (Stack) | Структура данных, работающая по принципу LIFO (Last In, First Out). Добавление и извлечение происходят с одного конца (вершины). |
| Очередь (Queue) | Структура данных, работающая по принципу FIFO (First In, First Out). Элементы добавляются в конец, а извлекаются из начала. |
| Дек (Deque) | Двусторонняя очередь, позволяющая добавлять и удалять элементы как в начало, так и в конец. |
| Приоритетная очередь (Priority Queue) | Структура, в которой элементы извлекаются в порядке убывания/возрастания приоритета. Реализуется на основе двоичной кучи (Binary Heap). |
| Поиск минимума за O(1) | Достигается путём хранения дополнительной информации (например, второго стека с текущими минимумами или использования двух стеков для очереди/дека). |

---

## Полный код решения

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>
#include <locale.h>

using namespace std;

// 1. СТЕК С MIN ЗА O(1)
template <typename T>
class MinStack {
    T* val, * mn; int top, cap;
    void resize() {
        T* nv = new T[cap * 2], * nm = new T[cap * 2];
        for (int i = 0; i <= top; i++) { nv[i] = val[i]; nm[i] = mn[i]; }
        delete[] val; delete[] mn; val = nv; mn = nm; cap *= 2;
    }
public:
    MinStack(int c = 10) : top(-1), cap(c) { val = new T[cap]; mn = new T[cap]; }
    ~MinStack() { delete[] val; delete[] mn; }
    void push(T v) {
        if (top >= cap - 1) resize();
        val[++top] = v;
        mn[top] = (top == 0) ? v : (v < mn[top - 1] ? v : mn[top - 1]);
    }
    void pop() { if (top == -1) throw underflow_error("Empty"); top--; }
    T getTop() const { if (top == -1) throw underflow_error("Empty"); return val[top]; }
    T getMin() const { if (top == -1) throw underflow_error("Empty"); return mn[top]; }
    bool isEmpty() const { return top == -1; }
};

// 2. ОЧЕРЕДЬ С MIN ЗА O(1) (На двух стеках)
template <typename T>
class MinQueue {
    MinStack<T> in, out;
    void transfer() {
        if (out.isEmpty()) while (!in.isEmpty()) { out.push(in.getTop()); in.pop(); }
    }
public:
    void enqueue(T v) { in.push(v); }
    void dequeue() { if (in.isEmpty() && out.isEmpty()) throw underflow_error("Empty"); transfer(); out.pop(); }
    T getMin() const {
        if (in.isEmpty() && out.isEmpty()) throw underflow_error("Empty");
        if (in.isEmpty()) return out.getMin();
        if (out.isEmpty()) return in.getMin();
        return in.getMin() < out.getMin() ? in.getMin() : out.getMin();
    }
};

// 3. ДЕК С MIN ЗА O(1) (На двух стеках)
template <typename T>
class MinDeque {
    MinStack<T> front, back;
public:
    void push_front(T v) { front.push(v); }
    void push_back(T v) { back.push(v); }
    void pop_front() {
        if (front.isEmpty() && back.isEmpty()) throw underflow_error("Empty");
        if (front.isEmpty()) while (!back.isEmpty()) { front.push(back.getTop()); back.pop(); }
        front.pop();
    }
    void pop_back() {
        if (front.isEmpty() && back.isEmpty()) throw underflow_error("Empty");
        if (back.isEmpty()) while (!front.isEmpty()) { back.push(front.getTop()); front.pop(); }
        back.pop();
    }
    T getMin() const {
        if (front.isEmpty() && back.isEmpty()) throw underflow_error("Empty");
        if (front.isEmpty()) return back.getMin();
        if (back.isEmpty()) return front.getMin();
        return front.getMin() < back.getMin() ? front.getMin() : back.getMin();
    }
};

// 4. ПРИОРИТЕТНАЯ ОЧЕРЕДЬ (КУЧА) ДЛЯ СЛИЯНИЯ K СПИСКОВ
struct Node { int val, listIdx, elemIdx; };
class MinHeap {
    Node* h; int sz, cap;
    void heapifyUp(int i) {
        while (i > 0 && h[i].val < h[(i - 1) / 2].val) { swap(h[i], h[(i - 1) / 2]); i = (i - 1) / 2; }
    }
    void heapifyDown(int i) {
        while (true) {
            int l = 2 * i + 1, r = 2 * i + 2, s = i;
            if (l < sz && h[l].val < h[s].val) s = l;
            if (r < sz && h[r].val < h[s].val) s = r;
            if (s != i) { swap(h[i], h[s]); i = s; }
            else break;
        }
    }
public:
    MinHeap(int c = 100) : sz(0), cap(c) { h = new Node[cap]; }
    ~MinHeap() { delete[] h; }
    void push(Node n) { if (sz >= cap) return; h[sz] = n; heapifyUp(sz++); }
    Node pop() { Node minN = h[0]; h[0] = h[--sz]; heapifyDown(0); return minN; }
    bool empty() const { return sz == 0; }
};

vector<int> mergeKLists(vector<vector<int>>& lists) {
    MinHeap pq; vector<int> res;
    for (int i = 0; i < lists.size(); i++)
        if (!lists[i].empty()) pq.push({ lists[i][0], i, 0 });

    while (!pq.empty()) {
        Node n = pq.pop(); res.push_back(n.val);
        if (n.elemIdx + 1 < lists[n.listIdx].size())
            pq.push({ lists[n.listIdx][n.elemIdx + 1], n.listIdx, n.elemIdx + 1 });
    }
    return res;
}

// ==================== ГЛАВНАЯ ФУНКЦИЯ ====================
int main() {
    setlocale(LC_ALL, "ru");

    cout << "1. Стек (Min O(1)): ";
    MinStack<int> s; s.push(5); s.push(2); s.push(8);
    cout << s.getMin() << " (после pop: "; s.pop(); cout << s.getMin() << ")" << endl;

    cout << "2. Очередь (Min O(1)): ";
    MinQueue<int> q; q.enqueue(10); q.enqueue(4); q.enqueue(7);
    cout << q.getMin() << " (после dequeue: "; q.dequeue(); cout << q.getMin() << ")" << endl;

    cout << "3. Дек (Min O(1)): ";
    MinDeque<int> d; d.push_back(15); d.push_front(3); d.push_back(8);
    cout << d.getMin() << " (после pop_front: "; d.pop_front(); cout << d.getMin() << ")" << endl;

    cout << "4. Слияние списков: ";
    vector<vector<int>> lists = { {1, 4, 5}, {1, 3, 4}, {2, 6} };
    vector<int> merged = mergeKLists(lists);
    for (int v : merged) cout << v << " ";
    cout << endl;

    return 0;
}
```

## Вывод программы

```
1. Стек (Min O(1)): 2 (после pop: 2)
2. Очередь (Min O(1)): 4 (после dequeue: 4)
3. Дек (Min O(1)): 3 (после pop_front: 8)
4. Слияние списков: 1 1 2 3 4 4 5 6
```

---

## Результаты и выводы
В ходе выполнения работы были изучены и реализованы четыре фундаментальные структуры данных. Мы научились использовать дополнительные структуры (например, второй стек) для оптимизации поиска минимума до O(1) в стеке, очереди и деке. Также была реализована приоритетная очередь на основе кучи и решена классическая алгоритмическая задача — слияние K отсортированных списков за O(N log K).

---

## Преимущества и недостатки
Использование двух стеков для очереди и дека позволяет достичь амортизированной сложности O(1) для основных операций и O(1) для поиска минимума, но требует дополнительной памяти под сами стеки.

Приоритетная очередь на основе двоичной кучи обеспечивает вставку и извлечение за O(log K), что оптимально для задачи слияния K списков. Основной недостаток — необходимость ручного управления памятью и реализации просеивания (heapify).

---

**Ссылка на репозиторий:**  
[https://github.com/galyatvasilevna/studies](https://github.com/galyatvasilevna/studies)

---

**Контрольные вопросы**

1.В чём заключается принцип работы стека (LIFO)?
Последний пришёл — первый ушёл. Добавление и удаление происходят только с вершины.

2.В чём заключается принцип работы очереди (FIFO)?
Первый пришёл — первый ушёл. Добавление в конец, извлечение из начала.

3.Чем дек отличается от обычной очереди?
В деке можно добавлять и удалять элементы с обоих концов (и в начало, и в конец).

4.Что такое приоритетная очередь? Где она применяется?
Структура, где извлекается элемент с наивысшим приоритетом. Применяется в планировщиках задач, алгоритмах Дейкстры, слиянии списков.

5.Как реализовать стек на основе динамического массива?
Нужен массив data, счётчик top (индекс вершины). При переполнении массив расширяется (удваивается).

6.Как реализовать очередь на основе кольцевого буфера?
Массив фиксированного размера, индексы head и tail. При добавлении/удалении индексы смещаются по кругу с помощью операции % capacity.

7.Почему приоритетную очередь обычно реализуют на куче?
Куча обеспечивает вставку и извлечение максимума/минимума за O(log n), что быстрее, чем O(n) при использовании обычного массива.

8.Какова сложность операций push и pop в стеке?
O(1) (амортизированное для push при расширении).

9.Какова сложность операций enqueue и dequeue в очереди?
O(1).

10.Какова сложность операций в приоритетной очереди? Почему?
O(log n), так как операции связаны с просеиванием элемента по дереву кучи (высота дерева log n).

11.Как проверить правильность расстановки скобок с помощью стека?
Идём по строке: если открывающая — кладём в стек; если закрывающая — проверяем, чтобы вершина стека была парной открывающей, и удаляем её. В конце стек должен быть пуст.

12.Как реализовать очередь на двух стеках? Какова сложность?
Первый стек для входящих, второй для исходящих. При dequeue, если второй пуст, перекладываем туда всё из первого. Амортизированная сложность — O(1).

13.Как реализовать дек на основе двусвязного списка?
Каждый узел хранит указатели prev и next. Операции вставки/удаления с обоих концов выполняются за O(1) путём изменения указателей.

14.Как реализовать дек на основе кольцевого буфера?
Массив, индексы front и back. Добавление сзади: back = (back + 1) % cap. Добавление спереди: front = (front - 1 + cap) % cap.

15.Как реализовать стек с поддержкой получения минимума за О(1)?
Использовать второй стек (или массив), который хранит текущий минимум для каждого уровня основного стека (как в реализованном MinStack).
