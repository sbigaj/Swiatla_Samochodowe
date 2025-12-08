# System sterowania światłami samochodowymi w AADL

Model systemu sterowania oświetleniem pojazdu zaprojektowany w języku Architecture Analysis & Design Language (AADL).

---

## Dane studenta

**Imię i nazwisko:** Szymon Bigaj  
**E-mail:** sbigaj@student.agh.edu.pl  

---

## Opis modelowanego systemu

### Opis ogólny

System sterowania światłami samochodowymi składa się z:

- **Urządzeń wejściowych** - przełączniki, dźwignie, czujniki (9 urządzeń)
- **Jednostki sterującej ECU** - procesor przetwarzający sygnały
- **Magistrali CAN** - medium komunikacyjne (500 kbit/s)
- **Logiki sterowania** - 6 niezależnych wątków przetwarzających
- **Urządzeń wyjściowych** - sterowniki świateł (10 typów świateł)

Model implementuje rzeczywistą architekturę systemu świateł pojazdu samochodowego zgodnie ze standardami branżowymi, z podziałem odpowiedzialności na moduły funkcjonalne i uwzględnieniem krytycznych wymagań czasowych.

---

### Opis dla użytkownika

System obsługuje następujące funkcje oświetlenia pojazdu:

#### **Oświetlenie zewnętrzne:**

1. **Światła pozycyjne** - włączane dźwignią świateł, sygnalizują obecność pojazdu
2. **Światła mijania (krótkie)** - podstawowe oświetlenie drogi, włączane automatycznie lub ręcznie
3. **Światła drogowe (długie)** - zwiększone oświetlenie z możliwym automatycznym wyłączeniem przy zbliżającym się pojeździe
4. **Światła dzienne (DRL)** - włączane w trybie automatycznym podczas jazdy dziennej
5. **Kierunkowskazy** - sygnalizacja zmiany kierunku (lewy/prawy) lub światła awaryjne
6. **Światła przeciwmgielne przednie** - dodatkowe oświetlenie w mglistych warunkach
7. **Światła przeciwmgielne tylne** - zwiększona widoczność z tyłu w gęstej mgle
8. **Światła STOP** - sygnalizacja hamowania 
9. **Światła cofania** - oświetlenie podczas jazdy wstecz
10. **Oświetlenie tablicy rejestracyjnej** - związane ze światłami pozycyjnymi

#### **Dodatkowe funkcje:**

- **Auto-wyłączanie długich świateł** - gdy czujnik wykryje zbliżający się pojazd
- **Automatyczne światła mijania** - czujnik światła otoczenia (zmierzch/tunel)
- **Priorytet świateł awaryjnych** - przycisk awaryjny przejmuje kontrolę nad kierunkowskazami
- **Logika zależności** - światła przeciwmgielne oraz drogowe działają tylko przy włączonych światłach mijania

---

## Spis komponentów AADL

### **Device (Urządzenia wejściowe)**

Fizyczne urządzenia generujące sygnały wejściowe od użytkownika i czujników:

| Urządzenie | Opis |
|------------|------|
| `LightSwitchLever` | Dźwignia sterująca trybem świateł (OFF/POSITION/LOW_BEAM/AUTO) |
| `AmbientLightSensor` | Czujnik natężenia światła otoczenia - automatyczne włączanie świateł |
| `IndicatorAndHighBeamLever` | Dźwignia kierunkowskazów i długich świateł |
| `FogLightFrontSwitch` | Przełącznik świateł przeciwmgielnych przednich |
| `FogLightRearSwitch` | Przełącznik świateł przeciwmgielnych tylnych |
| `HazardButton` | Przycisk świateł awaryjnych |
| `BrakePedalSwitch` | Czujnik pedału hamulca - wykrywa naciśnięciec |
| `RearGearSwitch` | Czujnik biegu wstecznego |
| `ApproachingCarSensor` | Czujnik zbliżających się pojazdów (radar/kamera) |

**Wspólna cecha:** 
- Każde urządzenie komunikuje się z ECU przez magistralę CAN.
- Każde urządzenie ma zdefiniowany flow source dla analizy przepływów

---

### **Device (Urządzenia wyjściowe)**

Sterowniki i moduły świateł realizujące funkcje oświetleniowe:

| Urządzenie | Opis |
|------------|------|
| `PositionLights` | Światła pozycyjne przednie - sygnalizacja obecności pojazdu |
| `LowBeamLights` | Reflektory mijania - podstawowe oświetlenie drogi |
| `HighBeamLights` | Reflektory drogowe - zwiększone oświetlenie |
| `IndicatorLights` | Kierunkowskazy (4 lampy: lewy/prawy przód/tył) |
| `DayLights` | Światła dzienne LED - widoczność w dzień |
| `FrontFogLights` | Światła przeciwmgielne przednie |
| `StopLights` | Światła hamowania - sygnalizacja zatrzymywania |
| `ReverseLights` | Światła cofania - oświetlenie przy jeździe wstecz |
| `RearFogLights` | Światła przeciwmgielne tylne |
| `PlatesLights` | Oświetlenie tablicy rejestracyjnej - świeci z pozycyjnymi |

**Wspólna cecha:** 
- Każde urządzenie odbiera komendy sterujące z ECU przez magistralę CAN.
- Każde urządzenie ma zdefiniowany flow sink dla analizy przepływów

---

### **Thread (Wątki - logika sterowania)**

Niezależne wątki przetwarzające realizujące logikę biznesową systemu:

| Wątek | Funkcja | Właściwości czasowe |
|-------|---------|---------------------|
| `MainLightControl` | Zarządzanie podstawowym oświetleniem (pozycyjne, mijania, dzienne) | Period: 100ms, Exec: 2-5ms, Deadline: 100ms |
| `HighBeamControl` | Kontrola długich świateł z opcjonalnym automatycznym wyłączaniem przy zbliżającym się pojeździe | Period: 50ms, Exec: 1-3ms, Deadline: 50ms |
| `IndicatorsControl` | Sterowanie kierunkowskazami i światłami awaryjnymi | Period: 50ms, Exec: 1-2ms, Deadline: 50ms |
| `FogLightsControl` | Zarządzanie światłami przeciwmgielnymi (weryfikacja dostępności mijania) | Period: 100ms, Exec: 1-2ms, Deadline: 100ms |
| `StopLightControl` | sterowanie światłami hamowania z minimalnym opóźnieniem | Period: 20ms, Exec: 0.5-1ms, Deadline: 20ms |
| `ReverseLightControl` | Sterowanie światłami cofania | Period: 50ms, Exec: 0.5-1ms, Deadline: 50ms |

**Kluczowe cechy:**
- Każdy wątek ma zdefiniowane flow paths dla analizy przepływów
- Period określa częstotliwość wykonywania wątku
- Exec (Compute_Execution_Time) to rzeczywisty czas przetwarzania
- Deadline określa maksymalny dopuszczalny czas na zakończenie
- StopLightControl ma najwyższy priorytet (najkrótszy Period = 20ms)

---

### **Process (Proces główny)**

| Proces | Opis |
|--------|------|
| `CarLightsController` | Główny proces integrujący wszystkie wątki sterowania. Zawiera porty wejściowe/wyjściowe, zarządza połączeniami między wątkami (np. przekazywanie stanu świateł mijania do kontrolera długich i przeciwmgielnych) oraz implementuje flow paths dla analizy end-to-end. |

**Subkomponenty:**
- 6 wątków: `MainLightControl`, `HighBeamControl`, `IndicatorsControl`, `FogLightsControl`, `StopLightControl`, `ReverseLightControl`

---

### **Processor (Procesor)**

| Procesor | Opis |
|----------|------|
| `ECU` | Electronic Control Unit - jednostka sterująca światłami. Fizyczne urządzenie wykonujące proces CarLightsController. Wszystkie wątki są bindowane do tego procesora. Posiada dostęp do magistrali CAN przez dedykowany port. |

---

### 🚌 **Bus (Magistrala komunikacyjna)**

| Magistrala | Opis |
|------------|------|
| `CAN_Bus` | Controller Area Network - standard komunikacyjny w automotive. Łączy wszystkie urządzenia wejściowe z ECU oraz ECU z urządzeniami wyjściowymi. Prędkość transmisji: 500 kbit/s, rozmiar ramki: 8 bajtów, czas transmisji: 2-5ms. Wszystkie połączenia danych są fizycznie realizowane przez tę magistralę (Actual_Connection_Binding). |

---

### **End-to-End Flows (Przepływy end-to-end)**

Kluczowe ścieżki przepływu danych przez cały system:

| Flow | Ścieżka | Budżet czasu |
|------|---------|--------------|
| `brake_to_stop_etef` | Pedał hamulca → CAN → ECU/StopLightControl → CAN → Światło STOP | **10-80ms** |
| `position_lights_etef` | Dźwignia świateł → CAN → ECU/MainLightControl → CAN → Światła pozycyjne | 50-150ms |
| `high_beam_etef` | Dźwignia długich → CAN → ECU/HighBeamControl → CAN → Długie | 30-100ms |
| `hazard_lights_etef` | Przycisk awaryjnych → CAN → ECU/IndicatorsControl → CAN → Kierunkowskazy | **20-80ms** |

---

### **System (System główny)**

| System | Opis |
|--------|------|
| `CarLightingControlSystem` | Główny system integrujący wszystkie komponenty modelu. Zawiera procesor ECU, magistralę CAN, 9 urządzeń wejściowych, 10 urządzeń wyjściowych oraz proces CarLightsController. Definiuje wszystkie połączenia danych między urządzeniami a procesorem oraz połączenia magistrali (bus access). Zawiera właściwości bindingu określające, że proces wykonuje się na ECU, a wszystkie połączenia danych przechodzą przez CAN. |

---

## Model - rysunek

Na rysunku przedstawiono diagram modelu systemu sterowania światłami samochodowymi:

https://github.com/sbigaj/Swiatla_Samochodowe/blob/main/Diagram_modelu.png

---

## 📊 Analiza opóźnień (Flow Latency Analysis)

Model zawiera pełną specyfikację przepływów umożliwiającą analizę czasową:

### Przykładowe wyniki (teoretyczne):

**Światła STOP (brake_to_stop_etef):**
```
Sensor → CAN(2-5ms) → ECU(0.5-1ms) → CAN(2-5ms) → Actuator
Total: ~15-70ms ✅ (< 80ms budget)
```

**Światła pozycyjne (position_lights_etef):**
```
Switch → CAN(2-5ms) → ECU(2-5ms) → CAN(2-5ms) → Lights
Total: ~55-115ms ✅ (< 150ms budget)
```

### Jak uruchomić analizę:

1. Otwórz model w OSATE
2. Zainstancjonuj system: prawym na `CarLightingControlSystem.impl` → **Instantiate**
3. Uruchom analizę: **Analyses → Timing → Check flow latency**
4. Wyniki pokażą szczegółowy breakdown opóźnień dla każdego flow

---

## 📁 Struktura projektu

```
.
├── README.md                          # Ten plik
└── Swiatla_Samochodowe.aadl          # Model AADL
```

---

## 🚀 Jak używać

### Wymagania:
- OSATE 2.x (https://osate.org/download-and-install.html)
- Plugin: Flow Latency Analysis (zazwyczaj wbudowany)

### Kroki:

1. **Import projektu do OSATE:**
   - File → New → AADL Project
   - Skopiuj plik `.aadl` do folderu projektu

2. **Walidacja modelu:**
   - Otwórz plik `.aadl`
   - OSATE automatycznie sprawdzi składnię

3. **Instancjonowanie:**
   - Prawym na `system implementation CarLightingControlSystem.impl`
   - Wybierz **Instantiate**
   - Powstanie plik `.aaxl2` z instancją

4. **Analiza przepływów:**
   - Prawym na instancję (plik `.aaxl2`)
   - **Analyses → Timing → Check flow latency**
   - Wyniki w oknie **Problems** i **Flow Latency Analysis**

5. **Wizualizacja:**
   - Użyj widoku **AADL Diagrams** do graficznej reprezentacji
   - Flow paths można wyświetlić za pomocą **Show Flow Path**

---

## 📚 Bibliografia i standardy

- **AADL Standard:** SAE AS5506C - Architecture Analysis & Design Language
- **Automotive Standards:** 
  - ISO 26262 (Functional Safety)
  - ECE R7 (Światła pozycyjne, hamowania, kierunkowskazy)
  - ECE R48 (Instalacja urządzeń oświetleniowych)
- **CAN Protocol:** ISO 11898-1 (Controller Area Network)

---

## 📄 Licencja

Model stworzony na potrzeby projektu akademickiego - AGH Kraków.

---

## 📧 Kontakt

**Szymon Bigaj**  
✉️ sbigaj@student.agh.edu.pl

---

*Model opracowany w ramach projektu z zakresu modelowania systemów wbudowanych w języku AADL.*
