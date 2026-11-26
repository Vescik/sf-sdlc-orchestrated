# skills-thin: draft zestawu 6 skilli i pętli delivery

`skills-thin/` odpowiada katalogowi `.github/` w repo. Testuj na osobnym branchu.

Pętla to dodatkowy skill `deliver-design` i agent `sdlc-conductor`. Wdrażaj ją dopiero, gdy podstawowy zestaw przejdzie exit criteria.

## Pochodzenie treści

| Skill | Na czym oparty |
|---|---|
| `solution-design`, `development`, `continue-development` | Obecne pliki z projektu, odchudzone |
| `fetch-ado-item` | Tylko wzmianki w innych skillach. Nie widziałem komendy |
| `deployment` | Wzmianki w `development` i `copilot-instructions`. Nie widziałem skilla |
| `git-workflow` | Jak wyżej |

## Do uzupełnienia przed testem

- `fetch-ado-item`: wywołanie pobierające ticket i nagłówki `ado-context.md`.
- `deployment`: domyślny test level.
- `git-workflow`: sekcja `To copy from the current files`.
- Potwierdź alias `intDev`.

## Nowe lub zmienione zachowanie

- Potwierdzenie w chacie dla record delete, bulk, package install/uninstall i usunięcia orga. Wcześniej nie było wymagane. Usuń punkt, jeśli go nie chcesz.
- Brak evidence statuses, identity query, self-checku i zakazu odczytów orga na etapie opcji.
- Brak sprawdzania wersji package względem `docs/package-concept.md`.
- `deployment` po tym samym błędzie dwa razy pyta użytkownika (moja propozycja).

## Zmiany w repo

1. Zastąp sześć skilli i `copilot-instructions.md`.
2. Usuń: `org-read`, `sf-metadata-discovery`, `org-discovery`, `investigate-config-records`, `work-item-context`, `check-against-principles`, `handoff`, `sf-soql-authoring`, `sf-apex-log-analysis`. Przed usunięciem przenieś treść review do agenta Reviewer.
3. Komendy spoza zestawu: `/prepare-qa-test-plan`, `/prepare-delivery-feature`, `/archive-work-items`. Jeśli to prompt files, Copilot harness ich nie załaduje.
4. Usuń prompt file `fetch-ado-item`, jeśli istnieje, żeby nie dublować nazwy.
5. Agenci: zaktualizuj `skills:` (spróbuj bez tego pola, żeby skille ładowały się na żądanie). Dopisz w nich format Handoff.
6. Hooki: sprawdź, czy role guard i handoff gate nie odwołują się do usuniętych skilli ani ich formatów.
7. Jeśli coś parsuje format z `development/references/org-change-log.md`, zachowaj ten plik i podlinkuj go z `development`.

## Pętla delivery

Źródło: `deliver-requirement` i `sdlc-conductor` z `Vescik/sf-agentic-sdlc` (commit `e265cba`). Różnica: pętla startuje od gotowego designu, bez intake, designu i design review.

Wymagane, zanim ruszy:

1. Skrypty `check_delivery.py`, `check-delivery.mjs`, `sdlc.py` i globalny safety hook z tego repo (bramka sub-agentów, grant autonomii, reguły push i draft PR). Obejmuje też moduły policy, które hook importuje.
2. `check_delivery.py` czyta AC z `requirements.md`. Zmień `REQUIREMENTS_FILE` na `ac-coverage.md`. Każde `AC-n` w tym pliku staje się wtedy wymagane, także wymienione jako out of scope.
3. SC5, hook push i `sdlc.py` akceptują tylko `feat/*`. Dostosuj je do `work-item/*` albo używaj `feat/*` w delivery run.
4. `sdlc.py` w kroku kończącym wskazuje `design.md`, `requirements.md` i nazwy stacji z `deliver-requirement`. Zaktualizuj prompty.
5. `config/harness.local.json`: `salesforce.agentOrg` = `intDev`, environment `dev`.
6. Agent `reviewer` musi mieć w sobie procedurę review.

Ryzyka:

- Wg `docs/plan05-w8-integration.md` w live runach 8 i 9 (Copilot CLI) conductor osiągnął PASS, ale pominął review i PR. Repo naprawiło to krokiem w `sdlc.py`. Pętlę docelowo trzymałbym w skrypcie.
- Przy współdzielonym dev orgu autonomiczny deploy i `RunLocalTests` dotykają całego zespołu: cudzy czerwony test daje BLOCKED.
- Wg `docs/copilot-cli-port.md` hooki z frontmattera agenta nie działały na CLI 1.0.112. Zakazy muszą siedzieć w globalnym hooku. Copilot harness w VS Code nie był tam zweryfikowany live.
- Wg tego samego dokumentu safety hook przepuszcza nieznane narzędzia MCP, w tym GitHub MCP, więc merge lub PR inny niż draft mógłby tamtędy przejść. Dodaj policy przed włączeniem tego serwera.
- Hook draft PR ufa `delivery-check.json`. Edycja edytorem jest blokowana, zapis przez shell tylko heurystycznie.

## Exit criteria

- `git grep -nE "org-read|sf-metadata-discovery|org-discovery|investigate-config-records|work-item-context|check-against-principles|sf-soql-authoring|sf-apex-log-analysis|skills/handoff" -- .github scripts config` nic nie zwraca.
- `git grep -n "<TODO" -- .github` nic nie zwraca.
- `sf project deploy start -o prod` zostaje zablokowane przez globalny hook, w każdej roli.
- W roli designera `sf config list` zostaje zablokowane. Jeśli przechodzi, role guard na tym hoście nie działa.
- Pętla: jeden run kończy się `delivery-check.json` PASS dla HEAD, wynikiem review i draft PR, bez ręcznego uruchamiania stacji.
- To samo zadanie, model i commit: obecny zestaw kontra nowy, na obu harnessach. Mierz czas do pierwszej edycji, liczbę odczytów z orga i poprawność wyniku.
