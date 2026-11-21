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