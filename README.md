# Symulator QKD z wykorzystaniem protokołu BB84 oraz BBM92
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1aX3ddTcxR1mtZ0ziGCE7nBVGq5sxOoUY#scrollTo=0Taowo3f47d6)

# Analiza Transpilacji i Kosztów Obwodów Kwantowych na IBM Quantum

**Autor:** Grzegorz Ciapa  
**Technologie:** Python, Qiskit SDK, IBM Quantum Platform (Hardware: `ibm_fez`)

## Cel Projektu
Celem projektu było zbadanie "kosztu" uruchomienia algorytmu kwantowego w symulacji oraz na rzeczywistym sprzęcie IBM Quantum. Analiza skupia się na wpływie poziomów optymalizacji transpilera (Levels 0-3) na głębokość obwodu i liczbę bramek, weryfikując hipotezę, że najwyższy poziom optymalizacji nie zawsze gwarantuje najlepsze wyniki fizyczne.

## Metodyka
Badania przeprowadzono na dwóch typach obwodów dla 5, 10 i 20 kubitów:
1.  **Structured:** Obwody o uporządkowanej strukturze (niskie splątanie, łatwe mapowanie).
2.  **Random:** Obwody losowe o wysokiej złożoności (wymagające intensywnego routingu/SWAP-ów).

## Wykorzystano metryki:
* **Głębokość obwodu (Depth):** Czas trwania algorytmu.
* **Liczba bramek 2-kubitowych:** Głównym źródło błędów w architekturze nadprzewodzącej.
* **ESP (Estimated Success Probability):** Szacowana wierność wyniku w obecności szumu.

## Kluczowe Wnioski
1.  **Paradoks Optymalizacji:** Dla obwodów losowych (Random) poziom optymalizacji 3 (Level 3) często generował **większą głębokość** niż Level 2 (np. 1774 vs 1632 dla 20 kubitów). Wynika to z "over-optimization" i agresywnej syntezy unitarnej niedopasowanej do topologii Heavy Hex.
2.  **Struktura ma znaczenie:** Dla obwodów strukturalnych Level 3 działał poprawnie, redukując głębokość z 171 (Level 0) do 62 (Level 3).
3.  **Realny Sprzęt (Hardware):** Rzeczywiste procesory (jak `ibm_fez`) narzucają duży narzut z powodu braku natywnej bramki CNOT (zastępowanej przez ECR) oraz ograniczonej łączności kubitów.

## Zawartość Repozytorium
* `Inf_kw_temat_2.ipynb` - Główny kod projektu (generowanie obwodów, symulacja, wysyłka zadań do IBM Quantum).
* `Temat 2 QI.pdf` - Pełny raport z opisem teoretycznym i analizą wyników.
* `Transpilacja_krotka_prezentacja_30min.pptx` - Prezentacja podsumowująca projekt.
