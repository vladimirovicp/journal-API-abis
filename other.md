
## Get https://mmi.ddev.site/api/generate-abis/journal-number

```
[
    {
        "id": "4925",
        "title_field": "Известия Саратовского университета. Новая серия. Серия «Математика. Механика. Информатика» 2007, Т. 7, вып. 1",
        "title_field_en": "Izvestiya of Saratov University. New Series. Series: Mathematics. Mechanics. Informatics 2007, vol. 7, iss. 1",
        "field_year": "2007",
        "field_volume": "7",
        "field_number": "1",
        "field_part": "",
        "field_journal_no_start": "1",
        "field_journal_no_end": "94"
    },

    ...
]
```

Надо будет писать проверку на существование данных.

if title_field  пустое, то выводится сообщение: "Выбранный журнал не содержит заголовка на русском языке"
if title_field_en пустое, то выводится сообщение: "Выбранный журнал не содержит заголовка на английском языке"
if field_year пустое, то выводится сообщение: "Выбранный журнал не содержит данных о годе"
if field_volume пустое, то выводится сообщение: "Выбранный журнал не содержит данных о томе"
if field_number пустое, то выводится сообщение: "Выбранный журнал не содержит данных о выпуске"

if field_journal_no_start пустое, то выводится сообщение: "Выбранный журнал не содержит данных о первой странице"
if field_journal_no_end пустое, то выводится сообщение: "Выбранный журнал не содержит данных о последней странице"



Формат XML на выходе.

Нужно будет передать 

    $titleid = variable_get("elibrary_titleid", "");
  	$issnPrint = variable_get("issnPrint_citation", "");
  	$issnOnline = variable_get("issnOnline_citation", "");



Переменные journal-number

```
jn_title = title_field
jn_title_en = title_field_en
jn_volume = field_volume
jn_number = field_number
jn_year = field_year

jn_journal_no_start = field_journal_no_start
jn_journal_no_end = field_journal_no_end

```

```xml
<?xml version="1.0" encoding="utf-16" standalone="no"?>
<journal>
    <titleid>  $titleid </titleid>
    <issn> $issnPrint </issn>
    <eissn> $issnOnline </eissn>
    <journalInfo lang="RUS">
        <title> {jn_title} </title>
    </journalInfo>
    <journalInfo lang="ENG">
        <title> jn_title_en </title>
    </journalInfo>
    <issue>
        <volume> jn_volume </volume>
        <number> jn_number </number>
        <dateUni> jn_year </dateUni>
        <pages> jn_journal_no_start - jn_journal_no_end </pages>

```

далее берутся данные из статьи

```xml
        <articles>
            <section>
				<secTitle lang="RUS"> $secTitleRu </secTitle>
				<secTitle lang="ENG"> $secTitleEn </secTitle>
			</section>
```




потом надо добавить
```php
function generate_abis_journal_number_callback() {
  // Отключаем кэш страницы drupal, чтобы ответ всегда был актуальным
  drupal_page_is_cacheable(FALSE);
  drupal_add_http_header('Content-Type', 'application/json');  // эту строку!!!
```