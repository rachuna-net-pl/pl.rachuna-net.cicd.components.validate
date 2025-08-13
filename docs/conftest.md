# 🛠 Job template: 🔬 Validate files (conftest)

> [!NOTE]
>
> Job **`Validate files (conftest)`** pozwala na **automatyczną walidację plików YAML/JSON** w oparciu o polityki OPA/Conftest. Pipeline pobiera repozytorium z politykami, przygotowuje środowisko i uruchamia testy w zadanym `namespace`. Idealnie sprawdza się na etapie `validate` w pipeline’ach CI/CD.

## Wymagania wstępne

- [conftest](https://www.conftest.dev/)

---
## ⚙️ Parametry wejściowe (`inputs`)

| Nazwa                        | Typ    | Domyślna wartość                                                                            | Opis                                             |
| ---------------------------- | ------ | ------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `docker_image`               | string | `registry.gitlab.com/pl.rachuna-net/containers/conftest:1.0.0`                              | Obraz Dockera z narzędziem Conftest              |
| `logo_url`                   | string | `https://gitlab.com/pl.rachuna-net/cicd/gitlab-ci/-/raw/main/_configs/_logo?ref_type=heads` | Adres URL logo wyświetlanego w logach            |
| `repository_policies`        | string | `git@gitlab.com:pl.rachuna-net/discipline.git`                                              | Repozytorium GitLab z politykami OPA/Conftest    |
| `repository_policies_branch` | string | `main`                                                                                      | Branch repozytorium z politykami                 |
| `repository_policies_path`   | string | `policies/conftest`                                                                         | Ścieżka katalogu polityk w repozytorium          |
| `source_files`               | string | `*.yml`                                                                                     | Pliki w repozytorium, które mają być testowane   |
| `namespace_policy`           | string | *(puste)*                                                                                   | Przestrzeń nazw dla polityk Conftest (namespace) |

---
## 🧬 Zmienne środowiskowe ustawiane w jobie

* `CONTAINER_IMAGE_CONFTEST` – obraz Dockera Conftest (na podstawie `inputs.docker_image`)
* `LOGO_URL` – ścieżka do logo (na podstawie `inputs.logo_url`)
* `REPOSITORY_POLICIES` – repozytorium z politykami
* `REPOSITORY_POLICIES_BRANCH` – branch repozytorium z politykami
* `REPOSITORY_POLICIES_PATH` – katalog polityk w repozytorium
* `SOURCE_FILES` – pliki do testowania
* `NAMESPACE_POLICY` – przestrzeń nazw polityk Conftest

---
## 📤 Output

Job **wykonuje następujące kroki**:

1. Pobranie logo (jeżeli `LOGO_URL` jest ustawiony).
2. Wyświetlenie informacji o przekazanych parametrach (`_function_print_row`).
3. Klonowanie repozytorium z politykami (`repository_policies`) w odpowiednim branchu.
4. Uruchomienie **Conftest** z wykorzystaniem wskazanych plików (`source_files`) i polityk (`repository_policies_path`) w przestrzeni nazw `namespace_policy`.

Przykład logu:

```
===> 🔬 Conftest Validate - Parameters
┌───────────────────────────────────┬─────────────────────────────────────────────────────┐
│ Variable                          │ Value                                               │
├───────────────────────────────────┼─────────────────────────────────────────────────────┤
│ REPOSITORY_POLICIES               │ git@gitlab.com:pl.rachuna-net/discipline.git        │
│ REPOSITORY_POLICIES_PATH          │ policies/conftest                                   │
│ SOURCE_FILES                      │ *.yml                                               │
└───────────────────────────────────┴─────────────────────────────────────────────────────┘
```

---
## 🧪 Dostępne joby

### **🔬 Validate files (conftest)**

Waliduje wskazane pliki repozytorium względem polityk Conftest:

```yaml
🔬 Validate files (conftest):
  stage: validate
  image: $CONTAINER_IMAGE_CONFTEST
  before_script:
    - git config --global --add safe.directory ${CI_PROJECT_DIR}
    - [ ! -z "${LOGO_URL}" ] && curl -s "${LOGO_URL}"
    - !reference [.validate:_function_print_row]
    - !reference [.validate:conftest_input_variables]
    - !reference [.validate:conftest_prepare]
  script:
    - conftest test $SOURCE_FILES --policy $REPOSITORY_POLICIES_PATH --namespace $NAMESPACE_POLICY
```

---
## 🧪 Przykład użycia w pipeline

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/validate/conftest@$COMPONENT_VERSION_VALIDATE
    inputs:
      repository_policies: git@gitlab.com:pl.rachuna-net/discipline.git
      repository_policies_path: policies/k8s
      source_files: "config.yml"
      namespace_policy: image_builder
```

---
⚡ Dzięki temu szablonowi **szybko weryfikujesz zgodność konfiguracji z politykami OPA/Conftest** bez ręcznej konfiguracji za każdym razem w pipeline’ach.