# 🦠 clamavcvdgenerate

[![Update CVD](https://img.shields.io/github/actions/workflow/status/TestingAvail/clamavcvdgenerate/update-cvd.yml?branch=main&label=update%20cvd&logo=github-actions)](https://github.com/TestingAvail/clamavcvdgenerate/actions/workflows/update-cvd.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Зеркало официальных баз сигнатур ClamAV (`.cvd` / `.cld`). Раз в сутки GitHub Actions скачивает свежие базы, проверяет их через `sigtool` и публикует в релизе `latest`.

## 🎯 Зачем это нужно

Если сервер не может напрямую обращаться к серверам обновлений ClamAV, например из закрытой сети или при ограниченном доступе в интернет, базы можно брать отсюда. Здесь каждый день выкладываются актуальные файлы с SHA-256 суммами.

## 📥 Скачивание баз

Файлы лежат в [последнем релизе](https://github.com/TestingAvail/clamavcvdgenerate/releases/tag/latest). В описании релиза указаны размер и SHA-256 каждого файла.

```bash
# Пример: скачать базы в каталог ClamAV (путь зависит от дистрибутива)
sudo curl -sSL -o /var/lib/clamav/main.cvd https://github.com/TestingAvail/clamavcvdgenerate/releases/download/latest/main.cvd
sudo curl -sSL -o /var/lib/clamav/daily.cvd https://github.com/TestingAvail/clamavcvdgenerate/releases/download/latest/daily.cvd
```

Проверить файл:

```bash
sigtool --info /var/lib/clamav/daily.cvd
sha256sum /var/lib/clamav/daily.cvd   # сравнить с описанием релиза
```

Затем перезапустите сервис ClamAV, например `sudo systemctl restart clamav-daemon`.

## ⚙️ Как это работает

1. Workflow `.github/workflows/update-cvd.yml` запускается каждый день в 03:08 UTC или вручную.
2. Устанавливается [`cvdupdate`](https://pypi.org/project/cvdupdate/) и `clamav`, после чего выполняется `cvd update`.
3. Для каждого `.cvd` / `.cld` вызывается `sigtool --info`. Если проверка не прошла, сборка останавливается.
4. Генерируются заметки к релизу с размерами и SHA-256 суммами.
5. Старый релиз `latest` удаляется, создаётся новый с файлами баз. Те же файлы публикуются через GitHub Pages.

> ⚠️ Из-за пересоздания релиза на короткое время файлы в `latest` могут быть недоступны. Если нужна стабильная ссылка, сохраняйте базы у себя.

## 🛠️ Локальный запуск

```bash
pip install cvdupdate
sudo apt-get install -y clamav
mkdir -p cvd-out
cvd config set --dbdir "$(pwd)/cvd-out"
cvd update
```

## 📄 Лицензия

Распространяется под лицензией MIT. См. [LICENSE](LICENSE). Сами базы ClamAV распространяются по собственным условиям проекта ClamAV.
