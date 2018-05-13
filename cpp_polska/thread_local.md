# Mało znany kuzyn statica - thread\_local
C++11 wprowadził wiele funkcjonalności, bez których nie wyobrażamy sobie pisania nowoczesnego kodu. Należą do nich na przykład move semantics, lambdy czy smart pointery. Są też ficzery, z których korzysta się raczej rzadko, ale warto je znać i wiedzieć co potrafią. Jednym z takich dodatków jest słowo kluczowe thread_local.

## Cykl istnienia obiektów*


Zanim przejdziemy dalej, stwórzmy sobie pomocniczą strukturę, która wypisze na standardowe wyjście swój moment tworzenia i niszczenia:

    #include <iostream>

    struct Foo {
      Foo(int i) : i_(i) { std::cout << "Foo(" << i_ << ")\n"; }
      ~Foo() { std::cout << "~Foo(" << i_ << ")\n"; }
      int i_;
    };

Mając taką strukturę, możemy przetestować w jakiej kolejności wywoływane są konstruktory i destruktory. 

### Automatyczny
Dla normalnie zadeklarowanych zmiennych, kolejność tworzenia i niszczenia jest zgodna z naszymi oczekiwaniami:

    int main() {
      Foo foo1(1); 
      {
        Foo foo2(2);
      }
      return 0;
    }

Na ekranie pojawi się :

    Foo(1)
    Foo(2)
    ~Foo(2)
    ~Foo(1)

Zgodnie z naszą intuicją, najpierw zostanie stworzona zmienna foo1, potem foo2, następnie natrafiamy na koniec zakresu zmiennej foo2, więc zmienna ta zostaje zniszczona. Na końcu, niszczy się foo1. Taki cykl istnienia obiektu (ang. Storage duration) - od momentu zadeklarowania to końca zakresu - nazywamy automatycznym.

### Statyczny
Sprawdźmy co się stanie, jeśli foo2 będzie zmienną statyczną:

    int main() {
      Foo foo1(1); 
      {
        static Foo foo2(2);
      }
      return 0;
    }
Po skompilowaniu i uruchomieniu, na ekranie zobaczymy: 

    Foo(1)
    Foo(2)
    ~Foo(1)
    ~Foo(2)

Jak widać, zmienna foo2 została zniszczona później niż foo1. Słowo static w powyższym kontekście, oznacza statyczny cykl istnienia(ang. static storage duration). Oznacza to, że zmienna zostanie stworzona w kolejności zadeklarowania (czyli po foo1) a usunięta na końcu działania programu. Statyczny cykl istnienia posiadają również zmienne globalne oraz zmienne zadeklarowane jako extern.
Przenalizujmy trochę trudniejszy przykład:

    Foo foo3(3);

    int main() {
      Foo foo1(1); 
      {
        static Foo foo2(2);
      }
      return 0;
    }

Po uruchomieniu, na ekranie zobaczymy:

    Foo(3)
    Foo(1)
    Foo(2)
    ~Foo(1)
    ~Foo(2)
    ~Foo(3)

Zmienne zostały stworzone w kolejności deklaracji. Jeśli chodzi o niszczenie - najpierw wywołał się destruktor foo1 (wyjście z zakresu), dalej zmienne są niszczone w kolejności odwrotnej do tworzenia, czyli najpierw wywołał się destruktor zmiennej foo2 a potem zmiennej foo3. 

Warto znać niuans dotyczący inicjalizacji zmiennych ze statycznym cyklem istnienia. Rozważmy taki przykład: 

    extern int i;
    int j = i;
    int i = 2;

    int main() {
      return j;
    }

W powyższym przykładzie, mogłoby się wydawać że zmienna j zostanie zainicjalizowana wartością 0. Jednak cały program zwraca wartość 2. 
Spróbujmy napisać więc kontrprzykład, gdzie zmienna j zostanie zainicjalizowana zerem: 

    int foo() {
      return 2;
    }

    extern int i;
    int j = i;
    int i = foo();

    int main() {
      return j;
    }

Po tej niewielkiej zmianie, program zwraca wartość zero. Dlaczego tak się dzieje?
Zerknijmy więc do standardu C++, dokładniej do [tego](http://eel.is/c++draft/basic.start.static#2) paragrafu:

>All static initialization strongly happens before (...) any dynamic initialization.

`int i = 2` jest inicjalizacją statyczną, natomiast `int j = i` jest inicjalizacją dynamiczną. Standard gwarantuje nam więc, że najpierw wykona się przypisanie do i.
W drugim przypadku - obie inicjalizacje są dynamiczne, więc j będzie miało wartość 0 ([zero initialization](http://en.cppreference.com/w/cpp/language/zero_initialization)).

### Wątkowy
Wróćmy teraz do tytułowego zagadnienia, czyli do thread\_local. Dodajmy jeszcze jedną zmienną, foo4:

    Foo foo3(3);

    int main() {
      Foo foo1(1);
      {
        thread_local Foo foo4(4);
        static Foo foo2(2);
      }
      return 0;
    }
    
Na ekranie pojawi się: 

    Foo(3)
    Foo(1)
    Foo(4)
    Foo(2)
    ~Foo(1)
    ~Foo(4)
    ~Foo(2)
    ~Foo(3)

Zmienna zadeklarowana jako thread\_local posiada wątkowy cykl istnienia  (ang. thread storage duration). Niszczona jest w momencie niszczenia wątku, czyli w naszym przykładzie zmienna foo4 zostanie zniszczona wcześniej niż zmienna foo2.

    void foo() {
      static int i = 1;
      thread_local Foo foo2(i++);
    }

    int main() {
      std::thread th1(foo);
      th1.join();

      std::thread th2(foo);
      th2.join();
      return 0;
    }

Na ekranie zostanie wypisane:

    Foo(1)
    ~Foo(1)
    Foo(2)
    ~Foo(2)

Jak widać, została stworzona tylko jedna zmienna "i" dla obu wątków, natomiast dla każdego wątku została stworzona oddzielnie zmienna Foo.


Oprócz tego, że zmienna może być thread\_local, nic nie stoi na przeszkodzie, aby zadeklarować zmienną jako static thread local:

    static thread_local Foo foo(0);

W tym kontekście, słowo static odnosi się do typu linkowania i oznacza, że zmienna ma linkowanie wewnętrzne (ang. internal linkage). 
Słowo static jest tutaj zbędne, gdyż taki rodzaj linkowania jest domyślny. Alternatywą jest linkowanie zewnętrzne ze słowem kluczowym extern:

    extern thread_local Foo foo; //defined in other compilation unit;

Linkowanie zewnętrzne (ang. external linkage) oznacza, że zmienna foo2 jest zdefiniowana w innej jednostce kompilacji.

## Praktyczne zastosowanie

Kiedy więc może nam się przydać zmienna z takim cyklem istnienia? Ostatnio natrafiłem na miejsce, gdzie mogłem tego użyć.
Korzystałem z zewnęrznej bilbioteki, libmongocxx, i natrafiłem na problem z wielowątkowym dostępem do bazy danych. Przeszukując dokumentację trafiłem na zdanie: 

> In general each mongocxx::client object AND all of its child objects should be used by a single thread at a time.

Zamiast mutexować wszystko jak leci (btw, każy zna osobę, która wszystkie problemy wielowątkowe rozwiązuje dokładając kolejne mutexy :) ), 
wystarczyło zmienić klienta do bazy danych na membera typu thread\_local - i rozwiązało to problemy synchronizacyjne. 

## Podsumowanie

Język C++ posiada bardzo wiele funkcjonalności, z których na co dzień się nie używa. Jedną z nich jest thread\_local, które może być bardzo przydatne 
w aplikacjach wielowątkowych. Zamiast synchronizować zmienną statyczną, możemy użyć zmiennej thread\_local i żadna synchronizacja nie będzie potrzebna. 


* - tłumaczenie pochodzi z polskiej edycji książki "Język C++. Kompendium wiedzy" Bjarne'a Stroupstroupa, w tłumaczeniu Łukasza Piwko
