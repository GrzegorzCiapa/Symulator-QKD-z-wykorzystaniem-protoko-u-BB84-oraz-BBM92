[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1aX3ddTcxR1mtZ0ziGCE7nBVGq5sxOoUY#scrollTo=0Taowo3f47d6)
# Symulator Protokołów Dystrybucji Klucza Kwantowego (QKD) — BB84 & BBM92

**Autor:** Grzegorz Ciapa 
**Technologie:** Python 3, Qiskit, NumPy, Jupyter Notebook, Matplotlib  

## Cel Projektu
Celem projektu jest implementacja oraz analiza porównawcza symulatora warstwy kwantowej i klasycznej dla dwóch kluczowych protokołów dystrybucji klucza kwantowego:
* **BB84** (protokół bazujący na przygotowaniu i pomiarze pojedynczych stanów fotonów),
* **BBM92** (zaawansowany wariant oparty na zjawisku splątania kwantowego i generowaniu par stanów Bella – zadanie na ocenę celującą).

Symulator pozwala na pełną rekonstrukcję fizycznych procesów nadawania, transformacji w kanale kwantowym, symulacji ingerencji osób trzecich (podsłuch Ewy przy użyciu różnych strategii) oraz klasycznego post-processingu (uzgadnianie baz, korekcja błędów i wzmocnienie prywatności).

## Metodyka
Projekt został zrealizowany w architekturze modułowej, odzwierciedlającej rzeczywiste etapy przetwarzania w fizycznych systemach QKD:

1. **Sprzętowe Źródło Losowości (QRNG z pliku binarnego):**
   * Zaimplementowano klasę `QRNGFileGenerator` działającą w oparciu o odczyt surowych plików binarnych (`QNGFile.dat` do `QNGFile10.dat`), które stanowią twardy zapis rzeczywistych pomiarów zjawisk stochastycznych.
   * **Optymalizacja buforowania:** Dane pobierane są pakietami (1 bajt = 8 bitów danych kwantowych) do pamięci podręcznej RAM (`buffer`), a pojedyncze wartości wydzielane są przy użyciu szybkich operacji przesunięć bitowych i maskowania logicznego (`& 1`).
   * **Zapobieganie determinizmowi:** Wprowadzono mechanizm losowego przesunięcia wskaźnika startowego (`seek`), dzięki czemu każde uruchomienie symulacji korzysta z innej puli unikalnych bitów kwantowych.

2. **Przygotowanie i Kodowanie Stanów Kwantowych:**
   * **BB84:** Kodowanie pojedynczych kubitów w oparciu o pobrane bity wartości i baz. Wykorzystanie bramki logicznej NOT (`X`) do zmiany stanu $|0\rangle \rightarrow |1\rangle$ oraz bramki Hadamarda (`H`) do transformacji bazy prostoliniowej (Z) w diagonalną (X).
   * **BBM92:** Symulacja niezależnego źródła par fotonów splątanych w stanie Bella $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ przy użyciu bramek Hadamarda (`H`) oraz kontrolowanej negacji (`CNOT`). Kubity są dystrybuowane niezależnie do Alicji i Boba.

3. **Kanał Kwantowy z Podsłuchem (Ewa):**
   * Symulacja obecności aktywnego podsłuchiwacza przy użyciu strategii *Intercept-Resend* (Przechwyć-Wyślij Ponownie).
   * Zaimplementowano dwa zróżnicowane modele ataków w celu zbadania ich wpływu na bezpieczeństwo przesyłu:
     * **Atak interwałowy** (zastosowany w symulacji BB84),
     * **Atak probabilistyczny** (zastosowany w symulacji BBM92).
   * Intensywność podsłuchu Ewy była płynnie modyfikowana w zakresie od 0% do 100% przesyłanych fotonów.

4. **Pomiar i Klasyczne Przetwarzanie Końcowe (Post-processing):**
   * **Uzgadnianie baz (Sifting):** Publiczna wymiana informacji o użytych bazach pomiarowych i odrzucenie wyników nieskorelowanych.
   * **Korekcja błędów (Information Reconciliation):** Klasyczna analiza parzystości bloków w celu eliminacji szumów tła.
   * **Wzmocnienie prywatności (Privacy Amplification):** Symetryczne skracanie klucza za pomocą operacji logicznej XOR na sąsiednich parach bitów, sprowadzające wiedzę Ewy do poziomu asymptotycznie bliskiego zeru.

## Wykorzystane Metryki
Eksperymenty symulacyjne przeprowadzono w szerokim spektrum scenariuszy (zmienna intensywność podsłuchu od 0% do 100% z krokiem co 10%). Weryfikacji poddano następujące wskaźniki:
* **QBER (Quantum Bit Error Rate):** Stosunek błędnych bitów do całkowitej długości klucza przesianego.
* **Wiedza Ewy (Eve's Info / Leakage):** Statystyczny procentowy stopień dopasowania bitów poznanych przez Ewę do oryginalnego klucza.
* **Długość klucza przesianego i końcowego:** Analiza wydajności ilościowej generowania bezpiecznego zasobu kryptograficznego po procesie skracania metodą XOR.

## Kluczowe Wnioski
* **Teoretyczna Granica Bezpieczeństwa (Próg Shora-Preskilla):** Symulacje jednoznacznie potwierdziły, że przekroczenie progu **~11% QBER** uniemożliwia efektywne odcięcie podsłuchiwacza za pomocą klasycznej korekcji. System automatycznie przerywa generowanie klucza przy wykryciu tej wartości anomalii.
* **Charakterystyka Ataku Pełnego (100%):** Zarówno dla protokołu BB84 (Atak Interwałowy), jak i BBM92 (Atak Probabilistyczny), w scenariuszu pełnego podsłuchu Ewy, wskaźnik QBER zbiega się precyzyjnie wokół wartości **25%** (z uwzględnieniem dodatkowego szumu sprzętowego tła na poziomie ok. 2-4%, co daje odczyty rzędu 25-29%).
* **Maksymalna Wiedza Asymptotyczna:** Przy 100% podsłuchu, wiedza Ewy o kluczu stabilizuje się na poziomie **~75%** (odczyty z symulacji w przedziale ~72-76%), co idealnie odzwierciedla matematyczne ograniczenia mechaniki kwantowej wynikające z twierdzenia o zakazie klonowania stanów kwantowych oraz statystycznego 50% ryzyka wyboru złej bazy pomiarowej przez hakera.
* **Monogamia Splątania w BBM92:** Wykazano, że jakakolwiek próba dokonania pomiaru na kanale splątanym przez Ewę bezpowrotnie niszczy korelację EPR, co skutkuje natychmiastowym, drastycznym skokiem QBER u legalnych stron komunikacji i pozwala na natychmiastową detekcję obecności intruza.

## Zawartość Repozytorium
* `QKD_simulator_GC.ipynb` — Kompletny kod źródłowy symulatora w formie interaktywnego notatnika Jupyter Notebook (zawiera implementację klas QRNG, obwodów Qiskit, algorytmów klasycznych oraz generatory wykresów).
* `Symulator QKD.pdf` — Oficjalny raport końcowy z projektu zawierający pełne opisy matematyczne, zrzuty ekranu z wynikami, tabele statystyczne oraz wykresy zależności QBER od poziomu infiltracji kanału.
* `QNGFile1.dat`, `QNGFile2.dat`, `QNGFile3.dat` — Pliki binarne z surowym ciągiem kwantowych danych losowych wykorzystywane jako fizyczne źródło entropii.
