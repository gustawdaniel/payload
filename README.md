payload
=======

A Symfony project created on February 22, 2017, 9:35 pm.


# Zarządzanie poziomami błędów w Symfony

[TOC]

## Opis projektu

W tym wpisie pokazuję czym jest opcja `payload` dostępna we wszystkich typach walidatorów Symfony oraz jak wykorzystać ją do określania poziomu istotności błędów walidacji. Przy okazji nadpisujemy domyślne ustawienia internacjonalizacji i pozwalamy ją konfigurować użytkownikowi.

## Wprowadzenie

Zanim przejdziemy do kodu, w kilku zdaniach streszczę sposób w jaki Symfony domyślnie przekazuje dane wprowadzane przez użytkownika przeglądarki do bazy danych.

W najbardziej typowej konfiguracji mamy bazę danych, której tabele są spięte z obiektami Php dzięki modelowi obiektowo relacyjnemu - na przykład Doctrine. Obiekty nazywane też encjami znajdują się w katalogu Entity. Każda własność encji może mieć w określone warunki, które powinna spełniać, aby móc ją uznać za zgodną z modelem. Za sprawdzanie tych warunków odpowiedzialne są walidatory. Można je wywoływać bezpośrednio w kontrolerach, albo pośrednio, ponieważ metoda do odczytywania danych z requesta i przekazywania ich do formularza `handleRequest` robi to automatycznie.

Kiedy użytkownik wprowadzi błędne dane do formularza, mogą one zostać zwalidowane na następujących poziomach:

1) na froncie

Wykorzystanie JavaScriptu do walidacji jest najbardziej przyjazne dla użytkownika. Pozwala mu jeszcze przed wysłaniem, w czasie rzeczywistym widzieć, czy jego dane będą odbite, czy zostaną przyjęte przez serwer. Nie można na niej polegać jako na walidacji danych wchodzących. Jest to raczej pomocny wskaźnik świadczący o przyjazności interfejsu. Jednak warto ją stosować również dlatego, że oszczędza serwerowi pracy. Niestety domyślna frontowa walidacja w Symfony tworzona w formularzach Twiga jest słaba, Nie wykorzystuje naszych komunikatów i używa komunikatów domyślnych dla przeglądarki i jeśli nie napisze się jej samemu, to najlepiej jest wyłączyć tą domyślną frontową walidację w Twigu.

2) na walidatorach

Przyjmowanie danych może zostać odbite również podczas przetwarzania formularza w kontrolerze. Brzmi to dość intuicyjnie, ale walidatory to najlepsze miejsce do walidacji danych po stronie serwera.

3) na zapisie do bazy

Zrzucania wyjątków przy próbie zapisu do bazy powinniśmy raczej unikać poza sytuacjami, kiedy walidacja wymaga znajomości stanu bezy danych. 

### Komunikacja walidatorów z formularzem

Teraz opiszę jak walidatory komunikują się z formularzami.

Jeśli walidator wywołany w metodzie `handleRequest` wykryje niezgodność wprowadzonych danych z warunkami jakie nałożyliśmy na model danych, to wyśle do formularza tablicę z błędami. Jeśli obsługą formularza zajmuje się Twig, to po stronie serwera zostanie przygotowany kod html zawierający wiadomości opisujące rodzaj błędów. 

Ta tablica z formularzem generowana przez jego metodę `createView` zawiera 27 zmiennych dla każdego pola oraz tyle samo dla całego formularza. Najbardziej interesująca dla nas zmienna `errors` zawiera tablicę z błędami. Każdy z jej elementów poza dość często stosowaną własnością `message` ma również własność `cause`, która dzięki własności `constraint` pozwala dostać się do zmiennej `payload`. Ze względu na złożoność tej struktury poniżej zamieszczam zrzut ekranu ze wycinkiem zbioru zmiennych określających jak ma zostać wyświetlone dane pole formularza.

[![error.png](https://s18.postimg.org/6q8psckmx/error.png)](https://postimg.org/image/6dhbm62d1/)

O ile wykorzystanie własności message jest implementowane domyślnie, o tyle własność `payload` nie ma żadnego wpływu na to, jak wygląda formularz do puki sami tego nie zaprogramujemy. W dalszej części przeczytasz jak sprawić, żeby dane, które wysyła twoja walidacja w zmiennej `payload` wpływały na wygląd komunikatów o błędach w formularzach.

## Instalacja

Podobnie jak w projekcie z FOSUserBundle zdecydowałem się tutaj na podążanie krok po kroku za sposobem w jaki piszę dany projekt od zera. Jeśli chcesz, możesz oczywiście ściągnąć projekt z repozytorium na githubie i pobrać go i przetestować za pomocą komend

```
Komendy do instalacji i testowania!!!
```

Pamiętaj jednak, żeby przeczytać i rozumieć zawartość skryptu instalacyjnego przed wykonaniem powyższych komend.

Jeśli chcesz tworzyć projekt krok po kroku, zacznij od komendy:

```
symfony new payload
```

Po zainstalowaniu projektu przechodzimy do katalogu z projektem

```
cd payload
```

Tworzymy obiekt, który będziemy walidować 

```
php bin/console doctrine:generate:entity
```

Nadajemy mu nazwę i wybieramy adnotacje jako sposób zapisu konfiguracji:

```
AppBundle:Product
ENTER
```

Nadajemy nazwy trzem atrybutom:

```
name
ENTER x 4
description
text
ENTER x 2
additionalInfo
ENTER x 5
```



