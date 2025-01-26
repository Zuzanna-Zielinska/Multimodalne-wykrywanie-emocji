# Multimodalne-wykrywanie-emocji
Repozytorium zawiera skrypty służące do wykonania pracy magisterskiej, której celem
było przeprowadzenie badania zbierającego dane biometryczne uczestników
podczas odczuwania emocji oraz analiza wypełnionych przez nich ankiet dotyczących
przeżywanych uczuć i na tej podstawie zaprojektowanie i wykonanie prototypu systemu
wykrywania emocji.

## Badanie
Jednym z kluczowych problemów w badaniach nad emocjami jest tzw. reaktywność, czyli
zmiana zachowania osoby badanej na skutek świadomości bycia obserwowaną. Żeby zaradzić temu problemowi została zrobiona triangulacja danych, czyli korzystanie z wielu źródeł informacji.
W badaniu zostały nagrane 3 modalności – ścieżka wzrokowa, mimika i puls. Dodatkowo
badani opiszą swoje uczucia w ankiecie.

Podczas badania nagrywano reakcje uczestników na bodźce za pomocą kamery, pulsomierza i eye-trackera.
Bazą bodźców była baza obrazów NAPS.
Po wstępnych przygotowaniach na monitorze przed badanym wyświetlała się seria obrazów
w formacie:

• krzyż skupiający wzrok na środku ekranu – 5 sekund,

• obraz wywołujący emocje – aż do kliknięcia przez badanego,

• plansza z napisem „ankieta”, podczas której badany opisywał swoje emocje – aż do
kliknięcia przez badanego,

• pusta plansza pozwalająca badanemu na wyciszenie się przed kolejnym obrazkiem –
10 sekund.

Cały eksperyment dla jednego badanego trwał od 30 minut do godziny.

<p align="center">
<img src=".\images\diagramy-stanowisko pomiarowe.drawio.png" alt="test-bench">
</p>

## Ankieta

W ankiecie badani byli proszeni, żeby podali słownie emocje, jakie odczuwają oraz wypełnili ankietę SAM.

Ankieta SAM (ang. Self-assesment manikin) opiera się na założeniu, że emocje można
opisać za pomocą trzech dziewięciostopniowych skal:

• skali wartościowości – czy badany odczuwa negatywne czy pozytywne uczucia,
<p align="center">
<img src=".\images\wartościowość.JPG" alt="wartosciowosc">
</p>
• skali pobudzenia – czy badany jest spokojny czy pobudzony,
<p align="center">
<img src=".\images\pobudzenie.JPG" alt="pobudzenie">
</p>
• skali dominacji – czy badany w pełni kontroluje daną emocję, czy jest przez nią
kontrolowany.
<p align="center">
<img src=".\images\dominacja.JPG" alt="dominacja">
</p>

## Baza obrazów
W badaniu została użyta baza NAPS (ang. Nencki Affective Picture System). Jest to baza danych składająca się z 1356 obrazów wywołujących emocje oraz ich średnie
wyniki wartościowości, pobudzenia i dominacji. Każdy obraz jest podzielony na 5 kategorii:
ludzie, twarze, krajobrazy, zwierzęta, przedmioty. Dodatkowo spośród wszystkich zdjęć
wybrano 510 i zbadano jakie, z sześciu podstawowych, wywołują emocje.

<p align="center">
<img src=".\images\NAPS.png" alt="NAPS">
</p>

Spośród ww. 510 zdjęć zostały wybrane dwa zestawy obrazów o najbardziej
zróżnicowanych wynikach według ankiety SAM. Pierwszy zawiera 20 obrazów,
drugi 30. W mniejszym ułożono zdjęcia pod względem wartościowości
w kolejności rosnącej i wybrano 5 obrazów równooddalonych od siebie. Powtórzono czynność
dla pobudzenia i dominacji. Pozostałe obrazy dobrano tak, aby w zestawie znajdowały się
obrazy wywołujące każdą z sześciu podstawowych emocji. Większy zbiór został wybrany
analogicznie, tylko zamiast dla 5, to dla 7 równooddalonych od siebie obrazów.

## Format surowych danych
### Mimika
Każdemu filmowi z nagraniem mimiki była przyporządkowana tabela w pliku CSV, która
w każdym wierszu zapisywała numer klatki i odpowiadający jej znacznik czasowy.
### Puls
W każdym wierszu była zarejestrowana przez pulsomierz wartość tętna oraz odpowiadający
jej znacznik czasowy.
### Ścieżka wzrokowa
Nagrania ścieżki wzrokowej były importowane z programu Tobii Pro Lab w pliku TSV.
Tabela miała 102 kolumny. Zawierały one kompleksowe informacje dotyczące eksperymentu
(np. znaczniki czasowe, nazwa sensora, czas rozpoczęcia eksperymentu), uczestnika
(np. nazwa, płeć, wiek), wyświetlanego obrazu (np. nazwa, rozdzielczość), wyników kalibracji,
ścieżki wzrokowej, wielkości źrenicy oraz stopnia otwarcia oka badanego.

## Przygotowanie danych do uczenia sieci
Przygotowanie danych składało się z następujących kroków:

• Odrzucenie nieistotnych danych.

• Sprawdzenie, czy każda próbka ma wszystkie modalności.

• Ekstrakcja punktów charakterystycznych mimiki.

• Synchronizacja danych.

• Wypełnienie pustych kolumn zerami.

• Zakodowanie słów.

• Normalizacja danych.

Żeby klatkę filmu z mimiką zamienić na wektor punktów charakterystycznych najpierw wykonano detekcję twarzy za pomocą algorytmu HOG (Histogram of gradients) i wytrenowanego klasyfikatora SVM.
Później ekstrakcję wykonano za pomocą algorytmu Kazemiego-Sullivan.

<p align="center">
<img src=".\images\punkty.png" alt="punkty">
</p>

<p align="center">
<img src=".\images\projekt rozłożony na części2.png" alt="a1">
</p>

## Architektura sieci

## Pliki
