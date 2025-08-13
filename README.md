# Bioinformatyka 
## Semestr: 6

## Projektu: Sekwencjonowanie przez hybrydyzację (SBH)

## Autor: Szymon Warguła 158037

## Linki:
do kodu programów:  

https://github.com/Kosinir/Bioinformatyka

do całego arkusza kalkulacyjnego: 

https://docs.google.com/spreadsheets/d/1j3mLXDZVZH6OMmuPjRM1Vn6ATKVlT9rWCe5lAFwuHHQ/edit?usp=sharing

## Opis algorytmu

Algorytm rozwiązujący problem Sequencing by Hybridization (SBH) ma na celu rekonstrukcję oryginalnej sekwencji DNA na podstawie zbioru oligonukleotydów (k-merów) o ustalonej długości. 

Problem biologiczny zostaje sprowadzony do problemu grafowego, w którym wierzchołkami są k-mery, a krawędzie odpowiadają ich możliwym nakładkom. 

Dzięki zastosowaniu metaheurystyki kolonii mrówek (ACO), wzmacnianej lokalną optymalizacją i algorytmem Dijkstry, możliwe jest skuteczne odtworzenie sekwencji, nawet w obecności błędów.

## Dane wejściowe

Program otwiera plik tekstowy i czyta z niego:

n – długość sekwencji DNA do odtworzenia.

k – długość oligonukleotydów (k-merów) w spektrum (7 ≤ k ≤ 10).

start_oligo – znany początkowy k-mer (wierzchołek startowy grafu).

num_neg_errors – liczba brakujących k-merów (błędy negatywne).

has_repeats – flaga: czy wśród błędów są te wynikające z powtórzeń (wtedy część k-merów można odwiedzić więcej niż raz).

num_pos_errors – liczba fałszywych k-merów (błędy pozytywne).

lista k-merów – zbiór oligonukleotydów długości k.

## Budowa grafu
Algorytm rozpoczyna się od przekształcenia spektrum k-merów w graf: każdy fragment długości k staje się wierzchołkiem, a między dowolnymi dwiema sekwencjami, które zachodzą na siebie przynajmniej jedną literą, wstawiana jest skierowana krawędź wyznaczona przez długość ich nakładki. 

Waga tej krawędzi odpowiada liczbie nowych nukleotydów, które musimy dodać do odtwarzanej sekwencji (czyli k minus długość nakładki). 

Do każdego istniejącego połączenia przypisywana jest początkowa wartość feromonu, co pozwala na późniejsze wzmacnianie lub osłabianie poszczególnych dróg w procesie optymalizacji.

## Inicjalizacja


Następnie inicjalizowane są “mrówki” – każda z nich przechowuje własną trasę (kolejność odwiedzanych wierzchołków), bieżącą długość odtworzonej sekwencji, sumaryczny koszt (suma wag krawędzi) oraz licznik odwiedzin poszczególnych wierzchołków, uwzględniający limit dopuszczalnych powtórzeń węzłów wynikający z liczby błędów negatywnych i informacji o powtórzeniach. 

Wszystkie mrówki startują z tego samego wierzchołka początkowego, który odpowiada znanemu kawałkowi sekwencji startowej.


## Etap 1 działania algorytmu

Każda iteracja algorytmu składa się z etapu „budowy trasy” dla każdej mrówki oraz etapu aktualizacji feromonów. 

Podczas budowy trasy mrówka porusza się po grafie, krok po kroku dopisując do rekonstruowanej sekwencji kolejne fragmenty. 

W pierwszej kolejności próbuje zawsze wykorzystać łuki o wadze 1 (najczęściej, zależności od parametru Beta) (czyli  maksymalne nakładanie się fragmentów), najpierw preferując te, które prowadzą do węzłów jeszcze nieodwiedzonych („świeże”), a jeśli takich zabraknie, akceptując wejście do węzła już użytego, o ile nie przekracza on wyznaczonego limitu powtórzeń. O ile jeszcze możliwe, porusza się po takich maksymalnych nakładkach aż do momentu, gdy żaden z węzłów docelowych nie spełnia tych kryteriów. 

Wtedy w obrębie łuków o malejącej nakładce (1, 2, 3, …, k–1) ponownie wyszukuje najpierw węzły świeże, a następnie dopuszcza re-usage, aż znajdzie jakąkolwiek krawędź prowadzącą dalej.


## Etap 2 działania algorytmu (tzw. jak algorytm radzi sobie z pułapkami)

Gdy i to zawiedzie – czyli mrówka utknie w węźle, z którego nie wychodzą żadne dopuszczalne krawędzie o nakładce ≥ 1 – używany jest algorytm Dijkstry: z bieżącego węzła budowana jest najkrótsza ścieżka do kilku wybranych kandydatów, spośród tych jeszcze nieodwiedzonych lub dopuszczalnych do ponowienia, wybierając tę o najmniejszym koszcie sumarycznym. 

Znaleziona podścieżka do wybranego węzła jest w całości doklejona do trasy mrówki, co pozwala przełamać lokalne zastoje i wykorzystać krawędzie o większej wadze do przesunięcia się w nowe obszary grafu. 

Jeśli i ta metoda nie doprowadzi do wznowienia postępu (nie ma żadnych dopuszczalnych celów), wreszcie wykonywany jest tzw. „skok wirtualny” – przejście do dowolnego węzła nie przekraczającego limitu powtórzeń, traktowane jak nakładka zerowa, co pozwala uniknąć zapętlenia lub przedwczesnego zakończenia budowy sekwencji.


## Etap 3 działania algorytmu - optymalizacja

Budowa trasy dla danej mrówki trwa tak długo, aż długość odtworzonej sekwencji osiągnie docelowe **n** nukleotydów (liczone jako początkowe **k** liter plus suma wag kolejnych przejść) lub osiągnięty zostanie limit kroków równy liczbie wszystkich k-merów. 

Na koniec, jeżeli trasa faktycznie osiągnęła długość co najmniej **n**, stosowana jest lokalna optymalizacja typu 2-opt, w której losowo wybrane fragmenty trasy są odwracane i sprawdzane pod kątem tego, czy mimo zmienionej kolejności krawędzie nadal istnieją (nakładka ≥ 1) i czy łączny koszt spada. Jeśli tak, trasa zostaje zastąpiona lepszą wersją.


## Finalizacja

Po zakończeniu budowy tras wszystkich mrówek obliczana jest statystyka ukończonych rozwiązań, a feromony na wszystkich krawędziach częściowo odparowują (mnożenie przez 1–ρ). 

Następnie każda mrówka, która odtworzyła pełną sekwencję, wnosi wkład w postaci deponowania feromonów proporcjonalnie do odwrotności całkowitego kosztu trasy – im lepsza (niższa suma wag), tym więcej feromonu. Dodatkowo stosowany jest depozyt elitarystyczny z najlepszej w dotychczasowej historii trasy, co pozwala przyspieszyć konwergencję. 

Cały cykl budowy tras i aktualizacji feromonów powtarza się wielokrotnie, aż zostanie przekroczony limit czasu lub liczba iteracji (w testach jest limit czasu co ma swoje wady i zalety ale o tym później).

Dzięki tej wieloetapowej strategii łączącej deterministyczne przechodzenie po maksymalnych nakładkach, globalne poszukiwanie za pomocą Dijkstry, agresywne przełamywanie zastoju „skokami wirtualnymi”, heurystykę ACO oraz lokalne poprawki 2-opt, algorytm radzi sobie zarówno z idealnymi zestawami k-merów, jak i z instancjami zawierającymi zarówno braki fragmentów, jak i fałszywe dodatki, przybliżając w praktyce oryginalną sekwencję DNA.

## Testy i pomiary algorytmu

Mamy 2 rodzaje testów: Testy parametrów algorytmu i Testy parametrów instancji.

Wszystkie pomiary (danego n) to wyniki średnich 50 instancji.

Oś Y reprezentuje miarę Levenshteina, a oś X badany parametr.

Każdy test jest dla instancji n = [300, 500, 700] czyli małe, średnie i duże instancje.

Linia trendu - pomarańczowa linia.

# Testowanie parametrów algorytmu 

## Badany parametr: Ilość Mrówek (ants)

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>Ants/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, num_neg = 30 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, time = 10 (s)</td>
    <td>50</td>
    <td>35,78</td>
    <td>149,84</td>
    <td>279,66</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, num_neg = 50 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, time = 10 (s)</td>
    <td>100</td>
    <td>35</td>
    <td>154,06</td>
    <td>291,38</td>
  </tr>
  <tr>
    <td>n = 700, k = 8, num_neg = 70 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, time = 10 (s)</td>
    <td>150</td>
    <td>37,58</td>
    <td>173,6</td>
    <td>285,48</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>250</td>
    <td>39,52</td>
    <td>173,94</td>
    <td>281,92</td>
  </tr>
</table>

<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/85fadf32-5160-48b5-b43d-44d17badf6cc" />



<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/0a81f4cc-629e-4939-a841-ef6f4c6940b2" />



<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/e585884d-ade4-4103-b151-4b0322ba502a" />


## Wnioski i obserwacje:


Dla wszystkich instancji optymalna liczba mrówke to 50, im mniejsza liczba mrówek tym więcej iteracji może wykonać algorytm w ciągu 10 s. (co jest dość małym czasem) ale dla n = 700 można zauważyć że najmniejszy wyniki ma dla ants = 50 i podobny wynik dla ants = 250.

Dla ants = 50, algorytm w ciągu 10 sekund może wykonać najwięcej iteracji (prawdopodobnie do 3 iteracji), w pozostałych przypadkach tylko 1 iteracje (wina sprzętu, algorytmu i  parametrów), ale dla tej 1 iteracji, 250 mrówek jest w stanie osiągnąć podobne wyniki co dla 50 mrówek.

## Badany parametr: Czas (time)

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>Time/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, num_neg = 30 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100</td>
    <td>5</td>
    <td>63,6</td>
    <td>180,3</td>
    <td>291,88</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, num_neg = 50 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100</td>
    <td>20</td>
    <td>40,56</td>
    <td>138,44</td>
    <td>270,02</td>
  </tr>
  <tr>
    <td>n = 700, k = 8, num_neg = 70 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100</td>
    <td>35</td>
    <td>29,78</td>
    <td>127,08</td>
    <td>258,08</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>50</td>
    <td>27,88</td>
    <td>123,28</td>
    <td>243,82</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>65</td>
    <td>25,2</td>
    <td>114,94</td>
    <td>240,88</td>
  </tr>
</table>

<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/4744008d-5571-4e59-8ec6-2e3630b29ec0" />

<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/673e14f4-f776-4f0a-9517-647751c46785" />

<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/7d0edb3d-2fbd-47bd-9bce-b79ef8141b0c" />

## Wnioski i obserwacje:

Wniosek prosty i oczywisty - im więcej czasu tym więcej iteracji, tym lepsze wyniki, gdzie pomiędzy 50 a 65 sekund algorytm znajduje optimum lokalne.

## Badany parametr: Beta

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>Beta/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, num_neg = 30 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>1</td>
    <td>125,86</td>
    <td>238,72</td>
    <td>334,76</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, num_neg = 50 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>3</td>
    <td>73,46</td>
    <td>229,54</td>
    <td>339,56</td>
  </tr>
  <tr>
    <td>n = 700, k = 8, num_neg = 70 (10%), repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>5</td>
    <td>56,04</td>
    <td>196,18</td>
    <td>325,2</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>7</td>
    <td>52,86</td>
    <td>164,08</td>
    <td>304,42</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>9</td>
    <td>48,54</td>
    <td>162,28</td>
    <td>282,9</td>
  </tr>
</table>

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/12ba8e52-2f3c-4918-870d-7b92017c684b" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/bbe97880-96ef-4b42-ac0f-52d7b72feef0" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/9de24aaf-05d2-43c4-82fd-6ea233d3f292" />


## Wnioski i obserwacje:

Tutaj również, im większy parametr (Beta) tym lepsze wyniki - dla małych wartości Beta, mrówki są bardziej skłonne do eksploracji najgorszych wierzchołków, ale im większa Beta tym mniej je eksplorują (te najgorsze) i jak widzimy dla Beta = 9 otrzymujemy najlepsze wyniki bo mrówki częściej wybierają wierzchołki o wadze 1 - jest to najbardziej widoczne dla instancji n = 700 bo tutaj algorytm jest w stanie 10 sekund wykonać 1-2 iteracji, gdzie dla n = [300, 500] szybko znajduje optimum lokalne.

# Testowanie parametrów Instancji

## Badany Parametr: błędy negatywne (num_neg)

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>num_neg(%)/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>3</td>
    <td>30,74</td>
    <td>147,3</td>
    <td>288,04</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>6</td>
    <td>34,3</td>
    <td>149,42</td>
    <td>286,76</td>
  </tr>
  <tr>
    <td>n = 700, repeat = True, num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>9</td>
    <td>33,1</td>
    <td>164,98</td>
    <td>289,68</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>12</td>
    <td>45,38</td>
    <td>158,74</td>
    <td>292,18</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>15</td>
    <td>46,12</td>
    <td>157,82</td>
    <td>294,74</td>
  </tr>
</table>

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/b41b39ce-2559-4b41-b27b-f0c958f9fb14" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/a49b1019-812d-4217-939a-86f7b5a05bd9" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/0d6c41b8-5e0e-4b77-b167-429068cdc29e" />


## Wnioski i obserwacje:

Im więcej błędów w instacji tym gorsze wyniki - co jest oczywiste ale dlaczego wyniki podobne? 

Prawdopodobnie wina za wysokiego parametru Beta (= 8) który sprawia że mrówki bardzo silnie preferują najkrótsze przejścia - najbardziej oczywiste ścieżki w grafie przez co poruszają się po podobnych trasach - co oznacza że te błędy są w pewnym stopniu “ignorowane” bo mrówki trzymają się lokalnie najlepszych przejść.

## Badany Parametr: błędy pozytywne (num_pos)

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>num_pos(%)/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, repeat = True, num_neg = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>3</td>
    <td>30,12</td>
    <td>164,42</td>
    <td>296,78</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, repeat = True, num_neg = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>6</td>
    <td>52,24</td>
    <td>173,88</td>
    <td>301,74</td>
  </tr>
  <tr>
    <td>n = 700, repeat = True, num_neg = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>9</td>
    <td>68,14</td>
    <td>188,62</td>
    <td>305,16</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>12</td>
    <td>78,62</td>
    <td>190,24</td>
    <td>307,56</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>15</td>
    <td>85,9</td>
    <td>197,24</td>
    <td>314,5</td>
  </tr>
</table>

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/44f0661e-d854-45f9-aa6a-8bef9631bca1" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/f2822cbf-e01b-447b-9f20-53b89712180a" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/be6528cb-e1e2-4bdd-a116-eb76f15eb557" />





## Wnioski i obserwacje:

Tutaj podobna sytuacja - im więcej błędów tym gorsze wyniki, tylko tutaj są one bardziej widoczne.

Dlaczego? 

Fałszywe k-mery dodają szum do grafu i tworzą nieistniejące połączenia, które zmyłkowo przyciągają mrówki - zwłaszcza przy wysokiego parametru beta. 

Im więcej takich błędów, tym większa szansa że algorytm zbuduje niepoprawną trasę. Sam efekt jest silniejszy niż przy błędach negatywnych, ponieważ mrówka może zupełnie “odpłynąć” od poprawnej sekwencji, zamiast tylko chwilowo się zatrzymać.

## Badany Parametr: K

<table>
  <tr>
    <th>Instancja</th>
    <th>Input</th>
    <th colspan="4">Average Levenshtein</th>
  </tr>
  <tr>
    <th></th>
    <th></th>
    <th>num_pos(%)/n</th>
    <th>300</th>
    <th>500</th>
    <th>700</th>
  </tr>
  <tr>
    <td>n = 300, k = 8, repeat = True, num_neg = 30 (10%), num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>7</td>
    <td>79,12</td>
    <td>204,66</td>
    <td>325,1</td>
  </tr>
  <tr>
    <td>n = 500, k = 8, repeat = True, num_neg = 50 (10%), num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>8</td>
    <td>42,46</td>
    <td>157,78</td>
    <td>288,62</td>
  </tr>
  <tr>
    <td>n = 700, repeat = True, num_neg = 70 (10%), num_pos = 0</td>
    <td>alpha = 0.7, beta = 8.0, rho = 0.8, q = 20.0, tau0 = 0.7, ants = 100, time = 10 (s)</td>
    <td>9</td>
    <td>15,72</td>
    <td>116,44</td>
    <td>241,56</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>10</td>
    <td>6,18</td>
    <td>64,74</td>
    <td>181,66</td>
  </tr>
</table>

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/2aa476b0-deec-47ea-9863-506a88790ad8" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/34d1db64-bd0b-4ea8-a591-1d0286a4ab76" />

<img width="594" height="346" alt="image" src="https://github.com/user-attachments/assets/7c5e3b8d-4d4d-4e0a-a151-98dff6caf2ed" />

## Wnioski i obserwacje

Wraz ze wzrostem długości k-merów, jakość rekonstrukcji wyraźnie się poprawia. 

Dlaczego?

Dłuższe k-mery są bardziej unikalne, bardziej odporny na błędy pozytywne i negatywne, więc graf ma mniej mylących połączeń i łatwiej prowadzi mrówki właściwą ścieżką.
