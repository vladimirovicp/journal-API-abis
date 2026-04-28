
## Get https://mmi.ddev.site/api/generate-abis/journal-number

```
[
    {
        "id": "4923",
        "title_field": "Известия Саратовского университета. Новая серия. Серия «Математика. Механика. Информатика» 2006, Т. 6, вып. 1",
        "title_field_en": "Izvestiya of Saratov University. New Series. Series: Mathematics. Mechanics. Informatics 2006, vol. 6, iss. 1",
        "field_year": "2006-01-01 00:00:00",
        "field_volume": "6",
        "field_number": "1",
        "field_part": ""
    },

    ...
]
```


потом надо добавить
```php
function generate_abis_journal_number_callback() {
  // Отключаем кэш страницы drupal, чтобы ответ всегда был актуальным
  drupal_page_is_cacheable(FALSE);
  drupal_add_http_header('Content-Type', 'application/json');  // эту строку!!!
```