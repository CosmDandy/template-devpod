# Шаблон для проектов с DevPod

[![Open in GitHub Codespaces][codespaces]](https://codespaces.new/CosmDandy/template-devpod)

[![github.dev][github.dev]](https://github.dev/CosmDandy/template-devpod) [![license][license]](LICENSE)

Этот репозиторий является отправной точкой для проектов при работе с которыми я использую [DevPod](https://devpod.sh/)

```bash
devpod up git@github.com:CosmDandy/template-devpod.git --id template-devpod-[...] --provider [...]
```

## [Docker in docker](https://github.com/devcontainers/features/tree/main/src/docker-in-docker)

```
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
```

## [Docker outside of docker](https://github.com/devcontainers/features/tree/main/src/docker-outside-of-docker)

```
  "mounts": [
    "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind"
  ],
  "features": {
    "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {}
  },
  "runArgs": [
    "--privileged",
    "--pid=host",
    "--network=host"
  ],
```

## Секреты: sops + direnv

В корне лежит `.envrc` с функцией `use_sops`. Она расшифровывает
`secrets.sops.yaml` и раскладывает пары в переменные окружения — по одному
обращению к хранилищу на файл, а не на каждое значение.

```bash
# 1. правила шифрования: кому доступен файл
cat > .sops.yaml <<'YAML'
creation_rules:
  - path_regex: secrets\.sops\.yaml$
    age: age1...          # публичный ключ получателя
YAML

# 2. завести секреты (откроется редактор)
sops secrets.sops.yaml

# 3. разрешить direnv — один раз на каталог и после каждой правки .envrc
direnv allow
```

Дальше `cd` в каталог сам поднимает окружение, а `watch_file` перечитает его,
когда `secrets.sops.yaml` изменится.

Зашифрованный файл **коммитится** — без ключа он бесполезен, и в `.gitignore`
его нет намеренно.

### Несколько окружений в одном репозитории

Вложенный `.envrc` подключает функции из корневого первой строкой:

```bash
# terraform/live/prod/.envrc
source_up
use_sops
```

### Функции определяются в репозитории, а не в личном direnvrc

`~/.config/direnv/direnvrc` есть только на твоих машинах, а `.envrc` уезжают в
git. Если объявить `use_sops` там, у всех остальных direnv оборвётся на
`command not found` и не экспортирует ничего — а `terraform` после этого не
упадёт, а пойдёт без `TF_VAR_*`. Поэтому определения живут здесь и опираются
только на штатную stdlib direnv (`has`, `log_error`, `direnv_load`,
`watch_file`).


## Бейджи для нового проекта

Ряд собран так, чтобы читаться как одна система: левая половина у всех одна и
та же серая (`labelColor=21262d`), цвет несёт справа. Живой статус оставляет
семантику — зелёный при passing, красный при failing; статика красится в
пурпур `7828dc` и циан `00a8c8` из палитры og-картинки.

Всё идёт через shields.io намеренно: родные бейджи GitHub и Scorecard рисуются
другими сервисами, и в одном ряду у них разная высота и разные скругления.

Ссылки вынесены в reference-style в конец файла — иначе строка с логотипом
SLSA занимает больше тысячи символов прямо в тексте.

Замени `REPO` на имя репозитория и `WORKFLOW` на файл нужного workflow:

```markdown
[![Open in GitHub Codespaces][codespaces]](https://codespaces.new/CosmDandy/REPO)

[![build][build]](https://github.com/CosmDandy/REPO/actions/workflows/WORKFLOW) [![scorecard][scorecard]](https://scorecard.dev/viewer/?uri=github.com/CosmDandy/REPO) [![license][license]](LICENSE)

[codespaces]: https://github.com/codespaces/badge.svg
[build]: https://img.shields.io/github/actions/workflow/status/CosmDandy/REPO/WORKFLOW?branch=master&style=flat&label=build&labelColor=21262d&logo=githubactions&logoColor=8b949e
[scorecard]: https://img.shields.io/ossf-scorecard/github.com/CosmDandy/REPO?style=flat&label=scorecard&labelColor=21262d
[license]: https://img.shields.io/github/license/CosmDandy/REPO?style=flat&label=license&labelColor=21262d&color=484f58&logo=opensourceinitiative&logoColor=8b949e
```

Дополнительные, по смыслу проекта:

```markdown
[ghcr.io]: https://img.shields.io/badge/ghcr.io-IMAGE-00a8c8?style=flat&labelColor=21262d&logo=docker&logoColor=8b949e
[terraform]: https://img.shields.io/badge/terraform-1.x-7828dc?style=flat&labelColor=21262d&logo=terraform&logoColor=8b949e
[cloudflare]: https://img.shields.io/badge/cloudflare-workers-00a8c8?style=flat&labelColor=21262d&logo=cloudflare&logoColor=8b949e
```

Логотипа OpenSSF в simple-icons нет, поэтому scorecard идёт без иконки.
Логотип SLSA тоже отсутствует — он вшивается data-URI, готовую строку возьми
из README любого репозитория с образами (wallcal, soniox-openai-shim).

[codespaces]: https://github.com/codespaces/badge.svg
[github.dev]: https://img.shields.io/badge/edit-github.dev-7828dc?style=flat&labelColor=21262d&logo=github&logoColor=8b949e
[license]: https://img.shields.io/github/license/CosmDandy/template-devpod?style=flat&label=license&labelColor=21262d&color=484f58&logo=opensourceinitiative&logoColor=8b949e
