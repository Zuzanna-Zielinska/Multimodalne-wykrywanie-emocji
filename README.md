# Multimodal emotion detection
The repository contains a description of my master’s thesis, the result of which is a recurrent neural network for emotion detection. The work involved designing and conducting a study that collected recordings of emotional reactions to stimuli using an eye tracker, camera, and heart rate monitor; processing raw data into training datasets; designing and training neural networks. [The full thesis text is available here.](https://github.com/Zuzanna-Zielinska/Multimodalne-wykrywanie-emocji/blob/main/Multimodalne%20wykrywanie%20emodji.pdf)

Study
During the study, eye movement, facial expressions and heart rate were recorded while participants viewed images from the NAPS database. After each image, participants reported their emotions in a questionnaire.



## Study
During the study, eye movement, facial expressions and heart rate were recorded while participants viewed images from the NAPS database. After each image, participants reported their emotions in a questionnaire.

<p align="center">
<img src=".\images\diagramy-stanowisko pomiarowe.drawio.png" alt="test-bench">
</p>

# Questionnaire
In the questionnaire, participants were asked to describe their emotions verbally and complete the SAM questionnaire.
The SAM questionnaire (Self-Assessment Manikin) is based on the assumption that emotions can be described using three nine-point scales:

•	pleasure – whether the participant feels negative or positive emotions,
<p align="center">
<img src=".\images\wartościowość.JPG" alt="wartosciowosc">
</p>
•	arousal – whether the participant is calm or excited,
<p align="center">
<img src=".\images\pobudzenie.JPG" alt="pobudzenie">
</p>
•	dominance – whether the participant is in control of the emotion or controlled by it.
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

## Architektura sieci
Celem sieci jest podanie wartości ankiety SAM na podstawie tabeli z nagraniami
wyrażającymi emocje. Wartościami wejściowymi są tabele etykiet oraz tabele
danych o 208 kolumnach i różnej liczbie wierszy. Wynika to z tego, że badany
sam decydował, kiedy przejść z obrazka wywołującego emocje do odpowiadania na ankietę.
Dane zostały podzielone na zbiór treningowy (258 próbek), testowy (54 próbek)
oraz walidacyjny (45 próbek).

<p align="center">
<img src=".\images\projekt rozłożony na części2.png" alt="a1">
</p>

Ustawiono rozmiar paczki danych (ang. batch size) na 1 przez różną długość tablic
wejściowych oraz sposób napisania biblioteki tensorflow.

Każda tablica miała różną liczbę wierszy. Żeby to skompensować, użyto ważonych metryk
podczas treningu. Waga była odwrotnie proporcjonalna do długości tablicy.

Wartościami wyjściowymi sieci są 3 parametry od 1 do 9. Ten problem można potraktować
na dwa sposoby. Wartościowość, pobudzenie i dominacja przyjmują dyskretne wartości,
dlatego można stworzyć sieć do klasyfikacji. Z drugiej strony każda z tych wartości
reprezentuje punkt na skali, do przewidywania czego odpowiednia jest regresja. Dodatkowo
jedna sieć może przewidywać wszystkie 3 wartości lub można stworzyć 3 sieci przewidujące
tylko jeden parametr. Biorąc to pod uwagę wytrenowano 20 architektur sieci regresyjnej, które
przewidywały trzy wartości oraz 20 architektur trzech sieci klasyfikujących które
przewidywały jedną wartość. Dla uproszczenia uczenia i testowania sieci, te ostatnie były
połączone jednym wejściem, które rozgałęziało się w trzy sekwencje takich samych warstw,
które nie były później ze sobą połączone.

<p align="center">
<img src=".\images\architektura1.png" alt="arch1">
</p>

<p align="center">
<img src=".\images\architektura2.png" alt="arch2">
</p>

Modele regresyjne były skompilowane z parametrami:

• optymalizator – Adam,

• współczynnik uczenia – 8*10-7,

• funkcja kosztu – błąd średnio-kwadratowy,

• ważona metryka – błąd średnio-kwadratowy.

Modele klasyfikujące były skompilowane z parametrami:

• optymalizator – Adam,

• współczynnik uczenia – 8*10-7,

• funkcja kosztu – rzadka kategoryczna entropia krzyżowa,

• ważona metryka – dokładność.

Oba typy modeli miały na początku warstwę wejściową. Modele regresyjne były
zakończone warstwą w pełni połączoną, zawierającą trzy neurony i sigmoidalną funkcję
aktywacji oraz warstwę skalującą, która przekształca dane z przedziału <0, 1> do <1, 9>. Wynikiem tych sieci były bezpośrednio wyniki ankiety SAM. Ostatnią
warstwą modeli klasyfikujących były trzy warstwy w pełni połączone zawierające dziewięć
neuronów i sigmoidalną funkcję aktywacji. Warstwa ta dawała
prawdopodobieństwo, że dane należą do jednej z klas. Jako odpowiedź sieci była
traktowana klasa z największym prawdopodobieństwem wystąpienia.

Środkowe warstwy różniły się między modelami. Składały się z przynajmniej
jednej warstwy LSTM oraz potencjalnie z różnych kombinacji kolejnych warstw LSTM,
warstwy dropout, która porzuca 0,2 połączeń między jej wejściem a wyjściem, warstwy
konwolucyjnej z jądrem o wielkości 5 oraz warstwy dense – kolejnej warstwy w pełni
połączonej, której funkcją aktywacji jest tangens hiperboliczny. Parametry wszystkich warstw
przedstawia tabela poniżej.

| Nazwa   | Architektura                                   |
|---------|-----------------------------------------------|
| model1  | lstm                                          |
| model2  | lstm, dense, dense                            |
| model3  | dense, dense, lstm                            |
| model4  | dense, dense, lstm, dense, dense              |
| model5  | dense, dense, lstm, dense, dropout, dense     |
| model6  | conv, lstm                                    |
| model7  | conv, conv, lstm                              |
| model8  | conv, dense, conv, lstm                       |
| model9  | conv, dense, conv, dropout, lstm             |
| model10 | conv, lstm, dense, dense                      |
| model11 | conv, conv, lstm, dense, dense                |
| model12 | lstm, lstm                                    |
| model13 | lstm, lstm, dense, dense                      |
| model14 | dense, dense, lstm, lstm                      |
| model15 | dense, dense, lstm, lstm, dense, dense        |
| model16 | dense, dense, lstm, lstm, dense, dropout, dense |
| model17 | dense, dense, lstm, dense, lstm, dense, dense |
| model18 | dense, dense, lstm, dense, lstm, dense, dropout, dense |
| model19 | conv, lstm, lstm                              |
| model20 | lstm, lstm, lstm                              |


Celem warstwy LSTM jest skumulowanie predykcji dla całej tabeli danych w jeden wektor
cech. Jeśli sieć zawierała więcej niż jedną warstwę LSTM, to zamiast tego poprzednie warstwy
przekazywały dalej swoje predykcje dla każdego wiersza tabeli.

Parametr wielkości w bibliotece tensorflow dla warstwy wejściowej i środkowych przed
i włącznie z ostatnią warstwą LSTM wynosił (1, None, 208). Po niej rozmiar warstw
środkowych wynosił (1, 208). Wyjściowe warstwy dla regresji miały wielkość (1, 3)
a dla klasyfikacji (1, 9). Wartość None oznacza różną dla każdej próbki liczbę wierszy tabeli.
Dodatkowo długość tabeli etykiet musi odpowiadać długości tabeli wejściowej, dlatego
przy wczytywaniu danych powielano jej wiersze.

Każdy model trenowano dla 50 epok. Po każdej z nich były wywoływane dwie funkcje. Jeśli
wyniki sieci dla zbioru walidacyjnego były lepsze od poprzednich według odpowiadającej
metryki, to wagi sieci były zapisywane. Metryką dla modeli regresyjnych był błąd średniokwadratowy
a dla klasyfikacji wartość funkcji kosztu. Jeśli wyniki dla zbioru walidacyjnego
nie poprawiły się od 3 iteracji, uczenie sieci było przedwcześnie przerywane. Uczenie każdego
modelu było powtarzane 5 razy i zapisywany był ten, z najlepszymi wynikami.

Dodatkowo te same architektury wytrenowano dla 50 epok, ale bez funkcji wcześniejszego
przerywania uczenia. Uczenie jednego modelu wykonywano tylko raz.

| Warstwa                 | Liczba parametrów sieci |
|-------------------------|-------------------------|
| Input                   | 0                       |
| LSTM                    | 346,944                 |
| Dense (1, None, 208)    | 43,472                  |
| Dense (1, 208)          | 43,472                  |
| Dense (1, 9)            | 1,881                   |
| Dense (1, 3)            | 627                     |
| Dropout                 | 0                       |
| Conv                    | 216,528                 |
| Rescale                 | 0                       |

## Pliki
