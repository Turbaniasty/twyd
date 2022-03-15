# URVE Smart Office
## Logowanie do systemu: 
### Lokalnie
1. Pobranie danych dotyczących logowania.

***Request***
>{server}/atman/login

***Response***
```json
{
  "auth": "internal",
  "css": "api/2.0/css",
  "layoutRefresh": 1200000,
  "logotype": {
  "type": "options",
  "id": 1,
  "name": "eveo_przezroczyste.png"
  }
}
```

2. Logujemy się do systemu przesyłając ten sam request ale z danymi.

***Request***

>{server}/atman/login

***Request data***
```jsonc
{
  "username": "nazwa_użytkownika/email", //case sensitive
  "password" : "hasło_użytkownika"
}
```
***Response***

Dostajemy w nim podstawową konfiugrację, typ logowania. Potrzebne do odpowiedniego wyświetlenia funkcji. Interesuje nas tutaj accessToken, należy go dodac do nagłówków każdego kolejnego requesta.

```jsonc
{
  "version": 3,
  "result": "success",
  "provider": "local",
  "username": "jkowalski",
  "email": "jan.kowalski@smartoffice.expert",
  "name": "Jan Kowalski",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsInVzbyI6Mn0...96dDC0DOsWZ3lnyBKFLpn7wqo5Y3nFo-Sc5H5YbisdA",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...iaWF0IjoxNjM1NDg5MTIxfQ.jYjayiECGusEaqhILDiGQZq6syZSN2DMR8q-e9BiTEU",
  "config": {/*...*/},
  /*...*/
}
```


### AzureAD / O365

1. Pobieramy konfigurację dla logowania. Mamy tutaj dokładne URL dla logowania i pobrania tokenów oraz z jakimi parametrami powinno zostać to wysłane.
 
***Request***

>{server}/atman/login

***Response***

```jsonc
{
  "auth": "openid",
  "css": "api/2.0/css",
  "layoutRefresh": 1200000,
  "nice_name": "Office365",
  "provider": "azuread",
  "logoutUrl": "https://login.microsoftonline.com/.../oauth2/v2.0/logout",
  "authUrl": "https://login.microsoftonline.com/.../oauth2/v2.0/authorize",
  "authUrlData": {
    "response_type": "code",
    "client_id": "...",
    "redirect_uri": "https://smartoffice.expert/token",
    "scope": "user.read email offline_access openid profile",
    "code_challenge": "{code_challenge}",
    "code_challenge_method": "S256"
  },
  "tokenUrl": "https://login.microsoftonline.com/.../oauth2/v2.0/token",
  "tokenUrlData": {
    "grant_type": "authorization_code",
    "code": "{code}",
    "client_id": "...",
    "redirect_uri": "https://smartoffice.expert/token",
    "code_verifier": "{code_verifier}",
    "scope": "user.read email offline_access openid profile"
  },
  "redirect_uri": "https://smartoffice.expert/token",
  "client_id": "..."
}
```

2. Logujemy się na stronie Microsoft, wracamy na stronę, za pomoca code pobieramy token.
3. Logujemy się do USO za pomoca tokenu.

***Request***

>{server}/atman/login

***Request headers***

W nagłówkach powinien znaleźć się nasz token, który otrzymaliśmy od Microsoft.

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6Ijl0SWUwTW0xUUhPNUthVU5Mcjc4akhCUjg5VnlpaUI4SEZjd2IxNWNYU...qAlnBcIiZs-6J6EJadujxt0wbEiXOAA1-oCw
```

***Request data***

Wysyłamy ewentualnego użytkownika w danych.

```jsonc
{
  "username": "jan.kowalski@smartoffice.expert"
}
```

***Response***

Dane zwrotne są podobne do logowania lokalnego. 

### Keycloak 

1. Pobieramy konfigurację dla logowania.
 
***Request***

>{server}/atman/login

***Response***

```jsonc
{
  "auth": "openid",
  "css": "api/2.0/css",
  "layoutRefresh": 1200000,
  "nice_name": "Keycloak",
  "provider": "openid",
  "logoutUrl": "https://smartoffice.expert/auth/realms/master/protocol/openid-connect/logout",
  "authUrl": "https://smartoffice.expert/auth/realms/master/protocol/openid-connect/auth",
  "authUrlData": {
  "response_type": "code",
  "client_id": "uso",
  "redirect_uri": "https://smartoffice.expert/token"
  },
  "tokenUrl": "https://smartoffice.expert/auth/realms/master/protocol/openid-connect/token",
  "tokenUrlData": {
  "grant_type": "authorization_code",
  "code": "{code}",
  "client_id": "uso",
  "redirect_uri": "https://smartoffice.expert/token"
  },
  "redirect_uri": "https://smartoffice.expert/token",
  "client_id": "uso"
}
```

2. Logujemy się na stronie Keycloak, wracamy na stronę za pomoca code pobieramy token.
3. Logujemy się do USO za pomoca tokenu.

***Request***

>{server}/atman/login

***Request headers***

W nagłówkach powinien znaleźć się nasz token, który otrzymaliśmy od Keycloak.

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6Ijl0SWUwTW0xUUhPNUthVU5Mcjc4akhCUjg5VnlpaUI4SEZjd2IxNWNY...qAlnBcIiZs-6J6EJadujxt0wbEiXOAA1-oCw
```

***Request data***

Wysyłamy ewentualnego użytkownika w danych

```jsonc
{
  "username": "jkowalski"
}
```

***Response***

Przy Keycloak nie jest zwracany *Access Token*. Przy autoryzacji zapytań, należy używać access token, który został dostarczony przez serwer Keycloak.

```jsonc
{
  "version": 3,
  "result": "success",
  "provider": "openid",
  "username": "jkowalski",
  "email": "jan.kowalski@smartoffice.expert",
  "name": "Jan Kowalski",
  "config": {/*...*/},
}
```

## Rezerwacja

1. Wyszukujemy dostępne zasoby w wybranych godzinach

***Request***

>{server}/atman/api/2.0/search-resource

***Request data***

```jsonc
{
  "startTime": "2021-11-01T09:00:00+01:00",
  "endTime": "2021-11-01T09:30:00+01:00",
  "type": "desk" // [desk, room, parking]
}
```

***Response***

```jsonc
{
  "settings": {/*...*/}, //ustawienia wyświetlania formularza
  "locations": {
  "3": {
    "name": "London Green Office, I piętro",
    "parent_name": "London Green Office",
    "id": 3,
    "resources": [
    {
      "id": 2,
      "name": "HR",
      "concurrent_reservations": null,
      "extras": null,
      "booked": false,
      "status": "free",
      "capacity": null,
      "leaflet": {/*...*/},
      "image": null,
      "dates": {
      "busy": null
      },
    }
    ]
  }
  },
}
```

2. Wybieramy interesujący nas zasób i rezerwujemy go.

***Request***

>{server}/atman/api/2.0/create-event

***Request data***

```jsonc
{
  "startTime": "2021-11-01T09:00:00+01:00",
  "endTime": "2021-11-01T09:30:00+01:00",
  "resources": [2] // tablica, id rezerwowanego zasobu
}
```

***Response***

```jsonc
{
  "result": "success",
  "dates": [
  {
    "startTime": "2021-11-01T08:00:00.000Z",
    "endTime": "2021-11-01T08:30:00.000Z"
  }
  ]
}
```

## Lista rezerwacji

***Request***

>{server}/atman/reservation-list

***Request data***

```jsonc
{
  "emailAddress": "adres_mailowy_uzytkownika"
}
```

***Response***
```jsonc
{
  "reservations": [
    {
      "id": 45919,
      "location_id": 3,
      "desk_id": 2,
      "date": "2021-11-01",
      "start_time": "2021-11-01T08:00:00.000Z",
      "end_time": "2021-11-01T08:30:00.000Z",
      "status": null,
      "location_name": "I piętro",
      "rbsubject": null,
      "platenumber": null,
      "created_by": null,
      "extras": null,
      "type": 1,
      "reservation_hash": "mWObyQK9ALMIopLKk8n2MgoLXTwPtcHBeUj9QqsFmFE=",
      "parent_name": "London Green Office",
      "desk_name": "HR",
      "leaflet": {
        "type": "Feature",
        "properties": {},
        "geometry": {/*...*/},
    }
  ],
  "pages": 1
}
```

## Edycja rezerwacji

Edytujemy rezerwację w taki sam sposób jak się ją tworzy, dodajemy reservationHash, które dostajemy pobierając listę rezerwacji.

***Request***

>{server}/atman/api/2.0/create-event

***Request data***

```jsonc
{
  "startTime": "2021-11-02T09:00:00+01:00",
  "endTime": "2021-11-02T09:30:00+01:00",
  "resources": [2], // tablica, id zasobu
  "reservationHash": "mWObyQK9ALMIopLKk8n2MgoLXTwPtcHBeUj9QqsFmFE="
}
```

***Response***

```jsonc
{
  "result": "success",
  "dates": [
  {
    "startTime": "2021-11-02T08:00:00.000Z",
    "endTime": "2021-11-02T08:30:00.000Z"
  }
  ]
}
```

## Potwierdzanie rezerwacji

***Request***

>{server}/atman/confirm-reservation

***Request data***

```jsonc
{
  "reservationId": 6087,
  "emailAddress": "jan.kowalski@smartoffice.expert"
}
```

***Response***

```jsonc
{
  "result": "success",
  "msg": "reservationConfirmed",
  "reservationId": 6087
}
```

## Usuwanie rezerwacji


***Request***

>{server}/atman/cancel-reservation

***Request data***

```jsonc
{
  "reservationId": 6087
}
```

***Response***

```jsonc
{
  "result": "success",
  "msg": "reservationReleased"
}
```

## Wyszukiwanie wcześniej zalogowanego pracownika (Find person)

***Request***

>{server}/atman/find-person

***Request data***

```jsonc
{
  "search": "Jan"
}
```

***Response***

```jsonc
[
  {
    "displayName": "Jan Kowalski",
    "email": "jan.kowalski@smartoffice.expert"
  },
  {
    "displayName": "Jan Adamski",
    "email": "jan.adamski@smartoffice.expert"
  }
]
```

## Sprawdzenie rezerwacji innego pracownika (Find person)

***Request***

>{server}/atman/find-person

***Request data***

```jsonc
{
  "emailAddress": "jan.kowalski@smartoffice.expert"
}
```

***Response***

```jsonc
{
  "reservations": [
    {
      "id": 6088,
      "status": null,
      "location_name": "I",
      "leaflet": {/*...*/},
      "location_id": 4,
      "image_h": 768,
      "image_w": 768,
      "mapimage": "aa.jpg",
      "rbsubject": null,
      "type": 1,
      "parent_name": "Kraków",
      "start_time": "2021-11-01T08:20:00.000Z",
      "end_time": "2021-11-01T08:35:00.000Z",
      "desk_name": "desk1"
    }
  ]
}
```