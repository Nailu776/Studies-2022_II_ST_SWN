# Algorytm detekcji zagubienia tokenu w przetwarzaniu typu Token - Ring 

## Model:
- Kanały zawodne FIFO;
- N procesów;
- Komunikacja (wszystkie wiadomości w tym Token) - przesyłane w jednym kierunku (Ring);
- Nie ma procesów wyróżnionych;
- Tokeny są podpisywane przez procesy (np.: nadawca tokenu: n), które je wysyłają ze swoim ID;
- Procesy posiadające token mogą wejść do sekcji krytycznej;
- Procesy są świadome z pewnym przybliżeniem, jak długo wiadomość nieblokowalna (ACK) przechodzi przez cały ring;

## Komunikacja:
W kanale przesyłane są 2 typy wiadomości:
- Token - w skrócie opisywany Tnm, gdzie n to ID procesu, który wysyła token, a m to liczba wejść do sekcji krytycznej ostatnio widziana przez proces n;
- Acknowledge - w skrócie opisywana ACKnm, gdzie n to ID procesu, który wysyła acknowledge, a m to liczba wejść do sekcji krytycznej ostatnio widziana przez proces n;

## Opis:
Procesy wysyłają sobie token, a po otrzymaniu tokenu wysyłają wiadomość ACK.  
Procesy zapamiętują liczbę wejść do sekcji krytycznej jako CSC (Critical Section Counter) w celu zidentyfikowania przestarzałych wiadomości.  
Wiadomości T lub ACK z nieprawidłową (przestarzałą) liczbą wejść do sekcji krytycznej są ignorowane.  
Procesy przesyłają dalej (nieblokująco) wiadomości ACK, sprawdzane są tylko wiadomości ACK dotyczące siebie i następnika.  

## Rysunek poglądowy:


## Szczegółowy opis algorytmu:
### Oznaczenia:
- N - liczba procesów biorących udział w przetwarzaniu Token - Ring;
- n - ID procesu (dowolnego - aktualnie rozważanego);
- k - ID procesu następnego po n (n + 1 % N) w kolejności ring;
- j - ID procesu następnego po k (k + 1 % N) w kolejności ring;
- p - ID procesu poprzedzającego n (n - 1 % N) w kolejności ring;
- CSC - Critical Section Counter - licznik wejść do sekcji krytycznej ostatnio widziany przez rozważany proces;
- T - token, gdzie Txm oznacza token wysłany przez proces o ID x z CSC (widzianym z procesu x) wynoszącym m;
- ACK - wiadomość acknowledge, dzie ACKxm oznacza ACK wysłane przez proces o ID x z CSC (widzianym z procesu x) wynoszącym m;


### Algorytm:
- Procesy nieposiadające tokenu oraz nieoczekujące na wiadomość ACK - oczekują wiadomości z tokenem,  
a wiadomości ACK przesyłają dalej bez blokowania.
- Proces n wysyła do procesu k token TnCSC i przechodzi w stan oczekiwania na wiadomość ACKkm.
- Proces k, jeżeli [!]:
    - Otrzyma wiadomość ACKxm - przesyła nieblokująco ACKxm dalej w komunikacji ring.
    - Otrzyma Token Tnm - porównuje m z CSC i jeżeli:
        - Token jest przestarzały (m < CSC) - wiadomość jest ignorowana.
        - Token jest nowy (m >= CSC) to ustawia CSC na m (CSC = m), a następnie:
            - Jeżeli proces k oczekiwał na ACKjm to oczekiwanie na ACKjm jest wyłączone, ponieważ dostał nowy token  
            (proces k ma pewność, że token przeszedł przez cały token).
            - Jeżeli proces k nie musi wchodzić do sekcji krytycznej, to przesyła token dalej (wysyła TkCSC do procesu j) i oczekuje wiadomości ACKjm.
            - Jeżeli proces k chce wejść do sekcji krytycznej to zwiększa CSC o 1 (CSC += 1), przechodzi do sekcji krytycznej i przesyła wiadomość ACKkCSC przez cały ring (do siebie), 
            aby poinformować proces n, że token został pomyślnie przesłany, a następnie otrzymać swoje ACK z powrotem, aby się upewnić, że retransmisja Tokenu jest zbędna.
    - Nie otrzyma Tokenu - nie podejmuje działań, cierpliwie czeka, aż ostatecznie dostanie Token.
    - Otrzyma ACKkm - porównuje m z CSC i jeżeli:
        - ACK jest przestarzałe (m < CSC) - wiadomość jest ignorowana.
        - ACK jest najnowsze (m == CSC) ma pewność, że wymiana tokenu zakończyła się pomyślnie.  
        Note: Jeżeli mamy kolejność wiadomości FIFO to nie dostaniemy ACKkm z m > CSC, gdyż CSC to największa liczba wejść do sekcji krytycznej jaką widział proces k w momencie wysyłania wiadomości ACK.
    - Nie otrzyma ACKkm w określonym z pewnym przybliżeniem czasem przetwarzania wiadomości ACK przez cały ring - jeżeli:
        - Nadal posiada token - wysyła retransmisje wiadomości ACKkCSC.
        - Nie posiada już tokenu - przekazanie tokenu z procesu n do k jest uznane, za pomyślnie zakończone.
    - Otrzyma wiadomość ACKjm - porównuje m z CSC i jeżeli:
        - ACK jest przestarzałe (m < CSC) - wiadomość jest ignorowana.
        - ACK jest nowe (m >= CSC) - proces k ma pewność, że przesłanie tokenu (z k do j) zakończyło się pomyślnie i przechodzi w stan oczekiwania tylko na token (oczekiwanie na wiadomość ACK zostaje wyłączone), a następnie przesyła ACKjm do procesu j.
    - Po wysłaniu tokenu nie otrzyma w określonym z pewnym przybliżeniem czasem wiadomości ACKjm - Wysyła token TkCSC ponownie (retransmisja tokenu TkCSC - domniemanie zgubienia).
- Proces n po wysłaniu tokenu, jeżeli:
    - Otrzyma wiadomość ACKkm - porównuje m z CSC i jeżeli:
        - ACK jest przestarzałe (m < CSC) - wiadomość jest ignorowana.
        - ACK jest nowe (m >= CSC) - proces n ma pewność, że przesłanie tokenu (z n do k) zakończyło się pomyślnie i przechodzi w stan oczekiwania tylko na token (oczekiwanie na wiadomość ACK zostaje wyłączone), a następnie przesyła ACKkm do procesu k.
    - Nie otrzyma w określonym z pewnym przybliżeniem czasem wiadomości ACKkm - Wysyła token TnCSC ponownie (retransmisja tokenu TnCSC - domniemanie zgubienia).
    - Proces n pozostałe sytuacje (niewypisane) rozważa identycznie jak proces k - dla sprawdzenia w punkcie algorytmu "Proces k, jeżeli [!]" można k zamienić na n, a n na p, natomiast j na k, aby rozważyć wszystkie przypadki.

    
## Aktualny stan algorytmu:
- Init 
## Pytania:
- Czy razem z Tokenem należy wysyłać ID procesu?  
Opis: Jeżeli token możemy tylko otrzymać od poprzednika, to interesuje nas tylko CSC.  
Odpowiedź: Prawdopodobnie nie - do przetestowania.

- Jak zauważane jest zgubienie tokenu?  
Odpowiedź: Proces który wysłał Token nie otrzymał ACK, co może oznaczać zgubienie ACK lub zgubienie tokenu, dlatego ponowna retransmisja tokenu odtwarza token przy domniemaniu jego zgubienia. Dwóch tokenów w ringu nie będzie, ponieważ przestarzałe tokeny za pomocą CSC są ignorowane. 

- Czy potrzebne jest uszeregowanie wiadomości FIFO w kanale?  
Opis: Jeżeli CSC definiuje nam, czy wiadomości ACK/token są przestarzałe to czy kolejność ACK ma znaczenie?  
Odpowiedź: Prawdopodobnie rozluźnienie tego założenia może spowodować zbędne retransmisje tokenu - do przetestowania.

- Czy poprawnie są identyfikowane przestarzałe wiadomości?  
Opis: Czy przestarzała wiadomość to m < CSC czy m <= CSC?  
Odpowiedź: Chyba jest pewnego rodzaju błąd, jeżeli dostaniesz retransmisje tokena od poprzednika, a sam nie wszedłeś do sekcji krytycznej to m == CSC, a następne procesy mogą być w sekcji / lub nie, ale w każdym razie może być więcej tokenów wtedy z różnym lub identycznym CSC.  
Rozwiązanie? - Może warto CSC zwiększać o 1 zawsze, bez względu na wejście do sekcji krytycznej - zmiana definicji CSC z liczba wejść do CS na liczba przekazań tokenu.

Autorzy:
- Julian Helwig 139940
- Seweryn Kopeć 139959

Specjalność: Systemy rozproszone  
Rok: 2022
