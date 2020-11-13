# GET
existing article, no translations
`GET /jsonapi/node/article/{id}`

existing article, default translation
`GET /jsonapi/node/article/{id}`

existing article, non-default translation
Using canonical URL
```
GET /jsonapi/node/article/{id}
Accept-Language: fr

// If it exists
Content-Language: fr
Content-Location: /jsonapi/node/article/{id}?tbd=fr

// If it doesn't exist
Content-Language: en-US
```

Using translation-specific URL
```
GET /jsonapi/node/article/{id}?tbd=fr

// If it exists
Content-Language: fr

// If it doesn't exist
404 Not found
```

# POST
new article in default language
`POST /jsonapi/node/article`

new article in non-default language
```
POST /jsonapi/node/article
Content-Language: fr

// plach notes that the language is a field value

// Alternative
POST /jsonapi/node/article
{type: "node--article", attributes: {langcode: "fr"}}

// If the above is permitted... the following must return an error
POST /jsonapi/node/article
Content-Language: en-US
{type: "node--article", attributes: {langcode: "fr"}}

// gabe wonders if it should be permitted to provide a langcode attribute without a content-language header

// The case where there is no langcode and no content-language header should use the default behavior from /admin/config/regional/content-language
POST /jsonapi/node/article
{type: "node--article"}
```
```
POST /jsonapi/node/article
{type: "node--article"}

204 Created
Location: /jsonapi/node/article/{id}
```

new translation of existing article
```
POST /jsonapi/node/article/{id}
Content-Language: fr

// plach notes that he fears that using the URL above will cause a conflict in the future if JSON:API supports other kinds of entity variants
// gabe thinks it would be okay to use custom headers in the case above

204 Created
Location: /jsonapi/node/article/{id}?tbd=fr
```

new translation of existing article with non-default source
```
POST /jsonapi/node/article/{id}
Content-Language: nl
{type: "node--article": id: "id", attributes: {content_translation_source: "fr"}}

// plach notes that he fears that using the URL above will cause a conflict in the future if JSON:API supports other kinds of entity variants
// gabe thinks it would be okay to use custom headers in the case above

204 Created
Location: /jsonapi/node/article/{id}?tbd=nl
```

# PATCH
change article, no translations
change article, w/ translations, default translation
change article, w/ translations, non-default translation

# DELETE
remove article, no translations
remove article, all translations
remove default translation of existing article
remove non-default translation of existing article
