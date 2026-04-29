
## Get https://mmi.ddev.site/api/generate-abis/journal-number

```json
[
    {
        "id": "4923",
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


## Get https://mmi.ddev.site/api/generate-abis/journal-data

```json
{
    "titleid": "11982",
    "issnPrint": "1816-9791",
    "issnOnline": "2541-9005"
}
```


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
				<secTitle lang="RUS"> heading.ru </secTitle>
				<secTitle lang="ENG"> heading.en </secTitle>
			</section>

            <article>
                <pages>{page_no}-{page_no_to}</pages>
                <artType>{typersci.abbreviation}</artType>
                <authors>
                   <author num="001"> 
                        <authorCodes>
                            <researcherid></researcherid>
                            <spin>8732-2104</spin>
                            <scopusid></scopusid>
                            <orcid>0000-0003-3820-9523</orcid>
                        </authorCodes>
                        <individInfo lang="RUS">
                            <surname>Григорьев</surname>
                            <initials>Алексей Александрович</initials>
                            <orgName>Саратовский национальный исследовательский государственный университет имени Н. Г. Чернышевского</orgName>
                            <address>Россия, г. Саратов, ул. Астраханская, 83</address>
                        </individInfo>
                        <individInfo lang="ENG">
                            <surname>Grigoriev</surname>
                            <initials>Alexey Alexandrovich</initials>
                            <orgName>Saratov State University</orgName>
                            <address>Astrahanskaya str., 83, Saratov, Russia</address>
                        </individInfo>
                    </author>
                </authors>

```


generate_abis_journalarticle_id_callback($nid)  при выводе autor": [
        "3477",
        "3476"
    ], это  3477 и 3476 - это uid user 

Пример получения полей пользователя user_load($autor_target_id)


результат

"autor": [
    {
        uid: uid
        num: ' id трехзначний начиная с 001, генерируется он количества пришедших авторов, каждый на 1 больше, например второй автор будет иметь 002'
        surname: field_lastname
        name: field_name2
        middle_name: field_middle_name
        initials: field_name2 + ' ' + field_middle_name
        company: 'tid таксономии'

        researcherid: field_researcherid  // текст
        spin: field_spin_elibrary // текст
        scopusid: field_scopusid // Текстовое поле
        orcid: 	field_orcid // Текстовое поле


        },
    {
        uid: uid,
        num: '002'
                surname: field_lastname
        name: field_name2
        middle_name: field_middle_name
        initials: field_name2 + ' ' + field_middle_name
        company: 'tid таксономии'

        researcherid: field_researcherid  // текст
        spin: field_spin_elibrary // текст
        scopusid: field_scopusid // Текстовое поле
        orcid: 	field_orcid // Текстовое поле
        },
    ...
]


        orgName: 
        address: 



нужно исправить generate_abis_journalarticle_id_callback($nid)

"autor": [{

    следующее

        surname: {
            ru: 'фамилия на русском',
            en: 'фамилия на английском',
        }

        name: {
            ru: 'имя на русском',
            en: 'имя на английском',
        }
        middle_name: {
            ru: 'отчество на русском',
            en: 'отчество на английском',
        }

        initials: {
            ru: 'имя отчество на русском',
            en: 'имя отчество на английском',
        }

}




потом надо добавить
```php
function generate_abis_journal_number_callback() {
  // Отключаем кэш страницы drupal, чтобы ответ всегда был актуальным
  drupal_page_is_cacheable(FALSE);
  drupal_add_http_header('Content-Type', 'application/json');  // эту строку!!!
```