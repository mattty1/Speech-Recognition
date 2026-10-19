# Szablon sprawozdania laboratoryjnego

**Przedmiot:** Uczenie głębokie w systemach rozpoznawania i przetwarzania mowy
**Zajęcia nr / temat:** 01, Sekwencje i sygnał mowy: od fali dźwiękowej do reprezentacji
**Autor / grupa:** Mateusz Pluta, GL5.03
**Data:** 15.10.2026r.

## 1. Cel ćwiczenia

Wygenerowanie i porównanie log-mel i MFCC dla trzech nagrań (samogłoska, wyraz, zdanie) oraz zbadanie wpływu parametrów n_fft i hop_length na wygląd spektrogramów

## 2. Metoda i implementacja

**Wykorzystane narzędzia i modele.**
librosa, numpy, matplotlib
**Najważniejsze fragmenty kodu (lub link do repozytorium/notatnika).**
https://github.com/mattty1/Speech-Recognition.git
**Konfiguracja (parametry, dane).**
Nagrania wczytałem z częstotliwością próbkowania 16 kHz (mono). Spektrogram log-mel dla 64 pasm mel oraz MFCC dla 13 pierwszych współczynników.
## 3. Wyniki

**Samogłoska ("a"):**
Samogłoska ("a"): Przy zwykłych ustawieniach na wykresie widać było równe i poziome paski. Kiedy zmieniłem n_fft na 256 i hop na 300, paski zlały się w grube, rozmyte linie, a cały obraz zrobił się kanciasty i "rozpikselowany" wzdłuż osi czasu.
**Wyraz:**
Wyraz: Falujące paski (formanty) i ciemne pionowe przerwy oznaczające krótkie zwarcia spółgłosek. Kiedy zmieniłem ustawienia na: n_fft=128 i hop=100, paski częstotliwości zupełnie zniknęły, ale za to dokładnie było widać, kiedy kończy się i zaczyna każdy dźwięk. Sama oś czasu stała się za to gładsza.
**Zdanie:**
Zapis całego zdania był pełny szumów i szybkich zmian. Zmiana ustawień na n_fft=1024 i hop=500 sprawiła, że paski częstotliwości były bardzo dokładne, ale obraz stał się kanciasty w czasie, że trudno było cokolwiek z niego na oko odczytać.

## 4. Analiza i interpretacja

Samogłoska (zmiana na :n_fft=256, hop=300): Zmniejszenie okna (n_fft) sprawilo, że poziome paski częstotliwości stały się zamazane i niewyraźne. Z kolei bardzo duży krok przesuwania (hop) spowodował, że program rzadziej "robił zdjęcia" dźwiękowi, przez co cały wykres stal się kanciasty i "rozpikselowany" w poziomie.

Wyraz (zmiana na: n_fft=128, hop=100): Bardzo krótkie okno (n_fft) całkowicie zniszczyło paski częstotliwości (pojawiły się czarne luki na wykresie), ale za to idealnie ostro pokazalo, w ktorej milisekundzie zaczyna się i kończy dany dźwięk. Mały krok (hop) zagęścił siatkę ramek, przez co wykres w osi czasu stał sie bardzo gładki i płynny.

Zdanie (zmiana na: n_fft=1024, hop=500): Długie okno (n_fft) świetnie uwydatniło paski częstotliwościowe, ale przez to zatarły się granice w czasie między szybko wypowiadanymi słowami. Ekstremalnie duży krok (hop) w połączeniu ze skomplikowaną mową ciągłą sprawił, że wykres stał się bardzo blokowy i dla ludzkiego oka zupelnie nieczytelny.   

## 5. Wnioski

Wybór okna (n_fft) to kompromis, duże okno świetnie pokazuje częstotliwości, ale rozmywa czas. Krótkie okno daje świetny czas, ale gubi częstotliwości. Krok przesuwania (hop_length) zmienia tylko płynność obrazka. Mały krok wygładza wykres, duży robi go kanciastym, ale w żaden sposób nie poprawia fizycznej jakości pokazanych częstotliwości. Obie testowane metody (log-mel i MFCC) pozbywaja sie części zbędnych danych, tworzac z nagrania uproszczony obraz.

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
