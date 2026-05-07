
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
				<secTitle lang="RUS"> {heading.ru} </secTitle>
				<secTitle lang="ENG"> {heading.en} </secTitle>
			</section>

            <article>
                <pages>{page_no}-{page_no_to}</pages>
                <artType>{typersci.abbreviation}</artType>
                <authors>
                   <author num={num}> 
                        <authorCodes>
                            <researcherid>{researcherid}</researcherid>
                            <spin>{spin}</spin>
                            <scopusid>{scopusid}</scopusid>
                            <orcid>{orcid}</orcid>
                        </authorCodes>
                        <individInfo lang="RUS">
                            <surname>{autor.surname.ru}</surname>
                            <initials>{autor.initials.ru}</initials>
                            <orgName>{company.data.orgName.ru}</orgName>
                            <address>{company.data.address.ru}</address>
                        </individInfo>
                        <individInfo lang="ENG">
                            <surname>{autor.surname.en}</surname>
                            <initials>{autor.initials.en}</initials>
                            <orgName>{company.data.orgName.en}</orgName>
                            <address>{company.data.address.en}</address>
                        </individInfo>
                    </author>
                </authors>
                <artTitles>
                    <artTitle lang="RUS">{ title.ru }</artTitle>
                    <artTitle lang="ENG">{ title.en}</artTitle>
                </artTitles>

                <abstracts>
                    <abstract lang="RUS">{ body.ru }</abstract>
                    <abstract lang="ENG"> { body.en }</abstracts>
                    <text lang="ANY"> { fulltext }</text>
                    <codes>
                        <udk> {udk}</udk>
                        <doi>{doi}</doi>
                        <edn> {edn}</edn>
                    </codes>

                    <keywords>
                        <kwdGroup lang="RUS">
                            for key_word in key_words.ru_page
                            <keyword>{key_word.ru}</keyword>
                        </kwdGroup>
                        <kwdGroup lang="ENG">
                            for key_word in key_words.en_page
                            <keyword>{key_word.en}</keyword>
                        </kwdGroup>
                    </keywords>

                    <dates>
                        <dateReceived>{date_received}</dateReceived>
                        <dateAccepted>{accepted}</dateAccepted>
                        <datePublication>{published}</datePublication>
                    </dates>


                    <references>
                        <reference>
                            for lit in literature
                            <refInfo lang="ANY">
                                <text>{lit.text}</text>
                            </refInfo>
                    </reference>

                    <files>
                        <file desc="fullText">{text_pdf.filename}</file>
                    </files>


```

generate_abis_journalarticle_id_callback($nid)

    "body": {
        "value": "<div class=\"tex2jax\"><p class=\"rtejustify\">Одно из свойств периодической функции на вещественной оси состоит в том, что она полностью определяется своими значениями на периоде. Этот факт допускает следующее нетривиальное обобщение на многомерный случай: если функция $f\\in C^\\infty (\\mathbb R^n)$ $(n\\ge 2)$ с нулевыми интегралами по всем сферам (или шарам) фиксированного радиуса $r$ равна нулю в некотором шаре радиуса $r$, то $f$ является нулевой на $\\mathbb R^n$. Условие бесконечной гладкости функции $f$ в этом утверждении ослабить нельзя. В данной работе изучается подобное явление для решений уравнений свертки, связанных с оператором обобщенного сдвига Бесселя. Сначала рассматривается случай, когда свертывателем уравнения является индикатор отрезка, симметричного относительно нуля. Показано, что решения такого уравнения определяется своими значениями на указанном отрезке. Далее приводится обобщение этого свойства для общего уравнения свертки Бесселя. Полученные результаты являются аналогами известных теорем единственности для периодических в среднем функций, принадлежащих Ф. Йону, Ю. И. Любичу и А. Ф. Леонтьеву.</p>\n</div>",
        "summary": "<div class=\"tex2jax\"></div>"
    },

    нужно переделать
    "body": {
        ru: данные не должны содержать теги html,
        en: данные не должны содержать теги html,
    }

    body.ru = должно происходить форматирование html тегов, они должны удаляться



Пример получения полей пользователя user_load($autor_target_id)





----


Добавь новый items $items['api/generate-abis/journal-list/%']

который будет по полученому значению id node просматривать все ноды типа материала journalarticle и сравнивать содержит ли поле 
field_journal_link Entity Reference такой же id как мы указали.

в результате должен получиться вывод статей относящихся к журналу

[
    {
        "id": 7253,
        "title": {
            "ru": "Теорема единственности для периодических в среднем функций на гипергруппе Бесселя – Кингмана",
            "en": "A uniqueness theorem for mean periodic functions on the Bessel – Kingmann hypergroup"
        },
    },
    {
        "id": 7254,
        "title": {
            "ru": "Теорема единственности для периодических в среднем функций на гипергруппе Бесселя – Кингмана2",
            "en": "A uniqueness theorem for mean periodic functions on the Bessel – Kingmann hypergroup2"
        },
    },
]




---



потом надо добавить
```php
function generate_abis_journal_number_callback() {
  // Отключаем кэш страницы drupal, чтобы ответ всегда был актуальным
  drupal_page_is_cacheable(FALSE);
  drupal_add_http_header('Content-Type', 'application/json');  // эту строку!!!
```




