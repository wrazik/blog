Wielu programistów korzystających z gcc używa standardowego zestawu flag ostrzegających przed błędami, 
czyli tytułowego -Wall -Wextra -pedantic. Użytkownicy clanga mają jeszcze dodatkowo flagę -Weverything. 
A jakich dodatkowych flag mogą  użyć osoby korzystające z gcca?

Gcc
Poniżej znajduje się lista najbardziej flag, które uważam za najbardziej przydatny.

* `-Wshadow`
Kompilator ostrzega, gdy zmienna zostanie przysłonięta przez zmienną o tej samej nazwie.

```cpp
int main() {
  int i {0};
  {
      int i{1}; // warning: declaration of 'i' shadows a previous local
  }
  return i;
}
```

Interaktywny przykład: [https://godbolt.org/g/LCs9ED](https://godbolt.org/g/LCs9ED)

* -Wnon-virtual-dtor
Kompilator ostrzega, jeśli jakaś klasa posiada metodę wirtualną i nie dostarcza wirtualnego destruktora

class Foo { // warning: <source>:1:7: warning: 'class Foo' has virtual functions and accessible non-virtual destructor
  virtual void foo() {}
};

```cpp
int main() {} 
```
Interaktywny przykład: https://godbolt.org/g/GsgTXd

-Wold-style-cast 
Kompilator ostrzega, gdy będziemy używać rzutowania w stylu C

```cpp
int main() {
  (double)4; //warning: use of old-style cast
} 
```
Interaktywny przykład: https://godbolt.org/g/Dia7Gf

* -Wcast-align
Ostrzega, jeśli 

* -Wunused warn on anything being unused
Ostrzega, jeśli 
* -Woverloaded-virtual warn if you overload (not override) a virtual function
Ostrzega, jeśli 
* -Wpedantic warn if non-standard C++ is used
Ostrzega, jeśli 
* -Wconversion warn on type conversions that may lose data
Ostrzega, jeśli 
* -Wsign-conversion warn on sign conversions
Ostrzega, jeśli 
* -Wmisleading-indentation warn if identation implies blocks where blocks do not exist
Ostrzega, jeśli 
* -Wduplicated-cond warn if if / else chain has duplicated conditions
Ostrzega, jeśli 
* -Wduplicated-branches warn if if / else branches have duplicated code
Ostrzega, jeśli 
* -Wlogical-op warn about logical operations being used where bitwise were probably wanted
Ostrzega, jeśli 
* -Wnull-dereference warn if a null dereference is detected
Ostrzega, jeśli 
* -Wuseless-cast warn if you perform a cast to the same type
Ostrzega, jeśli 
* -Wdouble-promotion warn if float is implicit promoted to double
Ostrzega, jeśli 
* -Wformat=2 warn on security issues around functions that format output (ie printf)
Ostrzega, jeśli 


Clang
Tak jak wspominałem na początku - clang posiada flagę -Weverything, która włącza wszystkie dostępne flagi, Jak się zapewne domyślacie - powoduje to bardzo dużo false-positive'ów, dlatego można zastosowań inne podejście: użyć flagi -Weverithing i wyłączyć te flagi, na których nam nie zależy.

MSVS
MSVS okazuje się być najbardziej uczciwym kompilatorem. Po właczeniu flagi /W4 (TODO: upewnij się), dostaniemy również warningi z biblioteki standardowej! 



A wy, znacie jeszcze jakieś ciekawe flagi, które pomagają we wczesnym znajdowaniu błędów?



[Źródło: https://github.com/lefticus/cppbestpractices/blob/master/02-Use_the_Tools_Available.md#gcc--clang]
