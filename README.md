# pve_post_install

Ansible-роль для канонического post-install Proxmox VE / PBS / PMG.

## v1 — что делает

1. **Установка `pve-fake-subscription`** (Jamesits) для убирания
   «No valid subscription» nag-окна. Не патчит системные файлы, переживает
   обновления `proxmox-widget-toolkit`.
2. **Disable enterprise APT repo** — append `Enabled: false` в
   `pve-enterprise.sources` / `pbs-enterprise.sources` / `pmg-enterprise.sources`
   (deb822 формат PVE 8+/PBS 3+/PMG 8+). Fake-subscription НЕ решает `401
   Unauthorized` при `apt update` — это отдельная граблина.

No-subscription repo прописывается PVE-installer'ом сам (`pve-install-repo.sources`)
— роль его не трогает.

## Opt-in

Роль **по умолчанию no-op** (`pve_post_install_enabled: false`). Чтобы
включить — задать в `host_vars` или `group_vars`:

```yaml
pve_post_install_enabled: true
```

## Переменные

| Var | Default | Что |
|---|---|---|
| `pve_post_install_enabled` | `false` | Главный switch — без него роль скипает всё. |
| `pve_post_install_install_fake_subscription` | `true` | Ставить pve-fake-subscription. |
| `pve_post_install_disable_enterprise_repo` | `true` | Append `Enabled: false` в enterprise `.sources`. |
| `pve_fake_subscription_version` | `'0.0.11'` | Версия пакета (без 'v'). |
| `pve_fake_subscription_url` | вычисляется из version | URL deb-файла на GitHub release. |
| `pve_fake_subscription_cache` | `/var/cache/apt/archives/…` | Путь под скачанный deb. |
| `pve_fake_subscription_sha256` | `''` | Опциональный SHA256 verify. |

## Поведение

1. Скипает всё если `pve_post_install_enabled=false`.
2. Asserts на Debian-family.
3. Авто-детект profile: PVE / PBS / PMG (по наличию `pveversion` /
   `proxmox-backup-manager` / `pmgversion`).
4. Asserts что profile один из pve/pbs/pmg.
5. Скачивает deb если версия не совпадает с уже установленной (или пакет
   не установлен).
6. `apt deb=...` для установки.
7. Verify сервис активен.

## Caveat

⚠ После установки **не жать «Check» на Subscription page** в Proxmox UI —
инвалидирует кэш fake-subscription.

## Применимость

Все 8+ PVE/PBS-нод фабрики + 2 PMG (когда дойдём). На Arch-хостах роль
fail-fast (assert Debian).
