# Generate ABIS

Модуль Drupal 7 для предоставления REST API данных журнала и научных статей. Предназначен для интеграции с внешними системами (например, eLibrary, ABIС).

## Содержание

- [Установка](#установка)
- [Зависимости](#зависимости)
- [Очистка кэша](#очистка-кэша)
- [API эндпоинты](#api-эндпоинты)
  - [1. GET /api/generate-abis/journal-number](#1-get-apigenerate-abisjournal-number)
  - [2. GET /api/generate-abis/journal-data](#2-get-apigenerate-abisjournal-data)
  - [3. GET /api/generate-abis/journal-list/%nid](#3-get-apigenerate-abisjournal-listnid)
  - [4. GET /api/generate-abis/journalarticle/%nid](#4-get-apigenerate-abisjournalarticlenid)
- [Внутренние функции](#внутренние-функции)
- [Типы материалов](#типы-материалов)
- [Таксономия](#таксономия)
- [Поля пользователей](#поля-пользователей)
- [Drupal Variables](#drupal-variables)

---

## Установка

1. Скопировать директорию `generate_abis` в `sites/all/modules/custom/`
2. Включить модуль на странице `admin/modules`
3. Очистить кэш: `drush cc all`

## Зависимости

- **Drupal 7**
- **Entity API** — для `EntityFieldQuery`
- **i18n (Internationalization)** — для получения переводов таксономии (`i18n_get_object`)
- **Entity Translation** — для мультиязычных полей (ru/en)

## Очистка кэша

После любых изменений в модуле:

```bash
drush cc all
```

---

## API эндпоинты

Все эндпоинты возвращают JSON с `Content-Type: application/json; charset=utf-8`. Кириллические символы выводятся без Unicode-escape кодировки (`JSON_UNESCAPED_UNICODE`).

---

### 1. GET /api/generate-abis/journal-number

Возвращает список всех опубликованных выпусков журнала (тип материала `journal_number`).

#### Параметры

Нет.

#### Пример запроса

```
GET https://mmi.ddev.site/api/generate-abis/journal-number
```

#### Пример ответа

```json
[
  {
    "id": 4923,
    "title": {
      "ru": "Известия Саратовского университета. Новая серия. Серия «Математика. Механика. Информатика» 2006, Т. 6, вып. 1",
      "en": "Izvestiya of Saratov University. New Series. Series: Mathematics. Mechanics. Informatics 2006, vol. 6, iss. 1"
    },
    "year": "2006",
    "volume": "6",
    "number": "1",
    "part": "",
    "journal_no_start": "",
    "journal_no_end": ""
  }
]
```

#### Описание полей

| Поле | Тип | Описание | Источник (поле Drupal) |
|------|-----|----------|----------------------|
| `id` | int | NID ноды выпуска | `node->nid` |
| `title` | object | Название выпуска | `title_field` (ru), `title_field` (en) |
| `title.ru` | string | Название на русском | `title_field` (язык по умолчанию) |
| `title.en` | string | Название на английском | `title_field` (en) |
| `year` | string | Год выпуска (4 цифры) | `field_year` (обрезается до 4 символов) |
| `volume` | string | Том | `field_volume` |
| `number` | string | Номер выпуска | `field_number` |
| `part` | string | Часть | `field_part` |
| `journal_no_start` | string | Начальный номер страниц | `field_journal_no_start` |
| `journal_no_end` | string | Конечный номер страниц | `field_journal_no_end` |

---

### 2. GET /api/generate-abis/journal-data

Возвращает глобальные данные журнала (titleid, ISSN) из переменных Drupal.

#### Параметры

Нет.

#### Пример запроса

```
GET https://mmi.ddev.site/api/generate-abis/journal-data
```

#### Пример ответа

```json
{
  "titleid": "12345",
  "issnPrint": "1816-9791",
  "issnOnline": "2500-2198"
}
```

#### Описание полей

| Поле | Тип | Описание | Drupal Variable |
|------|-----|----------|-----------------|
| `titleid` | string | Идентификатор в eLibrary | `elibrary_titleid` |
| `issnPrint` | string | Печатный ISSN | `issnPrint_citation` |
| `issnOnline` | string | Онлайн ISSN | `issnOnline_citation` |

---

### 3. GET /api/generate-abis/journal-list/%nid

Возвращает список всех статей (`journalarticle`), привязанных к указанному выпуску журнала через поле `field_journal_link` (Entity Reference).

#### Параметры

| Параметр | Тип | Описание |
|----------|-----|----------|
| `nid` | int | NID ноды типа `journal_number` |

#### Пример запроса

```
GET https://mmi.ddev.site/api/generate-abis/journal-list/4923
```

#### Пример ответа

```json
[
  {
    "id": 7253,
    "title": {
      "ru": "Теорема единственности для периодических в среднем функций на гипергруппе Бесселя – Кингмана",
      "en": "A uniqueness theorem for mean periodic functions on the Bessel – Kingman hypergroup"
    }
  },
  {
    "id": 7254,
    "title": {
      "ru": "Другая статья",
      "en": "Another article"
    }
  }
]
```

#### Описание полей

| Поле | Тип | Описание |
|------|-----|----------|
| `id` | int | NID ноды статьи |
| `title` | object | Название статьи |
| `title.ru` | string | Название на русском |
| `title.en` | string | Название на английском |

#### Ошибки

| Код | Условие | Ответ |
|-----|---------|-------|
| 400 | `nid` не является положительным числом | `{"error": "Invalid node id."}` |

---

### 4. GET /api/generate-abis/journalarticle/%nid

Возвращает полную информацию о статье (тип материала `journalarticle`) по её NID.

#### Параметры

| Параметр | Тип | Описание |
|----------|-----|----------|
| `nid` | int | NID ноды типа `journalarticle` |

#### Пример запроса

```
GET https://mmi.ddev.site/api/generate-abis/journalarticle/7253
```

#### Пример ответа

```json
{
  "id": 7253,
  "language": "ru",
  "title": {
    "ru": "Теорема единственности для периодических в среднем функций",
    "en": "A uniqueness theorem for mean periodic functions"
  },
  "language": "ru",
  "heading": {
    "tid": 156,
    "ru": "Математический анализ",
    "en": "Mathematical Analysis"
  },
  "typersci": {
    "tid": 9912,
    "ru": "Научная статья",
    "en": "Research Article",
    "abbreviation": "NA"
  },
  "udk": "517.5",
  "edn": "XFGHJK",
  "doi": "10.18500/1816-9791-2024-24-1-12-25",
  "autor": [
    {
      "uid": 3477,
      "num": "001",
      "surname": {
        "ru": "Иванов",
        "en": "Ivanov"
      },
      "name": {
        "ru": "Иван",
        "en": "Ivan"
      },
      "middle_name": {
        "ru": "Иванович",
        "en": "Ivanovich"
      },
      "initials": {
        "ru": "Иван Иванович",
        "en": "Ivan Ivanovich"
      },
      "company": {
        "target_id": 6977,
        "data": {
          "orgName": {
            "ru": "Саратовский государственный университет",
            "en": "Saratov State University"
          },
          "address": {
            "ru": "г. Саратов, ул. Астраханская, 83",
            "en": "83 Astrakhanskaya St., Saratov"
          }
        }
      },
      "researcherid": "A-1234-5678",
      "spin": "1234-5678-9012-3456",
      "scopusid": "571910 uniqueness theorem",
      "orcid": "0000-0001-2345-6789"
    }
  ],
  "key_words": {
    "ru_page": [
      {
        "tid": 12492,
        "ru": "обобщенный сдвиг",
        "en": "generalized translation"
      }
    ],
    "en_page": [
      {
        "tid": 11821,
        "ru": "generalized translation",
        "en": "dfs dsfdsf9"
      }
    ]
  },
  "acknowledgments": "Исследование поддержано грантом...",
  "funding": "РНФ, проект № 24-11-00000",
  "date_received": "28.09.2023",
  "accepted": "13.03.2024",
  "published": "28.02.2025",
  "page_no": "12",
  "page_no_to": "25",
  "body": {
    "ru": "Одно из свойств периодической функции на вещественной оси...",
    "en": "One of the properties of a periodic function on the real axis..."
  },
  "literature": [
    { "text": "GBD 2019 Diseases and Injuries Collaborators. Global burden of 369 diseases..." },
    { "text": "Bui F. Q., Almeida-da-Silva C. L. C. Association between periodontal pathogens..." }
  ],
  "short_text_pdf": {
    "fid": 1234,
    "filename": "short_text.pdf",
    "uri": "public://short_text.pdf",
    "url": "https://mmi.ddev.site/sites/default/files/short_text.pdf",
    "description": "",
    "display": 1
  },
  "fulltext": "",
  "text_pdf": {
    "fid": 5678,
    "filename": "article.pdf",
    "uri": "public://article.pdf",
    "url": "https://mmi.ddev.site/sites/default/files/article.pdf",
    "description": "",
    "display": 1
  },
  "text_pdf_en": "",
  "journal_link": "4923",
  "status": "published"
}
```

#### Описание полей

| Поле | Тип | Описание | Источник (поле Drupal) |
|------|-----|----------|----------------------|
| `id` | int | NID ноды статьи | `node->nid` |
| `language` | string | Язык ноды | `node->language` |
| `title` | object | Название статьи | `title_field` |
| `title.ru` | string | Название на русском | `title_field` (ru) |
| `title.en` | string | Название на английском | `title_field` (en) |
| `heading` | object/null | Рубрика статьи (таксономия) | `field_heading` (tid) |
| `heading.tid` | int | TID термина таксономии | — |
| `heading.ru` | string | Название рубрики на русском | `taxonomy_term->name` |
| `heading.en` | string | Название рубрики на английском | `i18n localize('en')` |
| `typersci` | object/null | Тип научной работы (таксономия) | `field_typersci` (tid) |
| `typersci.tid` | int | TID термина таксономии | — |
| `typersci.ru` | string | Название типа на русском | `taxonomy_term->name` |
| `typersci.en` | string | Название типа на английском | `i18n localize('en')` |
| `typersci.abbreviation` | string | Аббревиатура типа | `field_abbreviation` термина |
| `udk` | string | УДК | `field_udk` |
| `edn` | string | Идентификатор eLibrary (EDN) | `field_edn` |
| `doi` | string | DOI статьи | `field_doi` |
| `autor` | array | Список авторов статьи | `field_autor` (uid) |
| `autor[].uid` | int | UID пользователя (автора) | — |
| `autor[].num` | string | Порядковый номер автора (001, 002, ...) | Генерируется автоматически |
| `autor[].surname` | object | Фамилия | `user->field_lastname` |
| `autor[].surname.ru` | string | Фамилия на русском | `field_lastname` (ru) |
| `autor[].surname.en` | string | Фамилия на английском | `field_lastname` (en) |
| `autor[].name` | object | Имя | `user->field_name2` |
| `autor[].name.ru` | string | Имя на русском | `field_name2` (ru) |
| `autor[].name.en` | string | Имя на английском | `field_name2` (en) |
| `autor[].middle_name` | object | Отчество | `user->field_middle_name` |
| `autor[].middle_name.ru` | string | Отчество на русском | `field_middle_name` (ru) |
| `autor[].middle_name.en` | string | Отчество на английском | `field_middle_name` (en) |
| `autor[].initials` | object | Имя + Отчество | Генерируется: `name + ' ' + middle_name` |
| `autor[].initials.ru` | string | Инициалы на русском | — |
| `autor[].initials.en` | string | Инициалы на английском | — |
| `autor[].company` | object | Организация автора | `user->field_company_link` (Entity Reference на ноду организации) |
| `autor[].company.target_id` | int | NID ноды организации | — |
| `autor[].company.data` | object/null | Данные организации | Загружается через `node_load()` |
| `autor[].company.data.orgName` | object | Название организации | `title_field` ноды организации |
| `autor[].company.data.orgName.ru` | string | Название на русском | `title_field` (ru) |
| `autor[].company.data.orgName.en` | string | Название на английском | `title_field` (en) |
| `autor[].company.data.address` | object | Адрес организации | `field_address` ноды организации |
| `autor[].company.data.address.ru` | string | Адрес на русском | `field_address` (ru) |
| `autor[].company.data.address.en` | string | Адрес на английском | `field_address` (en) |
| `autor[].researcherid` | string | ResearcherID (Web of Science) | `user->field_researcherid` |
| `autor[].spin` | string | SPIN-код (eLibrary) | `user->field_spin_elibrary` |
| `autor[].scopusid` | string | Scopus Author ID | `user->field_scopusid` |
| `autor[].orcid` | string | ORCID | `user->field_orcid` |
| `key_words` | object | Ключевые слова (таксономия) | `field_key_words` |
| `key_words.ru_page` | array | Термины с русской версии страницы | `field_key_words` (ru) |
| `key_words.en_page` | array | Термины с английской версии страницы | `field_key_words` (en) |
| `key_words.*.[].tid` | int | TID термина таксономии | — |
| `key_words.*.[].ru` | string | Название термина (оригинал) | `taxonomy_term->name` |
| `key_words.*.[].en` | string | Название термина (перевод) | `i18n localize('en')` |
| `acknowledgments` | string | Благодарности | `field_acknowledgments` |
| `funding` | string | Источники финансирования | `field_funding` |
| `date_received` | string | Дата поступления (дд.мм.гггг) | `field_date_received` |
| `accepted` | string | Дата принятия (дд.мм.гггг) | `field_accepted` |
| `published` | string | Дата публикации (дд.мм.гггг) | `field_published` |
| `page_no` | string | Номер начальной страницы | `field_page_no` |
| `page_no_to` | string | Номер конечной страницы | `field_page_no_to` |
| `body` | object | Аннотация статьи (без HTML) | `body` |
| `body.ru` | string | Аннотация на русском (plain text) | `body` (ru), HTML-теги удаляются через `strip_tags()` |
| `body.en` | string | Аннотация на английском (plain text) | `body` (en), HTML-теги удаляются через `strip_tags()` |
| `literature` | array/null | Список литературы | `field_literature` |
| `literature[].text` | string | Строка литературы (plain text) | HTML-теги удаляются, сущности декодируются |
| `short_text_pdf` | object/string | PDF краткого текста | `field_short_text_pdf` |
| `fulltext` | string | Полный текст | `field_fulltext` |
| `text_pdf` | object/string | PDF полный текст (рус) | `field_text_pdf` |
| `text_pdf_en` | object/string | PDF полный текст (англ) | `field_text_pdf_en` |
| `journal_link` | string/int | NID привязанного выпуска журнала | `field_journal_link` (Entity Reference) |
| `status` | string | Статус статьи | `field_status` |

#### Формат файловых полей

Поля `short_text_pdf`, `text_pdf`, `text_pdf_en` при наличии файла возвращают объект:

```json
{
  "fid": 1234,
  "filename": "article.pdf",
  "uri": "public://article.pdf",
  "url": "https://mmi.ddev.site/sites/default/files/article.pdf",
  "description": "",
  "display": 1
}
```

Если файл отсутствует — возвращается пустая строка `""`.

#### Ошибки

| Код | Условие | Ответ |
|-----|---------|-------|
| 400 | `nid` не является положительным числом | `{"error": "Invalid node id."}` |
| 404 | Нода не найдена | `{"error": "Node not found."}` |
| 400 | Нода не является типом `journalarticle` | `{"error": "Node is not journalarticle."}` |

---

## Внутренние функции

### `generate_abis_json_output($var)`

Кастомный delivery callback. Устанавливает заголовок `Content-Type: application/json; charset=utf-8` и выводит JSON с флагом `JSON_UNESCAPED_UNICODE` (кириллица без escape-последовательностей).

### `generate_abis_get_field_value($node, $field_name, $langcode)`

Возвращает первое скалярное значение поля ноды. Извлекает значение по приоритету ключей: `safe_value`, `value`, `target_id`, `tid`, `nid`.

### `generate_abis_get_field_values($node, $field_name, $langcode)`

Возвращает нормализованное значение поля ноды. Если поле содержит одно значение — возвращает его скалярно. Если несколько — возвращает массив. Для файловых полей возвращает объект с `fid`, `filename`, `uri`, `url`, `description`, `display`.

### `generate_abis_normalize_field_item($item)`

Нормализует один элемент поля для API-вывода. Обрабатывает типы: файлы (`fid`), Entity Reference (`target_id`), таксономия (`tid`), ноды (`nid`), текстовые поля (`safe_value`/`value` с возможным `summary`).

### `generate_abis_format_date($value)`

Преобразует дату из формата `YYYY-MM-DD HH:MM:SS` в `DD.MM.YYYY`. Возвращает пустую строку при пустом значении.

### `generate_abis_build_autor_data($node)`

Строит массив данных авторов статьи. Для каждого автора загружает пользователя (`user_load()`), извлекает ФИО на русском и английском (Entity Translation), данные организации (`node_load()` через `field_company_link`), идентификаторы (ResearcherID, SPIN, Scopus, ORCID).

### `generate_abis_get_body_data($node)`

Возвращает аннотацию статьи (`body`) на русском и английском языках. HTML-теги удаляются через `strip_tags()`.

### `generate_abis_build_taxonomy_data($node, $field_name)`

Универсальная функция для получения данных таксономий с мультиязычной поддержкой. Возвращает объект с ключами:
- `ru_page` — термины с русской версии страницы
- `en_page` — термины с английской версии страницы

Каждый термин содержит `tid`, `ru` (имя термина), `en` (перевод через i18n).

### `generate_abis_build_literature($node)`

Парсит HTML-список литературы из поля `field_literature`. Извлекает содержимое каждого `<li>`, декодирует HTML-сущности (`&ndash;` → `–`, `&rsquo;` → `'` и т.д.), удаляет HTML-теги, сохраняя LaTeX-формулы. Возвращает массив объектов `{text: "..."}` или `null`, если поле пустое.

---

## Типы материалов

### journal_number

Выпуск журнала. Содержит информацию о конкретном выпуске (том, номер, год).

**Поля:**

| Машинное имя | Описание |
|-------------|----------|
| `title_field` | Название выпуска (мультиязычное) |
| `field_year` | Год выпуска (datetime) |
| `field_volume` | Том |
| `field_number` | Номер |
| `field_part` | Часть |
| `field_journal_no_start` | Начальный номер страниц |
| `field_journal_no_end` | Конечный номер страниц |

### journalarticle

Научная статья. Содержит все метаданные статьи.

**Поля:**

| Машинное имя | Описание |
|-------------|----------|
| `title_field` | Название статьи (мультиязычное) |
| `field_language` | Язык статьи |
| `field_heading` | Рубрика (Entity Reference на таксономию `heading`) |
| `field_typersci` | Тип научной работы (Entity Reference на таксономию) |
| `field_udk` | УДК |
| `field_edn` | EDN |
| `field_doi` | DOI |
| `field_autor` | Авторы (Entity Reference на пользователя) |
| `field_key_words` | Ключевые слова (Entity Reference на таксономию `key_words`) |
| `field_acknowledgments` | Благодарности |
| `field_funding` | Финансирование |
| `field_date_received` | Дата поступления |
| `field_accepted` | Дата принятия |
| `field_published` | Дата публикации |
| `field_page_no` | Начальная страница |
| `field_page_no_to` | Конечная страница |
| `body` | Аннотация (мультиязычное, long text) |
| `field_literature` | Список литературы (HTML) |
| `field_short_text_pdf` | PDF краткого текста (file) |
| `field_fulltext` | Полный текст |
| `field_text_pdf` | PDF полный текст (рус) |
| `field_text_pdf_en` | PDF полный текст (англ) |
| `field_journal_link` | Привязка к выпуску журнала (Entity Reference на `journal_number`) |
| `field_status` | Статус статьи |

---

## Таксономия

### heading (Рубрика)

Словарь таксономии для рубрик статей. Переводы получаются через `i18n`.

### key_words (Ключевые слова)

Словарь таксономии для ключевых слов. Термины на русской и английской версиях страницы — разные сущности (разные tid). Перевод каждого термина получается через `i18n localize('en')`.

---

## Поля пользователей

Модуль использует следующие поля профиля пользователя (Entity Translation):

| Машинное имя | Описание | Языки |
|-------------|----------|-------|
| `field_lastname` | Фамилия | ru, en |
| `field_name2` | Имя | ru, en |
| `field_middle_name` | Отчество | ru, en |
| `field_company_link` | Организация (Entity Reference на ноду) | und |
| `field_researcherid` | ResearcherID | und |
| `field_spin_elibrary` | SPIN-код eLibrary | und |
| `field_scopusid` | Scopus Author ID | und |
| `field_orcid` | ORCID | und |

---

## Drupal Variables

Модуль использует следующие переменные Drupal (`variable_get`):

| Переменная | Описание |
|-----------|----------|
| `elibrary_titleid` | Идентификатор журнала в eLibrary |
| `issnPrint_citation` | Печатный ISSN журнала |
| `issnOnline_citation` | Онлайн ISSN журнала |
