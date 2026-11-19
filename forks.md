# Procesy, wątki i synchronizacja w C++

## 1. Teoria – procesy i wątki

### 1.1. Czym jest proces?

Proces to uruchomiony program wraz z zasobami przydzielonymi mu przez system operacyjny.

Każdy proces posiada:
- własną przestrzeń adresową pamięci;
- identyfikator PID;
- zasoby systemowe;
- co najmniej jeden wątek wykonania.

Przykład: gdy uruchamiasz dwie instancje tej samej gry, system może utworzyć dwa niezależne procesy.

### 1.2. Czym jest wątek?

Wątek (thread) to pojedyncza ścieżka wykonywania kodu wewnątrz procesu.

Wątki tego samego procesu współdzielą:
- pamięć globalną;
- stertę (heap);
- otwarte pliki i inne zasoby procesu.

Każdy wątek ma natomiast własny stos (stack).

Przykład w grze:
- jeden wątek obsługuje sterowanie postacią;
- drugi oblicza fizykę;
- trzeci zajmuje się AI przeciwników;
- czwarty obsługuje zadania w tle.

### 1.3. Proces vs wątek

| Cecha | Proces | Wątek |
|---|---|---|
| Pamięć | Oddzielna przestrzeń | Współdzielona w procesie |
| PID | Własny | Wspólny dla procesu |
| Stos | Własny | Własny |
| Komunikacja | Wymaga IPC | Może korzystać ze wspólnych danych |
| Koszt utworzenia | Zwykle większy | Zwykle mniejszy |
| Awaria | Zwykle izolowana | Może zakończyć cały proces |

### 1.4. Współbieżność i równoległość

**Współbieżność (concurrency)** oznacza, że kilka zadań może wykonywać się w przeplatający się sposób.

**Równoległość (parallelism)** oznacza faktyczne wykonywanie zadań w tym samym czasie, np. na różnych rdzeniach procesora.

Utworzenie czterech wątków nie oznacza automatycznie, że będą działać na czterech rdzeniach. System operacyjny decyduje o przydzielaniu czasu procesora.

---

## 2. Procesy – funkcja fork()

`fork()` to funkcja systemowa POSIX, która tworzy nowy proces będący potomkiem procesu wywołującego.

Po wykonaniu `fork()` istnieją dwa procesy:
- rodzic (parent);
- dziecko (child).

Oba procesy kontynuują wykonywanie programu od miejsca wywołania `fork()`.

Funkcja zwraca:
- `0` – w procesie potomnym;
- dodatnią wartość – w procesie rodzica (PID dziecka);
- `-1` – błąd utworzenia procesu.

### 2.1. Pierwszy program z fork()

Utwórz plik `fork_example.cpp`.

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    std::cout << "Start programu" << std::endl;

    pid_t pid = fork();

    if (pid == 0) {
        std::cout << "Jestem dzieckiem!" << std::endl;
        std::cout << "PID: " << getpid() << std::endl;
    }
    else if (pid > 0) {
        std::cout << "Jestem rodzicem!" << std::endl;
        std::cout << "PID dziecka: " << pid << std::endl;

        wait(nullptr);
        std::cout << "Dziecko zakonczylo prace." << std::endl;
    }
    else {
        std::cerr << "Blad fork()" << std::endl;
        return 1;
    }

    return 0;
}
```

Kompilacja:

```bash
g++ fork_example.cpp -o fork_example
```

Uruchomienie:

```bash
./fork_example
```

### 2.2. Zadania do wykonania

1. Uruchom program kilka razy. Czy kolejność komunikatów jest zawsze taka sama?
2. Sprawdź PID rodzica i dziecka.
3. Dodaj `getppid()` i sprawdź PID rodzica z poziomu dziecka.
4. Usuń `wait(nullptr)`. Co się zmieni?
5. Dodaj zmienną globalną `int counter = 0`.
6. Zwiększ jej wartość w procesie potomnym. Sprawdź, czy zmiana jest widoczna u rodzica.

**Pytanie:** Dlaczego proces potomny nie zmienia wartości zmiennej w procesie rodzica?

Wskazówka: procesy mają oddzielne przestrzenie adresowe. Linux wykorzystuje mechanizm Copy-on-Write.

### 2.3. Obserwacja w systemie

Uruchom program i obserwuj procesy w `htop` lub `btop`.

Zadania:
- spróbuj zauważyć dwa procesy;
- odczytaj ich PID;
- sprawdź, który proces jest rodzicem;
- uruchom kilka kopii programu;
- zaobserwuj, jak zmienia się liczba procesów.

---

## 3. Wątki – std::thread

W przeciwieństwie do `fork()`, wątki działają w ramach jednego procesu i mogą korzystać ze wspólnej pamięci.

W C++11 wprowadzono standardową bibliotekę obsługującą wątki.

### 3.1. Pierwszy program wielowątkowy

Utwórz plik `thread_example.cpp`.

```cpp
#include <iostream>
#include <thread>

void zadanie(int id) {
    for (int i = 0; i < 5; i++) {
        std::cout << "Watek " << id
                  << " wykonuje krok " << i << std::endl;
    }
}

int main() {
    std::thread t1(zadanie, 1);
    std::thread t2(zadanie, 2);

    t1.join();
    t2.join();

    std::cout << "Koniec programu!" << std::endl;

    return 0;
}
```

Kompilacja:

```bash
g++ thread_example.cpp -o thread_example -pthread
```

Uruchomienie:

```bash
./thread_example
```

### 3.2. Ważne pojęcia

- `std::thread` – tworzy nowy wątek.
- `join()` – czeka, aż wskazany wątek zakończy pracę.
- `detach()` – odłącza wątek od obiektu zarządzającego nim.
- `std::this_thread::sleep_for()` – wstrzymuje wykonywanie aktualnego wątku.
- `std::thread::hardware_concurrency()` – podaje wskazówkę dotyczącą liczby dostępnych wątków sprzętowych.

### 3.3. Zadania do wykonania

1. Utwórz 4 wątki zamiast 2.
2. Każdy wątek powinien wypisać 20 komunikatów.
3. Dodaj `sleep_for()` i opóźnienie 500 ms.
4. Wypisz identyfikator każdego wątku za pomocą `std::this_thread::get_id()`.
5. Sprawdź, czy kolejność komunikatów jest zawsze identyczna.
6. Sprawdź, ile wątków sprzętowych zgłasza Twój komputer.

Przykładowy fragment:

```cpp
std::cout << "Liczba watkow sprzetowych: "
          << std::thread::hardware_concurrency()
          << std::endl;
```

### 3.4. Obserwacja w Task Managerze / btop

Uruchom program z większą liczbą wątków i obserwuj system.

W Linuxie:

```bash
ps -T -p PID
```

W Windowsie sprawdź kolumnę Threads w zakładce Szczegóły Menedżera zadań.

Zwróć uwagę na:
- liczbę wątków procesu;
- wykorzystanie CPU;
- liczbę procesów;
- identyfikatory wątków w Linuxie.

**Pytania:**
- Czy liczba wątków w systemie wzrosła?
- Czy wszystkie wątki wykonują się jednocześnie?
- Czy wykorzystanie CPU wzrosło?
- Czy zwiększenie liczby wątków zawsze przyspiesza program?

---

## 4. Race Condition – wyścig danych

Jednym z najważniejszych problemów programowania wielowątkowego jest sytuacja, w której kilka wątków jednocześnie korzysta z tych samych danych.

Wyścig danych (data race) występuje, gdy dochodzi do niezsynchronizowanych, konfliktujących operacji na tej samej lokalizacji pamięci.

### 4.1. Przykład problemu

Wyobraź sobie, że dwa wątki zwiększają tę samą zmienną.

Oczekiwany wynik: **2 000 000**

```cpp
#include <iostream>
#include <thread>

int licznik = 0;

void zwiekszaj() {
    for (int i = 0; i < 1000000; i++) {
        licznik++;
    }
}

int main() {
    std::thread t1(zwiekszaj);
    std::thread t2(zwiekszaj);

    t1.join();
    t2.join();

    std::cout << "Wynik: " << licznik << std::endl;

    return 0;
}
```

### 4.2. Dlaczego pojawia się problem?

Operacja:

```cpp
licznik++;
```

Można koncepcyjnie rozłożyć na:

1. Odczyt aktualnej wartości.
2. Zwiększenie wartości.
3. Zapis nowej wartości.

Przykładowa sytuacja:

| Krok | Wątek 1 | Wątek 2 | Licznik |
|---|---|---|---:|
| 1 | Odczytuje 10 | | 10 |
| 2 | | Odczytuje 10 | 10 |
| 3 | Zapisuje 11 | | 11 |
| 4 | | Zapisuje 11 | 11 |

Zamiast 12 otrzymujemy 11.

To uproszczony model utraconej aktualizacji. W rzeczywistym C++ taki program ma data race i niezdefiniowane zachowanie (undefined behavior), więc nie należy zakładać, że zawsze otrzymamy konkretny błędny wynik.

### 4.3. Zadania

1. Uruchom program kilka razy.
2. Zwiększ liczbę wątków do 4.
3. Zwiększ liczbę iteracji do 10 milionów.
4. Wyjaśnij, dlaczego program może zwracać różne wyniki.
5. Zastanów się, dlaczego problem nie występuje, kiedy każdy wątek ma osobną zmienną.

---

## 5. Mutex – zabezpieczenie współdzielonych danych

Mutex (mutual exclusion) pozwala ograniczyć dostęp do określonego fragmentu kodu tak, aby w danym momencie wykonywał go tylko jeden wątek.

Można to porównać do jednej toalety z kluczem: osoba, która ma klucz, może wejść, a pozostałe muszą poczekać.

### 5.1. Rozwiązanie problemu za pomocą mutexa

Utwórz `mutex_example.cpp`.

```cpp
#include <iostream>
#include <thread>
#include <mutex>

int licznik = 0;
std::mutex m;

void zwiekszaj() {
    for (int i = 0; i < 1000000; i++) {
        std::lock_guard<std::mutex> lock(m);
        licznik++;
    }
}

int main() {
    std::thread t1(zwiekszaj);
    std::thread t2(zwiekszaj);

    t1.join();
    t2.join();

    std::cout << "Wynik: " << licznik << std::endl;

    return 0;
}
```

Kompilacja:

```bash
g++ mutex_example.cpp -o mutex_example -pthread
```

Oczekiwany wynik:

```
Wynik: 2000000
```

### 5.2. Jak działa lock_guard?

```cpp
std::lock_guard<std::mutex> lock(m);
```

- Zajmuje mutex.
- Jeśli inny wątek już go posiada, czeka.
- Po wyjściu z zakresu zmiennej automatycznie zwalnia mutex.

Jest to bezpieczniejsze niż ręczne wywoływanie `lock()` i `unlock()`, ponieważ zwolnienie blokady następuje również przy wyjściu z zakresu wskutek wyjątku.

### 5.3. Zadania

1. Dodaj 4 wątki zwiększające licznik.
2. Sprawdź, czy wynik jest prawidłowy.
3. Przenieś deklarację `lock_guard` poza pętlę.
4. Zmierz czas działania obu wersji.
5. Zastanów się, dlaczego zabezpieczenie całej pętli może ograniczać współbieżność.
6. Sprawdź, czy mutex zawsze zwiększa wydajność programu.

---

## 6. Eksperyment – obserwacja CPU i procesów

Przygotuj trzy programy:

- program jednowątkowy;
- program wykorzystujący 4 wątki;
- program wykorzystujący 8 wątków.

Każdy powinien wykonywać podobne obliczenia przez odpowiednio długi czas.

Do pomiaru możesz wykorzystać `std::chrono`.

### 6.1. Przykładowy kod pomiarowy

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>

void obliczenia() {
    volatile long long wynik = 0;

    for (long long i = 0; i < 100000000; i++) {
        wynik += i % 7;
    }
}

int main() {
    auto start = std::chrono::high_resolution_clock::now();

    std::vector<std::thread> watki;

    for (int i = 0; i < 4; i++) {
        watki.emplace_back(obliczenia);
    }

    for (auto& t : watki) {
        t.join();
    }

    auto koniec = std::chrono::high_resolution_clock::now();

    auto czas =
        std::chrono::duration_cast<std::chrono::milliseconds>(
            koniec - start
        ).count();

    std::cout << "Czas: " << czas << " ms\n";
}
```

### 6.2. Eksperymenty

Uruchom program z 1, 2, 4 i 8 wątkami.

| Liczba wątków | Czas wykonania | Wykorzystanie CPU |
|---:|---:|---:|
| 1 | | |
| 2 | | |
| 4 | | |
| 8 | | |

Obserwuj jednocześnie `btop` lub Menedżer zadań Windows.

**Pytania do analizy:**

1. Przy ilu wątkach program działał najszybciej?
2. Czy zwiększenie liczby wątków zawsze skraca czas?
3. Co się dzieje, gdy liczba wątków przekracza liczbę rdzeni logicznych?
4. Czy każdy wątek musi mieć osobny rdzeń?
5. Dlaczego synchronizacja mutexem może spowolnić obliczenia?

Uwaga: `volatile` w tym przykładzie ogranicza pewne optymalizacje kompilatora, ale nie jest mechanizmem synchronizacji wątków. Właściwe zabezpieczenie danych wymaga m.in. mutexów lub atomików.

---

## 7. Zadanie końcowe – symulacja wielowątkowa

Stwórz program symulujący pracę prostego świata gry.

Wymagania:

1. Program ma posiadać wspólną zmienną określającą liczbę punktów gracza.
2. Cztery wątki mają symulować czterech przeciwników.
3. Każdy przeciwnik wykonuje 100 000 operacji zwiększających liczbę punktów.
4. Program ma wyświetlić końcowy wynik.
5. Początkowo wykonaj program bez mutexa.
6. Następnie zabezpiecz wspólną zmienną za pomocą mutexa.
7. Porównaj wyniki i wyjaśnij różnicę.

**Dodatkowe wyzwanie:**

Zamiast jednego wspólnego licznika każdy przeciwnik powinien posiadać własną liczbę punktów. Na końcu główny wątek powinien zsumować wyniki wszystkich przeciwników.

Zastanów się, czy w takim przypadku mutex jest jeszcze potrzebny.

---

## 8. Pytania sprawdzające

1. Czym różni się proces od wątku?
2. Co zwraca `fork()` w procesie potomnym?
3. Czy procesy utworzone przez `fork()` współdzielą zwykłe zmienne globalne?
4. Do czego służy `join()`?
5. Co się stanie, jeśli zakończymy `main()`, zanim dołączymy utworzone wątki?
6. Czym jest race condition?
7. Dlaczego `licznik++` nie jest bezpieczne przy jednoczesnym dostępie z kilku wątków?
8. Jak działa mutex?
9. Czym różni się mutex od `std::atomic`?
10. Czy większa liczba wątków zawsze oznacza większą wydajność?

---

## 9. Przydatne materiały i dokumentacja

### Procesy i fork()

- [Linux man-pages – fork()](https://man7.org/linux/man-pages/man2/fork.2.html)
- [Linux man-pages – wait()](https://man7.org/linux/man-pages/man2/wait.2.html)
- [Beej's Guide to Unix IPC](https://beej.us/guide/bgipc/)

### Wątki w C++

- [cppreference – std::thread](https://en.cppreference.com/w/cpp/thread/thread)
- [Microsoft Learn – thread](https://learn.microsoft.com/en-us/cpp/standard-library/thread-class)
- [cppreference – Thread support library](https://en.cppreference.com/w/cpp/thread)

### Race condition i synchronizacja

- [cppreference – Multi-threaded executions and data races](https://en.cppreference.com/w/cpp/language/multithread)
- [cppreference – std::mutex](https://en.cppreference.com/w/cpp/thread/mutex)
- [cppreference – std::atomic](https://en.cppreference.com/w/cpp/atomic/atomic)

### Obserwacja systemu

- [btop – GitHub](https://github.com/aristocratos/btop)
- [htop](https://htop.dev/)
- [Microsoft – Task Manager](https://support.microsoft.com/windows)

---

*Zalecane środowisko: C++17, G++ oraz Linux/WSL2. Kolejne zagadnienia do poznania: komunikacja międzyprocesowa (pipe, shared memory), condition_variable oraz thread pool.*