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
- Token - w skrócie opisywany T(n,m), gdzie n to ID procesu, który wysyła token, a m to liczba przejść tokenu między procesami ostatnio widziana przez proces n;
- Acknowledge - w skrócie opisywana ACK(n,m), gdzie n to ID procesu, który wysyła acknowledge, a m to liczba przejść tokenu między procesami ostatnio widziana przez proces n;

## Opis:
Procesy wysyłają sobie token, a po otrzymaniu tokenu wysyłają wiadomość ACK.  
Procesy zapamiętują liczbę przejść tokenu między procesami jako lokalną zmienną TPC (Token Passing Counter) w celu zidentyfikowania przestarzałych wiadomości.  
Wiadomości T lub ACK z nieprawidłową (przestarzałą) liczbą TPC są ignorowane.  
Procesy przesyłają dalej (nieblokująco) wiadomości ACK, sprawdzane są tylko wiadomości ACK dotyczące następnika.  

## Rysunek poglądowy:
![Rysunek https://gitlab.repozytoriumwiedzy.tech/studies/swn-2022/-/blob/main/Algorytm/example.png](example.png "Rysunek poglądowy")

## Szczegółowy opis algorytmu:
### Oznaczenia:
- n - ID procesu (dowolnego - aktualnie rozważanego);
- TPC - Token Passing Counter - licznik przejść tokenu między procesami ostatnio widziana przez rozważany proces;
- T - token, gdzie T(m) oznacza token wysłany z TPC wynoszącym m;
- ACK - wiadomość acknowledge, gdzie ACK(x,m) oznacza ACK wysłane przez proces o ID x z TPC (widzianym z procesu x) wynoszącym m;


### Algorytm:
- Procesy są w stanie wejść do strefy krytycznej jedynie podczas posiadania tokenu.
- Każdy proces cały czas nasłuchuje wiadomości pochodzacych od swojego poprzednika w pierścieniu i przetrzymuje wartość TPC.
- Proces n, jeżeli:
    - Otrzyma T(m) - porównuje m z TPC i jeżeli:
        - m <= TPC, token jest przestarzały - wiadomość jest ignorowana/usuwana.
        - m > TPC, token jest nowy - TPC w procesie jest aktualizowane do wartości m + 1, a następnie:
            - Proces n przestaje oczekiwać na ACK(n+1,m), ponieważ dostał z powrotem token  
            (proces n ma pewność, że token przeszedł przez cały pierścień).
            - Proces n przesyła wiadomość ACK(n, TPC) w przód przez cały ring, aby poinformować proces n-1, że token został pomyślnie przesłany.
            - Proces n po zakończeniu pracy przesyła token dalej (wysyła T(TPC) do procesu n+1) i oczekuje wiadomości ACK(n+1,x).  
    - Otrzyma ACK(n+1,m) - porównuje m z TPC i jeżeli:
        - m <= TPC, wiadomość ACK jest przestarzała - wiadomość jest ignorowana/usuwana.
        - m > TPC, wiadomość ACK jest nowa i informuje, że wymiana tokenu zakończyła się pomyślnie. Proces n zmienia wartość TPC na m.          
    - Otrzyma ACK(x,m), gdzie x!=(n+1), to przesyła wiadomość dalej i jeżeli:
        - m > TPC, wiadomość ACK jest nowa i informuje, że wymiana tokenu zakończyła się pomyślnie. Proces n zmienia wartość TPC na m.   
        (oznacza to, że potwierdzenie z n+1 do n zostało zgubione lub wyprzedzone przez potwierdzenie z n+k+1 do n+k, gdzie k>1, co również potwierdza dostarczenie tokenu z n do n+1)
    - Nie otrzyma ACK(n+1,m) w ustalonym czasie propagacji wiadomości przez cały pierścień podczas oczekiwania na potwierdzenie odebrania tokenu, to:
        - Retransmituje token T(TPC)
        - Zaczyna oczekiwanie na wiadomość ACK(n+1,TPC) na nowo.
    
## Aktualny stan algorytmu:
- Pytania dot. kompletnosci i optymalizacja
## Pytania:
- Czy razem z Tokenem należy wysyłać ID procesu?  
Opis: Jeżeli token możemy tylko otrzymać od poprzednika, to interesuje nas tylko CSC.  
Odpowiedź: Nie, ponieważ i tak tokeny zawsze dostajemy od poprzednika.

- Jak zauważane jest zgubienie tokenu?  
Odpowiedź: Proces, który wysłał Token, a nie otrzymał ACK lub Tokenu uznaje token za zgubiony i dokonuje retransmisji tokenu, która w przypadku faktycznego zgubienia tokenu odtwarza go w ringu. Natomiast dwóch tokenów w ringu nie będzie, ponieważ przestarzałe tokeny są ignorowane.

- Czy potrzebne jest uszeregowanie wiadomości FIFO w kanale?  
Opis: Jeżeli CSC definiuje nam, czy wiadomości ACK/token są przestarzałe to czy kolejność ACK ma znaczenie?  
Odpowiedź: Prawdopodobnie rozluźnienie tego założenia może spowodować zbędne retransmisje tokenu - do przetestowania.

- Czy poprawnie są identyfikowane przestarzałe wiadomości?  
Opis: Czy przestarzała wiadomość to m < CSC czy m <= CSC?  
Odpowiedź: Chyba jest pewnego rodzaju błąd, jeżeli dostaniesz retransmisje tokena od poprzednika, a sam nie wszedłeś do sekcji krytycznej to m == CSC, a następne procesy mogą być w sekcji / lub nie, ale w każdym razie może być więcej tokenów wtedy z różnym lub identycznym CSC.  
Rozwiązanie? - Może warto CSC zwiększać o 1 zawsze, bez względu na wejście do sekcji krytycznej - zmiana definicji CSC z liczba wejść do CS na liczba przekazań tokenu.

Zmiana CSC na TPS Token Passing Counter i zwiększanie TPS z każdym otrzymaniem tokenu.

- Czy wysłanie ACK oprócz trafienia do procesu, od którego dostaliśmy token musi trafiać do naszego procesu, który to ACK wysłał?  
Opis: Ta wiadomość ostatnio wysyłana daje tylko i wyłącznie "pewność", że przekazanie się powiodło i nie będzie zbędnych transmisji przestarzałego tokenu,  
ale jeżeli nie będzie retransmisji przestarzałego tokenu i tak (bo ACK dostał proces, który nam wysłał token) to co nam daje ta "pewność".

Nie trzeba wysyłać do samego siebie.

- Czy ACK potrzebuje ID?
Opis: Użycie TTL zamiast ID w ACK. W momencie otrzymania ACK z TTL 1 proces n wie, że jego token pomyślnie został przekazany. Procesy otrzymujące ACK(TTL, TPC) zmniejszają TTL o jeden i przekazują ACK dalej.

Autorzy:
- Julian Helwig 139940
- Seweryn Kopeć 139959

Specjalność: Systemy rozproszone  
Rok: 2022
