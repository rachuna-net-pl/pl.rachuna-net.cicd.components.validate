# 🛠 Job template: 🕵 yamllint

Job, który wykonuje walidację składni plików YAML w repozytorium przy użyciu narzędzia **yamllint**.

---
## ⚙️ Parametry wejściowe (`inputs`)

| Nazwa                     | Typ    | Domyślna wartość                                                                            | Opis                                                     |
| ------------------------- | ------ | ------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `docker_image`            | string | `registry.gitlab.com/pl.rachuna-net/containers/python:2.0.0`                                | Obraz Dockera z interpreterem Pythona lub shellem.       |
| `logo_url`                | string | `https://gitlab.com/pl.rachuna-net/cicd/gitlab-ci/-/raw/main/_configs/_logo?ref_type=heads` | URL logotypu wyświetlanego w logach joba.                |
| `validate_repo_namespace` | string | `pl.rachuna-net/cicd/components/validate`                                                   | Namespace i ścieżka do repozytorium komponentu validate. |
| `validate_repo_branch`    | string | `main`                                                                                      | Gałąź komponentu validate, z której pobierane są pliki.  |
| `yamllint_path`           | string | `_configs/validate/.yamllint.yml`                                                           | Ścieżka do pliku konfiguracyjnego **yamllint**.          |

---
## 🧬 Zmienne środowiskowe obsługiwane przez skrypt

Job ustawia i wykorzystuje poniższe zmienne środowiskowe:

* `CONTAINER_IMAGE_PYTHON` – wybrany obraz Dockera.
* `LOGO_URL` – adres logotypu, który jest opcjonalnie pobierany w trakcie uruchamiania joba.
* `VALIDATE_REPO_NAMESPACE` – namespace repozytorium komponentu validate.
* `VALIDATE_REPO_BRANCH` – gałąź repozytorium komponentu validate.
* `YAMLLINT_PATH` – ścieżka do pliku konfiguracyjnego yamllint.

---
## 📤 Output

Job wykonuje linting plików YAML i wypisuje wyniki w logach pipeline’u w formie standardowej dla narzędzia **yamllint**, np.:

```
===> 🔍 YAML lint results
source/ci-template.yml
  10:3      warning  missing document start "---"  (document-start)
  22:1      error    too many blank lines (1 > 0)  (empty-lines)
```

W przypadku błędów job kończy się statusem `failed`.