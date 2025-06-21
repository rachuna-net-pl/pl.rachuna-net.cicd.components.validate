### ⚙️ Parametry wejściowe (`inputs`)

| Nazwa           | Typ    | Domyślna wartość                                                | Opis                                                                 |
| --------------- | ------ | --------------------------------------------------------------- | -------------------------------------------------------------------- |
| `docker_image`  | string | `registry.gitlab.com/pl.rachuna-net/containers/terraform:1.0.0` | Obraz Dockera z zainstalowanym Terraformem                           |
| `tf_state_name` | string | `default`                                                       | Nazwa pliku stanu Terraform (używana podczas inicjalizacji backendu) |
| `debug`         | string | `"false"`                                                       | Czy włączyć tryb debugowania (`TF_LOG=debug`)                        |

---
### 🧬 Zmienne środowiskowe

| Nazwa zmiennej              | Wartość                       |
| --------------------------- | ----------------------------- |
| `CONTAINER_IMAGE_TERRAFORM` | `$[[ inputs.docker_image ]]`  |
| `TF_STATE_NAME`             | `$[[ inputs.tf_state_name ]]` |
| `TF_VAR_gitlab_token`       | `$GITLAB_TOKEN`               |
| `DEBUG`                     | `$[[ inputs.debug ]]`         |

---

### 🧱 Zależności

* Pliki lokalne:

  * `/source/input_variables_terraform.yml` – ustawienia dodatkowych zmiennych
  * `/source/logo.yml` – logo komponentu
  * `/source/terraform_init.yml` – inicjalizacja backendu Terraform

* Wymaga zmiennej `GITLAB_TOKEN` (przekazywanej jako `TF_VAR_gitlab_token`)

---
### 🧪 Joby walidujące

#### 🕵 Job: `terraform fmt`

* Sprawdza poprawność formatowania plików `.tf`
* Nie wykonuje zmian – tylko tryb walidacyjny (`-check`)
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
terraform fmt -recursive -check
```

---
#### ✅ Job: `terraform validate`

* Waliduje pliki Terraform (`.tf`) względem składni, zależności i zmiennych
* Poprzedzony `terraform init`
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
terraform validate
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