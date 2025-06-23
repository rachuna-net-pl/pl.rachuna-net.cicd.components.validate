### ⚙️ Parametry wejściowe (`inputs`)

| Nazwa               | Typ    | Domyślna wartość                                              | Opis                                                 |
| ------------------- | ------ | ------------------------------------------------------------- | ---------------------------------------------------- |
| `docker_image`      | string | `registry.gitlab.com/pl.rachuna-net/containers/ansible:1.0.0` | Obraz Dockera zawierający Ansible                    |
| `ansible_inventory` | string | `inventory/hosts.yml`                                         | Ścieżka do pliku inwentarza Ansible                  |
| `ansible_vars`      | string | `""`                                                          | Dodatkowe zmienne przekazywane do `ansible-playbook` |

---
### 🧬 Zmienne środowiskowe

| Nazwa zmiennej              | Wartość                           | Opis                                             |
| --------------------------- | --------------------------------- | ------------------------------------------------ |
| `CONTAINER_IMAGE_ANSIBLE`   | `$[[ inputs.docker_image ]]`      | Obraz Dockera z Ansible                          |
| `ANSIBLE_HOST_KEY_CHECKING` | `false`                           | Wyłącza weryfikację kluczy hostów SSH            |
| `ANSIBLE_FORCE_COLOR`       | `true`                            | Wymusza kolorowy output                          |
| `ANSIBLE_INVENTORY`         | `$[[ inputs.ansible_inventory ]]` | Ścieżka do inwentarza                            |
| `ANSIBLE_VARS`              | `$[[ inputs.ansible_vars ]]`      | Zmienna `--extra-vars` przekazywana do playbooka |

---
### 🧱 Zależności

**Lokalne pliki include:**

* `/source/input_variables_ansible.yml` – mapowanie zmiennych wejściowych
* `/source/logo.yml` – logo komponentu
* `/source/ansible_init.yml` – inicjalizacja środowiska Ansible

---
### ✅ Job: `ansible-playbook check`

* Przechodzi po wszystkich plikach `playbooks/*.yml`.
* Wykonuje `ansible-playbook` w trybie `--check`.
* Przekazuje zdefiniowany inwentarz i zmienne.

#### 📜 Skrypt

```bash
for file in playbooks/*.yml; do
  printf "┌ %-10s ┬ %-130s ┐\n" "──────────" "──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────"
  printf "| %-10s | %-130s |\n" "Playbook" "$file"
  printf "└ %-10s ┴ %-130s ┘\n" "──────────" "──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────"
  ansible-playbook -i $ANSIBLE_INVENTORY $file --check --extra-vars "$ANSIBLE_VARS"
done
```

---
### 🧪 Przykład użycia

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/validate/ansible-playbook-check@$COMPONENT_VERSION_VALIDATE
    inputs:
      ansible_inventory: "inventory/prod.yml"
      ansible_vars: '{"env": "prod"}'

✅ ansible-playbook check:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
```
