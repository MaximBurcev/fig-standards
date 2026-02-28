# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Проект

Русскоязычный перевод стандартов PHP-FIG (PSR — PHP Standards Recommendations), опубликованный на https://php-psr.ru/. Статический сайт собирается с помощью [MkDocs](https://www.mkdocs.org/).

## Сборка и локальный запуск

```bash
# Установить MkDocs и плагины (если не установлены)
pip install mkdocs mkdocs-material mkdocs-meta-descriptions-plugin

# Локальный сервер с живой перезагрузкой
mkdocs serve

# Собрать статический сайт в папку site/
mkdocs build
```

## Структура документации

- `docs/accepted/` — принятые стандарты PSR (основной контент для перевода)
- `docs/proposed/` — стандарты в статусе черновика/предложения
- `docs/bylaws/` — регламенты PHP-FIG
- `docs/index.md` — главная страница с индексом всех PSR
- `mkdocs.yml` — конфигурация сайта, навигация и тема

Каждый PSR, как правило, имеет два файла: сам стандарт (`PSR-N-name.md`) и мета-документ с обоснованием решений (`PSR-N-name-meta.md`).

## Правила работы с контентом

- Язык сайта — **русский**. Все переводы должны быть на русском языке.
- Не добавлять новые PSR в навигацию `mkdocs.yml`, не убедившись, что файл перевода существует в `docs/`.
- Папка `site/` — артефакт сборки, не редактировать вручную.
- Все изменения через pull request; прямые коммиты в `master` запрещены (ветка защищена).
- Никогда не делать force push в репозиторий php-fig organisation.
