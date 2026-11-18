# Szablon sprawozdania laboratoryjnego

**Przedmiot:** Uczenie głębokie w systemach rozpoznawania i przetwarzania mowy
**Zajęcia nr / temat:** 5 / Wpływ korpusu na tokenizację BPE i embeddingi Word2Vec
**Autor / grupa:** Mateusz Pluta / GL 5.03
**Data:** 12.11.2026r.

## 1. Cel ćwiczenia

Celem ćwiczenia jest zbadanie wpływu rozmiaru słownika na fragmentację słów w algorytmie tokenizacji BPE (ze szczególnym uwzględnieniem polskich form fleksyjnych, zarówno widzianych, jak i niewidzianych w treningu) oraz ocena wpływu zmiany proporcji kontekstów w korpusie na reprezentację wektorową słowa wieloznacznego w modelu Word2Vec.

## 2. Metoda i implementacja

**Wykorzystane narzędzia, modele i zbiory danych:** Wykorzystano biblioteki `tokenizers` (Hugging Face) do BPE oraz `gensim` do Word2Vec. Zbiór danych stanowił ręcznie przygotowany polski korpus (52 zdania), podzielony na 3 pule: zdania ze słowem "zamek" w znaczeniu budowli, "zamek" w znaczeniu mechanizmu oraz zdania tła (bez tego słowa).

**Link do repozytorium/notatnika:** https://github.com/mattty1/Speech-Recognition.git

**BPE:** `pre_tokenizer = Whitespace()`, badane limity wielkości słownika: `vocab_size` = 60, 100, 200, `min_frequency=1`.

**Word2Vec:** Architektura Skip-gram (`sg=1`), wektory o wymiarze `vector_size=20`, okno kontekstu `window=3`, epoki = 500, `min_count=1`.

## 3. Wyniki

**Metryki (tokenizacja BPE oraz podobieństwo cosinusowe Word2Vec):**
 **Hipoteza BPE:** Zwiększenie limitu słownika `vocab_size` spowoduje scalanie częstszych par symboli w dłuższe podsłowa i całe wyrazy, co obniży średnią liczbę tokenów na słowo zarówno dla form widzianych (zbliżając wynik do 1.0), jak i niewidzianych (dzięki wydzielaniu całych tematów fleksyjnych).

| Limit `vocab_size` | Faktyczny rozmiar $\|V\|$ | Grupa form | Średnia liczba tokenów / słowo | Liczba form z `[UNK]` |
|---|---|---|---|---|
| 60 | 60 | widziane | 3.50 | 0 |
| 60 | 60 | niewidziane | 4.67 | 0 |
| 100 | 100 | widziane | 3.00 | 0 |
| 100 | 100 | niewidziane | 4.00 | 0 |
| 200 | 200 | widziane | 1.50 | 0 |
| 200 | 200 | niewidziane | 3.00 | 0 |

  - **Podobieństwo cosinusowe dla grup tematycznych w Modelu 1 (pełne wektory 20D):**
    - Średnie podobieństwo wewnątrz Grupy 1 (Budowla: `król`, `rycerz`, `mur`, `wieżę`, `brama`): **0.898**
    - Średnie podobieństwo wewnątrz Grupy 2 (Mechanizm: `klucz`, `drzwiach`, `nowy`, `zepsuty`, `naprawy`): **0.871**
    - Średnie podobieństwo między Grupą 1 a Grupą 2: **0.856**

- **Wykresy / przykładowe predykcje (podziały BPE oraz rzut PCA 2D):**
  - **Podział dwóch niewidzianych form (`koty` oraz `kluczem`) dla wszystkich limitów:**
    - Limit 60: `'koty'` $\rightarrow$ `['ko', 't', 'y']`, `'kluczem'` $\rightarrow$ `['k', 'lu', 'cz', 'e', 'm']`
    - Limit 100: `'koty'` $\rightarrow$ `['ko', 'ty']`, `'kluczem'` $\rightarrow$ `['klu', 'cz', 'e', 'm']`
    - Limit 200: `'koty'` $\rightarrow$ `['ko', 'ty']`, `'kluczem'` $\rightarrow$ `['klucz', 'e', 'm']`
  - **Rzut PCA 2D dla Modelu 1 (grupy tematyczne i badany wyraz `zamek`):**
    *(Tutaj wklej wygenerowany wykres z rzutem PCA 2D)*
    `![Rzut PCA 2D - Model 1](sciezka_do_wykresu_pca.png)`

- **Porównanie z baseline (zmiana sąsiadów słowa `'zamek'` w Word2Vec):**
  - **Model 1 (baseline – przewaga kontekstu budowli, 12:4):** `ten` (0.965), `rycerz` (0.936), `mur` (0.935), `piękny` (0.925), `jest` (0.920).
  - **Model 2 (przewaga kontekstu mechanizmu, 4:12):** `klucz` (0.943), `nowy` (0.935), `ten` (0.929), `zepsuty` (0.920), `kurtka` (0.919).

## 4. Analiza i interpretacja

  **Tokenizacja BPE i weryfikacja hipotezy:** Wyniki potwierdzają przyjętą hipotezę. Zwiększenie limitu słownika z 60 do 200 zmniejszyło średnią liczbę tokenów z 3.50 do 1.50 dla form widzianych oraz z 4.67 do 3.00 dla form niewidzianych. Token `[UNK]` nie pojawił się ani razu, ponieważ cały alfabet podstawowy wystąpił w korpusie treningowym. W wyrazie `'kluczem'` wraz ze wzrostem słownika odtwarzany jest rdzeń: przy $|V|=60$ wyraz dzieli się na `['k', 'lu', 'cz', 'e', 'm']`, przy $|V|=100$ powstaje podsłowo `'klu'`, a przy $|V|=200$ wyodrębnia się cały temat `'klucz'` (pochodzący ze słowa treningowego *„klucz”* w zdaniach np. *„Nowy klucz otwiera zamek”*), podczas gdy niewidziana końcówka pozostaje rozbita na `'e'` i `'m'`.

  **Zmiana sąsiadów w Word2Vec:** Zmiana proporcji zdań między pulami całkowicie przebudowała ranking najbliższych sąsiadów słowa `'zamek'`. W Modelu 1 najwyżej znajdują się wyrazy opisujące warownię (`rycerz` – 0.936, `mur` – 0.935, `piękny` – 0.925), wynikające ze zdań *„Rycerz chroni ten zamek”*, *„Zamek ma wielki mur”* oraz *„Stary zamek jest piękny”*. W Modelu 2 wyrazy te (poza zaimkiem `ten`, 0.929) wypadają z pierwszej piątki – nadal posiadają one swoje wektory w Modelu 2 (wystąpiły w 4 zdaniach Puli A przy `min_count=1`), ale ustępują pojęciom z Puli B: `klucz` (0.943), `nowy` (0.935), `zepsuty` (0.920) oraz `kurtka` (0.919), zaczerpniętym m.in. ze zdań *„Nowy klucz otwiera zamek”* i *„Kurtka ma zepsuty zamek”*.

  **Grupy tematyczne na rzucie PCA 2D:** Przypisanie słów do grup wynika bezpośrednio z kontekstu zdań: Grupa 1 (`król`, `rycerz`, `mur`, `wieżę`, `brama`) opisuje średniowieczną budowlę, natomiast Grupa 2 (`klucz`, `drzwiach`, `nowy`, `zepsuty`, `naprawy`) dotyczy działania i usterki zamka w drzwiach. Na wykresie PCA widoczna jest separacja obu grup: pojęcia mechaniczne skupiły się po lewej stronie (`klucz`, `zepsuty`, `nowy`, `drzwiach`) oraz u góry (`naprawy`), natomiast rdzeń pojęć budowlanych (`wieżę`, `mur`, `król`, `rycerz`) utworzył centralne skupisko wokół słowa `zamek`, co odzwierciedla przewagę Puli A w Modelu 1. Jedynie słowo `brama` odstaje w prawym dolnym rogu wykresu, co jest efektem utraty części wariancji przy rzutowaniu przestrzeni 20-wymiarowej na 2 wymiary. Obliczenia na pełnych wektorach potwierdzają wnioski z wizualizacji: średnie podobieństwo wewnątrz grup (0.898 dla Budowli oraz 0.871 dla Mechanizmu) przewyższa średnie podobieństwo między grupami (0.856).

## 5. Wnioski

Tokenizacja BPE skutecznie rozwiązuje problem słów spoza słownika (OOV) w języku polskim, dekomponując nieznane formy fleksyjne na znane tematy i końcówki, a dobór limitu słownika pozwala kontrolować stopień kompresji sekwencji. Model Word2Vec poprawnie odwzorowuje statystykę sąsiedztwa w korpusie i grupuje pojęcia tematycznie, jednak przypisuje każdemu słowu pojedynczy, statyczny wektor uśredniający wszystkie konteksty występowania. Aby rozróżniać znaczenia wyrazów wieloznacznych (polisemii, takich jak *zamek*) w zależności od konkretnego zdania, należy stosować modele kontekstowe oparte na mechanizmie uwagi (np. architekturę Transformer / BERT).

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
