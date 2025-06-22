### ⚙️ Parametry wejściowe (`inputs`)

| Nazwa          | Typ    | Domyślna wartość                                             | Opis                                    |
| -------------- | ------ | ------------------------------------------------------------ | --------------------------------------- |
| `docker_image` | string | `registry.gitlab.com/pl.rachuna-net/containers/packer:1.0.0` | Obraz Dockera z zainstalowanym Packerem |

---
### 🧬 Zmienne środowiskowe

| Nazwa zmiennej           | Wartość                      |
| ------------------------ | ---------------------------- |
| `CONTAINER_IMAGE_PACKER` | `$[[ inputs.docker_image ]]` |
| `PACKER_LOG`             | `0`                          |

---
### 🧱 Zależności

* Pliki lokalne:

  * `/source/input_variables_packer.yml` – dodatkowe zmienne środowiskowe
  * `/source/logo.yml` – logo komponentu

* Packer musi mieć dostęp do plików `.pkr.hcl` i folderu `pkrvars/`

---
### 🧪 Joby walidujące

#### 🕵 Job: `packer fmt (syntax check)`

* Sprawdza poprawność formatowania plików `.hcl` z katalogu `pkrvars/`
* Używa polecenia `packer fmt -check`
* Nie wykonuje zmian
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
packer fmt -check -var-file=pkrvars/example.pkrvars.hcl .
```

---
#### ✅ Job: `packer validate`

* Waliduje konfigurację Packer dla każdego pliku `.hcl` w katalogu `pkrvars/`
* Poprzedzony `packer init`
* Nie uruchamia się automatycznie (`rules: when: never`)

```bash
packer validate -var-file=pkrvars/example.pkrvars.hcl .
```

---
### 🧪 Przykład użycia

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/validate/packer@$COMPONENT_VERSION_VALIDATE
    inputs:
      docker_image: $CONTAINER_IMAGE_PACKER

✅ packer validate:
  rules: !reference [.rule:validate:packer, rules]

🕵 packer fmt (syntax check):
  rules: !reference [.rule:validate:packer, rules]
```