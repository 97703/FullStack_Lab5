<img src="https://github.com/97703/DockerLab3/blob/main/rysunki/loga_weii.png?raw=true" style="width: 40%; height: 40%" />

> **Programowanie Full-Stack w Chmurze Obliczeniowej**

      dr inż. Sławomir Wojciech Przyłucki

<br>
Termin zajęć:

      środa, godz. 11:30,

Imię i nazwisko:

      Paweł Pieczykolan,
      II rok studiów magisterskich, WOiSI 2.3,


# Limity zasobów w K8s
<p align="justify">W zadaniu utworzono dwie przestrzenie nazw K8s z kontrolą zasobów: ns-dev o ograniczonych limitach oraz ns-prod z dwukrotnie większą pulą CPU i RAM. Zweryfikowano działanie Podów nginx spełniających oraz przekraczających limity, potwierdzając poprawność konfiguracji przedziałów zasobów.</p>

<p align="justify">Zadanie zostało zrealizowano na maszynie wirtualnej na której działał system Ubuntu 24.04 LTS.</p>

## 1. Utworzenie przestrzeni nazw
<p align="justify">W pierwszym etapie zadania utworzone zostały dwie przestrzenie nazw:
ns-dev dla środowiska deweloperskiego oraz ns-prod dla środowiska produkcyjnego.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen1.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 1. Utworzenie wymaganych przestrzeni nazw</i>
</p>

## 2. Konfiguracja limitu zasobów
<p align="justify">W środowisku <code>ns-dev</code> ograniczono zasoby do 1 rdzenia CPU i 1024 megabitów pamięci RAM maksymalnie oraz 10 POD-ów. W środowisku <code>ns-prod</code> zasoby (CPU, RAM) zostały ustawione na dwukrotność wartości z <code>ns-dev</code>. Dodatkowo przygotowano obiekt <code>LimitRange</code> z pliku <code>limitrange.yaml</code>, który wymusza limity dla Podów uruchamianych bez zdefiniowanych żądań i limitów zasobów.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen3.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 2. Utworzenie obiektów Quota oraz LimitRange ograniczających zasoby dla przestrzeni nazw</i>
</p>

<p align="justify">Obiekt typu <code>LimitRange</code> o nazwie `memlimits` dla przestrzeni nazw <code>ns-dev</code> definiuje limity zasobów dla kontenerów w następujący sposób:</p>

- default – domyślny limit, czyli ile kontener może maksymalnie używać, jeśli nie poda własnego limitu: 100m CPU i 128Mi pamięci.
- defaultRequest – domyślne żądanie zasobów, czyli ile kontener dostanie gwarantowane, jeśli nie poda własnego request: 100m CPU i 128Mi pamięci.
- max – maksymalny limit, czyli górna granica zasobów możliwych do przydzielenia kontenerowi: 200m CPU i 256Mi pamięci.

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen2.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 3. Zawartość pliku limitrange.yaml</i>
</p>

## 3. Deployment no-test – przekroczenie limitów
<p align="justify">Utworzono obiekt <code>Deployment</code> o nazwie <code>no-test</code> w przestrzeni nazw <code>ns-dev</code> na bazie obrazu nginx. Deployment został celowo skonfigurowany tak, aby przekraczał ustalone limity zasobów CPU i pamięci. Celem tego działania było sprawdzenie, czy wprowadzone ograniczenia w ramach <code>LimitRange</code> i <code>ResourceQuota</code> skutecznie blokują uruchamianie kontenerów, które próbują wykorzystać więcej zasobów, niż zostało przydzielone. Dzięki temu można zweryfikować, że mechanizmy kontroli zasobów w Kubernetes działają zgodnie z oczekiwaniami.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen4.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 4. Zawartość pliku no-test.yaml</i>
</p>

<p align="justify">Po uruchomieniu Deploymentu sprawdzono jego status. Ze względu na przekroczenie limitów CPU i pamięci, Pod-y nie mogły zostać uruchomione poprawnie. W logach i wynikach komend widać było błędy dotyczące braku możliwości przydzielenia żądanych zasobów. To zachowanie potwierdziło skuteczność konfiguracji LimitRange i ResourceQuota, a także pokazało, że Kubernetes efektywnie chroni środowisko przed nadmiernym zużyciem zasobów w przestrzeni <code>ns-dev</code>.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen6.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 5. Uruchomienie Deploymentu no-test</i>
</p>

<p align="justify">Po wydaniu polecenia uruchamiającego Deployment widać było, że Pod-y nie mogą zostać uruchomione. Kontenery próbowały zużyć więcej CPU i pamięci, niż pozwalały na to limity zdefiniowane w <code>LimitRange</code> oraz <code>ResourceQuota</code>. Ten etap pozwolił zweryfikować, że mechanizm kontroli zasobów skutecznie blokuje uruchomienie nieprawidłowo skonfigurowanych kontenerów i chroni środowisko przed nadmiernym obciążeniem.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen5.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 6. Przekroczenie limitu kontenera w deploymencie no-test</i>
</p>

## 4. Deployment yes-test – spełnienie limitów
<p align="justify">Kolejnym krokiem było utworzenie Deploymentu <code>yes-test</code>, który spełniał wszystkie wymagania dotyczące wykorzystania zasobów CPU i pamięci. Deployment ten miał na celu pokazanie, że Pod-y mieszczące się w zadeklarowanych limitach uruchamiają się poprawnie i działają stabilnie w przestrzeni <code>ns-dev</code>.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen8.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 7. Zawartość pliku yes-test.yaml</i>
</p>

<p align="justify">W trakcie uruchamiania sprawdzano status Pod-ów, aby upewnić się, że działają poprawnie.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen7.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 8. Uruchomienie Deploymentu yes-test</i>
</p>

<p align="justify">W kolejnym kroku użyto polecenia <code>kubectl describe</code>, aby szczegółowo zweryfikować przydzielone żądania i limity zasobów dla każdego Poda.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen9.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 9. Wynik polecenia describe dla yes-test</i>
</p>

## 5. Deployment zero-test – brak deklaracji żądań i limitów
<p align="justify">Ostatnim testem był Deployment <code>zero-test</code>, który nie posiadał deklaracji żądań i limitów zasobów. Celem było sprawdzenie, czy obiekt <code>LimitRange</code> automatycznie przypisuje domyślne wartości do Pod-ów uruchamianych bez własnych deklaracji. Test ten pozwala upewnić się, że nawet w przypadku braku jawnych wartości zasoby są kontrolowane i mieszczą się w przyjętych granicach, co zwiększa stabilność środowiska.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen10.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 10. Zawartość pliku zero-test.yaml</i>
</p>

<p align="justify">Deployment został uruchomiony w przestrzeni <code>ns-dev</code> przy użyciu polecenia <code>kubectl apply -f zero-test.yaml</code>. Po jego uruchomieniu sprawdzono status Pod-ów oraz ich przydzielone zasoby. Dzięki <code>LimitRange</code>, każdy kontener otrzymał domyślne wartości CPU i pamięci, nawet jeśli nie zostały one jawnie zadeklarowane w pliku YAML. To pozwoliło upewnić się, że mechanizm automatycznego przydzielania zasobów działa zgodnie z założeniami i chroni środowisko przed niekontrolowanym zużyciem zasobów.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen11.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 11. Uruchomienie Deploymentu zero-test</i>
</p>

<p align="justify">Opis Deploymentu potwierdził, że Pod-y zostały poprawnie uruchomione. Dodatkowo sprawdzono szczegóły pojedynczego Poda, aby zweryfikować, że przydział domyślnych wartości CPU i pamięci jest prawidłowy, co zapewnia bezpieczeństwo i przewidywalność środowiska.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen12.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 12. Wynik polecenia describe dla zero-test</i>
</p>
  
<p align="justify">Szczegółowe sprawdzenie pojedynczego Poda pokazało, że domyślne wartości CPU i pamięci zostały poprawnie przypisane zgodnie z <code>LimitRange</code>, co potwierdza skuteczność mechanizmu automatycznego przydzielania zasobów w przestrzeni <code>ns-dev</code>.</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/97703/FullStack_Lab5/main/rysunki/screen13.png" style="width: 70%; height: 70%" /></p>
<p align="center">
  <i>Rys. 13. Wynik polecenia describe dla Poda zero-test</i>
</p>

## 6. Podsumowanie
<p align="justify">W ramach zadania utworzono dwie przestrzenie nazw K8s: <code>ns-dev</code> i <code>ns-prod</code>, z różnymi limitami zasobów. W przestrzeni <code>ns-dev</code> ograniczono liczbę Pod-ów oraz maksymalne zużycie CPU i pamięci. Przestrzeń nazw <code>ns-prod</code> posiadała dwukrotnie większe zasoby. Przeprowadzone testy Deploymentów <code>ns-dev</code>  pokazały, że: Deployment przekraczający limity <i>no-test</i> nie uruchomił się poprawnie, Deployment spełniający limity <i>yes-test</i> działał stabilnie, a Deployment bez deklaracji limitów <i>zero-test</i> działał poprawnie oraz otrzymał domyślne wartości zasobów. Zadanie potwierdziło skuteczność konfiguracji <code>ResourceQuota</code> i <code>LimitRange</code> oraz prawidłowe działanie mechanizmów kontroli zasobów w Kubernetes.</p>
