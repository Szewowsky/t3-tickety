---
name: t3-tickety
description: Użyj, gdy co najmniej dwa gotowe tickety można wykonać niezależnie lub użytkownik prosi o podział prac w T3. Pomaga wybrać wykonawców i odebrać zmiany z osobnych worktree.
---

# Tickety w T3

1. **Wybierz tickety.** Sprawdź zależności, wspólne pliki i kontrakty. Zaproponuj równolegle tylko te prace, które można zaimplementować i przetestować na obecnej bazie. Gdy jeden ticket ustala API dla drugiego, ustaw je kolejno. Jeśli cały zakres jest sekwencyjny, wykonuj go po kolei.

2. **Daj użytkownikowi wybór.** Przed pierwszą delegacją sprawdź raz w tym wątku `orchestrator_capabilities`. Poleć konkretną liczbę agentów i dla każdego wskaż ticket, dostawcę, model oraz obsługiwany poziom effortu. Uzasadnij propozycję kosztem, złożonością lub czasem integracji. W jednej wiadomości daj wybór: „Uruchomić tę konfigurację równolegle, robić po jednym tickecie czy zmienić dostawców, modele lub effort?”. Poczekaj na wybór. Jeśli użytkownik już określił te ustawienia dla bieżących prac, zastosuj je od razu. Dopytaj tylko o brakującą decyzję.

3. **Uruchom wybraną konfigurację.** Przy pracy sekwencyjnej kończ odbiór jednego ticketu przed rozpoczęciem następnego. Dla każdego delegowanego ticketu użyj `delegate_task` z `isolation: "worktree"` i wybranymi ustawieniami w `target`. Worktree tworzy T3. W briefie podaj bazowy commit, ścieżkę ticketu, zakres zmian, kryteria odbioru i testy. Zapisz powiązanie ticketu z zadaniem T3. Jeśli wybrany model jest niedostępny, przedstaw dostępne zamienniki użytkownikowi.

4. **Scal i sprawdź.** Po powrocie agenta sprawdź jego diff, SHA i wyniki względem kryteriów ticketu. Scalaj odebrane commity pojedynczo do gałęzi integracyjnej. Na połączonym kodzie uruchom build, testy ticketu i testy regresji dotkniętych ścieżek. W razie błędu zachowaj worktree do diagnozy, popraw kod i powtórz testy.

5. **Posprzątaj po odbiorze.** Gdy testy na gałęzi integracyjnej przejdą, sprawdź, że zadanie T3 się zakończyło, SHA z worktree jest przodkiem bieżącego `HEAD` i w katalogu nie ma niezapisanej pracy. Usuń tylko ten worktree poleceniem `git worktree remove <ścieżka>`. Sprawdź `git worktree list`. Jeśli Git odmawia usunięcia, ustal przyczynę i zachowaj cudze pliki. Raportuj commit integracyjny, wyniki testów i każdy worktree pozostawiony do wyjaśnienia.
