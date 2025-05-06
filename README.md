
# Adaptive Cruise Control (ACC) – Model AADL

## 1. Dane studenta
### Jakub Karoń
### jakubkaron@student.agh.edu.pl


## 2. Opis ogólny  
Adaptive Cruise Control (ACC) to zaawansowany system wspomagający kierowcę, który utrzymuje zadaną prędkość pojazdu oraz – w trybie adaptacyjnym – bezpieczny odstęp od poprzedzającego samochodu. Nasz model w AADL obejmuje trzy główne elementy:
1. **Menedżer trybów (ModeManager)**  
   – odpowiada za przełączanie między trybami: OFF, HOLD (klasyczny tempomat), ADAPTIVE (ACC) i STOP (zerowa prędkość).  
2. **Wątek ACC (ACCThread)**  
   – monitoruje dystans do poprzedzającego pojazdu i na tej podstawie modyfikuje prędkość zadawaną regulatorowi.  
3. **Regulator prędkości (PIDThread)**  
   – w pętli co 50 ms odczytuje prędkość aktualną, oblicza błąd względem prędkości zadanej (z klasycznego tempomatu lub ACC) i wysyła komendę do przepustnicy.

Całość zestawiona jest w systemie `AdaptiveCCSystem`, który łączy sensory (prędkości i dystansu), jednostkę sterującą i aktuatorem przepustnicy.

---

## 3. Wymagania funkcjonalne  
1. **Monitorowanie prędkości i dystansu**  
   - Odczyt prędkości z czujnika co 50 ms.  
   - Odczyt odległości do poprzedzającego pojazdu co 50 ms.  
2. **Tryby pracy**  
   - **OFF** – system wyłączony.  
   - **HOLD** – klasyczny tempomat utrzymujący zadaną prędkość.  
   - **ADAPTIVE** – ACC dostosowujący prędkość, by utrzymać bezpieczny odstęp (_d_target_).  
   - **STOP** – zatrzymanie pojazdu, gdy dystans < minimalnego progu.  
3. **Przejścia między trybami**  
   - SET → HOLD → (jeśli pojazd w zasięgu) → ADAPTIVE.  
   - Hamulec lub OFF → powrót do OFF.  
   - W ADAPTIVE, gdy dystans < próg, → STOP; gdy dystans ≥ próg, → ADAPTIVE.  
4. **Algorytm regulacji**  
   - PID w pętli zamkniętej:  
     1. odczyt prędkości,  
     2. obliczenie błędu względem docelowej prędkości,  
     3. generacja sygnału do przepustnicy.  
   - W trybie ADAPTIVE: regulator przyjmuje jako prędkość zadaną wynik z ACCThread.

---

## 4. Wymagania niefunkcjonalne  
1. **Czas rzeczywisty**  
   - **PIDThread**: okres 50 ms, WCET maks. 2 ms, deadline = 50 ms, jitter ≤ ±1 ms.  
   - **ACCThread**: okres 50 ms, WCET maks. 2 ms, deadline = 50 ms.  
   - **ModeManager**: okres 10 ms, WCET maks. 1 ms, deadline = 10 ms.  
2. **Bezpieczeństwo i niezawodność**  
   - Natychmiastowe przejście do OFF po sygnale hamowania.  
   - Tryb STOP w sytuacjach krytycznych (dystans < próg).  
   - Obsługa utraty pomiaru (timeouty, powrót do HOLD lub OFF).  
