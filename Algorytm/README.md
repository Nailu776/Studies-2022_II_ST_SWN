# Algorytm detekcji zagubienia tokenu w przetwarzaniu typu Token - Ring 

## Model:
- Kanały zawodne;
- N procesów;
- Komunikacja (wszystkie wiadomości w tym Token) - przesyłane w jednym kierunku (Ring);
- Nie ma procesów wyróżnionych;
- Tylko procesy posiadające aktualny token mogą wejść do sekcji krytycznej;
- Procesy znają oszacowany czas propagacji nieblokowanej wiadomości przez pierścień. 

## Komunikacja:
W kanale przesyłane są 2 typy wiadomości:
- Token - w skrócie opisywany T(m), gdzie m to liczba przejść tokenu między procesami ostatnio widziana przez proces wysyłający token;  
- Acknowledge - w skrócie opisywana ACK(TTL,m), gdzie TTL to wartość Time To Live, a m to liczba przejść tokenu między procesami ostatnio widziana przez proces, który wysłał dane ACK; 

## Opis:  
Procesy zapamiętują liczbę przejść tokenu między procesami, jako lokalną zmienną TPC (Token Passing Counter) w celu zidentyfikowania przestarzałych wiadomości.  
Procesy wysyłają sobie token, a po otrzymaniu tokenu wysyłają wiadomość ACK(N-1,TPC).  
Wiadomości T lub ACK z nieprawidłową (przestarzałą) liczbą TPC są ignorowane.  
Procesy przesyłają dalej (nieblokująco) wiadomości ACK(TTL,m), jeśli TTL > 1.  

## Rysunek poglądowy:
![Rysunek https://gitlab.repozytoriumwiedzy.tech/studies/swn-2022/-/blob/main/Algorytm/example.png](example.png "Rysunek poglądowy")

## Szczegółowy opis algorytmu:
### Oznaczenia:
- N - liczba procesów biorących udział w komunikacji;
- n - ID procesu (dowolnego - aktualnie rozważanego);
- TPC - Token Passing Counter - licznik przejść tokenu między procesami ostatnio widziana przez rozważany proces;
- T - token, gdzie T(m) oznacza token wysłany z TPC wynoszącym m;
- ACK - wiadomość acknowledge, gdzie ACK(TTL,m) oznacza ACK z Time To Live wynoszącym TTL oraz z TPC (widzianym z procesu wysyłającego ACK) wynoszącym m;

### Algorytm:
- Procesy są w stanie wejść do strefy krytycznej jedynie podczas posiadania tokenu.
- Każdy proces stale nasłuchuje wiadomości pochodzacych od swojego poprzednika w pierścieniu i przetrzymuje wartość TPC.
- Proces n, jeżeli:
    - Otrzyma T(m) - porównuje m z TPC i jeżeli:
        - m <= TPC, token jest przestarzały - wiadomość jest ignorowana/usuwana.
        - m > TPC, token jest nowy - TPC w procesie jest aktualizowane do wartości m+1, a następnie:
            - Proces n przestaje oczekiwać na ACK, ponieważ dostał z powrotem token  
            (proces n ma pewność, że token dotarł do procesu n+1).
            - Proces n przesyła wiadomość ACK(N-1, TPC) w przód przez cały ring, aby poinformować proces n-1, że token został pomyślnie przesłany.
            - Proces n po zakończeniu pracy przesyła token dalej (wysyła T(TPC) do procesu n+1) i oczekuje wiadomości ACK.  
    - Otrzyma ACK(TTL,m) - sprawdza wartości TTL oraz m i jeżeli:
        - TTL > 1 - proces n przesyła dalej ACK(TTL-1,m). 
        - m > TPC to wiadomość ACK jest nowa i informuje, że wymiana tokenu zakończyła się pomyślnie. 
        (dla TTL > 1 oznacza to, że potwierdzenie z n+1 do n zostało zgubione lub wyprzedzone przez potwierdzenie z n+k+1 do n+k, gdzie k>1, co również potwierdza dostarczenie tokenu z n do n+1)
    - Nie otrzyma informacji, że wymiana tokenu zakończyła się pomyślnie, w ustalonym czasie propagacji wiadomości przez cały pierścień, to:
        - Retransmituje token T(TPC)
        - Oczekuje na potwierdzenie na nowo.
    
Autorzy:
- Julian Helwig 139940  
- Seweryn Kopeć 139959  

Specjalność: Systemy rozproszone    
Rok: 2022  
