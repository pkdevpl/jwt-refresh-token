# JWT Refresh Token

## Wordpress REST API

JWT Refresh Token dodaje do Wordpress'a punkty REST, które pozwalają:
* Zalogować się do serwera poprzez zapytanie `POST` i uzyskać `JSON Web Token (JWT)`.
* Zarejestrować nowego użytkownika.
* Zweryfikować istniejący token JWT.
* Wymienić token JWT na nowy, za pomocą `Secure Cookie`.
* Wysłać link resetujący hasło do wskazanego użytkownika.

## Dwie wtyczki

JWT Refresh Token jest rozszerzeniem wtyczki [JWT Auth](https://wordpress.org/plugins/jwt-auth/) stworzonej przez [Useful Team](https://usefulteam.com/) i potrzebuje jej do poprawnego działania.

## JWT Auth
Podstawowa wtyczka dodaje do strony 2 punkty REST:
* /wp-json/jwt-auth/v1/token
* /wp-json/jwt-auth/v1/token/validate

Pierwszy z nich pozwala zalogować się do strony za pomocą zapytania `POST`.  Drugi pozwala zweryfikować wygenerowany wcześniej token JWT.

Logowanie odbywa się poprzez wysłanie zapytania `POST`, zawierającego pola `username` i `password`. Jeżeli użytkownik istnieje w bazie, zapytanie zwróci tymczasowy `JSON Web Token (JWT)`, który aplikacja może wykorzystywać w dalszej komunikacji, umieszczając go jako header zapytania HTTP `Authorization: Bearer JWT_TOKEN`.

Weryfikacja z kolei odbywa się poprzez wysłanie pustego zapytania `POST` z ustawionym nagłówkiem (header) `Authorization: Bearer JWT_TOKEN`.

## JWT Refresh Token
JWT Refresh Token (rozszerzenie) dodaje możliwość:
* Automatycznego odświeżania tokena JWT (przedłużania ważności).
* Rejestrowania nowych kont użytkowników.
* Resetowania hasła użytkowników.

**Odświeżanie** odbywa się automatycznie, wraz z logowaniem przez punkt `wp-json/jwt-auth/v1/token`. Po wygenerowaniu podstawowego tokena JWT, serwer automatycznie zapisuje w przeglądarce ciasteczko `Secure Cookie` zawierające Refresh Token.   Wysłanie pustego zapytania `GET` pod adres `/jwt-auth/v1/token/refresh` automatycznie sprawdzi bezpieczne ciasteczko i wymieni token na nowy, bez konieczności ponownego logowania. 

Dzięki takiemu mechanizmowi, podstawowy token JWT zachowuje ważność jedynie przez 15 min i może być przechowywany w dowolnej formie w pamięci podręcznej. Ciasteczko `Secure Cookie` zawierające Refresh Token jest niedostępne dla skryptów JS w przeglądarce i dzięki temu nie może zostać pozyskany przez złośliwy skrypt.

**Rejestrowanie użytkowników** polega na wysłaniu zapytania `POST` pod adres `/jwt-auth/v1/register-user`. Zapytanie musi zawierać `username`, `email` oraz `password`. Strona automatycznie utworzy konto użytkownika i odeśle nowy token JWT.

**Resetowanie hasła** wymaga wysłania zapytania `POST` pod adres `/jwt-auth/v1/reset-password` zawierającego `email` użytkownika. Po sprawdzeniu danych, Wordpress wyśle link do zresetowania hasła dla wskazanego użytkownika. Link resetujący hasło umożliwia przekierowanie użytkownika do niestandardowego formularza ustawiania hasła. Przekierowanie zachowuje jednak te same standardy bezpieczeństwa co standardowy formularz Wordpress.

## Wersja DEMO
Wtyczkę można przetestować w aplikacji [Postman](https://postman.com), kierując zapytania pod adresy:

**Logowanie** - zapytanie `POST` zawierające `username` i `password`:  
```http://wp.pkdev.pl/wp-json/jwt-auth/v1/token```

**Weryfikacja** - zapytanie `POST` z headerem `Authorization: Bearer JWT_TOKEN`:  
```http://wp.pkdev.pl/wp-json/jwt-auth/v1/token/validate```

**Rejestracja** - zapytanie `POST` zawierające `email`, `username` i `password`:  
```http://wp.pkdev.pl/wp-json/jwt-auth/v1/register-user```

**Odświeżenie** - zapytanie `GET` po zalogowaniu:  
```http://wp.pkdev.pl/wp-json/jwt-auth/v1/refresh```

**Resetowanie hasła** - zapytanie `POST` zawierające `email` pod adres:  
```http://wp.pkdev.pl/wp-json/jwt-auth/v1/refresh```

Głównym celem wtyczki jest udostępnienie mechanizmu logowania i komunikacji dla stron Single Page Application napisanych z użyciem biblioteki React, Vue, czy Angular.
