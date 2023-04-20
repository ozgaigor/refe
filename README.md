# START

# 1. Rozwiń skrót DOTS
  Jest ro zrezygnowanie z Monobehavior
# 2. Filozofia optymalizacyjna.
  Wszystko sprowadza się do zoptymalizowania zużycia CPU poprzez układanie danych w struktury które są szybko przetwarzane oraz wielowątkowość procesora.
# 3. Jest to zestaw paczek ( packet menager ) które możemy dodawać i odejmować. Pozostałe paczki powodują, że podstawowe pakiety z unity są kompatybline z DOTS.

- Psychic - Fizyka
- Animation -  Animacje
- Netcode metodą komunikacji między dwoma lub więcej komputerami, z których każdy ma uruchomioną tę samą grę.
- DSPGraph - System Audio
- DOTS Runtime - instant apps - Google play instant

# 4. Więc jak działa DOTS?
  Trzy paczki z ww. slajdów optymalizują wszystko pod względem zużycia CPU
  
# 5. ECS
  Tworzymy obiekt w Unity, tworzymy game object, tworzymy monoBehavior, co się tam znajduje?
  Informacja o stanie obiektu, 
  Piłka - [ pozycja, prędkość, kolor ]
  
  ESC rozbija informacje na komponenty, systemy i entity
  
  Systemy nie wiedzą nic o danych, nie wiedzą jaki jest stan. Wiedzą o typie i co zrobić z danymi
  Komponenty nie wiedzą jak dane będa przetwarzane ale wiedzą jakie dane zawierają i jaką maja wartość
  Zamiast jednej piłki która zawiera pozycje prędkość i kolor, mamy 3 komponenty.
  Encje są tagami
  
  Dlaczego układanie danych jest bardziej optymalne.
  Załóżmy że chcemy kopnąc naraz 100 piłek:
  W unity musimy wczytać każdy obiekt piłki ich prędkość pozycje i kolor. 
  Wczytujemy całość mimo że nie potrzebujemy.
  
  W CPU jeżeli wczyutujemy pamięc która jest po sobie jest ona dodatkowo zoptymalizowana.
  
  W ESC zamaist obiektów piłek mamy ułożone w jeden ciąg komponenty prędkość i komponenty pozycja.
  
  System pobiera tylko komponenty tego samego typu w jednym ciągu.
 
# 6. Archetypy
  Informacja o tym że encja ma konkretny zestaw komponentów
  
  Entities pakuje encje w chunki za pomocą instancjonowania archetypów. Chunki są dodatkowo optymalizowane w ten sposób że można jest w łatwy sposób     wciagnąc do pamięci CPU.
  Unity wie jakie systemy tworzymy.
  Tworzymy system kopnięcia piłki który używa tylko pozyji i prędkości. Utworzymy tym samym subarchetyp zawierający tylko te dwa komponenty. I osoby subarchetyp ktory stworzy encje dla koloru.
  
  Ale jak stworzyć Entity?
  Mamy klase IComponentData. Wszystko co dziedziczymy po IComponentData jest komponentem i może być używane jako nośnik dany dla encji. Ograniczeniem jest to że IComponentData musi być strukturą(struct)
  
  Brzmi to wszystko jak wyłącznie pisanie w kodzie lecz nie do końca. Unity udostępnia tak zwane Conversion Workflow.
  
  Conversion Workflow - umożliwia konwertowaniee obiektów na scenie na encje.
  
  Tag nad strukturą powoduje że unity generuje komponent który jak monoBehavior pozwoli nam podpiąc pod obiekt na scenie. Kiedy będzie ona konwertowana na encje automatycznie zostanie stworzona encja z obiektu z podpiętym komponentem.
  
  Subsceny - ograniczniem są właśnie subsceny. Jest to osobna scena podpoięta pod scene gdzie są tylko informacje które chcemy konwertować. Nie konwertujemy na encje np. oświetlenie czy też tło.
  
  
# 7. Job system
  Pozwala on nam na łatwy multithreading
  W unity są dwie rzeczy na które musimy uważać, gdyż są to spore problemy.
  
  Jednym z nich jest Context Switching - Tworzymy więcej wątków niz mamy rdzeni logicznych.
  Mamy powiedzmy 8 rdzeni logicznych i dodajemy 9. W tym momencie jeden z wątków zostanie zatrzymany, zapisany i wyrzucony na rzecz 9 wątku. W momencie kiedy 9 wątek się zakończy, zastąpiony zostanie wczytany i ponownie uruchomiony. Co powoduje że system zwalnia.
  
  Job system zamiast wątków tworzy joby, czyli małe pakiety i ustawia je w kolejke. Tworzy on wtedy tylko tyle wątków ile rdzeni ( 1 thread na core logiczny ) 
  
  Drugim problemem jest Race Conditions. Jest to przypadek w którym próbujemy wykonać 2 operacje na tych samych danych, mogą one wtedy trafić na ten sam wątek. W takim wypadku wynik będzie zależny od tego która operacja zakończy się jako pierwsza.
  
  " W banku mamy 500 złotych i musimy dokonać 2 przelewy(A i B) po 300 złotych. Nie możemy opłacić obu na raz. Przy użyciu sekwencyjnego przetwarzania danych jako pierwszy wykona się przelew A a B nie dojdzie do skutku. W przeciwnym wypadku nie używając sekwencyjnego przetwarzania danych obie operacje wykonają się jednocześnie co spowoduje przekroczenie 0 i wejście na ujemny stan konta."
  
  Takich konfliktów może byc sporo a z pomoca przychodzi Safety System.
  
  W momencie kiedy może dojść do Race Condition program wyrzuci nam na ekran błędy. Dodatkowo możemny ustawić Dependencies. Nasze joby mają wtedy wiedze które dane mają wykonać się jako pierwsze. 
  
  Native Containers są zastępstwem C# stuktur. Problemem jest że musimy sami zarządzać pamięcią.
  Ułatwia nam to Alokator w którym ustalamy na jak długo ma tą pamięć alokować.
  
  Temp: Klatka lub krócej
  TempJob: Czas trwania Joba
  Persistent: Zastosowanie między jobami
  
  DisponseSentinel - błędy jeżeli zapomnimy alokowac pamięc.
  
  Jak tworzyć Joby?
  
  Dziedziczymy po interfejsach IJob lub IParallelJobFor i na obiekcie Job wykonujemy metode Schedule(Skedżul) a Job System zrobi reszte sam. 
  
  IJob - stworzenie jednego joba
  IParallelJobFor - [Array] zamiast joba który iteruje przez Array tworzy x jobów aby to najlepiej zoptymalizować.
  
  # 8. Burst Compiler
  
  Kompilator, zmieniający kod napisany w języku C#, na kod maszynowy o wysokiej
efektywności, wykorzystujący możliwości nowoczesnych procesorów.
Burst Compiler działa w bardzo dobrej symbiozie z C# Job System.
W zależności od platformy, na której budowana jest aplikacja (PC, Android itd.),
istnieją pewne zabiegi optymalizacyjne, które można wykonać, aby przyspieszyć
działanie programu. Naturalnie Burst Compiler robi to za nas.


Jak użyć Burst Compilera.

Wystaczy dodać atrybut [BurstCompile] niestety jest ale...
ma wiele wyjąctków:

- wymaga uzycia job system, ale nie musi być on wielowątkowy
- wymaga natywnych narzędzi do zbudowania
- działa tylko z Unity.Mathematics
- nie działa z referencjami
- nie działa ze zmiennymi statycznymi

Jak to wszsytko spiąć ze sobą?

Entities.ForEach()

Każda encja która zawiera komponenty i ten konponent ma wartość większą niż jeden ustaw na 1. 

Unity przeiteruje po chunkach, znajdzie chunki z podanymi komponentami ,sprwadzi wartności i wykona operacja na każdej z nich. 
Wpisujemy na koniec ... w ten sposób tworzy on Parallel joba i automatycznie kompiluje to przez burst.
