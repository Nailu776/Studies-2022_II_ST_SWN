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

