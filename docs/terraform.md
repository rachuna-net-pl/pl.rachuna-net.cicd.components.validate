**Wymagania**
- Obraz kontenera dostępny jest [tutaj](https://gitlab.com/pl.rachuna-net/containers/terraform/container_registry/8516490).

#### `✅ terraform fmt`  
Job sprawdza poprawność formatowania plików Terraform w repozytorium. Jeśli pliki nie są zgodne ze standardem formatowania Terraform, pipeline zakończy się błędem.  

#### `✅ terraform validate`  
Job uruchamia `terraform init`, integrując się ze stanem zapisanym w repozytorium GitLab. Następnie, za pomocą polecenia `terraform validate`, przeprowadzana jest walidacja kodu.  
