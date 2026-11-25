# Szablon sprawozdania laboratoryjnego

**Przedmiot:** Uczenie głębokie w systemach rozpoznawania i przetwarzania mowy
**Zajęcia nr / temat:** 6 / Projekt generacyjny cz. 1 – przygotowanie danych dla Char-RNN
**Autor / grupa:** Mateusz Pluta / GL 5.03
**Data:** 19.11.2026r.

## 1. Cel ćwiczenia

Przygotowanie polskiego tekstu do przewidywania następnego znaku (Char-RNN): oczyszczenie korpusu, podział na train/val/test, słownik znaków oraz Dataset i DataLoader z oknami przesuniętymi o 1 znak. Dodatkowo sprawdzenie, jak długość okna SEKW wpływa na liczbę par.

## 2. Metoda i implementacja

**Narzędzia:** Python, PyTorch (Dataset, DataLoader), NumPy, Matplotlib

**notatnik** 
https://github.com/mattty1/Speech-Recognition.git

**Dane:** B. Prus, *Lalka*, tom I (Wolne Lektury)  

- **Konfiguracja:** podział 80/10/10 na trzy rozłączne, ciągłe fragmenty . Takie proporcje są standardowe, a val i test (po ok. 77 tys. znaków)  wystarczają na okna do SEKW = 160. Słownik zbudowany tylko na treningu (V = 101). Znaki spoza słownika miały być zamieniane na spację, ale w val i test takich znaków nie było. Okna: krok 1, SEKW = 40 batch_size = 32, train: shuffle=True, drop_last=True, val/test: bez mieszania.

## 3. Wyniki

**Liczebności zbiorów (SEKW = 40, batch = 32)**

| Zbiór | Znaków | Udział | Różnych znaków | Par | Batchy |
|---|---|---|---|---|---|
| train | 618105 | 80,0% | 101 | 618065 | 19314 |
| val | 77263 | 10,0% | 74 | 77223 | 2414 |
| test | 77264 | 10,0% | 81 | 77224 | 2414 |


**Liczba par w zależności od SEKW**

| SEKW | train | val | test |
|---|---|---|---|
| 20 | 618085 | 77243 | 77244 |
| 40 | 618065 | 77223 | 77224 |
| 80 | 618025 | 77183 | 77184 |
| 160 | 617945 | 77103 | 77104 |

![alt text](wykres_sekw-1.png)

![alt text](wykres_znaki-1.png)

![alt text](wykres_slowa-1.png)

## 4. Analiza i interpretacja

**SEKW:** liczba par to liczba znaków − SEKW, więc każdy dodatkowy znak okna odejmuje 1 parę. Od 20 do 160 różnica to tylko 140 par (ok. 0,02% treningu i 0,2% val/test), dlatego linie na wykresie są prawie płaskie. Okna się nakładają (krok 1), więc par jest tyle co znaków, a dłuższe okno daje tylko więcej kontekstu kosztem większego batcha.
**Znaki:** najczęstsza jest spacja (ok. 94 tys.), potem „a”, „i”, „e”, „o”, „z”. Z polskich liter w top 20 jest tylko „ł”, reszta (ą, ę, ż, ś…) jest rzadsza, więc model będzie miał na nich mniej przykładów. Val i test mają mniej różnych znaków (74 i 81) niż train (101), bo rzadkie znaki występują tylko w treningu.
**Słowa:** w treningu jest 93 276 słów, z czego 19 584 różnych. Najczęstsze to krótkie słowa funkcyjne („i”, „się”, „w”, „nie”, „na”), a także „pan” i „wokulski”, czyli słownictwo typowe dla tej książki. Większość słów to rzadkie słowa (długi ogon, jak w rozkładzie Zipfa).
**Ograniczenia:** to jedna książka jednego autora, więc model będzie naśladował jej styl.

## 5. Wnioski

- Podział jest ciągły i rozłączny, słownik pochodzi tylko z treningu, a przesunięcie celu o 1 sprawdzone w każdym zbiorze.

- `SEKW` prawie nie zmienia liczby par, więc długość okna warto dobierać pod potrzebny kontekst i koszt obliczeń.

- Do poprawy: większy korpus (więcej książek, podział po dokumentach), przy innym korpusie znak <UNK>, a także krok okna większy niż 1, żeby mniej się nakładały.

---

## Kryteria oceny (rubryka)

| Element | Waga |
|---|---|
| Poprawność implementacji | 40% |
| Wyniki i ich interpretacja | 30% |
| Czytelność kodu (PEP8, komentarze) | 20% |
| Wnioski | 10% |

Próg zaliczenia sprawozdań (O1): 90%. Oddanie: do tygodnia po zajęciach.

---

Prowadzący: dr Tomasz Cieplak / mgr inż. Patryk Marek, Rok akademicki: 2026/27
