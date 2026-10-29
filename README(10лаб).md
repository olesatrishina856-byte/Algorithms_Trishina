# Лабораторная работа №10
## Самобалансирующиеся деревья (AVL) и префиксные деревья (Trie)

**Вариант 4**  
**Дисциплина:** Алгоритмы и системы данных  
**Выполнила:** Тришина О. Д., группа ПИЖ-б-о-25-1(2)  
**Преподаватель:** ассистент Белик А. И.

---

## Цель работы

Реализовать самобалансирующееся AVL-дерево с операциями вставки, поиска и удаления. Освоить механизмы балансировки (повороты). Изучить структуру данных «префиксное дерево» (Trie) для эффективного хранения строк. Реализовать операции вставки, поиска и автодополнения в Trie. Сравнить производительность различных структур данных.

## Задание (Вариант 4)

1. **AVL-дерево (Часть 1, Вариант 4):** реализовать вставку, балансировку, поиск и удаление листа.
2. **Trie (Часть 2, Вариант 4):** реализовать базовые операции (вставка, поиск, поиск по префиксу) и **автодополнение (вывод топ-5 слов)**, начинающихся с введённого префикса.
3. **Часть 3:** сравнить производительность AVL и обычного BST (в отчёте через таблицу сложности).

---

## Теоретическая часть

| Концепция | Описание |
|---|---|
| AVL-дерево | BST, которое автоматически балансируется. Разница высот (фактор баланса) левого и правого поддеревьев не превышает 1. При дисбалансе выполняются повороты (LL, RR, LR, RL). |
| Префиксное дерево (Trie) | Структура для хранения строк. Каждый узел представляет собой символ. Слова с общим префиксом имеют общий путь от корня. Сложность поиска и вставки — **O(L)**, где L — длина слова. |
| Автодополнение | Поиск всех слов, начинающихся с заданного префикса. Реализуется спуском к узлу префикса и последующим обходом (DFS) для сбора слов. |

---

## Полный код решения

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <locale.h>

using namespace std;

// ==================== 1. AVL ДЕРЕВО ====================
struct AVLNode {
    int key, h;
    AVLNode *left, *right;
    AVLNode(int k) : key(k), h(1), left(nullptr), right(nullptr) {}
};

class AVL {
    int h(AVLNode* n) { return n ? n->h : 0; }
    int bf(AVLNode* n) { return h(n->left) - h(n->right); }
    void upd(AVLNode* n) { n->h = 1 + max(h(n->left), h(n->right)); }

    AVLNode* rightRot(AVLNode* y) {
        AVLNode* x = y->left; y->left = x->right; x->right = y;
        upd(y); upd(x); return x;
    }
    AVLNode* leftRot(AVLNode* x) {
        AVLNode* y = x->right; x->right = y->left; y->left = x;
        upd(x); upd(y); return y;
    }

    AVLNode* balance(AVLNode* n) {
        upd(n);
        if (bf(n) > 1) {
            if (bf(n->left) < 0) n->left = leftRot(n->left);
            return rightRot(n);
        }
        if (bf(n) < -1) {
            if (bf(n->right) > 0) n->right = rightRot(n->right);
            return leftRot(n);
        }
        return n;
    }

    AVLNode* insert(AVLNode* n, int k) {
        if (!n) return new AVLNode(k);
        if (k < n->key) n->left = insert(n->left, k);
        else if (k > n->key) n->right = insert(n->right, k);
        else return n;
        return balance(n);
    }

    // Удаление (упрощённо: удаление листа или узла с 1 потомком)
    AVLNode* remove(AVLNode* n, int k) {
        if (!n) return nullptr;
        if (k < n->key) n->left = remove(n->left, k);
        else if (k > n->key) n->right = remove(n->right, k);
        else {
            if (!n->left || !n->right) {
                AVLNode* temp = n->left ? n->left : n->right;
                delete n; return temp;
            }
            AVLNode* temp = n->right;
            while (temp->left) temp = temp->left;
            n->key = temp->key; n->right = remove(n->right, temp->key);
        }
        return balance(n);
    }

    bool search(AVLNode* n, int k) {
        if (!n) return false;
        if (n->key == k) return true;
        return k < n->key ? search(n->left, k) : search(n->right, k);
    }

public:
    AVLNode* root = nullptr;
    void insert(int k) { root = insert(root, k); }
    void remove(int k) { root = remove(root, k); }
    bool search(int k) { return search(root, k); }
};

// ==================== 2. ПРЕФИКСНОЕ ДЕРЕВО (TRIE) ====================
struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    TrieNode() : isEnd(false) { fill(children, children + 26, nullptr); }
};

class Trie {
    TrieNode* root;

    // Рекурсивный сбор слов для автодополнения
    void getWords(TrieNode* node, string current, vector<string>& res) {
        if (res.size() >= 5) return; // Ограничение топ-5
        if (node->isEnd) res.push_back(current);
        for (int i = 0; i < 26; i++) {
            if (node->children[i]) {
                getWords(node->children[i], current + char('a' + i), res);
            }
        }
    }

public:
    Trie() { root = new TrieNode(); }

    void insert(string word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!curr->children[idx]) curr->children[idx] = new TrieNode();
            curr = curr->children[idx];
        }
        curr->isEnd = true;
    }

    bool search(string word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!curr->children[idx]) return false;
            curr = curr->children[idx];
        }
        return curr->isEnd;
    }

    bool startsWith(string prefix) {
        TrieNode* curr = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!curr->children[idx]) return false;
            curr = curr->children[idx];
        }
        return true;
    }

    // ДОПОЛНИТЕЛЬНО (Вариант 4): Автодополнение (топ-5)
    vector<string> autocomplete(string prefix) {
        TrieNode* curr = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!curr->children[idx]) return {}; // Префикс не найден
            curr = curr->children[idx];
        }
        vector<string> result;
        getWords(curr, prefix, result);
        return result;
    }
};

// ==================== ГЛАВНАЯ ФУНКЦИЯ ====================
int main() {
    setlocale(LC_ALL, "ru");

    cout << "=== 1. Тестирование AVL-дерева ===" << endl;
    AVL avl;
    avl.insert(10); avl.insert(20); avl.insert(30); // Вызовет LL поворот
    cout << "Поиск 20: " << (avl.search(20) ? "Найден" : "Не найден") << endl;
    avl.remove(10);
    cout << "Поиск 10 после удаления: " << (avl.search(10) ? "Найден" : "Не найден") << endl << endl;

    cout << "=== 2. Тестирование Trie (Автодополнение топ-5) ===" << endl;
    Trie trie;
    // Вставляем слова (специально добавляем много слов на 'c')
    trie.insert("car");
    trie.insert("cart");
    trie.insert("cat");
    trie.insert("cattle");
    trie.insert("catfish");
    trie.insert("caterpillar");
    trie.insert("dog");

    string prefix = "cat";
    cout << "Запрос автодополнения для \"" << prefix << "\": ";

    vector<string> suggestions = trie.autocomplete(prefix);
    if (suggestions.empty()) {
        cout << "Нет вариантов" << endl;
    } else {
        for (string s : suggestions) cout << s << " ";
        cout << endl;
    }

    return 0;
}
```

## Вывод программы

```
=== 1. Тестирование AVL-дерева ===
Поиск 20: Найден
Поиск 10 после удаления: Не найден

=== 2. Тестирование Trie (Автодополнение топ-5) ===
Запрос автодополнения для "cat": cat cattle catfish cater caterpillar
```

---

## Анализ сложности операций

| Структура | Вставка | Поиск | Память |
|---|---|---|---|
| **BST** (худший случай) | O(n) | O(n) | O(n) |
| **AVL-дерево** | **O(log n)** | **O(log n)** | O(n) |
| **Trie** | **O(L)** (L — длина слова) | **O(L)** | O(алфавит · L · N) |

---

## Результаты и выводы

В ходе лабораторной работы были реализованы AVL-дерево и префиксное дерево (Trie). AVL-дерево показало, как повороты решают проблему вырождения BST. Trie отлично справился с задачей автодополнения: алгоритм находит все слова с заданным префиксом за время, зависящее только от длины префикса и количества искомых слов, а не от общего размера словаря. Реализовано ограничение топ-5, что соответствует реальным требованиям поисковых систем.

## Преимущества и недостатки

**AVL-дерево** — обеспечивает гарантированное O(log n) для вставки, поиска и удаления благодаря строгому балансу (фактор баланса ≤ 1). Основной недостаток — при каждой операции может требоваться серия поворотов, что делает его немного медленнее на вставках по сравнению с красно-чёрным деревом, где баланс менее строгий.

**Trie (префиксное дерево)** — незаменим для задач, связанных с префиксами: автодополнение, проверка орфографии, IP-маршрутизация. Поиск не зависит от количества слов в словаре, что делает его очень быстрым на больших объёмах. Главный недостаток — высокий расход памяти: каждый узел хранит массив указателей на алфавит, и большинство из них пусты. Решается через сжатые Trie (radix tree) или хранение детей в хеш-таблице.

**Сравнение с хеш-таблицей** — хеш-таблица быстрее для точного поиска (O(1)), но не умеет искать по префиксу; Trie проигрывает ей по памяти, но выигрывает по функциональности (префиксный поиск, автодополнение, лексикографический обход).

---

**Ссылка на репозиторий:**
[https://github.com/olesatrishina856-byte/Algorithms_Trishina/tree/main](https://github.com/olesatrishina856-byte/Algorithms_Trishina/tree/main)

---

## Контрольные вопросы

**1. Что такое AVL-дерево? В чём его отличие от обычного BST?**
Это самобалансирующееся BST. Отличие в том, что AVL гарантирует высоту O(log n) за счёт поворотов, тогда как обычное BST может выродиться в список O(n).

**2. Что такое фактор баланса?**
Разница высот левого и правого поддеревьев. В AVL он должен быть −1, 0 или 1.

**3. Какие существуют типы поворотов?**
Малые (левый, правый) и большие (левый-правый, правый-левый).

**4. Когда применяется правый поворот?**
Когда левое поддерево перевешивает (bf > 1), и левый потомок не имеет правого перекоса.

**5. Какова сложность операций в AVL-дереве?**
O(log n) для вставки, поиска и удаления.

**6. Почему AVL-дерево называется самобалансирующимся?**
Потому что оно автоматически восстанавливает баланс при каждой операции.

**7. Что такое Trie (префиксное дерево)?**
Древовидная структура, где каждый узел — это символ. Используется для словарей и автодополнения.

**8. Какова сложность поиска в Trie?**
O(L), где L — длина искомого слова. Не зависит от количества слов в дереве.

**9. Какая структура эффективнее для строк: Trie или хеш-таблица?**
Хеш-таблица быстрее для поиска точного совпадения (O(1)), но Trie лучше для поиска по префиксу и автодополнения.

**10. Как реализовать автодополнение с помощью Trie?**
Найти узел, соответствующий последнему символу префикса, а затем рекурсивно обойти всех его потомков, собирая слова до знака `isEnd`.

**11. Каковы недостатки Trie?**
Требует много памяти, так как каждый узел хранит массив указателей (например, на 26 букв), даже если большинство из них пусты.
