1) Annoying numeric literals - we need flexible literals pls.
2) `funptr`s are not C-compatible
3) I need defer!
4) const ESC: char = 27u8 as char
5) match nie działa z {}
6) W klasie albo wszystko ma public i private albo nic. `public Terminal.destroy()` też musi mieć
7) cptr nie ma arytmetyki wskaznikow
8) Formatter nie usuwa trailing whitespace z komentarzy jak tuz za nimi jest enter
9) Metoda i wolna funkcja o tej samej nazwie nie mogą się wywoływać nawzajem, nawet przy innej liczbie argumentów. Moze to akurat oczekiwane?
   ```
   import core.builtins.*;

   fun bar(a: i64, b: i64) -> i64 = a + b;

   class Foo {
       x: i64 = 1;
       fun bar(a: i64) -> i64 = bar(x, a);
   }
   ```

   ```
   error[E0]: Found a combination of function and non-function callables in a call expression.
   ```
10) ~~`public mut fun` nie parsuje się~~. Naprawione na `main` (#3996, nowy model mutowalności); dotyczyło starszego duckc.
11) Moduł nie widzi modułu-rodzeństwa po samej nazwie: z `src/tests/tests.dk` import `testing.check` daje `Module not found.` Lookup szuka tylko wśród dzieci, przodków i nazwy pakietu. Obejście: `import src.testing.check;` (root nazywa się `src`).
12) `str == str` się nie kompiluje (jest tylko `String == String`). Obejście: własny `strEq`.
13) String literal wewnątrz `{}` w f-stringu: `f"{paint(RED, \"FAIL\")}"` daje `String unclosed before end of line.` Obejście: stała albo zmienna.
14) `const` z literałem tablicy zawiesza duckc (100% CPU, bez końca), nawet gdy nieużyty:
    ```
    const A: u8[4] = [1u8, 1u8, 1u8, 1u8];
    ```
    `var` działa. Zgłoszone: https://github.com/ducktype-org/duckling/issues/4171
15) `expand` w ciele funkcji przyjmuje tylko jedną instrukcję. Kilka instrukcji daje `Unexpected additional tokens during parsing.` Obejście: wygenerować blok `{ ... }`.
16) Funkcja typu, która bake'uje template, nie liczy się w comp-time:
    ```
    fun Result(T: type, E: type) -> type = Ok[T] | Err[E];
    ```
    `Expression cannot be evaluated at compile-time.` Obejście: `template(T: type, E: type) const Result = Ok[T] | Err[E];`
17) Brak koercji z mniejszego wariantu do większego: `A | B` nie przechodzi jako `A | B | C`. Pojedynczy `A` do `A | B | C` działa.
18) Nie da się zrobić `move` pola z bindingu w `match`: `move e.error` daje `` `move` can only be applied to an owned local variable.`` Obejście: matchować przez `ref` i bindować `ref Err[E]`.
19) `()` jako alternatywa w typie wariantu: `fun f() -> () | RomTooBig` daje `Call failed because no matching functions were found.` (testy języka używają `T | ()`, nie sprawdzałem tej kolejności). `Ok[()]` działa.
20) Kolejny przypadek z 1): `match` z `case _ = 0;` i `case e = foo();` (zwraca i64) daje `All match cases have to be of the same type, but this one is i64 while an earlier one is i32.` Trzeba pisać `0i64`.
21) Literał zmiennoprzecinkowy bez typu to `f32`: `var term = 1.0;`, a potem `term = term * x` z `x: f64` daje `Type f64 cannot be converted to type f32.` Trzeba pisać `var term: f64 = 1.0;`.
22) Brak shadowingu nawet między rozłącznymi blokami: `var k` w bloku `if` i późniejsze `var k` w funkcji dają `Variable name is ambiguous, because it has been defined multiple times.`
23) Nie da się wstawić `{` ani `}` do f-stringa: `f"Expected '\{'"` daje `Escape sequence \{ is not recognised.` Obejście: zwykły string i `append`.
24) Ewaluacja comp-time (`expand`) nie radzi sobie z:
    - dzieleniem float przez zero (`0.0 / 0.0` dla NaN): `Compile time evaluation of this function call failed or returned unsupported result.` Obejście: `inf = 1.0e308 * 10.0`, `nan = inf - inf`.
    - `std.math` (libm przez FFI, np. `math.sqrt`).
    Błąd nie mówi, która instrukcja zawiodła, trzeba bisectować.
25) `match (result)` na lokalnej zmiennej z wariantem posiadającym `String` daje `Cannot implicitly copy a value of non-trivially-copyable type`. Trzeba `match (move result)` albo przekazać przez `ref` do funkcji.
26) `expand` na poziomie modułu nie widzi żadnych funkcji (ani z tego modułu, ani zaimportowanych): `expand gen();` daje `Call failed because no matching functions were found.` W `namespace N { expand gen(); }` działa. Do tego przyjmuje tylko jedną deklarację. Obejście: wygenerować `export { ... }` i rozwinąć w namespace (`src/chip8/isa/isa.dk`).
27) `if c then x else none` nie koerguje `none` do `?T`, nawet z `let v: ?str = ...` i `x as (?str)`: `Both branches ... the then branch is ?slice char and the else branch is Class None.` Obejście: dwa `return`.
28) Comp-time: metoda na tymczasowej liście, `const N: i64 = opcodes().length();`, daje `VMThread has panicked: Data was freed`. Z `var t = opcodes();` działa.
29) `const` z klasą zwróconą z funkcji (`const QQ: Q = q();`) też zawiesza duckc, jak 14).
30) Zła sygnatura `main`, np. `fun main(argc: i64, argv: manyptr cptr char) -> i64`, wywala kompilator: `terminate called after throwing an instance of 'std::bad_expected_access<...>'` zamiast błędu.
31) `for (x in ...)` działa tylko dla statycznych tablic: dla `List`/`str` daje `[Feature not implemented] for statements for non-array type`. Zmienna pętli to kopia, więc `for (b in mem) b = 0u8;` nie zeruje tablicy.
32) Tablica `bool[N]` jako pole klasy psuje układ struktury i duckc pada na asercji w backendzie LLVM. Frontend liczy `bool[N]` jako N bitów, LLVM jako N bajtów. Dotyczy każdego N (sprawdzone 2, 16, 2048) i każdej pozycji pola; `u8[N]` w tym samym miejscu oraz lokalne `var x: bool[N];` działają.
    ```
    class A { flags: bool[16]; after: i64 = 7; }
    fun main() -> i64 = { var a: A; a.flags[3] = true; return a.after; }
    ```
    ```
    Assertion failed: `expected_offset == actual_offset`
    LLVM struct layout mismatch for class '...A' at field index 1: expected offset 8B, got 16B
    ```
    Gdy tablica jest ostatnim polem: `LLVM struct size mismatch for class ...`. Obejście: `u8[N]` z wartościami 0/1 (`Chip8.display`, `Chip8.keys`); dostęp przez `pixel()` i `setKey()`.
33) Missing return statement nie daje source posititon
34) Brakuje konstruktorów użytkowanika