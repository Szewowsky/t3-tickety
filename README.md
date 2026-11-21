# t3-tickety

Skill dla agentów w [T3 Code](https://github.com/pingdotgg/t3code). Gdy masz kilka gotowych ticketów, agent sprawdza, które da się robić równolegle, pyta Cię o dostawcę, model i effort dla każdego, rozsyła je do osobnych worktree, a na końcu scala wyniki, puszcza testy i sprząta.

To rozszerzenie do paczki skilli Matta Pococka: [mattpocock/skills](https://github.com/mattpocock/skills). Paczka Matta prowadzi od pomysłu do ticketów, a ten skill przejmuje moment, w którym tickety są gotowe i trzeba je rozdać agentom. Dodatek jest niezależny, nie jest częścią oficjalnej paczki i nie kopiuje z niej żadnych plików.

## Wymagania

- **T3 Code.** Skill korzysta z jego narzędzi `orchestrator_capabilities` (lista dostępnych dostawców i modeli) oraz `delegate_task` (zlecenie pracy agentowi w osobnym worktree). W zwykłym Claude Code albo Codex bez T3 nie zadziała.
- **Paczka Matta Pococka**, bo to z niej pochodzą tickety. Instalacja jest opisana w [jego README](https://github.com/mattpocock/skills#installation-30-second-setup). W Claude Code:

  ```bash
  claude plugin install mattpocock-skills@claude-plugins-official
  ```

  Potem raz w każdym repo uruchom `/setup-matt-pocock-skills`. Skill zapyta, gdzie trzymasz tickety (GitHub, lokalne pliki Markdown albo coś innego).
- **Repo w Git** z testami, które da się uruchomić jedną komendą.

## Instalacja

```bash
npx skills add Szewowsky/t3-tickety
```

Albo ręcznie: skopiuj katalog `skills/t3-tickety/` do `.claude/skills/` (Claude Code) lub `.agents/skills/` (Codex) w swoim repo.

## Jak z tego korzystam

Cały przebieg od pomysłu do scalonego kodu. Kroki 1, 2, 4 i 6 to skille Matta, krok 5 to ten skill, a audyty robię sam innym modelem.

| Krok | Co uruchamiam | Co z tego wychodzi |
| --- | --- | --- |
| 1 | `/grilling` | Wspólne zrozumienie, co budujemy |
| 2 | `/to-spec` | Spec zapisany w trackerze |
| 3 | Audyt specu innym modelem | Lista luk i poprawiony spec |
| 4 | `/to-tickets`, potem audyt ticketów | Tickety z zależnościami |
| 5 | `/t3-tickety` | Kod z każdego ticketu scalony na jednej gałęzi |
| 6 | `/code-review` | Przegląd całości przed mergem |

### 1. Grill: ustalam, co chcę zrobić

Uruchamiam `/grilling` i opisuję pomysł. Agent zadaje pytania rundami, do każdego daje swoją rekomendację, a fakty z kodu sprawdza sam. Kończymy, gdy nie zostaje żadna decyzja podjęta po cichu.

### 2. Spec: plan na bazie grilla

W tym samym wątku uruchamiam `/to-spec`. Skill nie pyta od nowa, tylko spisuje to, co ustaliliśmy: problem, rozwiązanie, historyjki użytkownika, decyzje implementacyjne, decyzje o testach i to, co jest poza zakresem.

### 3. Audyt specu innym modelem

Otwieram nowy wątek z modelem innym niż ten, który pisał spec. Jeśli spec pisał Claude, audytuje GPT albo Gemini, i odwrotnie. Nowy wątek nie zna rozmowy z grilla, więc widzi tylko to, co naprawdę trafiło do pliku. Jeśli coś było oczywiste w rozmowie, ale nie zostało zapisane, wyjdzie właśnie tutaj.

Prompt, od którego możesz zacząć:

```text
Przeczytaj spec w <ścieżka do specu>. Nie poprawiaj go. Wypisz:
1. Luki: decyzje, których brakuje, żeby zacząć implementację.
2. Sprzeczności między sekcjami.
3. Założenia o kodzie, które nie zgadzają się z repo. Sprawdź je w plikach.
4. Rzeczy, które powinny być poza zakresem, a są w środku.
Do każdego punktu podaj cytat ze specu i propozycję poprawki.
```

Uwagi wklejam autorowi specu i decyduję, które przyjmuję. Spec poprawia autor, nie audytor.

### 4. Tickety i audyt ticketów

Uruchamiam `/to-tickets`. Skill tnie spec na pionowe plastry: każdy ticket przechodzi przez wszystkie warstwy, da się go sprawdzić osobno i ma wpisane, które tickety go blokują.

Potem drugi audyt, znowu innym modelem w nowym wątku:

```text
Przeczytaj spec w <ścieżka do specu> i tickety w <katalog z ticketami>. Sprawdź:
1. Czy każda historyjka ze specu ma swój ticket.
2. Czy każdy ticket da się zrobić i przetestować bez pozostałych, poza tymi z "Blocked by".
3. Czy zależności są kompletne i nie tworzą pętli.
4. Które tickety dotykają tych samych plików, więc nie powinny iść równolegle.
Do każdej uwagi podaj numer ticketu i propozycję poprawki.
```

Punkt 4 jest tu najważniejszy, bo od niego zależy, co w następnym kroku pójdzie równolegle.

### 5. Implementacja: `/t3-tickety`

Najpierw tickety, potem implementacja. Paczka Matta daje tu dwie drogi, a ten skill jest trzecią:

- **Jeden ticket albo łańcuch, w którym każdy czeka na poprzedni:** `/implement` w jednym wątku, po kolei.
- **Kilka niezależnych ticketów, ten sam agent i model do wszystkich:** `/implement-spec` Matta, który sam rozdaje pracę swoim podagentom.
- **Kilka niezależnych ticketów, a chcesz sam dobrać wykonawcę do każdego:** `/t3-tickety`.

Po uruchomieniu `/t3-tickety` agent:

1. Sprawdza zależności, wspólne pliki i kontrakty, i wybiera tickety, które da się zrobić równolegle na obecnej bazie.
2. Pokazuje propozycję: ilu agentów, który ticket dla którego, jaki dostawca, model i effort, z uzasadnieniem kosztem, złożonością albo czasem scalania. Wybierasz: uruchomić tak, robić po jednym, czy zmienić ustawienia.
3. Zleca każdy ticket przez `delegate_task` w osobnym worktree. Brief zawiera bazowy commit, ścieżkę ticketu, zakres, kryteria odbioru i testy.
4. Odbiera wyniki: sprawdza diff względem kryteriów ticketu, scala commity pojedynczo do gałęzi integracyjnej, uruchamia build i testy na połączonym kodzie.
5. Sprząta: usuwa tylko te worktree, których praca jest już na gałęzi, i raportuje to, co zostało do wyjaśnienia.

Dzięki temu prosty ticket może dostać tańszy model, a trudny mocniejszy z wyższym effortem.

### 6. Przegląd

Na scalonej gałęzi uruchamiam `/code-review`. Sprawdza dwie rzeczy naraz: czy kod trzyma standardy repo i czy robi to, co zapisano w specu.

## Język

Skill jest napisany po polsku, więc agent prowadzi tę część pracy po polsku. Jeśli pracujesz po angielsku, przetłumacz `SKILL.md`. To pięć akapitów.

## Licencja

MIT, zobacz [LICENSE](LICENSE). Paczka [mattpocock/skills](https://github.com/mattpocock/skills) ma własną licencję MIT i własnego autora.
