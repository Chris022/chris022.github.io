---
title: Der C Präprozessor
author: Christoph Weberbauer
layout: post
---

{% abstract %}
In diesem Artikel geht es um den C-Präprozessor und coole Dinge, die man damit machen kann.

Disclaimer: Ich bin kein C-Profi, habe mich aber gestern mit dem Präprozessor beschäftigt und wollte die Dinge, die ich dabei gelernt habe, hier etwas zusammenfassen.
{% endabstract %}


## Einleitung

Der [C-Präprozessor](https://gcc.gnu.org/onlinedocs/cpp/) ist ein Programm, welches grundsätzlich separat zum eigentlichen C-Compiler existiert. Für gewöhnlich wird dieses Programm allerdings als erster Schritt beim Compilieren von C-Code aufgerufen. Im Grunde genommen ist der Präprozessor eine eigene Programmiersprache, die Sourcecode eines Programmes noch vor dessen Kompilierung verändert.
Ein weiteres sehr bekanntes Beispiel für einen solchen Präprozessor ist [PHP](https://de.wikipedia.org/wiki/PHP).
Laut [Wikipedia](https://de.wikipedia.org/wiki/C-Pr%C3%A4prozessor) bearbeitet der C-Präprozessor "Anweisungen zum Einfügen von Quelltext (`#include`), zum Ersetzen von Makros (`#define`), und bedingter Übersetzung (`#if`)." Was genau dies heißt, will ich in dieser Notiz etwas genauer betrachten.


## Einfügen von Quelltext

In C ist es so, dass eine Funktion erst aufgerufen werden kann, nachdem sie definiert worden ist. Anders hat der C-Compiler keine Chance zu wissen, ob die Datentypen des Funktionsaufrufes korrekt sind. Betrachtet man also das folgende Stück Code:

```c
#include <stdio.h>

void main(){
	say_hello();
}

void say_hello(){
	printf("Hello World.");
}
```

So würde der C-Compiler einen Fehler erzeugen, da er zum Zeitpunkt des Aufrufs von `say_hello()` noch nichts über die Existenz dieser Funktion weiß. Als Lösung gibt es zwei Möglichkeiten. Möglichkeit 1 ist es, die Funktion vor dem ersten Aufruf zu definieren. Dies ist jedoch nicht immer praktikabel. Deshalb existiert die Möglichkeit, 2. Dabei weiß man den C-Compiler nur vor dem ersten Aufruf der Funktion auf deren Existenz hin. Also zum Beispiel so:

```c
#include <stdio.h>

void say_hello();

void main(){
	say_hello();
}

void say_hello(){
	printf("Hello World.");
}
```

In diesem Fall muss die Implantation der Funktion dann nicht einmal in derselben Datei passieren. So kann man auch folgendes machen

 
```c
//main.c
void say_hello();

void main(){
	say_hello();
}
```

```c
//other.c
# include <stdio.h>

void say_hello(){
	printf("Hello");
}
```

Beim Compilieren gibt man dann einfach alle zu compilierenden Dateien an. Also `gcc main.c other.c`. Wichtig, damit man die Funktion `say_hello()` aber in `main.c` aufrufen kann, muss der C-Compiler wissen, dass diese existiert. Daher auch die Deklaration: `void say_hello()`. Dies muss man nun also in alle Files geben, in denen die Funktion aufgerufen werden soll. Benutzt man viele Files, kann dies sehr umständlich werden. Deshalb werden diese Deklarationen meist in sogenannten Header Files gesammelt. 
Dies bedeutet, man erstellt ein File `other.h` welches alle Deklarationen des Files `other.c` enthält. Also so


```c
//other.h
void say_hello();
```

Und hier kommt nun der Präprozessor ins Spiel. Dieser stellt jetzt den Befehl `#include "file"` zur Verfügung. Dieser macht nichts anders, als den Inhalt des Files mit dem Namen `file` in das eigentliche File zu kopieren. Also kann man unser `main.c` wie folgt umschreiben.


```c
//main.c
#include "other.h"

void main(){
	say_hello();
}
```

Beim Compilieren wird nun im ersten Schritt der Präprozessor aufgerufen und fügt nun die Deklaration unserer Funktion `say_hello` (und alle anderen aus `other.h`) in `main.c` ein.

INFO: Es gibt zwei Arten von `#include`. Schreibt man den Filenamen in `<>` also z.B.: `#include <stdio.h>` so sucht der Präprozessor nur im Pfad des C-Compilers nach der Datei. 
Schreibt man den Filenamen in `""` also z.B.: `#include "other.h"` so sucht der Präprozessor zusätzlich noch im aktuellen Verzeichnis. 

Zusammengefasst macht der `#include` Befehl nichts anders, als den Inhalt einer Datei in die andere Datei zu kopieren. Dies werden wir später noch genauer untersuchen.

## Makros

Ein Makro ist einfach ein benanntes Stück Code. Jedes Vorkommen des Namens wird dann einfach durch den Code ersetzt. So kann man zum Beispiel ein Makro namens `PI` erstellen und dann im Code wie folgt nutzen:
```c
#define PI 3.1415

printf("%i", PI);
```

Der Unterschied zwischen einer Variable und einem Makro ist hierbei, dass das Makro schon vor dem Kompilieren ersetzt wird. Der C-Compiler sieht also obigen Code nie. Stattdessen sieht dieser lediglich
```c
printf("%i", 3.1415);
```


Es gibt dabei zwei Arten von Makros.
1) Makros ohne Parameter. Dies sind Makros wie oben, wobei ein Text einfach durch einen anderen Text ersetzt wird.
2) Makros mit Parametern. Diese Makros ähneln einer Funktion. Akzeptieren also Parameter. Basierend auf diesen Parametern wird ein Text dann durch unterschiedliche Texte ersetzt. Ein Beispiel ist z.B. das folgende Makro, welches die größere von zwei Zahlen findet. `#define MAX(a, b) ((a) < (b) ? (b) : (a))`. Ich will hierbei erneut anmerken, dass ein Aufruf `MAX(1,2)` kein wirklicher Funktionsaufruf ist! Stattdessen wird `MAX(1,2)` einfach durch `((1) < (2) ? (2) : (1))` ersetzt.

INFO: Will man mehrere Zeilen ersetzen, so endet man eine Zeile immer mit `\` ähnlich wie in Python.

Mithilfe dieser Makros kann man nun einige coole Dinge machen. Zum Beispiel kann man die gesamte Syntax von C verändern und neue Kontrollstrukturen erschaffen. So gibt es in C keine `foreach` Schleife. Mit Makros kann man diese aber nachbauen.

```c
#include <stdio.h>

#define foreach(from, to) \
	for(int i = from; i < to; i++)


void main(){
	foreach(1, 10){
		printf("%i", i);
	}
}
``` 

Eine Library, die dies sehr exzessiv nutzt, ist [Catch2](https://github.com/catchorg/Catch2).

Makros erlauben es einem auch generelle Funktionen zu schreiben. Angenommen, man will eine Funktion `add` schreiben, die zwei `Ints` addiert. So kann man dies natürlich einfach machen. Aber was, wenn man nun die gleiche Funktion mit `Doubles` schreiben will... Dann kann man dies natürlich auch händisch machen, dies wird aber evl. mit der Zeit zu aufwendig. Deshalb kann man auch ein Makro schreiben, welches eine add Funktion erzeugt.

```c
#include <stdio.h>

#define ADD(TYPE) \
	TYPE add_##TYPE(TYPE a, TYPE b){return a + b;}

ADD(int)
ADD(double)
ADD(char)

void main(){
	printf("%i", add_int(1,2));
}
```

Merke das `##` beim Namen der Funktion. Dies nutzt man, da der Präprozessor sonst das `TYPE` in `add_TYPE` nicht ersetzen würde, da er `add_TYPE` als ein einziges Literal sieht. Durch das `##` sagt man dem Präprozessor dann, dass `add_` und `TYPE` separate Literale sind. Für genauere Information Empfehle ich [Concatenation](https://gcc.gnu.org/onlinedocs/cpp/Concatenation.html) und [Stringizing](https://gcc.gnu.org/onlinedocs/cpp/Stringizing.html).
## Bedingte Ersetzung
Die folgenden Befehle können verwendet werden, um gewisse Teile von Code unter bestimmten Bedingungen zu ersetzen: `#if`, `#ifdef`, `#ifndef`, `#else`, `#elif` und `#endif`. 
Die Befehle funktionieren dabei immer ähnlich und können am besten anhand eines Beispiels erklärt werden:
```c 
#if VERBOSE >=2
	printf("test");
#else
	printf("other");
#endif
```

Hierbei prüft der Präprozessor, ob ein Makro namens `VERBOSE` einen Wert größer gleich 2 hat. Wenn ja, fügt er den Code `printf("test");` in den Output ein. Wenn nicht, wird der Codeteil aus dem else Zweig in den Output inkludiert. 

`#ifdef` prüft nun nicht den Wert eines Makros, sondern lediglich, ob das Makro definiert ist.
`#ifndef` prüft dann noch, ob das Makro nicht definiert ist.

In C kann dies praktisch sein, wenn man auf unterschiedlichen Betriebssystemen unterschiedliche Libraries inkludieren muss.
## C-Präprozessor als Programm
Zum Schluss will ich nun noch verdeutlichen, dass der Präprozessor nichts direkt mit dem C-Compiler zu tun hat! Dafür will ich den C-Präprozessor einfach mal in Python nutzen. Um den Präprozessor vor der Python-Interpretation aufzurufen, nutzten wir das gcc Flag `-E` und ein wenig Linux trixerrei.
```bash
cat file.py | gcc -E - | python3 -
```

Damit können wir nun die volle Power des Präprozessor nutzen und z.B.: ein Programm wie folgt schreiben: 

```python
#define PRINT_PY() \
	print(3.1415)

PRINT_PY()
```

Ob dies nun in Python sonderlich nützlich ist, würde ich eher bezweifeln. Aber theoretisch klappt es auf jeden Fall und der ein oder andere ist sicher etwas kreativer als ich und findet sicherlich coole arten, dies zu nutzen.