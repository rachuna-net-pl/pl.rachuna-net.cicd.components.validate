# <img src=".gitlab/avatar.png" alt="avatar" height="20"/> validate

[![](https://gitlab.com/pl.rachuna-net/cicd/components/validate/-/badges/release.svg)](https://gitlab.com/pl.rachuna-net/cicd/components/validate/-/releases)
[![](https://gitlab.com/pl.rachuna-net/cicd/components/validate/badges/main/pipeline.svg)](https://gitlab.com/pl.rachuna-net/cicd/components/validate/-/commits/main)

Komponent do automatycznej walidacji jakości i poprawności kodu w procesach CI/CD.

* [Terraform](#terraform)
* [YAMLLINT](#yamllint)

---
### Ansible
::include{file=docs/ansible.md}

---
## Packer
::include{file=docs/packer.md}

---
## Terraform
### ⚙️ Parametry wejściowe (`inputs`)

| Nazwa           | Typ    | Domyślna wartość                                                | Opis                                                                 |
| --------------- | ------ | --------------------------------------------------------------- | -------------------------------------------------------------------- |
| `docker_image`  | string | `registry.gitlab.com/pl.rachuna-net/containers/terraform:1.0.0` | Obraz Dockera z zainstalowanym Terraformem                           |
| `tf_state_name` | string | `default`                                                       | Nazwa pliku stanu Terraform (używana podczas inicjalizacji backendu) |
| `debug`         | string | `"false"`                                                       | Czy włączyć tryb debugowania (`TF_LOG=debug`)                        |

### 🧬 Zmienne środowiskowe

| Nazwa zmiennej              | Wartość                       |
| --------------------------- | ----------------------------- |
| `CONTAINER_IMAGE_TERRAFORM` | `$[[ inputs.docker_image ]]`  |
| `TF_STATE_NAME`             | `$[[ inputs.tf_state_name ]]` |
| `TF_VAR_gitlab_token`       | `$GITLAB_TOKEN`               |
| `DEBUG`                     | `$[[ inputs.debug ]]`         |

### 🧱 Zależności

* Pliki lokalne:

  * `/source/terraform_input_variables.yml` – ustawienia dodatkowych zmiennych
  * `/source/logo.yml` – logo komponentu
  * `/source/terraform_init.yml` – inicjalizacja backendu Terraform

* Wymaga zmiennej `GITLAB_TOKEN` (przekazywanej jako `TF_VAR_gitlab_token`)

### **Definicje jobów**

#### 🕵 Job: `terraform fmt`

* Sprawdza poprawność formatowania plików `.tf`
* Nie wykonuje zmian – tylko tryb walidacyjny (`-check`)
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
terraform fmt -recursive -check
```

#### ✅ Job: `terraform validate`

* Waliduje pliki Terraform (`.tf`) względem składni, zależności i zmiennych
* Poprzedzony `terraform init`
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
terraform validate
```

#### ✅ Job: `tflint`

Job odpowiedzialny za uruchomienie TFLint – narzędzia służącego do statycznej analizy kodu Terraform.
Pomaga wykrywać błędy, niespójności oraz niezalecane praktyki jeszcze przed wdrożeniem.

```bash
tflint
```

#### ✅ Job: `terraform-docs`

Job odpowiedzialny za automatyczne generowanie dokumentacji modułów Terraform z wykorzystaniem narzędzia terraform-docs.
Dzięki temu README.md zawsze pozostaje aktualne względem kodu modułów (zmienne, outputy itd.).

```bash
terraform-docs -c .terraform-docs.yml .
```

---
### 🧪 Przykład użycia

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/validate/terraform@$COMPONENT_VERSION_VALIDATE
    inputs:
      docker_image: $CONTAINER_IMAGE_TERRAFORM

✅ terraform validate:
  rules: !reference [.rule:validate:terraform, rules]


🕵 terraform fmt:
  rules: !reference [.rule:validate:terraform, rules]
```

---
## YAMLLINT

> [!NOTE]
>
> Komponent ma na celu przeprowadzenie walidacji wszystkich plików YAML w repozytorium. Wykorzystuje on narzędzie [`yamllint`](https://yamllint.readthedocs.io/en/stable/) oraz centralnie zdefiniowany plik konfiguracyjny `.yamllint.yml`.
Wyniki walidacji są zwracane w ramach joba w **stage** `validate`.

### **Wejścia (inputs)**

Komponent obsługuje parametryzację za pomocą zmiennych wejściowych:

| Zmienna                    | Domyślna wartość                                             | Opis                                                                                |
| -------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| `docker_image`             | `registry.gitlab.com/pl.rachuna-net/containers/python:2.0.0` | Obraz Dockera używany do uruchomienia joba. Powinien zawierać narzędzie `yamllint`. |
| `component_repo_namespace` | `pl.rachuna-net/cicd/components/validate`                    | Namespace i ścieżka do repozytorium z komponentem walidacyjnym.                     |
| `component_repo_branch`    | `main`                                                       | Gałąź repozytorium komponentu, z której korzysta pipeline.                          |
| `yamllint_path`            | `_configs/.yamllint.yml`                                     | Ścieżka do pliku konfiguracyjnego `yamllint`.                                       |

---

### **Zmienne środowiskowe**

Na podstawie `inputs` ustawiane są następujące zmienne środowiskowe:

* `CONTAINER_IMAGE_PYTHON` – obraz Dockera do użycia w jobie.
* `COMPONENT_REPO_NAMESPACE` – namespace repozytorium komponentu.
* `COMPONENT_REPO_BRANCH` – gałąź repozytorium komponentu.
* `YAMLLINT_PATH` – ścieżka do pliku konfiguracyjnego `yamllint`.

---

### **Struktura include**

Komponent do działania wymaga kilku lokalnych plików `include`:

* **`/source/function_print_row.yml`** – funkcje pomocnicze (logowanie/drukowanie wyników),
* **`/source/logo.yml`** – wyświetlanie logotypu w logach joba,
* **`/source/yamlint_input_variables.yml`** – wczytanie zmiennych wejściowych,
* **`/source/yamlint_prepare.yml`** – przygotowanie środowiska dla `yamllint`.

---

### **Definicje jobów**

**`🕵 YAML lint`**

Job wykonywany w **stage:** `validate`, który rozszerza `.validate:yamlint:base`.
W praktyce jest to główny job walidacji YAML w pipeline.

---

### **Przykład użycia**

```yaml
stages:
  - validate

include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/validate/yaml@$COMPONENT_VERSION_VALIDATE
    inputs:
      docker_image: $CONTAINER_IMAGE_PYTHON

🕵 YAML lint:
  stage: validate
  rules: !reference [.rule:validate:yamllint, rules]
```

---
## Contributions
Jeśli masz pomysły na ulepszenia, zgłoś problemy, rozwidl repozytorium lub utwórz Merge Request. Wszystkie wkłady są mile widziane!
[Contributions](CONTRIBUTING.md)

---
## License
Projekt licencjonowany jest na warunkach [Licencji MIT](LICENSE).

---
# Author Information
### &emsp; Maciej Rachuna
# <img src="https://gitlab.com/pl.rachuna-net/gitlab-profile/-/raw/main/assets/logo/website_logo_transparent_background.png" alt="rachuna-net.pl" height="100"/>


