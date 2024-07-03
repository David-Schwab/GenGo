
# Analyse zur Verwendung von Generics in Go

## Introduction
In der heutigen Zeit gewinnen generische Programmieransätze zunehmend an Bedeutung, insbesondere in stark typisierten Programmiersprachen. Seit Version 1.19 unterstützt die Programmiersprache Go Generics, eine lang erwartete Funktion, die es Entwicklern ermöglicht, flexibler und wiederverwendbarer zu programmieren. Diese Neuerung bringt jedoch nicht nur Vorteile, sondern auch Herausforderungen mit sich, insbesondere im Hinblick auf die praktische Umsetzung und Nutzung von Generics in realen Projekten.Bereits 24 Stunden nach der Veröffentlichung der Programmiersprache Go am 10. November 2009 wurde ein Kommentar zum Einsatz von Generics in der Golang-Mailingliste veröffentlicht. Es wurde behauptet, dass moderne Programmiersprachen Generics benötigen. Seitdem wurde viel über Generics für Go diskutiert und in den jährlich vom Go-Team durchgeführten Online-Umfragen zur Programmiersprache, den sogenannten User-Surveys (2016, 2017, 2018, 2019), wurden die Generics immer wieder als Wunschfeature der Entwickler aufgeführt. Im Jahr 2019 beantworteten 79% der Befragten zum Beispiel die Frage “Which critical language features do you need that are not available in Go?” mit: Generics. Schließlich werden seit Februar 2022 Generics von Go unterstützt.

Diese Fallstudie setzt sich zum Ziel, die Verwendung von Generics in Go-Programmen zu untersuchen und dabei etwaige Einschränkungen zu identifizieren. Dabei soll geklärt werden, welche und wie viele Go-Programme Generics tatsächlich nutzen und in welcher Form diese verwendet werden, sei es in generischen Strukturen, Funktionen oder anderen Konstrukten. Des Weiteren wird analysiert, welche praktischen Einschränkungen bei der Nutzung von Generics in Go bestehen, im Vergleich zur theoretischen Konzeption.
Um diese Ziele zu erreichen, wird eine systematische Analyse von Go-Programmen durchgeführt, die auf GitHub veröffentlicht sind. Hierfür kommt ein spezielles Tool zum Einsatz, das von einem Kommilitonen im Rahmen einer früheren Projektarbeit entwickelt wurde. Das Tool namens [goAnalyze](https://github.com/Mur-Ha/goAnalyze.git) ermöglicht es, Go-Programme nach der Verwendung von Generics zu durchsuchen und liefert somit eine solide Basis für die Datenerhebung und analyse dieser Fallstudie.



Diese Untersuchung trägt nicht nur zum besseren Verständnis der praktischen Anwendung von Generics in Go bei, sondern soll auch aufzeigen, in welchen Bereichen Verbesserungen und Weiterentwicklungen notwendig sind, um die Nutzung von Generics in der Go-Programmiersprache weiter zu optimieren.


# Einführung in die Untersuchung von Generics

Im ersten Abschnitt unserer Projektarbeit möchten wir für jeden Use Case von Generics, der in unserer Untersuchung beziehungsweise durch das Analyse-Tool GO ANALYZE betrachtet wird, ein klares und präzises Beispiel darstellen. Unser Ziel ist es, das Verständnis von Generics zu vertiefen, und um ebenfalls die Ergebnisse unserer Analyse mit den in den analysierten Go-Programmen von GitHub verwendeten Generics abgleichen zu können.

## Wir werden die folgenden Use Cases untersuchen und darstellen:

- 1. "Any" Type Bound: Die Verwendung des universellen Typs "any" als Typbeschränkung.
- 2. Generische Strukturen (Generic Structs)
- 3. Generische Methoden (Generic Methods)
- 4. Generische Funktionen (Generic Functions) mit weiteren Typbeschränkungen neben "any"
- 5. Verwendung von Typassertionen (Type Assertions)
- 6. Verwendung von Typesets als Typbeschränkungen


# "Any" Type Bound

Im ersten Use Case betrachten wir die Verwendung des universellen Typs "any" als Typbeschränkung. Dieser Ansatz ermöglicht es, eine Funktion so zu definieren, dass sie mit beliebigen Datentypen arbeiten kann. Ein anschauliches Beispiel hierfür ist die Funktion swap, die zwei Werte beliebigen Typs vertauscht.

```go
// Bound type parameter
func swap[T any](x *T, y *T) {
    tmp := *x
    *x = *y
    *y = tmp
}
```

In dieser Funktion wird T als Typparameter verwendet, der durch jeden beliebigen Typ ersetzt werden kann. Die Parameter x und y sind Zeiger auf den Typ T. Der Wert, auf den x zeigt, wird zunächst in der temporären Variablen tmp gespeichert. Anschließend wird der Wert, auf den y zeigt, dem Wert zugewiesen, auf den x zeigt. Zum Schluss wird der ursprünglich in tmp gespeicherte Wert in den Wert, auf den y zeigt, gespeichert.

## Eine Demonstration dieser Funktionalität erfolgt im testSwap-Beispiel:

```go
func testSwap() {
    x := 1
    y := 3

    fmt.Printf("\n x = %d, y = %d", x, y) // Ausgabe der ursprünglichen Werte von x und y

    swap[int](&x, &y) // Übergabe von x und y als Zeiger und Festlegung des Typs int

    fmt.Printf("\n x = %d, y = %d", x, y) // Ausgabe der vertauschten Werte von x und y
}
```

In diesem Test wird die swap-Funktion mit zwei Ganzzahlen (int) verwendet. Die Variablen x und y werden als Zeiger übergeben, was bedeutet, dass die Adressen, an denen x und y gespeichert sind, übergeben werden. Durch die Angabe von int als Typparameter wird festgelegt, dass die Funktion nur mit int-Werten ausgeführt wird. Die initialen Werte von x und y werden ausgegeben, die swap-Funktion vertauscht die Werte, und schließlich werden die neuen Werte von x und y ausgegeben.

Die Typbeschränkung any stellt dabei keine speziellen Anforderungen an die Typen, sodass die Funktion mit jedem Typ verwendet werden kann, was die Flexibilität und Wiederverwendbarkeit des Codes erhöht. In Go müssen Typparameter immer eine Typbeschränkung haben, und any ist die allgemeinste Form einer solchen Beschränkung.


# Generische Strukturen (Generic Structs)

Generische Strukturen in Go bieten eine flexible Möglichkeit, Datenstrukturen zu erstellen, die mit unterschiedlichen Datentypen arbeiten können. Ein anschauliches Beispiel dafür ist die Implementierung eines Interfaces und dessen Nutzung in generischen Funktionen.

In Go werden Interfaces verwendet, um eine Menge von Methoden zu definieren, die ein Typ implementieren muss. 

## Betrachten wir das folgende Interface Show:

```go
type Show interface {
    show() string
}
```

Das Show Interface verlangt, dass jeder Typ, der dieses Interface implementiert, eine Methode show definiert, die einen String zurückgibt.

## Nun implementieren wir dieses Interface in zwei verschiedenen Strukturen (boolean und number):

```go
type boolean struct {
    val bool
}

func (this boolean) show() string {
    if this.val {
        return "true"
    } else {
        return "false"
    }
}
```

Die Struktur boolean hat ein Feld val vom Typ bool. Die Methode show prüft den Wert von val und gibt entsprechend "true" oder "false" zurück. Da boolean die Methode show definiert, implementiert sie das Show Interface.

```go
type number struct {
    val int
}

func (this number) show() string {
    return strconv.Itoa(this.val)
}
```

Die Struktur number hat ein Feld val vom Typ int. Die Methode show konvertiert diesen ganzzahligen Wert in einen String mithilfe der Funktion strconv.Itoa und gibt ihn zurück. Auch hier wird das Show Interface implementiert.

- Nun betrachten wir zwei Funktionen, die das Show Interface verwenden. Die erste Funktion, showTwice1, nimmt zwei Parameter vom Typ Show und ruft die Methode show auf beiden auf, um das Ergebnis als String zurückzugeben:

```go
func showTwice1(x Show, y Show) {
    fmt.Printf("\n %s", x.show()+y.show())
}
```

- Die zweite Funktion, showTwice2, ist eine generische Funktion, die zwei Parameter desselben Typs erwartet, der das Show Interface implementiert:

```go
func showTwice2[T Show](x T, y T) {
    fmt.Printf("\n %s", x.show()+y.show())
}
```

## Diese Funktionen können wie folgt getestet werden:

```go
func testShow() {
    n := number{1}
    b := boolean{true}

    // showTwice1 kann verschiedene Typen akzeptieren, solange sie das Show Interface implementieren
    showTwice1(n, b)
    showTwice1(n, n)
    showTwice1(b, b)

    // showTwice2 erwartet, dass beide Parameter denselben Typ haben
    // showTwice2(n, b) würde nicht akzeptiert werden
    showTwice2(n, n) // ohne Instanziierung muss der Compiler den Typen "erraten"
    showTwice2[boolean](b, b) // explizite Instanziierung
}
```

In diesem Beispiel zeigt showTwice1, dass es mit unterschiedlichen Typen funktioniert, solange beide den Show Interface implementieren. showTwice2 hingegen verlangt, dass beide Parameter denselben Typ haben, der das Show Interface implementiert. Dieser Mechanismus wird als strukturelle Subtypisierung bezeichnet. In Go bedeutet das, dass ein Typ als Subtyp eines Interfaces betrachtet wird, wenn er alle Methoden dieses Interfaces implementiert. Dies geschieht automatisch basierend auf der Methodensignatur, ohne dass eine explizite Deklaration erforderlich ist.
Die Strukturen ermöglichen also die Wiederverwendbarkeit, wie im Fall der showTwice1 und showTwice2 Funktionen. Mit showTwice1 können wir verschiedene Typen kombinieren, die das Show Interface implementieren, ohne dass die Funktion für jeden spezifischen Typ neu geschrieben werden muss.Außerdem wird von der Funktion showTwice2 Typensicherheit garantiert. Es wird sichergestellt, dass beide Parameter denselben Typ haben, was vom Compiler zur Kompilierungszeit überprüft wird.

Insgesamt zeigt das Beispiel auch die Flexibilität von generischen Strukturen. Sie ermöglichen es uns, eine allgemeine Funktion zu schreiben, die mit verschiedenen Typen arbeiten kann, solange diese Typen das erforderliche Interface implementieren. Dies macht den Code anpassungsfähiger und erleichtert die Erweiterung um neue Typen, wie beispielsweise das Hinzufügen weiterer Strukturen.


## Generische Methoden und Funktionen (Generic Methods and Functions)

In diesem Abschnitt betrachten wir die Verwendung von generischen Methoden und Funktionen anhand eines Beispiels. Generische Methoden und Funktionen ermöglichen es, Code zu schreiben, der mit verschiedenen Datentypen funktioniert, ohne auf bestimmte Typen beschränkt zu sein. Sie bieten Flexibilität und Wiederverwendbarkeit, was besonders bei komplexeren Datenstrukturen und Algorithmen von Vorteil ist.

- Betrachten wir zunächst die Definition eines generischen Typs pair, der zwei Typ-Parameter T und S akzeptiert. Beide Typ-Parameter müssen das Interface Show implementieren, das eine Methode show definiert, die einen String zurückgibt:

```go
type Show interface {
    show() string
}

type boolean struct {
    val bool
}

func (this boolean) show() string {
    if this.val {
        return "true"
    } else {
        return "false"
    }
}

type pair[T Show, S Show] struct {
    left  T
    right S
}

func (this pair[T, S]) show() string {
    return "(" + this.left.show() + "," + this.right.show() + ")"
}
```

Die Struktur pair hat zwei Felder left und right, die jeweils Typen T und S sind. Diese Typen müssen das Show Interface implementieren, was bedeutet, dass sie eine show Methode haben, die einen String zurückgibt. Die Methode show der Struktur pair kombiniert die show Methoden der beiden Felder zu einem einzigen String.

## Ein Beispiel für die Verwendung dieser Struktur ist wie folgt:

```go
func testShow() {
    p := pair[boolean, boolean]{boolean{true}, boolean{false}}
    fmt.Printf("\n %s", p.show())
}
```

In diesem Beispiel erstellen wir eine Instanz von pair mit zwei boolean Typen. Die Methode show der Struktur pair ruft die show Methoden der beiden boolean Werte auf und gibt das Ergebnis als String aus.

- Nun betrachten wir eine alternative Definition der Struktur ppair, bei der die Typ-Parameter T und S keine Einschränkungen haben. Die Einschränkungen werden stattdessen nur in der Methode show definiert:

```go
type ppair[T any, S any] struct {
    left T
    right S
}

func (this ppair[T Show, S Show]) show() string {
    return "(" + this.left.show() + "," + this.right.show() + ")"
}
```

Hier können die Typen T und S beliebige Typen sein, aber die Methode show erfordert, dass T und S das Show Interface implementieren. Diese flexiblere Methode wird jedoch von Go 1.18 und 1.19 nicht unterstützt, was bedeutet, dass der Compiler die Methodendefinition im zweiten Beispiel ablehnt. In diesen Go-Versionen müssen Einschränkungen auf Typ-Parametern auf der Ebene des Typs definiert werden, wenn sie in Methoden verwendet werden sollen. Dies führt zu einer strikteren Typprüfung und verhindert die flexible Definition von Einschränkungen nur auf Methodenebene.

- Stattdessen können wir also eine Funktion showFunc definieren, die die Typ-Einschränkungen in der Funktionssignatur spezifiziert:

```go
func showFunc[T Show, S Show](this ppair[T, S]) string {
    return "(" + this.left.show() + "," + this.right.show() + ")"
}

func testShowFunc() {
    p := ppair[boolean, boolean]{boolean{true}, boolean{false}}
    fmt.Printf("\n %s", showFunc[boolean, boolean](p))
}
```

Hier zeigt showFunc, wie Typ-Einschränkungen in der Funktionssignatur definiert werden können, um sicherzustellen, dass die Typen T und S das Show Interface implementieren. Diese Funktionalität ermöglicht es, flexibelere generische Funktionen zu schreiben, die Typ-Einschränkungen nur dann anwenden, wenn sie tatsächlich benötigt werden.



# Type Assertions

In diesem Abschnitt gehen wir auf die Verwendung von Type Assertions in Go ein, um die Typensicherheit zur Laufzeit zu gewährleisten. Type Assertions sind ein mächtiges Werkzeug, um dynamische Typen zu überprüfen und sicherzustellen, dass ein Interface tatsächlich einen bestimmten konkreten Typ implementiert.

- Betrachten wir zunächst eine einfache Funktion assert1, die eine Type Assertion verwendet:

```go
func assert1(x any) {
    y := x.(boolean)
    fmt.Printf("\n%s", y.show())
}
```

Die Funktion assert1 akzeptiert ein Argument x vom Typ any, was bedeutet, dass x jeden Typ annehmen kann. Die Anweisung y := x.(boolean) versucht, x in den Typ boolean zu konvertieren. Wenn x tatsächlich vom Typ boolean ist, wird die Konvertierung erfolgreich sein und der Wert wird der Variablen y zugewiesen. Anschließend wird die Methode show auf y aufgerufen und das Ergebnis wird ausgegeben. Falls x nicht vom Typ boolean ist, wird zur Laufzeit ein Fehler ausgelöst.

```go
func assert1b(x Show) {
    y := x.(pair[boolean, boolean])
    fmt.Printf("\n%s", y.show())
}
```

Hier akzeptiert assert1b ein Argument x, das das Show Interface implementiert. Es wird versucht, x in den Typ pair[boolean, boolean] zu konvertieren. Dies funktioniert nur, wenn x tatsächlich eine Instanz von pair[boolean, boolean] ist.

## Beispiel für die Verwendung dieser Funktion :

```go
func testAssert() {
    assert1(boolean{true}) // Erfolgreiche Assertion
    // assert1(1) 
    Hier wird assert1 mit einem boolean Wert aufgerufen, was erfolgreich ist. Der Versuch, assert1 mit einem int Wert aufzurufen, würde jedoch einen Laufzeitfehler auslösen.t, dass sie in der run-time failt

    assert1b(pair[boolean, boolean]{boolean{false}, boolean{true}}) // Erfolgreiche Assertion
    // assert1b(boolean{false}) würde bei run-time failen, weil x.pair 2 elemente erwartet
}
```

## Einschränkungen und Fehler bei Type Assertions

Ein häufiges Problem tritt auf, wenn versucht wird, Type Assertions auf generische Typen anzuwenden. 

- Betrachten wir die Funktion assert2:

```go
func assert2[T any](x T) {
    y := x.(boolean)
    fmt.Printf("\n%s", y.show())
}
```

Diese Funktion ist generisch und akzeptiert ein Argument x vom Typ T, wobei T ein beliebiger Typ ist.
Allerdings wird die folgende Fehlermeldung ausgegeben: "invalid operation: cannot use type assertion on type parameter value x (variable of type T constrained by any)". 
Dies bedeutet, dass die Type Assertion auf einen Wert des Typs T, der durch any eingeschränkt ist, nicht angewendet werden kann. 
Go erlaubt keine Type Assertions auf Typ-Parameter, weil der Compiler zur Kompilierungszeit nicht weiß, welcher konkrete Typ für T eingesetzt wird.

- Und folgendes wird wie bei assert2 auch nicht akzeptiert aus selbem Grund:

```go
func assert2b[T Show](x T) {
        y := x.(pair[boolean,boolean])
    fmt.Printf("\n%s",y.show())
}
```

- Diese Funktion akzeptiert ein Argument x vom generischen Typ T, wobei T das Show Interface implementiert.

```go
func assert3[T Show](x pair[T,T]) {
        y := x.(pair[boolean,boolean])
    fmt.Printf("\n%s",y.show())
}
```

Die Funktion assert3 akzeptiert ein Argument x vom Typ pair[T, T], wobei T das Show Interface implementiert.
Auch hier versucht die Anweisung y := x.(pair[boolean, boolean]), x in den Typ pair[boolean, boolean] zu konvertieren.
Der Compiler lehnt die Funktionen assert2b und assert3 mit der selben Fehlermeldung ab.


# Type Sets

Type Sets sind eine Funktion in Go, die es ermöglicht, generische Funktionen mit eingeschränkten Typen zu definieren. Sie sind nützlich, um Funktionen zu erstellen, die nur mit bestimmten Typen arbeiten können. Type Sets werden durch die Verwendung des |-Operators definiert, um eine Liste von erlaubten Typen anzugeben.


```go
func plus3[T int | float32 | float64](x T, y T, z T) T {
    return x + y + z
}
```

Die Funktion plus3 ist generisch und akzeptiert drei Parameter vom gleichen Typ T, wobei T auf die Typen int, float32 oder float64 eingeschränkt ist. Diese Einschränkung wird durch ein Type Set realisiert. Das bedeutet, dass plus3 nur mit den angegebenen Typen verwendet werden kann. Dies stellt sicher, dass die Addition der Parameter x, y und z für alle zulässigen Typen gültig ist.

## Beispiel:

```go
type onOff struct {
    val bool
}

func (this onOff) show() string {
    if this.val {
        return "On"
    } else {
        return "Off"
    }
}

func showOnlyOnOffandPair[T boolean | onOff](x T) string {
    return x.show()
}
```

Die Funktion showOnlyOnOffandPair versucht, eine Typset-Einschränkung zu verwenden, um anzugeben, dass T entweder boolean oder onOff sein kann. Der Zweck der Funktion besteht darin, die show-Methode des übergebenen Typs T aufzurufen. Allerdings erlaubt Go die Verwendung von Typsets nur mit eingebauten primitiven Typen und nicht mit benutzerdefinierten Typen wie onOff. Daher wird diese Funktion abgelehnt und der Code wird nicht kompiliert.

## Testfunktion für Type Sets

```go
func testTypeSets() {
    fmt.Println(plus3(1, 2, 3))               // int
    fmt.Println(plus3(1.1, 2.2, 3.3))         // float32
    fmt.Println(plus3(1.1, 2.2, 3.3))         // float64

    // showOnlyOnOffandPair(boolean{true})     // Compilerfehler
    // showOnlyOnOffandPair(onOff{true})       // Compilerfehler
}
```

In testTypeSets wird plus3 mit int, float32 und float64 aufgerufen, was korrekt ist und keine Fehler verursacht. Der Versuch, showOnlyOnOffandPair mit boolean oder onOff aufzurufen, führt jedoch zu einem Compilerfehler, da benutzerdefinierte Typen in Typsets nicht verwendet werden können.


## Beschreibung der Verwendung des Tools

Im Folgenden wird der Ablauf zur Verwendung des Tools sowie die erzielten Ergebnisse beschrieben.

- Zuerst muss das Tool aus dem Verzeichnis aggregation im usage interactive Modus gestartet werden:

```go
C:\....\GoAnalyse\goAnalyze\aggregation> go run aggregation.go -usage interactive
```

- Der interaktive Modus startet und fordert zur Eingabe von Befehlen auf:

```go
Interactive mode...
Please enter commands:
(q or quit will exit this mode and end the program)
```

- Um eine Liste der bereits geladenen Go-Projekte von GitHub anzuzeigen, verwenden Sie den Befehl listProjects:

```go
2. > listProjects
```

- Da bisher keine Projekte geladen wurden, bleibt die Liste leer:

```go
Projects: -
```

- Die 10 beliebtesten Go-Projekte von GitHub können mit dem Befehl `getNewProjects 10` geladen werden:

```go
3.> getNewProjects 10
```

- Die Anfrage wird an die GitHub API gesendet:

```go
request: https://api.github.com/search/repositories?q=language:go&per_page=10&page=0
```

- Nach dem Laden der Projekte kann die aktualisierte Liste erneut mit listProjects angezeigt werden:

```go
4. listProjects
```

- Die Ausgabe zeigt die 10 neu geladenen Projekte:

```go
Projects:
- avelino/awesome-go [cloned: false]
- golang/go [cloned: false]
- kubernetes/kubernetes [cloned: false]
- fatedier/frp [cloned: false]
- gin-gonic/gin [cloned: false]
- gohugoio/hugo [cloned: false]
- ollama/ollama [cloned: false]
- moby/moby [cloned: false]
- junegunn/fzf [cloned: false]
- syncthing/syncthing [cloned: false]
```


- Um eines der Projekte zu laden, wird der Befehl `loadProject` gefolgt vom Projektnamen verwendet:

```go
5. > loadProject gin-gonic/gin
```

- Das Projekt wird aus dem angegebenen Repository heruntergeladen:

```go
https://github.com/gin-gonic/gin.git

Enumerating objects: 7734, done.
Counting objects: 100% (1297/1297), done.
Compressing objects: 100% (183/183), done.
Total 7734 (delta 1185), reused 1160 (delta 1113), pack-reused 6437
Download done!
gin-gonic/gin is now loaded and ready to be processed.
```

- Nun kann das gewählte Projekt, in diesem Fall gin-gonic/gin, mit dem Befehl processProject analysiert werden:

```go
6. processProject gin-gonic/gin
```

- Die Analyse wird durchgeführt und die Ergebnisse werden angezeigt:

```go
../analyzeTarget/gin-gonic/gin
full: 167, filtered: 91
--- ../analyzeTarget/gin-gonic/gin/doc.go ----
UsesGenerics: false
UsesTypeSet: false
UsesTypeAssertions: false
TypeNames: &{map[]}
TypeAlias:
map[]
90
--- ../analyzeTarget/gin-gonic/gin/version.go ----
UsesGenerics: false
UsesTypeSet: false
UsesTypeAssertions: false
TypeNames: &{map[]}
TypeAlias:
map[]
89
...
All results processed
```

- Um die Ergebnisse der Analyse übersichtlich anzuzeigen, wird der Befehl `evalResult` verwendet:

```go
7. > evalResult gin-gonic/gin
```

- Die Auswertung zeigt eine detaillierte Auflistung der verwendeten Generics und Typ-Assertions:


```go
Used 43 generics
The following Generics were used:
Most Certain Hits:

Uncertain Hits:
  - pos: 2
  - v.Key.: 1
  - idx: 2
  - i: 16
  - param.Key.: 1
  - newPos: 2
  - j: 1
  - end: 2
  - tagValue: 1
  - n: 1
  - path: 2
  - w: 4
  - r: 6
  - k: 2
The 'any' typebound was not used as TypeBound.
Lines using Generics:
  - ../analyzeTarget/gin-gonic/gin/internal/bytesconv/bytesconv_test.go: [56]
  - ../analyzeTarget/gin-gonic/gin/githubapi_test.go: [378]
  - ../analyzeTarget/gin-gonic/gin/context.go: [761 1215 1215 1218 1218]
  - ../analyzeTarget/gin-gonic/gin/binding/default_validator.go: [31 35]
  - ../analyzeTarget/gin-gonic/gin/tree.go: [78 78 130 131 137 137 367 498 774 836]
  - ../analyzeTarget/gin-gonic/gin/path.go: [62 66 70 74 83 87 102 103 133 149]
  - ../analyzeTarget/gin-gonic/gin/binding/default_validator_benchmark_test.go: [17]
  - ../analyzeTarget/gin-gonic/gin/recovery.go: [79 143]
  - ../analyzeTarget/gin-gonic/gin/gin.go: [462 639 642]
  - ../analyzeTarget/gin-gonic/gin/binding/form_mapping.go: [186 424 435]
  - ../analyzeTarget/gin-gonic/gin/errors.go: [138 152]
  - ../analyzeTarget/gin-gonic/gin/logger.go: [238 252]
  - ../analyzeTarget/gin-gonic/gin/utils.go: [159]

No TypeSets used
Used 64 TypeAssertions
The following TypeAssertions were used:
  - TypeCTX::GetText:map[string]any: 1
  - TypeName::IDENTIFIER:string: 16
  - TypeName::IDENTIFIER:int: 3
  - TypeCTX::GetText:[]string: 1
  - TypeName::GetText:http.Hijacker: 1
  - TypeCTX::GetText:*bindTestStruct: 1
  - TypeCTX::GetText:*Test_OptionalGroup: 1
  - TypeName::IDENTIFIER:float64: 6
  - TypeName::IDENTIFIER:uint64: 2
  - TypeName::IDENTIFIER:neutralizedReaddirFile: 1
  - TypeName::GetText:http.Flusher: 1
  - TypeCTX::GetText:[]byte: 2
  - TypeCTX::GetText:map[string]string: 2
  - TypeName::GetText:json.Number: 1
  - TypeCTX::GetText:*responseWriter: 2
  - TypeName::GetText:http.Pusher: 1
  - TypeCTX::GetText:*Context: 1
  - TypeName::IDENTIFIER:bool: 1
  - TypeCTX::GetText:*validator.Validate: 2
  - TypeName::GetText:http.CloseNotifier: 1
  - TypeCTX::GetText:*structModifyValidation: 1
  - TypeCTX::GetText:*os.File: 1
  - TypeCTX::GetText:*net.OpError: 1
  - TypeName::IDENTIFIER:error: 1
  - TypeName::IDENTIFIER:int64: 2
  - TypeCTX::GetText:map[string][]string: 2
  - TypeName::GetText:time.Duration: 1
  - TypeName::GetText:time.Time: 1
  - TypeName::IDENTIFIER:uint: 1
  - TypeCTX::GetText:*OnlyFilesFS: 1
  - TypeName::IDENTIFIER:float32: 1
  - TypeName::IDENTIFIER:int32: 1
  - TypeCTX::GetText:*Test: 1
  - TypeName::GetText:proto.Message: 2
Lines using TypeAssertions are:
  - ../analyzeTarget/gin-gonic/gin/context.go: [297 305 313 321 329 337 345 353 361 369 377 385 393 780 1280]
  - ../analyzeTarget/gin-gonic/gin/mode.go: [97]
  - ../analyzeTarget/gin-gonic/gin/tree_test.go: [549 883 891]
  - ../analyzeTarget/gin-gonic/gin/response_writer.go: [112 117 123 127]
  - ../analyzeTarget/gin-gonic/gin/binding/validate_test.go: [200 207 227 237]
  - ../analyzeTarget/gin-gonic/gin/binding/protobuf.go: [30]
  - ../analyzeTarget/gin-gonic/gin/binding/binding_test.go: [641 695 696 697 1126 1260]
  - ../analyzeTarget/gin-gonic/gin/context_test.go: [182 233 234 235 236 237 238 239 345]
  - ../analyzeTarget/gin-gonic/gin/recovery_test.go: [151 186 221]
  - ../analyzeTarget/gin-gonic/gin/auth_test.go: [89 108 128 146 165]
  - ../analyzeTarget/gin-gonic/gin/binding/plain.go: [50]
  - ../analyzeTarget/gin-gonic/gin/fs_test.go: [32]
  - ../analyzeTarget/gin-gonic/gin/logger.go: [227]
  - ../analyzeTarget/gin-gonic/gin/routergroup.go: [221]
  - ../analyzeTarget/gin-gonic/gin/gin.go: [602]
  - ../analyzeTarget/gin-gonic/gin/utils_test.go: [120]
  - ../analyzeTarget/gin-gonic/gin/binding/form_mapping.go: [419 430]
  - ../analyzeTarget/gin-gonic/gin/render/protobuf.go: [24]
  - ../analyzeTarget/gin-gonic/gin/testdata/protoexample/test.pb.go: [255 267]
  - ../analyzeTarget/gin-gonic/gin/recovery.go: [62 95]
  ```

# Zusammenfassung der Analyseergebnisse

## Defintion der Ergebnisse:

- Most Certain Hits: Dies sind Treffer, die eindeutig als Generics identifiziert wurden.

- Uncertain Hits: Diese Treffer sehen wie Generics aus, sind aber keine beziehungsweise sind nicht als solche idenifizierbar. 
Dazu gehören z.B. Aufrufe von Maps, Arrays oder Datentypen aus anderen Bibliotheken, deren interne Struktur nicht nachvollzogen werden kann.

- Jede Verwendung von Generics oder Type Assertions wird detailliert mit Datei- und Zeilenangaben dokumentiert

# Verfahren zur Identifikation von Generics in Go

## Syntax-Analyse mit ANTLR4

Das zu analysierende Projekt wird einer Syntax-Analyse mithilfe von ANTLR4 unterzogen. ANTLR parst den Code gemäß einer definierten Grammatik in einen Abstract Syntax Tree (AST), der aus Tokens besteht. Diese Tokens repräsentieren bestimmte Elemente im Code. ANTLR selbst macht dabei keine finale Klassifizierung, sondern liefert die Vorverarbeitung. Das Tool, das darauf aufbaut, arbeitet dann mit diesen Tokens, insbesondere mit den Identifiern bzw. Typen innerhalb von eckigen Klammern, die hier Namen für beispielsweise Typepounds oder Typinstanziierungen im Zusammenhang mit Generics sein könnten. Daraus werden dann potenzielle Generics identifiziert.

Generics können dabei an unterschiedlichen Stellen verwendet werden:

- Deklaration von Funktionen:

```go
func a[T any]() { ... } //Der Name "any" als Typebound
```

Selbiges gilt auch für Structs oder beim Aufrufen einer generischen Methode:

```go
showTwice2[boolean](b, b)//Der Name "boolean" als Typinstanziierung für showTwice2
```

Ein besonderes Problem bei der Identifikation von Generics ist die Unterscheidung zwischen Typen für Generics und Schlüsseln für Maps. Betrachten wir folgende Code-Beispiele:

- Code A:


```go
func main() {
  c := make(map[int]func())
  x := 0
  c[x]() //A
}
```

- Code B:


```go
func c[T any](){

}

func main() {
  c[int]() //B
}
```

Beide Code-Beispiele resultieren in den selben Tokens:



Es wird folgendermaßen vorgegangen, um dann zu entscheiden, ob es sich bei "A" und "B" jeweils um "most certain" also wahrscheinliche und "uncertain", also unsichere hits für Generics handelt:


1. Aufbau einer Typenliste

Das Tool baut eine Liste aufm, welche enthält:

- Projektdefinierte Typen: Alle definierten Typen des betrachteten Projekts.
- Standard-Basistypen: grundlegenden Datentypen wie z.B. (float, int, string, bool)

2. Kategorisierung der Typen

- Sicher erkannte Generics (Most Certain Hits):

Wenn der Typname oder Identifier innerhalb der eckigen Klammern bei "//B" in der aufgebauten Liste enthalten ist, wird er als "most certain" eingestuft. Da "int" in der Liste der Basistypen ist, trifft das zu.

- Beispiel 1: 

```go
func c[T any](){

}

func main() {
  c[int]() //B
}
```

- Beispiel 2: Projektspezifische Typen

Angenommen, im Projekt wurde ein Typ Show definiert:

```go
type Show interface {
    show() string
}


func showTwice2[T Show](x T, y T) {
    fmt.Printf("\n %s", x.show()+y.show())
}
```
Show wird als potenzieller Generic identifiziert, welcher nach dem Abgleich mit der aufgebauten Liste als most certain hit eingestuft wird, da Show im Projekt definiert ist und definierte Typen in der Liste enthalten sind.

Im Grunde werden alle anderen Fälle als uncertain hits eingestuft.

- Unsicher erkannte Generics (Uncertain Hits):
Wenn der Typname nicht in der Liste enthalten ist, bleibt die Zuordnung unsicher. Dies gilt insbesondere für mögliche Map-Zugriffe (z.B. map[x][y]), bei denen x und y oft keine generischen Typen, sondern Schlüssel- und Indexwerte sind.

- Beispiel 1:

```go
func main() {
  c := make(map[int]func())
  x := 0
  c[x]() //A
}
```
In diesem Beispiel könnte es sich bei "//A"" um einen Generic handeln, da aber "x" nicht in der Liste der bekannten Typen enthalten ist, und auch nicht als Typ vorher definiert wurde, bleibt die Zuordnung unsicher. Offensichtlich handelt es sich hier auch nicht um einen Generic sondern um einen Map Aufruf.

- Beispiel 2:

```go
m := map[string]map[int]string{
    "outer": {1: "value1", 2: "value2"},
}
value := m["outer"][1]
```

Hier könnten outer und 1 als potenzielle generische Typen erkannt werden, sind jedoch in diesem Kontext Zugriffe auf eine Map. Da diese Namen "outer" und "1" nicht in der Liste mit bekannten Typen enthalten sind, bleibt die Zuordnung unsicher("uncertain").

## Begründung für die Sicherheit bei der Identifikation von Generics
Die Sicherheit, dass ein Typname in eckigen Klammern wahrscheinlich bzw. "most certain" ein Generic ist, basiert auf mehreren Faktoren:

- Keywords und Typnamen als Variablennamen:
Generische Typen tauchen typischerweise in Deklarationen und Instanziierungen auf, die innerhalb von eckigen Klammern notiert sind. In Go können bestimmte Schlüsselwörter und Typnamen nicht als Variablennamen verwendet werden. Dies gilt sowohl für Sprachschlüsselwörter wie if oder for als auch für Basistypen wie int, float, string und bool. Diese Einschränkungen helfen dabei, generische Typen sicherer zu identifizieren, da diese Namen in eckigen Klammern sehr wahrscheinlich keine Variablennamen sind.

- Eindeutigkeit der Typen:
Basistypen und projektspezifische Typen sind eindeutig und werden in der Liste der bekannten Typen geführt. Wenn ein Typname in den eckigen Klammern mit einem dieser bekannten Typen übereinstimmt, ist es sehr wahrscheinlich, dass es sich um einen Generic handelt.

4. Manuelle Überprüfung von Hits
Sowohl "most certain hits" als auch "uncertain hits" sollten manuell überprüft werden, deshalb werden sie entsprechend auch als: "es sind wahrscheinlich Generics" oder "es sind unwahrscheinlich Generics" definiert:

- Most Certain Hits:
Es könnten false positives enthalten sein, daher ist eine Überprüfung notwendig.

- Uncertain Hits:
Es könnten false negatives enthalten sein. Diese Hits können durchaus sinnvolle und relevante Verwendungen von Generics darstellen. Es macht Sinn, diese zu überprüfen, um festzustellen, ob sie plausible Typebounds oder generische Instanziierungen sein könnten, insbesondere wenn sie aus anderen Bibliotheken stammen.

- Nach manueller Überprüfung und im Rahmen dieser Fallstudie wird davon ausgegangen, dass das Tool immer Recht hat und "most certain hits" eindeutige Treffer für Generics darstellen, und "uncertain hits" keine Treffer, also keine Generics sind. Dies dient dazu, um eine bessere Aussage über die allgemeine Verwendung von Generics in Go treffen zu können.

## Die Analyse ergab folgende Ergebnisse für das Projekt gin-gonic/gin:

- Release von gin-gonic: Mai 2015
- Letzte Änderungen: <1 Monat
- Platzierung: Platz 5 der beliebtesten Go-Projekte

Es wurden 43 Generics verwendet, jedoch alle als unsicher (uncertain hits) eingestuft, und keine als sicher (most certain hits) identifiziert.

- Most Certain Hits: Keine eindeutigen Treffer als Generics identifiziert.
- Uncertain Hits: Alle 43 Treffer.
- Verwendung von "any" und Typesets: Keine Verwendung festgestellt.
- Type Assertions: Insgesamt 64 Type Assertions wurden verwendet.



## Ergebnisse für das Projekt junegunn/fzf:

- Release von Junegunn: Oktober 2020
- Letzte Änderungen: <1 Monat
- Platzierung: Platz 9 der beliebtesten Go-Projekte

```go
> evalResult junegunn/fzf
Used 185 generics
The following Generics where used:
Most Certain Hits:

Uncertain Hits:
  - math.MaxUint16.: 1
  - char: 2
  - idx: 34
  - prevClass: 2
  - pidx: 3
  - event: 5
  - off: 18
  - cursor: 1
  - j: 14
  - minIdx: 2
  - asString: 2
  - end: 1
  - offset: 3
  - e: 5
  - pidx_: 2
  - r: 3
  - k: 1
  - idx2: 7
  - mg.cursors.: 1
  - i: 66
  - idx1: 7
  - partialResult.index.: 1
  - key: 4
The 'any' typebound was not used as TypeBound.
Lines using Generics:
  - ../analyzeTarget/junegunn/fzf/src/ansi.go: [95 98 101 114 144 156 306]
  - ../analyzeTarget/junegunn/fzf/src/options_test.go: [143 144 205 206 251 253 256 257 258 261 262]
  - ../analyzeTarget/junegunn/fzf/src/terminal_test.go: [524]
  - ../analyzeTarget/junegunn/fzf/src/tokenizer.go: [94 95 115 246 250]
  - ../analyzeTarget/junegunn/fzf/src/tui/light.go: [608]
  - ../analyzeTarget/junegunn/fzf/src/pattern.go: [88 153]
  - ../analyzeTarget/junegunn/fzf/src/merger.go: [133 155 165 165]
  - ../analyzeTarget/junegunn/fzf/src/algo/normalize.go: [486 488 488]
  - ../analyzeTarget/junegunn/fzf/src/util/util_windows.go: [119]
  - ../analyzeTarget/junegunn/fzf/src/result_test.go: [136]
  - ../analyzeTarget/junegunn/fzf/src/tui/eventtype_string.go: [121]
  - ../analyzeTarget/junegunn/fzf/src/options.go: [609 615 642 653 675 1277 1466 1526 1526 1888 1938 2851]
  - ../analyzeTarget/junegunn/fzf/src/proxy.go: [99]
  - ../analyzeTarget/junegunn/fzf/src/result.go: [134 142 143 143 226 226 226 226 230 231 243 243 243 243 247 247 258 258 258 258 262 262]
  - ../analyzeTarget/junegunn/fzf/src/chunklist.go: [102 110]
  - ../analyzeTarget/junegunn/fzf/src/merger_test.go: [40 43 46 64 77 78 85 86]
  - ../analyzeTarget/junegunn/fzf/src/actiontype_string.go: [133]
  - ../analyzeTarget/junegunn/fzf/src/algo/algo_test.go: [198 200]
  - ../analyzeTarget/junegunn/fzf/src/algo/algo.go: [210 214 263 310 369 389 389 404 408 419 480 483 493 496 497 502 511 512 522 524 526 529 558 575 577 581 582 583 594 600 607 628 631 674 679 772 834]
  - ../analyzeTarget/junegunn/fzf/src/ansi_test.go: [45 45 136 136 142 144 179]
  - ../analyzeTarget/junegunn/fzf/src/util/util.go: [215 218]
  - ../analyzeTarget/junegunn/fzf/src/terminal.go: [360 360 360 360 364 364 574 585 757 1220 1349 1357 1359 1362 1366 1370 1374 1377 1379 1379 1386 1386 1386 1386 1389 1389 1390 1390 1391 1391 1391 1392 1392 1392 1479 2201 2318 2319 2351 2352 2973 3011 4161 4167 4908 4913]
  - ../analyzeTarget/junegunn/fzf/src/matcher.go: [145 187 237]
  - ../analyzeTarget/junegunn/fzf/src/util/chars.go: [30 35 40 56 56 116 206 217]
  - ../analyzeTarget/junegunn/fzf/src/util/atexit.go: [25]

No TypeSetss used
Used 12 TypeAssertions
The following TypeAssertions where used:
  - TypeCTX::GetText:*syscall.Stat_t: 1
  - TypeCTX::GetText:*exec.ExitError: 2
  - TypeCTX::GetText:*string: 1
  - TypeName::IDENTIFIER:quitSignal: 1
  - TypeName::IDENTIFIER:previewRequest: 1
  - TypeName::IDENTIFIER:previewResult: 1
  - TypeName::IDENTIFIER:int64: 1
  - TypeName::IDENTIFIER:string: 3
  - TypeCTX::GetText:[]string: 1
Lines using TypeAssertions are:
  - ../analyzeTarget/junegunn/fzf/src/proxy_windows.go: [18]
  - ../analyzeTarget/junegunn/fzf/src/core.go: [312 341 399]
  - ../analyzeTarget/junegunn/fzf/src/terminal.go: [3395 3678 3697]
  - ../analyzeTarget/junegunn/fzf/src/tui/ttyname_unix.go: [17 36]
  - ../analyzeTarget/junegunn/fzf/src/proxy.go: [116]
  - ../analyzeTarget/junegunn/fzf/src/util/util_windows.go: [66 107]
```
Es wurden 185 Generics verwendet, davon 0 als sicher (most certain hits) identifiziert und 185 als unsicher (uncertain hits).

- Most Certain Hits: Keine eindeutigen Treffer als Generics identifiziert.
- Uncertain Hits: Alle 185 Treffer
- Verwendung von "any" und Typesets: Keine Verwendung festgestellt.
- Type Assertions: Insgesamt 12 Type Assertions wurden verwendet.



## Ergebnisse für das Projekt ollama/ollama:

- Release von ollama/ollama: Juli 2023
- Letzte Änderungen: <1 Monat
- Platzierung: Platz 7 der beliebtesten Go-Projekte

```go
> evalResult ollama/ollama
Used 190 generics
The following Generics where used:
Most Certain Hits:
  - bool: 3
  - any: 3
  - string: 2

Uncertain Hits:
  - uint64: 6
  - layer.Digest.: 2
  - gpuZeroID: 1
  - key: 11
  - k: 12
  - v.ID.: 1
  - digest: 2
  - servers: 1
  - g.i.: 6
  - resp.Digest.: 6
  - n: 3
  - i: 84
  - uint32: 9
  - int64: 3
  - int8: 3
  - float64: 3
  - int32: 3
  - name: 1
  - l.Digest.: 1
  - uint16: 3
  - jsonTag: 2
  - demandLib: 1
  - endIdx: 1
  - v: 1
  - uint8: 3
  - int16: 3
  - float32: 3
  - j: 7
The 'any' typebound was used 3 times overall and represents 37.500000 percent of all type bounds
Generics where used 6 times to declare a function or struct in this scope
Lines using Generics:
  - ../analyzeTarget/ollama/ollama/integration/concurrency_test.go: [55 61 61 100 100 215 215 236 243 244 244]
  - ../analyzeTarget/ollama/ollama/llm/ggml.go: [24]
  - ../analyzeTarget/ollama/ollama/gpu/assets.go: [136]
  - ../analyzeTarget/ollama/ollama/app/tray/wintray/tray.go: [395 395]
  - ../analyzeTarget/ollama/ollama/server/manifest.go: [159]
  - ../analyzeTarget/ollama/ollama/cmd/cmd.go: [129 132 437 440 729 802 805]
  - ../analyzeTarget/ollama/ollama/api/types.go: [423 428 496 599 606 617 619 631 638 645 647 650]
  - ../analyzeTarget/ollama/ollama/gpu/amd_windows.go: [96 180 189 189 189 190]
  - ../analyzeTarget/ollama/ollama/server/routes_list_test.go: [54]
  - ../analyzeTarget/ollama/ollama/gpu/gpu.go: [389 405]
  - ../analyzeTarget/ollama/ollama/envconfig/config.go: [114]
  - ../analyzeTarget/ollama/ollama/server/routes.go: [766 767]
  - ../analyzeTarget/ollama/ollama/llm/gguf.go: [139 147 149 151 153 155 157 159 161 163 165 167 191 198 198 204 209 257 263 320 325 334 336 338 340 342 344 346 348 350 352 354 375 380 389 391 393 395 397 399 401 403 405 407 409 425 516 520 524]
  - ../analyzeTarget/ollama/ollama/gpu/amd_linux.go: [324 421 425 425 425 425 426 426]
  - ../analyzeTarget/ollama/ollama/llm/memory.go: [99 167 167 168 171 172 172 178 198 200 201 221 223 224 238 242 244 256 264]
  - ../analyzeTarget/ollama/ollama/convert/tokenizer.go: [78 83]
  - ../analyzeTarget/ollama/ollama/convert/torch.go: [84 84 214 257 257]
  - ../analyzeTarget/ollama/ollama/server/images.go: [343 483 484 486 488 520 521 565 727 840 872 879 1099]
  - ../analyzeTarget/ollama/ollama/llm/server.go: [138 250 250 253 258]
  - ../analyzeTarget/ollama/ollama/server/sched.go: [428 428 429 429 429 430 432 432 433 434 434 438 438 440 440 440 440 458 600 600 600 600 603 603]
  - ../analyzeTarget/ollama/ollama/server/prompt.go: [157 157]
  - ../analyzeTarget/ollama/ollama/types/model/name.go: [326 331 342]
  - ../analyzeTarget/ollama/ollama/gpu/types.go: [82 82 114 114 114 114 115 115]
  - ../analyzeTarget/ollama/ollama/convert/safetensors.go: [96 183 252 252]
  - ../analyzeTarget/ollama/ollama/server/model.go: [186]

No TypeSetss used
Used 67 TypeAssertions
The following TypeAssertions where used:
  - TypeCTX::GetText:*SelfTestData: 1
  - TypeName::IDENTIFIER:float64: 6
  - TypeCTX::GetText:*State: 1
  - TypeCTX::GetText:*TrainerSpec: 1
  - TypeCTX::GetText:*ModelProto: 1
  - TypeName::IDENTIFIER:uint32: 2
  - TypeCTX::GetText:*parse.FieldNode: 1
  - TypeName::IDENTIFIER:torchWriterTo: 1
  - TypeCTX::GetText:[]rune: 3
  - TypeCTX::GetText:*types.Dict: 2
  - TypeName::IDENTIFIER:string: 13
  - TypeCTX::GetText:[]string: 3
  - TypeCTX::GetText:*ModelProto_SentencePiece: 1
  - TypeCTX::GetText:*NormalizerSpec: 1
  - TypeCTX::GetText:*parse.ActionNode: 1
  - TypeName::IDENTIFIER:rune: 6
  - TypeCTX::GetText:*pytorch.Tensor: 2
  - TypeName::IDENTIFIER:safetensorWriterTo: 4
  - TypeCTX::GetText:*slog.Source: 2
  - TypeCTX::GetText:*SelfTestData_Sample: 1
  - TypeCTX::GetText:*net.TCPAddr: 2
  - TypeCTX::GetText:*Termios: 2
  - TypeCTX::GetText:*Spinner: 1
  - TypeName::IDENTIFIER:bool: 3
  - TypeCTX::GetText:[]interface{}: 1
  - TypeCTX::GetText:[]int: 1
  - TypeCTX::GetText:*blobUpload: 1
  - TypeCTX::GetText:*blobDownload: 1
  - TypeCTX::GetText:[]any: 2
Lines using TypeAssertions are:
  - ../analyzeTarget/ollama/ollama/convert/torch.go: [52 53 57 58 75 80 98]
  - ../analyzeTarget/ollama/ollama/progress/progress.go: [32]
  - ../analyzeTarget/ollama/ollama/server/images.go: [483 484 485]
  - ../analyzeTarget/ollama/ollama/readline/term.go: [29]
  - ../analyzeTarget/ollama/ollama/app/lifecycle/logging.go: [41]
  - ../analyzeTarget/ollama/ollama/convert/mistral.go: [29]
  - ../analyzeTarget/ollama/ollama/readline/readline_unix.go: [10]
  - ../analyzeTarget/ollama/ollama/llm/ggml.go: [37 109 324 344]
  - ../analyzeTarget/ollama/ollama/integration/utils_test.go: [39]
  - ../analyzeTarget/ollama/ollama/cmd/cmd.go: [655 661 662 673 674 675 676 992]
  - ../analyzeTarget/ollama/ollama/convert/llama.go: [40 44]
  - ../analyzeTarget/ollama/ollama/api/types.go: [441 465 472 478 485 492]
  - ../analyzeTarget/ollama/ollama/readline/history.go: [110 119 144]
  - ../analyzeTarget/ollama/ollama/openai/openai.go: [162]
  - ../analyzeTarget/ollama/ollama/readline/term_windows.go: [36]
  - ../analyzeTarget/ollama/ollama/readline/buffer.go: [54 61 111 186 261 353 544]
  - ../analyzeTarget/ollama/ollama/convert/sentencepiece/sentencepiece_model.pb.go: [1396 1410 1424 1438 1452 1464]
  - ../analyzeTarget/ollama/ollama/llm/server.go: [267]
  - ../analyzeTarget/ollama/ollama/llm/gguf.go: [228]
  - ../analyzeTarget/ollama/ollama/server/prompt.go: [17 34]
  - ../analyzeTarget/ollama/ollama/convert/gemma.go: [50]
  - ../analyzeTarget/ollama/ollama/convert/mixtral.go: [29]
  - ../analyzeTarget/ollama/ollama/server/upload.go: [385]
  - ../analyzeTarget/ollama/ollama/server/download.go: [366]
  - ../analyzeTarget/ollama/ollama/server/routes.go: [295 766 1066 1154 1404]
```

Es wurden 190 Generics verwendet, davon 8 als sicher (most certain hits) identifiziert und 182 als unsicher (uncertain hits).

- Most Certain Hits: 8 Treffer, darunter:
- bool: 3
- any: 3
- string: 2
- Uncertain Hits: 182 Treffer

Verwendung von "any": 3 Mal verwendet, was 37.5% der Type Bounds ausmacht.

- Type Assertions: Insgesamt 67 Type Assertions wurden verwendet.

# Zusammenfassung der Beobachtungen

Die drei betrachteten Projekte stammen aus unterschiedlichen Zeitperioden:

- gin-gonic: Veröffentlicht weit vor der Einführung von Generics in Go.
- junegunn/fzf: Veröffentlicht kurz nach der Einführung von Generics in Go.
- ollama/ollama: Veröffentlicht einige Zeit nach der Einführung von Generics in Go.

Es fällt auf, dass in den beiden älteren Projekten gin-gonic und junegunn/fzf keine sicheren Generics verwendet wurden. Im aktuellsten Projekt ollama/ollama werden hingegen einige Generics verwendet, jedoch sind diese immer noch nicht in großer Anzahl zu finden. Insgesamt bleibt die Verwendung von Generics in Go-Projekten gering. Ähnliche Trends wurden auch in anderen großen Go-Projekten beobachtet.


# Ergebnisse aus eigenen Recherchen und Diskussionen in Foren

- Einbezogene Foren: Reddit, Hackernews,Stack Overflow

## Positive Erfahrungen und Anwendungsbeispiele:

- Fehlerbehandlung und Standardwerte: Häufige Verwendungen von zwei Funktionen mit folgender Funktion:

```go
must[T any](t T, err error) T: // Gibt den Wert zurück oder löst bei einem Fehler einen Error aus.
zero[T any]() T: //Gibt den Nullwert für jeden Typ zurück.

func must[T any](t T, err error) T {
    if err != nil {
        panic(fmt.Errorf("error in must[%T]: got error: %w", t, err))
    }
    return t
}
```

Verwendung: Zum Erzwingen eines Wertes t, der aus einer Funktion oder einem Prozess zurückgegeben wird, und um sicherzustellen, dass kein Fehler aufgetreten ist.

```go
func zero[T any]() T {
    var t T
    return t
}
```
Verwendung: Zur Initialisierung von Variablen oder als Standardwert für Typen, insbesondere wenn der Nullwert des Typs verwendet werden soll.

## Praktische Anwendungen
- Datenbank- und ORM-Generierung: Query Builder und ORM-Generator, um die Duplizierung von Abfragen für verschiedene Datenbankdialekte zu reduzieren.

- Prometheus: Ein Beispiel für die sinnvolle Nutzung von Generics ist ein Pull Request in Prometheus, der zeigt, wie Generics für eine effizientere Implementierung von Sortieralgorithmen verwendet werden können.

- HTTP-Client-Wrapper: Erstellung von HTTP-Client-Wrappern zu verallgemeiner. Dies reduziert erheblich den Boilerplate-Code für SQL-Handling.

- JSON-Parsing:  Generics werden oft verwendet, um das JSON-Parsen zu vereinfachen und sicherzustellen, dass die JSON-Daten immer in die erwarteten Strukturen umgewandelt werden:

```go
type JSONStreamReader[T any] struct {
    Scanner *bufio.Scanner
}

func (js JSONStreamReader[T]) ReadAll(dst *[]T) error {
    for js.Scanner.Scan() {
        var data T
        if err := json.Unmarshal(js.Scanner.Bytes(), &data); err != nil {
            return err
        }
        *dst = append(*dst, data)
    }
    return nil
}
```

## Allgemeine Einschätzungen

- Reduzierung von Boilerplate-Code:
Generics verringern die Notwendigkeit, redundanten Code zu schreiben oder Code-Generatoren zu verwenden. Sie helfen, allgemeinere und flexiblere Datenstrukturen zu erstellen.

- Erhöhte Typsicherheit:
Generics ermöglichen es, Typen zur Kompilierzeit zu überprüfen, was die Wahrscheinlichkeit von Laufzeitfehlern durch Typkonvertierungen verringert und die Zuverlässigkeit und Wartbarkeit des Codes verbessert.

- Verbesserte Bibliotheken:
Einige Bibliotheken und Projekte haben von der Einführung von Generics profitiert. Beispiel: 
das Paket golang.org/x/exp/maps, das nützliche Funktionen für Map-Datenstrukturen bereitstellt.

- Leistungsverbesserungen:
Von Nutzern wird hervorgehoben, dass Reflection langsamer ist als Generics, was ein wichtiger Grund für die Verwendung von Generics sein kann, um die Performance zu verbessern.


# Argumente gegen die Verwendung von Generics in Go 

## Kritische Meinungen und Herausforderungen 

- Eingeschränkte Nutzung: Einige Entwickler berichten, dass sie Generics selten brauchen oder verwenden, da sie in ihren Anwendungsfällen nicht notwendig sind und, dass Generics allgemein in der Praxis nur selten verwendet werden beziehungsweise hauptsächlich in speziellen, weniger verbreiteten Bibliotheken zu finden sind.

- Schwierigkeiten bei der Umsetzung: Einige Nutzer erwähnen, dass sie Schwierigkeiten haben, Generics in Go zu verstehen und korrekt zu implementieren, insbesondere im Vergleich zu anderen Sprachen wie Java.

- Refactoring-Kosten: Entwickler argumentieren, dass viele Open-Source-Projekte Generics nicht übernehmen, weil die Zeit und Ressourcen für das Refactoring begrenzt sind.

- Leistungsbedenken: Die Einführung von Generics in Go 1.18 hat die Kompilierungszeiten selbst für Codebasen, die keine Generics verwenden, verlangsamt. Obwohl Go 1.20 diese Zeiten wieder auf das Niveau von Go 1.17 gebracht hat, bleibt die Frage, ob die initiale Einführung notwendig war.

- Reduzierte Code-Lesbarkeit:
Die Verwendung von Generics kann die Lesbarkeit des Codes verschlechtern. Die zusätzlichen Klammern und Typdeklarationen könnten den Code komplexer erscheinen lassen.

- Komplexität des Compilers:
Die Hinzufügung von Generics hat die Komplexität des Go-Compilers erhöht, was zu mehr potenziellen Fehlern und einer langsameren Weiterentwicklung des Compilers führen könnte.


# Fazit zur Verwendung von Generics in Go

Die Einführung von Generics in Go war ein bedeutender Schritt zur Erweiterung der Sprachfunktionalität. Sie bieten zweifellos Vorteile, insbesondere für die Erstellung flexibler und typsicherer Bibliotheken.
Die Analyse mehrerer Projekte und Diskussionen in der Entwicklergemeinschaft zeigt jedoch, dass Generics trotz ihres Potenzials bislang nur begrenzt eingesetzt werden. Viele Entwickler finden Generics in ihren spezifischen Anwendungsfällen nicht notwendig oder haben Schwierigkeiten bei der Umsetzung. Die zusätzlichen Komplexitäten und möglichen Performanceeinbußen tragen ebenfalls dazu bei, dass Generics bisher nicht weit verbreitet sind.


Generics stellen eine nützliche Verbesserung dar, sind aber kein Allheilmittel. Ihr Nutzen wird sich mit der Zeit zeigen, wenn mehr Projekte und Bibliotheken die neuen Möglichkeiten nutzen und ihre Erfahrungen teilen. Bis dahin bleibt die Verbreitung von Generics in Go eher begrenzt und auf spezifische Anwendungsfälle beschränkt.


