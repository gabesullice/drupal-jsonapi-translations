# GET

* Existing article, no translations:
```
GET /jsonapi/node/article/{id}

200 OK
```

* Existing article, default translation:
```
GET /jsonapi/node/article/{id}

200 OK
```

* Existing article, non-default translation, using canonical URL:
```
GET /jsonapi/node/article/{id}
Accept-Language: fr

// If translation exists:
200 OK
Content-Language: fr
Content-Location: /jsonapi/node/article/{id}?lang=fr

// If translation does not exist:
// option a)
// provide a default without redirecting
200 OK
Content-Language: en

// option b)
// do not provide a default or redirect, simply provide alternatives
406 Not acceptable
Link: </jsonapi/node/article/{id}?lang=en>; rel="alternate" hreflang="en"
Link: </jsonapi/node/article/{id}?lang=nl>; rel="alternate" hreflang="nl"
{links: {// same links, in json}}

// option c)
// redirect to a default and also provide alternatives
300 Multiple choices
Location: /jsonapi/node/article/{id}?lang=en
Link: </jsonapi/node/article/{id}?lang=en>; hreflang="en"
Link: </jsonapi/node/article/{id}?lang=nl>; hreflang="nl"
{links: {// same links, in json}}
```

* Existing article, non-default translation, using translation-specific URL:
```
GET /jsonapi/node/article/{id}?lang=fr

// If translation exists:
200 OK
Content-Language: fr

// If translation does not exist:
404 Not found
```

# POST

* New article in default language:
```
POST /jsonapi/node/article
{type: "node--article"}
```

* New article in non-default language:
```
POST /jsonapi/node/article
{type: "node--article", attributes: {langcode: "fr", ...}}

201 Created
Location: /jsonapi/node/article/{id}
```
```
// The "Content-Language" header is just an alternative way to specify the

// entity "langcode" field.
POST /jsonapi/node/article
Content-Language: fr
{type: "node--article", attributes: {...}}

201 Created
Location: /jsonapi/node/article/{id}
```
```
// A mismatch between those two MUST return an error:
POST /jsonapi/node/article
Content-Language: en
{type: "node--article", attributes: {langcode: "fr"}}

400 Bad Request
```
```
// The case where there is no "langcode" and no "content-language" header should
// use the default behavior configured at "/admin/config/regional/content-language".
POST /jsonapi/node/article
{type: "node--article"}

201 Created
Location: /jsonapi/node/article/{id}
```

* New translation of existing article, only translatable field values can be
posted in this case:
```

POST /jsonapi/node/article/{id}
Content-Language: fr
{attributes: {...}}

201 Created
Location: /jsonapi/node/article/{id}?lang=fr
```
```
// Alternatively:
POST /jsonapi/node/article/{id}
{attributes: {langcode: "fr", ...}}

201 Created
Location: /jsonapi/node/article/{id}?lang=fr
```
```
// Either "article" or "field_event_date" are not translatable:
POST /jsonapi/node/article/{id}
Content-Language: fr
{attributes: {field_event_date: "2020-11-16", ...}}

422 Unprocessable entity
```
```
// A French translation already exsists:
POST /jsonapi/node/article/{id}
Content-Language: fr
{attributes: {...}}

409 Conflict
```
```
// Plach notes that he fears that using the URL above will cause a conflict in
// the future if JSON:API supports other kinds of entity variants.
// Gabe thinks it would be okay to use custom headers in the case above, for
// instance:
POST /jsonapi/node/article/{id}
Content-Language: fr
X-Drupal-Entity-Variant-User: 2

201 Created
Location: /jsonapi/node/article/{id}?lang=fr&user=2
```

* New translation of existing article with non-default source, only translatable
field values can be posted in this case:
```
POST /jsonapi/node/article/{id}
Content-Language: nl
{attributes: {content_translation_source: "fr", ...}}

201 Created
Location: /jsonapi/node/article/{id}?lang=nl
```

# PATCH

* Change article, no translations:
```

PATCH /jsonapi/node/article/{id}
{attributes: {...}}

200 OK
```

* Change article, default translation:
```

PATCH /jsonapi/node/article/{id}
{attributes: {...}}

200 OK
```

* Change article, non-default translation, only translatable field values can be
posted in this case:
```

PATCH /jsonapi/node/article/{id}?lang=fr
{attributes: {...}}

// If French translation exists:
200 OK

// If French translation does not exist:
404 Not Found

// If an unstranslatable field value is specified:
422 Unprocessable entity
```
```
// A mismatch between "langcode" and/or the Content-Language header and the
// translation language MUST result in an error. Translation language change is
// not supported at the moment:
PATCH /jsonapi/node/article/{id}?lang=fr
Content-Language: en
{attributes: {...}}

400 Bad Request
```
```
PATCH /jsonapi/node/article/{id}?lang=fr
{attributes: {langcode: "en", ...}}

400 Bad Request
```
```
PATCH /jsonapi/node/article/{id}?lang=fr
Content-Language: fr
{attributes: {langcode: "en", ...}}

400 Bad Request
```
```
PATCH /jsonapi/node/article/{id}?lang=fr
Content-Language: fr
{attributes: {langcode: "fr", ...}}

200 OK
// or
204 No content
// see https://jsonapi.org/format/#crud-updating-responses-204
```

# DELETE

* Remove article, no translations:
```

DELETE /jsonapi/node/article/{id}

204 No content
```

* Remove article, all translations:
```

DELETE /jsonapi/node/article/{id}

204 No content
```
```
DELETE /jsonapi/node/article/{id}
Content-Language: en

// If the default translation language is English:
204 No content

// If the default translation language is not English:
400 Bad Request
```

* Removing the default translation of an existing article is not supported:
```

DELETE /jsonapi/node/article/{id}?lang=en

400 Bad Request
```

* Remove non-default translation of existing article:

```

DELETE /jsonapi/node/article/{id}?lang=fr

// If French translation exists:
204 No content

// If French translation does not exist:
404 Not Found
```
```
// Specifying a Content-Language header does not make sense in this case, but if
// it is provided it MUST match the translation language:
DELETE /jsonapi/node/article/{id}?lang=fr
Content-Language: en

400 Bad Request
```
```
DELETE /jsonapi/node/article/{id}?lang=fr
Content-Language: fr

204 No content
```
