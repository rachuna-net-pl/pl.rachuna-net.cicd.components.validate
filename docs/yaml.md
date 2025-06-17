**Wymagania**  
- Obraz kontenera dostępny jest [tutaj](https://gitlab.com/pl.rachuna-net/containers/python/container_registry/8443096).
- Plik konfiguracyjny **yamllint** jest pobierany z:  
  [GitLab API](https://gitlab.com/api/v4/projects/68158616/repository/files/_configs%2F.yamllint.yml/raw?ref=main)  

#### `🕵 YAML lint`  
Job sprawdza poprawność składni oraz zgodność plików YAML z ustalonymi regułami. Weryfikacja formatowania odbywa się za pomocą narzędzia [`yamllint`](https://github.com/adrienverge/yamllint).  

- Jeśli repozytorium nie zawiera pliku `.yamllint`, zostanie on pobrany automatycznie z zdefiniowanego źródła (`yamllint_url`).  
- Weryfikacja wykonywana jest poleceniem:  
  ```sh
  yamllint .
  ```
- Jeśli jakikolwiek plik YAML zawiera błędy składniowe lub formatowanie nie jest zgodne z regułami `yamllint`, pipeline zakończy się błędem.  
