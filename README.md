# MODEL STEROWANIA ŚWIATŁAMI SAMOCHODOWYMI
System sterowania światłami samochodowymi napisany w języku AADL

Imię i nazwisko: Szymon Bigaj

e-mail: sbigaj@student.agh.edu.pl

# Opis ogólny modelu:
Model przedstawia System Sterowania Oświetleniem Samochodu (Car Lighting System) zbudowany w języku AADL. System odpowiada za zbieranie danych z urządzeń wejściowych (przełącznik świateł, czujnik natężenia światła otoczenia, manetka kierunkowskazów, przycisk świateł awaryjnych) oraz sterowanie urządzeniami wykonawczymi (jednostki świateł mijania/drogowych i kierunkowskazów).

Model obejmuje:

•	Urządzenia wejściowe reprezentujące fizyczne interfejsy sterowania i czujniki.

•	Proces przetwarzający (LightControlProcess), który zawiera dwa równoległe wątki:

  o	Logikę sterowania światłami (LightModeLogic)
  
  o	Logikę sterowania kierunkowskazami (TurnSignalController)
  
•	Urządzenia wykonawcze, które przyjmują komendy sterujące światłami.

•	Procesor i pamięć ECU, na których działa proces sterujący.

•	Szynę komunikacyjną, przez którą przepływają dane między komponentami.

Celem modelu jest odwzorowanie struktury architektury wbudowanego systemu świateł samochodowych, z oddzieleniem funkcji logicznych od urządzeń fizycznych i zasobów sprzętowych.


# Opis modelu dla użytkownika

System sterowania oświetleniem zapewnia automatyczne i manualne zarządzanie oświetleniem pojazdu. Kierowca korzysta z przełączników i czujników, a system dokonuje odpowiednich decyzji dotyczących oświetlenia:

•	Przełącznik świateł umożliwia ręczne włączenie świateł.

•	Czujnik światła otoczenia może automatycznie włączyć światła, gdy warunki tego wymagają (np. zmrok, tunel).

•	Manetka kierunkowskazów pozwala aktywować kierunkowskazy prawe/lewe.

•	Przycisk świateł awaryjnych włącza jednocześnie oba kierunkowskazy.

# Spis komponentów AADL

Dane (data):

1. LightCommand - Reprezentuje komendę wysyłaną do urządzeń wykonawczych (np. włącz/wyłącz	światła, tryb).
   
2. SensorData - Dane wejściowe z czujników i przełączników (np. pozycja przełącznika, poziom światła).
   
3. ConfigData - Służy do przechowywania konfiguracji systemu, używana przez proces i pamięć.
   
Urządzenia wejściowe i wyjściowe (device):

1. HeadlightSwitch - Przełącznik świateł – przekazuje dane pozycji manetki ustawianej przez użytkownika.
   
2. AmbientLightSensor - Czujnik światła otoczenia – informuje system czy otoczenie jest ciemne lub jasne.
   
3. TurnSignalLever - Manetka kierunkowskazów – sygnały dla kierunkowskazów lewy/prawy.
   
4. EmergencyButton – Przycisk świateł awaryjnych – włącza oba kierunkowskazy.
   
5. HeadlightUnit - Jednostka wykonawcza świateł pojazdu – odbiera polecenia sterujące
     
6. TurnSignalUnit - Jednostka wykonawcza kierunkowskazów – odbiera polecenia sterujące
    
Proces (process)

1. LightControlProcess - Główny proces sterowania oświetleniem. Łączy wejścia, logikę przetwarzania i wyjścia.
   
Podzespoły procesu:

1. ModeLogicThread (LightModeLogic) – logika trybów oświetlenia (auto/manual).
   
2. TurnSignalThread (TurnSignalController) – logika kierunkowskazów i awaryjnych.
   
3. ConfigDataStore (ConfigData) – lokalna pamięć konfiguracji.
   
Wątki (thread)

1. LightModeLogic - Obsługuje światła główne (np. włączanie w zależności od jasności i przełącznika).
   
2. TurnSignalController - Obsługuje kierunkowskazy oraz tryb awaryjny.
   
Pamięć (memory)

1. ECU_Memory - Reprezentuje pamięć jednostki sterującej (ECU), pozwala procesowi uzyskać dostęp do konfiguracji.
   
Procesor (processor)

1. LightingECU - Procesor ECU obsługujący proces sterowania oświetleniem, zdefiniowany jako działający w trybie FIFO.

Szyna komunikacyjna (bus)

1. CANBus - Moduł komunikacyjny, przez który przesyłane są dane między komponentami.
   
System (system)

1. CarLighting - Model całego systemu oświetlenia – łączy wszystkie urządzenia i komponenty przetwarzające.
