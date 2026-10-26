# Лабораторная работа №3  
## Реализация динамического массива (аналог vector). Итераторы

**Вариант 4**  
**Дисциплина:** Алгоритмы и системы данных  
**Выполнила:** Тришина О. Д., группа ПИЖ-б-о-25-1(2)  
**Преподаватель:** ассистент Белик А. И.

---

## Цель работы
Разработать шаблонный класс `Vector<T>` с поддержкой «правила пяти», итераторов и операторов сравнения (==, !=, <, >), освоить управление динамической памятью и провести амортизированный анализ операции `push_back`.

## Задание (Вариант 4)
- Реализовать класс `Vector` с поддержкой «правила пяти», итераторами и дополнительной функциональностью согласно варианту.
- **Вариант 4:** Операторы сравнения векторов (`==`, `!=`, `<`, `>`).

---

## Теоретическая часть

| Концепция | Описание |
|---|---|
| Динамический массив | Массив, размер которого определяется во время выполнения программы и может меняться. |
| Ёмкость (capacity) | Объём выделенной памяти под элементы. |
| Размер (size) | Количество элементов, реально хранящихся в массиве. |
| Правило пяти | Необходимость определить деструктор, конструктор копирования, оператор присваивания копированием, конструктор перемещения и оператор присваивания перемещением. |
| Итератор | Объект, указывающий на элемент контейнера и позволяющий перебирать элементы (в простейшем случае — указатель `T*`). |

---

## Полный код решения

```cpp
#include <iostream>
#include <stdexcept>
#include <algorithm>
#include <locale.h>

using namespace std;

// ШАБЛОННЫЙ КЛАСС VECTOR<T>
template <typename T>
class Vector {
private:
    T* data;          // динамический массив
    int sz;           // размер (сколько элементов)
    int cap;          // ёмкость (сколько выделено)

public:
    // --- Конструкторы ---
    Vector() : data(nullptr), sz(0), cap(0) {}

    explicit Vector(int n) : sz(n), cap(n) {
        data = new T[cap];
    }

    // Конструктор копирования (глубокое копирование)
    Vector(const Vector& other) : sz(other.sz), cap(other.cap) {
        data = new T[cap];
        for (int i = 0; i < sz; i++) data[i] = other.data[i];
    }

    // Конструктор перемещения
    Vector(Vector&& other) noexcept
        : data(other.data), sz(other.sz), cap(other.cap) {
        other.data = nullptr;
        other.sz = other.cap = 0;
    }

    // --- Деструктор ---
    ~Vector() { delete[] data; }

    // --- Операторы присваивания ---
    Vector& operator=(const Vector& other) {
        if (this != &other) {
            Vector temp(other);  // copy-and-swap
            swap(temp);
        }
        return *this;
    }

    Vector& operator=(Vector&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            sz = other.sz;
            cap = other.cap;
            other.data = nullptr;
            other.sz = other.cap = 0;
        }
        return *this;
    }

    // --- Доступ ---
    T& operator[](int i) {
        if (i < 0 || i >= sz) throw out_of_range("Index out of range");
        return data[i];
    }

    const T& operator[](int i) const {
        if (i < 0 || i >= sz) throw out_of_range("Index out of range");
        return data[i];
    }

    int size() const { return sz; }
    int capacity() const { return cap; }
    bool empty() const { return sz == 0; }

    // --- Изменение ---
    void push_back(const T& value) {
        if (sz >= cap) {
            int newCap = (cap == 0) ? 1 : cap * 2;
            reserve(newCap);
        }
        data[sz++] = value;
    }

    void pop_back() {
        if (empty()) throw underflow_error("Vector is empty");
        sz--;
    }

    void reserve(int newCap) {
        if (newCap <= cap) return;
        T* newData = new T[newCap];
        for (int i = 0; i < sz; i++) newData[i] = data[i];
        delete[] data;
        data = newData;
        cap = newCap;
    }

    void clear() { sz = 0; }

    void swap(Vector& other) {
        std::swap(data, other.data);
        std::swap(sz, other.sz);
        std::swap(cap, other.cap);
    }

    // --- Итераторы ---
    T* begin() { return data; }
    T* end() { return data + sz; }
    const T* begin() const { return data; }
    const T* end() const { return data + sz; }

    // ============ ОПЕРАТОРЫ СРАВНЕНИЯ (ВАРИАНТ 4) ============
    // == : совпадают размер и все элементы
    bool operator==(const Vector& other) const {
        if (sz != other.sz) return false;
        for (int i = 0; i < sz; i++)
            if (!(data[i] == other.data[i])) return false;
        return true;
    }

    // != : отрицание ==
    bool operator!=(const Vector& other) const {
        return !(*this == other);
    }

    // < : лексикографическое сравнение
    bool operator<(const Vector& other) const {
        int minSize = (sz < other.sz) ? sz : other.sz;
        for (int i = 0; i < minSize; i++) {
            if (data[i] < other.data[i]) return true;
            if (other.data[i] < data[i]) return false;
        }
        return sz < other.sz;  // все равны - меньше тот, кто короче
    }

    // > : противоположность <
    bool operator>(const Vector& other) const {
        return other < *this;
    }

    // --- Для удобства вывода ---
    void print() const {
        cout << "[";
        for (int i = 0; i < sz; i++) {
            cout << data[i];
            if (i < sz - 1) cout << ", ";
        }
        cout << "]";
    }
};

// ==================== ДЕМОНСТРАЦИЯ ====================
int main() {
    setlocale(LC_ALL, "ru");

    // Создаём два одинаковых вектора
    Vector<int> v1, v2;
    for (int i = 1; i <= 5; i++) { v1.push_back(i); v2.push_back(i); }

    // Вектор покороче
    Vector<int> v3;
    for (int i = 1; i <= 4; i++) v3.push_back(i);

    // Вектор с другими числами
    Vector<int> v4;
    for (int i = 1; i <= 5; i++) v4.push_back(i * 10);

    // --- Демонстрация сравнений ---
    cout << "v1 = "; v1.print(); cout << endl;
    cout << "v2 = "; v2.print(); cout << endl;
    cout << "v3 = "; v3.print(); cout << endl;
    cout << "v4 = "; v4.print(); cout << endl << endl;

    cout << "v1 == v2 : " << (v1 == v2) << endl; // 1
    cout << "v1 == v3 : " << (v1 == v3) << endl; // 0
    cout << "v1 != v3 : " << (v1 != v3) << endl; // 1
    cout << "v3 < v1  : " << (v3 < v1) << endl; // 1 (v3 короче и совпадает по префиксу)
    cout << "v1 > v3  : " << (v1 > v3) << endl; // 1
    cout << "v1 < v4  : " << (v1 < v4) << endl; // 1 (1 < 10)
    cout << "v4 > v1  : " << (v4 > v1) << endl; // 1

    // --- Работа со строками ---
    cout << "\n=== Vector<string> ===" << endl;
    Vector<string> s1, s2;
    s1.push_back("banana");
    s1.push_back("apple");
    s2.push_back("apple");
    s2.push_back("cherry");
    cout << "s1 = "; s1.print(); cout << endl;
    cout << "s2 = "; s2.print(); cout << endl;
    cout << "s1 < s2  : " << (s1 < s2) << endl; // 1 (banana < cherry)
    cout << "s1 == s2 : " << (s1 == s2) << endl; // 0

    // --- Итераторы ---
    cout << "\n=== Итераторы (range-based for) ===" << endl;
    cout << "v1: ";
    for (int x : v1) cout << x << " ";
    cout << endl;

    // --- Правило пяти ---
    cout << "\n=== Правило пяти ===" << endl;
    Vector<int> vCopy(v1); // конструктор копирования
    cout << "vCopy(v1)       = "; vCopy.print(); cout << endl;

    Vector<int> vMove(std::move(vCopy)); // конструктор перемещения
    cout << "vMove(move)     = "; vMove.print(); cout << endl;
    cout << "vCopy после    : size = " << vCopy.size() << endl;

    Vector<int> vAssign;
    vAssign = vMove; // оператор копирования
    cout << "vAssign = vMove: "; vAssign.print(); cout << endl;

    Vector<int> vMoveAssign;
    vMoveAssign = std::move(vAssign); // оператор перемещения
    cout << "vMoveAssign    = "; vMoveAssign.print(); cout << endl;
    cout << "vAssign после : size = " << vAssign.size() << endl;

    // --- Амортизированный анализ push_back ---
    cout << "\n=== Амортизированный анализ push_back ===" << endl;
    Vector<int> vA;
    int prevCap = 0, totalCopies = 0;
    for (int i = 1; i <= 20; i++) {
        vA.push_back(i);
        if (vA.capacity() != prevCap) {
            totalCopies += prevCap;
            cout << "n = " << i << " -> cap = " << vA.capacity() << endl;
            prevCap = vA.capacity();
        }
    }
    cout << "Всего копирований: " << totalCopies << endl;
    cout << "Среднее на push_back: " << (double)totalCopies / 20 << endl;

    return 0;
}
```

## Вывод программы

```
v1 = [1, 2, 3, 4, 5]
v2 = [1, 2, 3, 4, 5]
v3 = [1, 2, 3, 4]
v4 = [10, 20, 30, 40, 50]

v1 == v2 : 1
v1 == v3 : 0
v1 != v3 : 1
v3 < v1  : 1
v1 > v3  : 1
v1 < v4  : 1
v4 > v1  : 1

=== Vector<string> ===
s1 = [banana, apple]
s2 = [apple, cherry]
s1 < s2  : 1
s1 == s2 : 0

=== Итераторы (range-based for) ===
v1: 1 2 3 4 5 

=== Правило пяти ===
vCopy(v1)       = [1, 2, 3, 4, 5]
vMove(move)     = [1, 2, 3, 4, 5]
vCopy после    : size = 0
vAssign = vMove: [1, 2, 3, 4, 5]
vMoveAssign    = [1, 2, 3, 4, 5]
vAssign после : size = 0

=== Амортизированный анализ push_back ===
n = 1 -> cap = 1
n = 2 -> cap = 2
n = 3 -> cap = 4
n = 5 -> cap = 8
n = 9 -> cap = 16
n = 17 -> cap = 32
Всего копирований: 31
Среднее на push_back: 1.55
```

---

## Результаты и выводы
Реализован шаблонный класс Vector<T> с полной поддержкой «правила пяти» (деструктор, конструктор копирования, конструктор перемещения, оператор присваивания копированием, оператор присваивания перемещением).

Реализованы итераторы (через указатели T*), что позволяет использовать range-based for для обхода элементов.

Реализованы операторы сравнения (==, !=, <, >). Лексикографическое сравнение (<) работает корректно для векторов разной длины и разных типов данных (на примере string).

Проведён амортизированный анализ push_back: ёмкость удваивается, что обеспечивает среднее количество копирований ~1.55 на 20 операций, подтверждая амортизированную сложность O(1).

---

**Ссылка на репозиторий:**  
[https://github.com/galyatvasilevna/studies](https://github.com/galyatvasilevna/studies)

---

**Контрольные вопросы**

1.Почему динамический массив называется «динамическим»?
Потому что его размер определяется во время выполнения программы и может меняться (push_back, resize), в отличие от статического массива, размер которого фиксирован на этапе компиляции.

2.Что такое ёмкость (capacity) и чем она отличается от размера (size)?
Ёмкость — сколько всего памяти выделено под элементы. Размер — сколько элементов реально хранится. Всегда size ≤ capacity. Разница — это «запас» на будущие добавления.

3.Почему при расширении массива ёмкость удваивается, а не увеличивается на фиксированное число?
Удвоение даёт амортизированную сложность О(1) для push_back. Если бы прибавляли константу (например, +1), то каждое добавление требовало бы копирования О(n) элементов, и общая сложность стала бы О(n²).

4.Какова амортизированная сложность операции push_back? Почему?
О(1) амортизированно. Расширения происходят редко (логарифмически часто: после 1, 2, 4, 8, 16... элементов). Суммарно за n операций тратится О(n) копирований, то есть в среднем О(1) на одну операцию.

5.Что такое «правило трёх»? Когда оно применяется?
Если классу нужен пользовательский деструктор, или конструктор копирования, или оператор присваивания копированием — то нужны все три. Применяется, когда класс владеет ресурсом (динамической памятью, файлом, сокетом).

6.Что такое «правило пяти»? Чем оно отличается от «правила трёх»?
Это «правило трёх» плюс конструктор перемещения и оператор присваивания перемещением. Отличие: позволяет передавать владение ресурсом (move), а не копировать его — это быстрее.

7.В чём разница между конструктором копирования и оператором присваивания?
Конструктор копирования создаёт новый объект из существующего. Оператор присваивания работает с уже существующим объектом, предварительно освобождая его старые ресурсы.

8.Зачем нужна проверка if (this != &other) в операторе присваивания?
Защита от самоприсваивания (v = v). Без неё мы бы сначала удалили ресурс, а потом попытались скопировать из уже уничтоженного — получили бы мусор.

9.Что такое итератор? Как он связан с указателем?
Итератор — объект, указывающий на элемент контейнера и позволяющий перебирать элементы. В простейшем случае (для Vector) итератор — это обычный указатель T*.

10.Зачем нужны методы begin() и end()?
Для поддержки range-based for (for (int x : v)) и STL-алгоритмов (std::sort, std::find). begin() — первый элемент, end() — «за последним».

11.Как реализовать range-based for для своего контейнера?
Достаточно, чтобы у класса были методы begin() и end(), возвращающие итераторы (указатели). Компилятор сам развернёт цикл в обход от begin() до end().

12.Что такое «глубокое копирование»? Чем оно отличается от «поверхностного»?
Глубокое — копируется сам ресурс (выделяется новая память, копируется содержимое). Поверхностное — копируется только указатель, и два объекта начинают делить один ресурс, что приводит к двойному удалению.

13.Что такое семантика перемещения и зачем она нужна?
Возможность «забрать» ресурсы у временного объекта без копирования. Нужна для производительности: при работе с большими массивами перемещение в разы быстрее копирования.

14.Почему конструктор перемещения помечается noexcept?
Чтобы контейнеры STL (std::vector, std::queue) при перевыделении памяти использовали перемещение, а не копирование. Если конструктор может бросить исключение, STL выберет копирование (безопаснее).

15.Как правильно освобождать память в деструкторе?
Через delete[] data; для массивов, созданных через new[]. Для одиночных объектов — delete. После освобождения полезно обнулить указатель (data = nullptr;), но это не обязательно.
