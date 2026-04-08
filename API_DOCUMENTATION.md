FORMAT: 1A
HOST: https://api.aihelps.com/v1/

# AI Helps

This is official API for integration external applications with AI Helps products Beauty Pro and Fitness Pro.

API tends to follow REST principles.

API root URL for HTTP requests is: https://api.aihelps.com/v1/

API status page: https://api.aihelps.com/status

Questions about API: https://t.me/joinchat/J1-qj0gTp_i6EmE2rtvGvw (Telegram channel)

AI Helps API consultant: https://chatgpt.com/g/g-69a06e7b02908191818b18d2b77f1f40-ai-helps-api-consultant

## About API

### API hosts

Default API root URL for HTTP requests is: https://api.aihelps.com/v1/.
There are several (at current moment - 3) data servers worldwide, each client database located on the server, closiest to client.
When receiving access token you will also receive index of server (number greater or equal 1). All api methods are accessible via any server,
but the quickest way is to use server, where database is located. Host names are:
- `api.aihelps.com` for server 1 and 5 (Frankfurt, Germany)
- `api4.aihelps.com` for server 4 (Mumbai, India)
and so on.

Authorization requests better to call on server 1.

### API versioning

Current API version is "Version 1". Version information is mandatory in API URL and described as `v{number}`, where `{number}` is integer, starting from `1` (current version is `1`).

Versioning rules are following:
1) New API methods can be added without incrementing version number.
2) New parameters can be added to existing API methods if they don"t change API behavior with these new parameters missing.
3) Changing API URLs, parameters names or their behaviour will increment version number.
4) Removing some API URLs or parameters names will increment version number.

These rules guarantee that code, working with specific API version won"t broke on minor changes made. All changes that potentially can break code will be presented in new version.

Last API version is considered "public". All developers will be notified about API version incremented, if they provided `contact_email` (see Application account section).
Old API version will be available at least 2 months (it can happens that several old versions will be available) and this period can be extended by request.

All ideas, errors and propositions can be sent to Telegram channel https://t.me/joinchat/J1-qj0gTp_i6EmE2rtvGvw.

### HTTPS protocol

All API available only through HTTPS (HTTP not available for both public/private API).

Current server supports TLS 1.0, 1.1 and 1.2 (SSL 1-3 commonly recognized vulnerable)

## REST API with extensions

Current API implementation uses REST API guidelines. According to idea, that in most cases API is used for both get/update data, in many cases with local store, resources representation seems to be better choice than GraphQL "mostly read with complex views" idea and gRPC "all is call" idea (more useful in some-transaction-like APIs). According to that, REST API seems to be the most natural way for our API, taking into account our specifics, GraphQL has some promising features can be useful and gRPC usage is far from our case, so we didn't took it into account.

So, AI Helps API fulfills REST principles: client–server, stateless, cacheable (not important in our case), uniform interface and layered system.
Resources are identified in `/resources/{id}` way (subresources `/resources/{id}/subresources/{id2}` and so on), methods to work with resources are GET, POST, PUT and DELETE.

API URI usually represents either list of resources (e.g. `\clients`) or specific resource from list, given by id (e.g. `\clients\{id}`).
Next table describes what methods can be used to operate with data:

API url|GET|POST|PUT|DELETE
:------|:-:|:--:|:-:|:----:
List of resources (e.g. `\clients`)|get all resources|create new resource|nothing|nothing
Specific resource (e.g. `\clients\{id}`)|get resource by id|nothing|update resource|delete resource

Not all methods will be available for all lists or resources, see detailed documentation below.
If method not available for given API URL, response with HTTP status code 404 (Not Found) will be returned.

We also took best ideas from GraphQL and implemented in our API:
1) While retrieving data you forced to pass fields you want to get. This idea gives more efficient data loading and easier API usage - even if we add extra fields, you will not receive them until explicitly requested.
2) To add or update resource you can pass only part of fields - you do not need to pass all fields. While update, non-passed fields will leave unchanged (so, no more need for PATCH method). This greatly simplifies API usage and decrease API complexity. Along with previous idea this makes API much more flexible and easy to use (non-fixed data structure).
3) In some situations API can be used mostly to read complex data without close to zero changes. So we added complex read-only or read-write fields to our resources. Such "fields" can be retrieved and updated in same request with main resource, greatly decreasing complexity of client code, number of requests and speed. These "fields" are specific for each resource and fine-tuned for speed (contrary to GraphQL where complex structure is hard to tune), still reach enough to satisfy client needs.
4) We also implemented subscription for changes for all resources in API with WebHooks.

May be in future we also will add GraphQL wrapper around our REST API, so you will be able to use GraphQL client libraries.

### Data format

All API methods use JSON for both input and output. For some requests (all GET requests) body not needed, for DELETE server returns nothing.

For retrieving images, where response will be image itself, for updating image request body is image itself.

ContentType (`application/json`) is not mandatory for requests and can be skipped, but server always return content type
(`application/json` for 200/201 responces (do not return content type for 204 (No content)) and `image/jpeg` or `image/png` for images).

XML not supported now and no plans to support it in future.

All API requests should use UTF-8 encoding, all responses also in UTF-8.

We recommend you use Postman to test our API.
//https://www.getpostman.com/

### Data types

All items in response/request are JSON objects, each field can be either simple type shown below or array/set of subitems.

In requests and responses following simple data types can be used:
+ string - formatted as simple JSON string (`"abc"`). Empty string can be represented either as null or empty string. API always returns empty string as null. String parameters can be marked as `required`, that means null or empty string are not allowed for such strings and all required fields should be provided on creating new item.
+ number - formatted as simple JSON number (`12.3`)
+ boolean - formatted as simple JSON true or false (`true`)
+ identifier - all identifiers are represented as GUIDs. Should be formatted as string
with 36 characters: `"XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"`. Empty identifier can be
represented either as null, empty string or `"00000000-0000-0000-0000-000000000000"`
API always returns empty identifier as null.
+ datetime - is represented in ISO 8601 Extended format `"YYYY-MM-DDTHH:mm:ss.sssZ"`.
T - delimiter char; milliseconds shown, but in this API wlll always be zero; timezone is always UTC ("Z" char). E.g. `"2017-01-01T13:51:55.000Z"`.
This format is native for Javascript and used in Date.toJSON(), Date.parse(). API also can receive dates in several other formats, full list of available incoming formats (but output is still always the same):
    + `"YYYY-MM-DD"`
    + `"YYYY-MM-DDTHH:mm"`
    + `"YYYY-MM-DDTHH:mm:ss"`
    + `"YYYY-MM-DDTHH:mm:ss.sss"`
    + `"YYYY-MM-DDTHH:mm:ss.sssZ"`
+ image - in JSON formatted as base64, usually can be retrieved by direct link
+ html_text - text with additional html tags, used to format that text. There are some differences with a valid html document
    + There is no `<!DOCTYPE html>` comment
    + Content should not be wrapped with `<html>` or `<head>` or `<body>` tags
    + Content should contains only one of these kind of nodes:
        + `#text` - contains plain text
        + `<span>` - can contain `style` attribute, which can contain these CSS declarations:
            + `font-weight: bold` - for bold text
            + `font-style: italic` - for italic text
            + `text-decoration: underline` - for underlined text
            + `color: #FF0000` - text color. There can be any HEX-value color
        + `<h1>`, `<h2>`, `<h3>` - describes font size, can contains the same CSS declarations, as `<span>` node. Note, that headlines will be considered as NOT bold
        + `<div>` - describes paragraph with alignment. `<div>` should not contain another `<div>` inside it. Left-aligned text should NOT be wrapped with `<div>` tags. Alignment is set using `style` attribute, which contains one of two CSS declarations:
            + `text-align: center` - centered text
            + `text-align: right` - right-aligned text
        + `<br/>` - line break

### Get list of items format

In current and next sections `someitems` in URL will describe clients, products, sales or any other item type.

Request: GET `/someitems?fields=field_list[&filter1=value1&filter2=value2&..]`

Get `someitems` with fields specified in `fields` parameter (required parameter).

Example of `fields` parameter:
`fields=field1,field2,field_set3(subfield1,subfield2,..),..`
If some field is array/set, use parentheses to describe subfields. If some subfield is also array/set, use parentheses again: e.g.
`fields=id,name,history(date,name),relatives(name,history(date,name))`

Each item has unique `id` (identifier). `id` can be used in API URL for GET/PUT/DELETE operations. `id` field is always returned,
no need to specify it in `fields` parameter.
If you specify `fields` parameter empty, only ids will be returned.

If filter values are set, filters applied. More details about filters will be provided in specific filter description.

For all items list there is one universal filter: `ids` - provide list of ids, separated by comma, only items with provided ids will be returned.

Response example:

```
[
        {
            "id": "246db366-061f-45ac-984d-de8050c045a1",
            "name": "Item 1",
            "type": "Simple",
            "color": "Red"
        },
        {
            "id": "905e5ff6-9d41-465b-aced-7ea81a7e2ee3",
            "name": "Item 2",
            "type": "Complex",
            "color": "Green"
        },
        {
            "id": "cfbb963e-3ceb-4764-b3f4-805f3a3e056b",
            "name": "Item 3",
            "type": "Simple",
            "color": "Yellow"
        }
]
```

Items in response are sorted either by field "name" or field "date" (if "name" not exists, any item has either "name" or "date" fields).

### Get item by id format

Request: GET `/someitems/{id}?fields=field_list`

Get `item` with given `id` and fields specified in `fields` parameter (required parameter).

Example of `fields` parameter:
`fields=field1,field2,field_set3(subfield1,subfield2,..),..`
If some field is array/set, use parentheses to describe subfields. If some subfield is also array/set, use parentheses again: e.g.
`fields=id,name,history(date,name),relatives(name,history(date,name))`

`id` field is always returned, no need to specify it in `fields` parameter.

Response example:

```
{
    "id": "ecb3521a-3b50-4281-b947-34e7ca526b28",
    "name": "Item 1",
    "type": "Simple",
    "color": "Red"
}
```

### Create new item format

Request: POST `/someitems`

```
{
    "name": "Item 4",
    "type": "Simple",
    "color": "Blue",
    "category": "8c98a00b-1526-4dc0-9968-008439a78019"
}
```

Not all fields should be specified - missing fields will be initialized with default values.

Response will return status 201 (not 200) and id of newly created item:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137"
}
```

In some cases after item was created, some items fields except `id` needed. Making another GET request
(GET /someitems or GET /someitems/{id}) seems redundant. While making original request you can just describe
fields you want to receive in `fields` parameter (optional):

Request: POST `/someitems?fields=name,type,color,category`

```
{
    "name": "Item 4"
}
```

Response will look like:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137",
    "name": "Item 4",
    "type": "Simple",
    "color": "",
    "category": null
}
```

### Update item format

Request: PUT `/someitems/{id}`

```
{
    "type": "Complex"
}
```

Not all fields should be specified - missing fields will left field values unchanged.

In some cases after item was updated, some fields are needed. Making another GET request
(GET /someitems or GET /someitems/{id}) seems redundant. While making original request you can just describe
fields you want to receive in `fields` parameter (optional). Server always returns id of updated item 
and fields of `fields` parameter if specified:

Request: PUT `/someitems/{id}?fields=name,type,color,category`

```
{
    "type": "Complex"
}
```

Response will look like:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137",
    "name": "Item 1",
    "type": "Complex",
    "color": "Red",
    "category": null
}
```

### Delete item format

Request: DELETE `/someitems/{0}`

```
(no body needed)
```
For DELETE requests JSON body is not needed and no body can be sent at all.

Response: status 204 (No content)

### Operations with categories

Categories are present for several items types (products, services, groups, etc.), they are special
object types thus can be edited using special methods.

Each category 2 fields (except `id`):
+ name - category name,
+ parent - id of parent category.

Categories helps to make multi level hierarchies, like:

```
+ Top category (virtual)
    + Item 1
    + Category 1
        + Item 2
        + Subcategory 1
            + Item 3
```

#### Get list of categories format

Request: GET `/someitems/categories?fields=field_list[&filter1=value1&filter2=value2&..]`

Get categories with fields specified in `fields` parameter (required parameter). If filter values are set, filters applied.

Response example:

```
[
        {
            "id": "246db366-061f-45ac-984d-de8050c045a1",
            "name": "Subcategory 1",
            "parent": "8c98a00b-1526-4dc0-9968-008439a78019"
        },
        {
            "id": "5c8316ef-68fb-47d4-b99a-ef715a60bd16",
            "name": "Subcategory 2",
            "parent": "8c98a00b-1526-4dc0-9968-008439a78019"
        }
]
```

All categories in response are sorted by field "name".

#### Get category by id format

Request: GET `/someitems/categories/{id}?fields=field_list`

Get category with given `id` with fields specified in `fields` parameter (required parameter).

Response example:

```
{
    "id": "246db366-061f-45ac-984d-de8050c045a1",
    "name": "Subcategory 1",
    "parent": "8c98a00b-1526-4dc0-9968-008439a78019"
}
```

#### Create new category format

Same to creating new items:

Request: POST `/someitems/categories`

```
{
    "name": "Category 2",
    "parent": "8c98a00b-1526-4dc0-9968-008439a78019"
}
```

Not all fields should be specified - missing fields will be initialized with default values.

Response will return status 201 (not 200) and id of newly created category:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137"
}
```

If some category fields except `id` is needed, you can just describe fields you want to receive in `fields` parameter (optional):

Request: POST `/someitems/categories?fields=name,parent`

```
{
    "name": "Category 2"
}
```

Response will look like:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137",
    "name": "Category 2",
    "parent": null
}
```

#### Update category format

Request: PUT `/someitems/categories/{id}`

```
{
    "parent": null
}
```

Not all fields should be specified - missing fields will left field values unchanged.

Response: status 204 (No content)

If some category fields except `id` is needed, you can just describe fields you want to receive in `fields` parameter (optional):

Request: PUT `/someitems/categories/{id}?fields=name,parent`

```
{
    "parent": null
}
```

Response will look like:

```
{
    "id": "1bfcf663-3f5d-4a86-8e0c-12c5ea34c137",
    "name": "Category 2",
    "parent": null
}
```

#### Delete category format

Request: DELETE `/someitems/categories/{0}`

```
(no body needed)
```
For DELETE requests JSON body is not needed and no body can be sent at all.

Response: status 204 (No content)

Please mention, that removing category will remove all subcategories and all items in category and
all subcategories.


#### Get items with categories

Request: GET `/someitems/tree[?fields=fields_list,categories_fields=categories_fields_list,empty_categories&filter1=value1&filter2=value2&..]`

If some items are grouped in categories, in most cases most convenient will be to receive such items from API in
some sort of tree view. This request combine both items and categories and represent them in tree view:

1) Result root is "virtual" top category with empty id, name and parent. You can't update or delete this category.
2) Each category, including root, has two arrays: `categories` and `items`.
3) Items can be filtered using filters (same as for get items).
4) By default, empty categories (that has no subcategories or items) are not returned. You can force to return them,
setting `empty_categories` in parameters.

Response will look like:

```
{
    "id": null,
    "name": "",
    "parent": null,
    "categories": [
        {
            "id": "8c98a00b-1526-4dc0-9968-008439a78019",
            "name": "Category 1",
            "parent": null,
            "categories": [
                {
                    "id": "5c8316ef-68fb-47d4-b99a-ef715a60bd16",
                    "name": "Subcategory 1",
                    "parent": "8c98a00b-1526-4dc0-9968-008439a78019",
                    "categories": [
                    ],
                    "items": [
                        {
                            "id": "cfbb963e-3ceb-4764-b3f4-805f3a3e056b",
                            "name": "Item 3",
                            "type": "Simple",
                            "color": "Yellow",
                            "category": "5c8316ef-68fb-47d4-b99a-ef715a60bd16"
                        }
                    ]
                }
            ],
            "items": [
                {
                    "id": "905e5ff6-9d41-465b-aced-7ea81a7e2ee3",
                    "name": "Item 2",
                    "type": "Complex",
                    "color": "Green",
                    "category": "8c98a00b-1526-4dc0-9968-008439a78019"
                ]
            }
        }
    ]
    "items": [
        {
            "id": "246db366-061f-45ac-984d-de8050c045a1",
            "name": "Item 1",
            "type": "Simple",
            "color": "Red",
            "category": null
        }
    ]
}
```

Categories in "categories" arrays are sorted by field "name", items in "items" array in response are sorted either by field "name" or "date" (if "name" not exists).

One moment to keep in mind with empty categories. Assume we have next structure:

```
+ Top category (virtual)
    + Item 1
    + Category 1
        + Item 2
        + Subcategory 1
            + Item 3
        + Empty subcategory 1
    + Category 2
        + Empty subcategory 2
```

Empty categories, as in example above can happen in two cases:
1) Someone add category in program and forgot to add items inside (or adding items in progress).
2) You use filters to filter some items and originally non-empty categories become empty with your filters.
In most cases you don't need to receive empty categories, so API automatically filter empty categories and don't return them.
So, tree structure above will look like:

```
+ Top category (virtual)
    + Item 1
    + Category 1
        + Item 2
        + Subcategory 1
            + Item 3
```

Empty subcategories 1 and 2 was not shown, Category 2 become empty and was not shown also.

If you need to receive empty categories, set parameter `empty_categories` and all categories will be preserved.

### Getting/updating images 

Difference between images and other data types are that images are binary data type. Thus it is non-optimal to
get/update them via JSON (using base64 for example). Also, receiving images inside JSON makes not so comfortable
to set IMG/PICTURE tags.

That's why getting/receiving images (client photos, employee photos, products pictures, etc.) are always separate methods.
But they all have same format.

#### Get image

To get image, use GET request one level deeper than object which image you want to get: for example, `/items/{id}/photo`.
Result will always be of content mime type image/jpeg, image/png or image/gif (if no image found, 404 error will be returned).

The most common problem with images is resizing them to specific size. Usually browser is capable of doing this,
but it is better to do that on server to decrease size of data transferred and increase speed. So, on getting images,
these 3 optional parameters can be provided (please note both `width` and `height` should be provided for resizing):

Parameter|Possible values|Description
:--------|:--------------|:----------
width|integer|Width of resized image in pixels
height|integer|Height of resized image in pixels
resize|fit (default)/fit_center_transparent/stretch/crop|Type of operation in case proportions of original image will not match proportins of needed image.

List of possible `resize` values:

Value|Description
:----|:----------
fit (default)|Image will be resized constraining proportions to fit into provided width & height. One of dimensions can be less than needed.
fit_center_transparent|Same as previous but if one of dimensions is less than needed, additional transparent rows/colums added to image to make width and height equal to needed
stretch|Image will be resized to needed width & height without constraining proportions (width and height will be exact as needed but image will be distorted)
crop|Image will be resized constraining proportions to such scale that one of dimensions is equal to needed while another is equal or bigger than needed. If second dimension is bigger, image will be cropped to match needed width and height. In this case proportions are constrained, image size equals to needed but part of image can be cropped (for example, original size 2000x1000, needed 100x100: first scale to 200x100 than crop left 50 and right 50 pixels).

In all cases scaling is done only for bigger images. Scaling is skipped if needed size is bigger than original (otherwise artefacts can be seen on image).

In some cases it is preferable to use API URL inside IMG/PICTURE tag, without using JavaScript to fetch image. But all image requests need access token in
`Authorization` header. So for getting image requests (and only for them) access token can be specified as additional parameter `access_token` in URL, thus
`Authorization` header not needed for such requests. Resulting URL can be easily inserted as `src` parameter of `IMG` tag.

A little about caching images: all images should be cached for 10 minutes, after that ETag can be used to check if image was changed or not.
If you use images inside IMG/PICTURE tags, browser do all this magic for you, but in case you get images from JavaScript or some other language,
please use Cache-Control and ETag headers to decrease traffic and increase speed of your application.

<a name="updating-image"></a>
#### Update image

Updating image is very straightforward process.
Use same URL as for getting image but with PUT method.

Content should be image itself (mime types image/jpeg, image/png or image/gif are supported).
Mime type of an image will be defined by it's content, so `Content-Type` header will be ignored.

No parameters supported:
`width`, `height` and `resize` has no meaning: for each image type (client photo, product picture) there are custom max allowed size. If image is bigger,
it will be fitted in max allowed size.
`access_token` not allowed: access token should be passed as `Authorization` header.

There are three request types supported:

##### 1) Sending binary data (recommended)

Images are binary files. So passing them 'as is' is recommended. This method requires least memory and time resources in comparison with the other methods

* Request header
    ```
    Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    ```

* Request body
    ```
    FF D8 FF E0 00 10 4A 46 49 46 00 01 01 00 00 01
    00 01 00 00 FF DB 00 43 00 05 03 04 04 04 03 05
    ...
    00 51 45 14 00 51 45 14 00 51 45 14 00 51 45 14
    01 FF D9
    ```

##### 2) Sending base64 encoded image

Alternative way: use base64 encoding instead of pure binary data.
To send image as base64, `Content-Transfer-Encoding` header must be set to `base64`

* Request header
    ```
    Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    Content-Transfer-Encoding: base64
    ```

* Request body
    ```
    /9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAUDBAQEAwUEBAQFBQUGBwwIBwcHBw8LCwkMEQ8SEhEPERETFhwXExQaFRERGCEYGh0dHx8fExciJCIeJBweHx7...AUUUUAFFFFABRRRQB/9k=
    ```

##### 3) Sending resource url

You can also pass url, server will download image by itself.
To provide url, set `Content-Type` header to `text/plain`, then the request body will be considered as url.

Url should be well-formed in accordance with [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) and [RFC 2732](https://www.ietf.org/rfc/rfc2732.txt)
<!-- This standars are used in Uri.IsWellFormedUriString(uri, uriKind) method, .NET Framework 4.0 . If Server will run under .NET Framework 4.5 or later, then RFC 3986 and RFC 3987 will be used -->

* Request header 
    ``` 
    Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    Content-Type: text/plain
    ```

* Request body
    ```
    https://some.image.url.com/img.png
    ```

To delete image, just send request without content (content length 0 bytes) or use DELETE method with same URL.

Result is 204 (No content) on success.

## Scopes

Scopes describe which API is available for specific application (more about applications will be described later).

You can see needed scope in method description (only if `Database` or `Employee` tokens used, otherwise scope is not appliable).

Scope|Description
:----|:----------
full|Get full access to database
clients_module|Read prices, employees information and free time
database_token_by_password|Possibility to receive database token providing employee requsites and password (available only for AI Helps products)
employee_login|Possibility to login with employee requsites with password or sms code (available only for AI Helps products)
client_login|Possibility to login with client requsites with sms code (or create new client, available only for AI Helps products)

## Errors

Depending on the request method, server return different HTTP status codes:

+ GET:
    + 200 - OK: on success
    + 404 - Not Found: item not found by given id
+ POST:
    + 201 - Created: on success
    + 202 - Accepted: for long requests (see Long running requests queue section)
+ PUT:
    + 200 - OK: on success if item returned
    + 202 - Accepted: for long requests (see Long running requests queue section)
    + 204 - No content: on success if item not returned
    + 404 - Not Found: item not found by given id
+ DELETE: 200 (OK)
    + 204 - No content: on success
    + 404 - Not Found: item not found by given id

+ Common errors:
    + 400 - Bad Request: error in request parameters (bad parameters, required parameter not given)
    + 401 - Unauthorized: authorization required (if not specified) or failed (bad token, token expires etc.)
    + 403 - Forbidden: your scope does not allow running current API (check needed scope in documentation)
    + 409 - Conflict: your changes conflicts with data in database
    + 429 - Too Many Requests: server limits number of requests by ip for some methods (like authorization)
    + 500 - Internal Server Error: error happen on server while API method running (please write us at ag@aihelps.com)
    + 503 - Service Unavailable: server unavailable due to maintainance or some server problem (please try again later)

Success codes are 200 for GET, 201 for POST, 200/204 for PUT and 204 for DELETE requests.

In case of error (error codes >= 400) server response has next format (for all types of requests):
```
{
    "status": 401,
    "type": "INVALID",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Unknown employee login or password",
    "details": "http://docs.aihelps.apiary.io/errors/INVALID"
}
```

Where:
+ status: HTTP status code (described earlier)
+ type: error type, if provided
+ request: request id (you can use this id to get details from API developers via dev@aihelps.com)
+ message: message for developer, describing problem
+ details: link to documentation with more details

There are two types of errors:
1) errors like Bad Request which notifies that you use API in wrong way. These errors do not have `type` field, but provides detailed message. Such errors usually happens during development phase, thus developer see them and fix inplace. Such errors are: 400 Bad Request, 401 Unathorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests, 500 Internal Server Error and 503 Service Unavailable. All of them can happen at any request.
2) errors that happens in production like invalid password while logging in or token expired. Such errors always have specific `type` and should be handled in code that uses API. Usually they have additional information in JSON parameters that helps to handle this error. You need to write code to manage them in production. They are: 400 Bad Request - APPOINTMENT_VALIDATION, 401 Unathorized - UNKNOWN, EXPIRED, INVALID and BLOCKED; 403 Forbidden - LICENSE; 409 Conflict - ALREADY_EXISTS and TIME_CONFLICT; 503 Service Unavailable - REMOTE_DATABASE, SMS, SMS_AUTH and SMS_BALANCE. From them 401 Unathorized - UNKNOWN, EXPIRED; 403 Forbidden - LICENSE; 503 Service Unavailable - REMOTE_DATABASE can happen at any request, other - only at specific requests (all such requests will be marked).

### 400 Bad Request

These errors means that your request is incorrect - not enough parameters, bad parameter type, failed validation and so on.

```
{
    "status": 400,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Parameter `name` value should be equals or greater than zero",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/400-bad-request"
}
```

Can happen at any request.

### 400 Bad Request - APPOINTMENT_VALIDATION

These errors means some validation errors while changing appointment. List of these errors is long enough, details can be found in `Appointments` section. You can pass `force=true` parameter to ignore such validations.

```
{
    "status": 400,
    "type": "APPOINTMENT_VALIDATION",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Validation error(s) while updating appointment",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/400-bad-request-appointment-validation",
    "validations":
    [
        {
            "id": "da77cb7c-5725-4afa-abf4-c097a9e9081a",
            "type": "SERVICE_PROFESSIONAL_CANT_PROVIDE_SERVICE",
            "service": "ced508d8-b4cf-498b-9893-a976a912ba21",
            "professional": "6b98745c-d72d-4831-b7cc-104449d17617"
        }
    ]
}
```

**Can happen only at appointment insert/update endpoints.**

### 401 Unathorized

These errors means some problems with authorization. If no `type` specified, it's usually common problem, that happens during development.

```
{
    "status": 401,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Unknown authorization scheme: Basic",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized"
}
```

Can happen at any request.

### 401 Unathorized - UNKNOWN

These errors means some problems with authorization. `type=UNKNOWN` means that provided access token, refresh token, application id & password ot other requisites are unknown. There is no way to refresh token or continue session.

```
{
    "status": 401,
    "type": "UNKNOWN",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Unknown access token: 3227f712-eb6c-4003-8260-ca70fcad50d1",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-unknown"
}
```

Correct handling this error in production - treat session as logged out and redirect user to login page.

Can happen at any request.

### 401 Unathorized - EXPIRED

These errors means some problems with authorization. `type=EXPIRED` means that provided access token, auth id or other temporary access object is expired,

```
{
    "status": 401,
    "type": "EXPIRED",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Access token expired",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-expired"
}
```

Correct handling this error in production - silently refresh token, save new token and repeat original request.

Can happen at any request.

### 401 Unathorized - INVALID
<a name="error-INVALID"></a> 

These errors means some problems with authorization. `type=INVALID` means that during authorization process you provided wrong login/password, sms code or other authorization information.

```
{
    "status": 401,
    "type": "INVALID",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Wrong provided sms code",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-invalid"
}
```

Correct handling this error in production - notify user his credentials are invalid and provide possibility to enter credentials once more.

**Can happen only at few authorization requests, such request will contain info about this error.**

### 401 Unathorized - BLOCKED
<a name="error-BLOCKED"></a> 

These errors means some problems with authorization. `type=BLOCKED` means that during authorization process you overlimit attempts to enter password/sms code or other credentials and verification blocked for some time.

Additional parameter added `unblockAfter`, which contains number of seconds after which unblock occures and you can repeat attempts.
```
{
    "status": 401,
    "type": "BLOCKED",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Can't check password: blocked for 10 minutes because 5 wrong passwords provided, left 124 seconds.",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-blocked",
    "unblockAfter": 124
}
```

Another similar case: sms can be send only once a minute:
```
{
    "status": 401,
    "type": "BLOCKED",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Sms verification code was sent recently, minimum interval is one minute between sending smses, 17 seconds left.",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-blocked",
    "unblockAfter": 17
}
```

If you don't receive `unblockAfter` field, the only option for user is to start authentification process from start:
```
{
    "status": 401,
    "type": "BLOCKED",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "auth_id becomes invalid after sending 20 wrong codes",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/401-unathorized-blocked"
}
```

Correct handling this error in production - notify user his credentials are invalid, wait for specified amount of time (if provided) and provide possibility to enter credentials once more. If `unblockAfter` not specified, start authorization process from beginning.

**Can happen only at few authorization requests, such request will contain info about this error.**

### 403 Forbidden

These errors means your scope does not allow running current API (check needed scope in documentation). If no `type` specified, it's usually problem with scopes and easily found during development process.

```
{
    "status": 403,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Your scope (reports) does not allow running 'POST /clients'",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/403-forbidden"
}
```

Can happen at any request.

### 403 Forbidden - LICENSE

`type=LICENSE` means some problem with license, for example, "Licenses expired for all locations expired" or "This endpoint not allowed for your license type".

```
{
    "status": 403,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Licenses expired for all locations expired",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/403-forbidden-license"
}
```

Correct handling this error in production - notify user that his license is finished, he should prolonge it until continue using product.

Can happen at any request.

### 404 Not Found

These errors means that either URL was not found or some object was not found by id, phone, name etc.

Additional parameters added `object`, `type`, `value`, which describes what object was not found.

`object` can be either "url" or item from database: "client", "employee", "appointment", etc.
`type` describes type of requisites used for search: "url" (for "url" only), "id", "phone", "email", "name", "code" (for database)
`value` value of type `type` used for search.

If URL was not found:
```
{
    "status": 404,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "No API with URL POST /abc found",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/404-not-found",
    "object": "url",
    "type": "url",
    "value": "POST /abc"
}
```

If some object was not found:
```
{
    "status": 404,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Client with id ccac7904-dbaa-4373-95c2-464031fdb533 not found",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/404-not-found",
    "object": "client",
    "type": "id",
    "value": "ccac7904-dbaa-4373-95c2-464031fdb533"
}
```

Can happen at any request.

### 409 Conflict - ALREADY_EXISTS
<a name="error-ALREADY_EXISTS"></a>

These errors means request could not be completed due to a conflict with the current state of the target resource. `type=ALREADY_EXISTS` means that object with provided parameters already exists.

Additional parameters added `type`, `id`, `where`:
`itemType` - type of item already exists
`id` of object already exists
`where` - list of conditions to be unique, used to find item

```
{
    "status": 409,
    "type": "ALREADY_EXISTS",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Only one planned or confirmed appointment should exists for specified date + location + client + employee",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/409-conflict-already_exists",
    "itemType": "appointment",
    "id": "74e85faf-de73-4fe6-9530-3af1dd7c1b34",
    "where":
    {
        "date": "2020-01-01T00:00:00Z000",
        "location": "e0cd7be0-dcf1-4b49-b704-a5d2cbd59125",
        "client": "012b6f96-dbea-4e47-8c27-c474021c03f5",
        "employee": null,
        "state": "planned/confirmed"
    }
}
```

Correct handling this error in production - depending from endpoint, correct input data and make request once more if needed (in many cases item already exists and no need to repeat request).

**Can happen only at few requests, such request will contain info about this error.**

### 409 Conflict - TIME_CONFLICT
<a name="error-TIME_CONFLICT"></a>

These errors means request could not be completed due to a conflict with the current state of the target resource. `type=TIME_CONFLICT` means that object can be saved on proposed time because it will conflict with already stored items.

Additional parameter added `anotherItems`, which is array of items, each containing:
`id`: id of object which changes raised time conflict (id of group lesson or aappointment service when changing itself, id of specific appointment service when changing appointment)
`itemType`: `professional`, `hall`, `resource` or `appointment` - type of conflicted resource
`itemId`: id of conflicted resource
`objectType`: another object, that use resource, possible values `appointmentService`, `groupLesson`, or `nonWorkingTime` (for professional only)
`objectId`: appointment service id or group lesson id
`objectDescriptionId`: object that describes conflicted object: appointment client or employee id for `appointmentService`, group id for `groupLesson`
`objectStart`: start of object (date and time)
`objectDuration`: object duration in minutes

```
{
    "status": 409,
    "type": "TIME_CONFLICT",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Only one planned or confirmed appointment should exists for specified date + location + client + employee",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/409-conflict-time_conflict",
    "anotherItems":
    [
        {
            "id": "da77cb7c-5725-4afa-abf4-c097a9e9081a",
            "itemType": "professional",
            "itemId": "9d5437d2-b868-426c-a572-38d532d47987",
            "objectType": "appointmentService",
            "objectId": "e03ee732-2f5a-476f-afdb-0d53a16a072e",
            "objectDescriptionId": "4ac7591c-8811-430b-a3fb-d2e53f42c177",
            "objectStart": "2020-01-01T00:00:00Z000",
            "objectDuration": 120
        }
    ]
}
```

Correct handling this error in production - show received conflicts to user. If user really sure, most endpoints returning this error can be called once more with `force=true` parameter, ignoring this error.

**Can happen only at few requests, such request will contain info about this error.**

### 429 Too Many Requests

If you make too much requests, you can see bug like this. There are different limits on different parts of API, but in normal cases they should not be reached.

```
{
    "status": 429,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Up to 100 requests per second allowed from one IP (try again soon)",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/429-too-many-requests"
}
```

Can happen at any request.

### 500 Internal Server Error

These errors usually means some server bug. Provide `request` identifier to development team to solve the bug.
Details of error are not shown for security reasons.

```
{
    "status": 500,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Internal Server Error: error id 'a7dfb9a8-0383-4087-a1e6-606fc2bc714c'. You can use this id to get error details using api@aihelps.com",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/500-internal-server-error"
}
```

Can happen at any request.

### 503 Service Unavailable

These errors means some structural internal API problems, like problems with database. If no `type` specified, it's usually problems that can be solved only by AI Helps team in nearest future, you can do nothing with it.

```
{
    "status": 500,
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Service Unavailable (some of internal infrastructure is unavailable thus request can't be processed)",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/503-service-unavailable"
}
```

Can happen at any request.

### 503 Service Unavailable - REMOTE_DATABASE

In rare cases clients database is located not on AI Helps servers, but hosted on clients server. AI Helps can't guarantee such server will be always accessible. 
`type=REMOTE_DATABASE` means API can't connect to clients database and process request (either server/database unavailable or wrong credentials stored in AI Helps for remote server). You can push clients by yourself to guarantee stable connection to their database.

```
{
    "status": 500,
    "type": "REMOTE_DATABASE",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Remote database unavailable (client database located on other server is unavailable)",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/503-service-unavailable-remote_database"
}
```

Can happen at any request.

### 503 Service Unavailable - SMS
<a name="error-SMS"></a>

Clients usually have accounts in text messaging services to send smses to their clients. Some endpoints requires sending sms to proceed (like login via sms code). If some error happens during sending sms, errors with `type=SMS` wil be received. These errors are usually either problems with connectivity or situations when service changed their API and integration become invalid.

```
{
    "status": 503,
    "type": "SMS",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Sending SMS: can't connect to sending service",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/503-service-unavailable-sms"
}
```

Correct handling this error in production - notify user message can't be sent due to problens with sending service and he should try once more in some time.

**Can happen only at few requests, where sending sms is crucial, such request will contain info about this error.**

### 503 Service Unavailable - SMS_AUTH
<a name="error-SMS_AUTH"></a>

Clients usually have accounts in text messaging services to send smses to their clients. Some endpoints requires sending sms to proceed (like login via sms code). If during sending sms we discovered, that login, password or alpha name for sms sending service is incorrect, error with `type=SMS_AUTH` will happen. Contrary to error `type=SMS` this one can be easily fixed changing login/password/alpha name in program settings.

```
{
    "status": 503,
    "type": "SMS_AUTH",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Sending SMS: wrong login/password",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/503-service-unavailable-sms_auth"
}
```

Correct handling this error in production - notify user message can't be sent due to wrong credentials and he should try once more in some time.

**Can happen only at few requests, where sending sms is crucial, such request will contain info about this error.**

### 503 Service Unavailable - SMS_BALANCE
<a name="error-SMS_BALANCE"></a>

Clients usually have accounts in text messaging services to send smses to their clients. Some endpoints requires sending sms to proceed (like login via sms code). If during sending sms we discovered, that no more credits left, error with `type=SMS_BALANCE` will happen. Contrary to error `type=SMS` this one can be easily fixed by client, making payment to sms sending service.

```
{
    "status": 503,
    "type": "SMS_BALANCE",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Sending SMS: out of money",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/503-service-unavailable-sms_balance"
}
```

Correct handling this error in production - notify user message can't be sent due to lack of money and he should try once more in some time.

**Can happen only at few requests, where sending sms is crucial, such request will contain info about this error.**

## Change history

Here you can see history of changes in API and documentation.

Date|Description
-:|:-
2021-03-29|Field clientInside added to appointment
2021-01-25|Fields price, professionalPhone, hallName, clientName, clientPhone, cancelReason added to appointment service
2021-01-11|Merged and Original fields added to appointments
2020-12-02|HasAnySales field added to appointments
2020-12-02|Parent field added to positions
**2020-11-29**|**Prepayment settings and fields added**
**2020-11-18**|**Feedback field added to appointments**
**2020-11-15**|**New API to get movements & stocktakings in Products section, field `products` in sales**
2020-09-23|Settings & employee field `language` added
2020-09-17|Updated dashboard reports
**2020-09-10**|**New endpoint `/feedbacks` - new way to work with appointments feedbacks**
2020-09-09|Added error type 400 Bad Request: APPOINTMENT_VALIDATION
2020-08-29|Minor changes to appointments change behaviour in different states
2020-08-21|Added check appointment services overlaps in one appointment
**2020-08-19**|**`timeConflicts` and `validations` fields added to appointments and appointment items**
**2020-08-15**|**Creating appointment with location+date+client/employee already exists do not raise error but merge second appointment into existing one**
2020-08-05|Sending additional sms notification for appointment
**2020-08-01**|**Additional validations mechanism added to appointments: see details in Appointments section**
2020-07-27|Time conflict error - now shows id of object caused error
2020-07-27|Services: pricesInNativeCurrency, noProfessionalPriceInNativeCurrency fields. Products: packagePrice, packagePriceInNativeCurrency, portionPrice, portionPriceInNativeCurrency, unitPrice, unitPriceInNativeCurrency fields
2020-07-27|add field pictureUrl for several endpoints: services, groups, products, cards
2020-07-07|scope `clients_module` add permissions for getting specific fields for several endpoints: employees, services, grouplessons, positions, locations
2020-07-07|changing appointments: sms errors do not raise 503 Service Unavailable error, but send info in additional field
2020-07-07|update via webhooks added to `clients_module` scope
2020-07-07|sales: payments field added
2020-07-02|scope `services_aggregator` added
**2020-06-25**|**Promotions added to API**
2020-06-24|max length for text fields added
**2020-06-24**|**New errors handling implemented - see Errors section for details**
2020-06-05|appointments: referral_source field added
2020-06-05|employees: email field added
2020-06-05|products: extendent units field add items at enum
2020-06-01|storages: `sale_unit_types` field added
2020-05-29|Remove endpoints to delete all items
2020-05-13|storages: products_filter filed added; products: extended location_prices fields: original_automatic_price,original_automatic_portion_price,original_automatic_unit_price added
2020-04-29|Tasks period added in minutes and hours + `period_multiplier` field
2020-04-27|grouplessons: assistant, assistant_name, hall fields
2020-04-24|Endpoints to make NPS surveys
2020-04-08|New scope `online_store` added for integrations with online stores
2020-03-31|products: `supply_price_currency`, `supply_price` fields added
2020-03-19|clients: `photo` field added, employees: `photo` fields added
2020-03-14|helpers get country and city by ip
2020-03-12|clients: `comments` field
2020-03-10|appointment: `hall` field
2020-02-20|client: `status` field (readonly)
2020-02-18|Endpoint to estimate password quality and provide suggestions to improve password
**2020-02-16**|**Endpoints to view/change user information: /me/**
**2020-02-16**|**New employee authorization mechanism (old will become deprecated)**
2020-01-14|cards and clientcards: `discounts` field added (readonly)
2019-12-26|settings: fields: services_from_filter
**2019-12-24**|**Authorization now checks client licenses**
2019-12-24|Locations: field and filter `active` added
2019-12-24|Errors 403_02x & 403_04x changed and regrouped
2019-12-06|Each scheme has universal filter `ids` to filter by ids.
2019-10-25|501 Not Implemented error added to errors list
**2019-10-29**|**Long running requests queue section (in Tasks)**
2019-10-25|employees: fields: phone added
2019-10-11|products: fields: location_prices added
2019-10-10|`/group_appointments/id`: new endpoint
2019-09-25|grouplessons: fields: `filled_completely` added
2019-09-16|`/employees/pick_employee`: new endpoint, `/internal/v1/prices`: new endpoint, `/internal/v1/contacts`: new endpoint
2019-09-06|Cards: field `visits` added
2019-09-05|`/schedule` endpoint: field `assistant` added
2019-09-05|Appointments: field `cancelReceptionist` added
**2019-08-31**|**Reports\Dashboard section added (extracting data needed for dashboard report)**
**2019-08-30**|**Updates section added (receiving updates via Web sockets or Web hooks)**

## Deprecated endpoints and fields

This section contains information about deprecated endpoints or fields in endpoints. We usually deprecate endpoints or fields if we propose some better alternative for such fields or resources. After deprecation published we give at least 3 months to API users to make corresponding changes in their code.
According to IETF standard, we return 3 headers in response in case endpoint is deprecated:

Deprecation: true
Sunset: [date]
Link: <[link to subsection in this section]>; rel="deprecation"

Link leads to subsection of this section, where more details and alternatives are described.

In case of deprecated field, we add one more non-standard header to make it easier to distinguish problem:

X-Deprecated-Field: [field]
Sunset date is date when we plan to remove endpoint or field. We will not remove endpont or field before that date, but can do it in any moment after date happens (thus in some cases endpoint/field can be valid for months more, but no guarantee).

### Field 'price'
<a name="deprecation-appointment-service-price"></a>

Field 'price' doesn't take into account active cards and discounts for a client. Use 'predictedSum' field instead.

### Fields 'discount', 'bonus', 'available_locations' for client cards
<a name="deprecation-clientcard"></a>

These fields use old structure, use fields 'discounts', 'bonuses', 'availableLocations' instead.

# Group Authorization

This section describes authorization that should be done before most API calls. For each method in this documentation authorization type will be specified.

API based on OAuth 2.0 authorization model (with some significant changes in terminology, syntax and principles), see https://tools.ietf.org/html/rfc6749 for details.
To make most of requests, you need access token (you can see needed token type for each method in method description as "Authorization"). Access token is string that
represents access to client database (full or limited) given for some application for specific period of time. Access token has lifetime - 24 hours after moment it was
received. After that refresh token (received along with access token) can be used to receive new valid token.

For successful authorization you need to register your application and receive `application_id` and `application_secret` (this should be done only once).
Registering/updating/removing application is described in next section (Application).
This section implies that you already has `application_id` and `application_secret`.

There are 4 types of access tokens:
1) `Application` token - this token can be used to view/change application information and get application tokens list. This token does not allow any access to clients databases.
For each application only one valid `Application` token exists in given moment of time. On successful receiving new `Application` token, previous `Application`
token for given application, if exists, become unavailable. Receiving new `Application` token using API will be described later in this section.
2) `Database` token - this token grants access for your application to specific database. For access to client database you need to get permission
from one of database employees with chief rights (with red user icon in program). Typically used if you need to read and analyze some information from database or
add/update/delete information for all employees (e.g. add sales from other software).
For each application-database pair only one valid `Database` token exists in given moment of time. On successful receiving new `Database` token, previous
`Database` token for given application-database pair, if exists, become unavailable. New `Database` token can be received in three ways: using API
to show login page in browser (will be described later in this section), using API send employee requisites (need application `database_token_by_password` scope,
will be described later in this section) or through Beauty Pro/Fitness Pro application. In third case, user with chief rights should start the program,
select Settings->Marketplace, find your application and press "Grant access". Token will be generated. You can check if you already have access token,
using Access Tokens API (see token management below in this section).
Please mention, that `Database` token access is common for whole database and not depends from user, who grant access. Thus access can be revoked by another
user with chief rights.
3) `Employee` token - this token grants access for your application to specific database for specific employee. This token available only for specific
AIHelps products (application should have `employee_login` scope) and used for login. Receiving this token, in fact, means employee
login and all subsequent API requests will be executed as if employee done it itself in software. This token type should be used if you need to perform operations
from employee point of view (e.g. custom application with some additional functions or custom interface).
One employee can have several valid `Employee` tokens at once, what allows him to use application from several devices. On successful receiving new `Employee` token, previous
`Employee` tokens for given application-database-employee triple still remains valid. Receiving new `Employee` token using API will be described later in this section.
Token can be received either providing employee requsites with password or providing employee phone and sms code. In second code database can be omitted (but not all
employess will be found).
4) `Client` (beauty salon/gym client) token - this token grants access for your application to specific database for specific beauty salon/gym client.
This token available only for specific AIHelps products (application should have `client_login` scope) and used for login. 
Receiving this token, in fact, means client login and all subsequent API requests will be executed as if client done it itself in software. Client always login
by his phone number and code from SMS. This token type should be used for all types of client applications: booking systems, loyality/marketing applications etc.
One client can have several valid `Client` tokens at once, what allows him to use application from several devices. On successful receiving new `Client` token, previous
`Client` tokens for given application-database-client triple still remains valid. Receiving new `Client` token using API will be described later in
this section.

Received token information contains:
- access token (can be used for API queries for 24 hours)
- access token expire time
- scope (for `Database` and `Employee` tokens)
- refresh token (for `Database`, `Employee` and `Client` tokens)

There are two more useful procedures:

1) If access token expires, refresh token can be used to generate new valid token (described below).
2) If token become public or insecure or simply not needed any more, it can be revoked. Revoking tokens described below in this section.

## Application token [/auth/application]

### Receiving Application token [GET /auth/application{?application_id,application_secret}]

If you want get/update application information, delete application, manage application access tokens and some other operations, not connected to specific client database,
Application access token needed. Process of receiving such token very simple, cause you don't need any confirmation from end users - just send `application_id` and
`application_secret` (both are required) and receive token.

Please pay attention, such token need for one-time operations, thus `refresh_token` not generated.

**Authorization:** none

+ Parameters
    + application_id: `a47cbc05-5ce6-4551-9456-2855b52a17f0` (string, required) - application id
    + application_secret: `a3db644a-907a-4da2-bcdd-bc9f7068237d` (string, required) - application secret

+ Response 200 (application/json)

    + Attributes
        + access_token: `d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5` (string) - token used for authorizations
        + expires_at: `2016-08-12T14:18:27.000Z` (datetime) - access expiration date

    + Body

            {
                "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
                "expires_at": "2017-06-01T00:00:00.000Z"
            }

## Database token [/auth/database]

### Receiving Database token [GET /auth/database{?application_id,application_secret,database_code,location}]

This method creates request to access specified database for specific application if not yet created and returns access token information if access already granted.

This method can be used in two cases:

Access to private application (default state for all applications):
1) Call this method with specified `application_id`, `application_secret` and `database_code` will create access request.
2) Login into desktop application (BeautyPro, FitnessPro, DentaPro), go to Settings->Marketplace, select needed application and press `Grant access` (in marketplace only public applications and applications that asked for access are shown).
3) Call this method once again with specified `application_id`, `application_secret` and `database_code` will return access token information.

Access to public application (public applications are generally used by many clients, switching application to public is made manually by developer request if application is ready and has some value for clients):
1) User login into desktop or web application, go to Settings->Marketplace, select needed application and press `Grant access`.
2) Call this method with specified `application_id`, `application_secret` and `database_code` will return access token information.

If access was granted, method will return access token information, if not granted - JSON with one field `status` either "pending" or "refused".

**Authorization:** none

+ Parameters
    + application_id: `a47cbc05-5ce6-4551-9456-2855b52a17f0` (identifier, required) - application id
    + application_secret: `a3db644a-907a-4da2-bcdd-bc9f7068237d` (identifier, required) - application secret
    + database_code: `123456` (string, required) - database for which access should be granted
    + location: `a47cbc05-5ce6-4551-9456-28ccb52bbb11` (identifier, optional) - The default location for the access token. If the location not specified in URL parameter it takes the location for the current database_code. 

+ Response 200 (application/json)

    + Attributes
        + status: `pending` (enum[string]) - status of current request
            + Members
                + `pending` - request still pending
                + `refused` - request refused by user

    + Body

            {
                "status": "pending"
            }


+ Response 200 (application/json)

    + Attributes
        + server: 1 (number) - server index where client database is located, preffered for requests
        + access_token: `d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5` (string) - token used for API operations
        + database: `123456` (string) - database for which access granted
        + scope: `clients`, `sales` (array[string]) - set of rights granted
        + expires_at: `2017-06-01T00:00:00.000Z` (datetime) - access expiration date
        + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string) - token used to update access token

    + Body

            {
                "server": 1,
                "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
                "database": "123456",
                "scope": [ "full", "employee_login" ],
                "expires_at": "2017-06-01T00:00:00.000Z",
                "location": "a71923bd-84c5-49a3-a7fb-a08fbecc2be1"
            }

## Token refresh [/auth/refresh]

### Refresing expired token [GET /auth/refresh{?application_id,refresh_token}]

Access token is valid only 24 hours. After that period token should be refreshed. Call this method to refresh token.
Please pay attention that refresh token is also not perpetual. It can become invalid (all information about access token & refresh token will be vanished)
in several months if not used (but validity period much longer than 24 hours for access token). In this case you need to receive new token.

On success new `access_token` and `refresh_token` will be generated and `expires_at` updated. Old `access_token` and `refresh_token` will be unavailable.

**Authorization:** none

+ Parameters
    + application_id: `a47cbc05-5ce6-4551-9456-2855b52a17f0` (string, required) - application id
    + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string, required) - refresh token for updating access token

+ Response 200 (application/json)

    + Attributes
        + access_token: `d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5` (string) - new access token
        + expires_at: `2016-08-12T14:18:27.000Z` (datetime) - new access expiration date
        + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string) - new refresh token

    + Body

            {
                "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
                "expires_at": "2017-06-01T00:00:00.000Z",
                "refresh_token": "13baeeb8-ebe7-4751-86a6-5c960bfe1776"
            }

## Token revoke [/auth/revoke]

### Revoke token [GET /auth/revoke{?application_id,refresh_token}]

If access token become public or insecure in some other way or just not needed any more, you can revoke it.
Revoking makes both access token unavailable for API calls and refresh token unavailable for generating new token.
Revoking is valid for all types of tokens.

Revoke should be done using `refresh_token`, not `access_token`.

Please pay attention that refresh token is also not perpetual. It can become invalid (all information about access token & refresh token will be vanished)
in several months if not used (but validity period much longer than 24 hours for access token). In this case tokens already deleted and nothing more should be done.

**Authorization:** none

+ Parameters
    + application_id: `a47cbc05-5ce6-4551-9456-2855b52a17f0` (string, required) - application id
    + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string, required) - refresh token to revoke

+ Response 204

# Group Applications

To receive access token you need to register your application and receive `application_id` and `application_secret`. 
In AI Helps API one developer account have only one application account, so "Application" and "Developer" are synonyms in terms of current API. For simplicity sake we
would call it "Application" later in this document.

## Application [/applications]

+ Attributes
    + application_name (string, required) - Name of application as it will be shown to end user. Can be non unique (do not try to give unique name, leave it meaningful). Max length is 500 characters.
    + application_url (string, optional) - Link to application site (without leading `https://`, HTTP not allowed, will be shown to end users)
    + developer_name (string, optional) - Application developer name as it will be shown to end user. Can be non unique (one developer can register several accounts). Max length is 500 characters.
    + contact_email (string, optional) - E-mail used for notifications, account recovery etc.
    Important, because this e-mail will be used to notify about switching to new API version (do not miss it!). Not visible to end users.
    + application_id (string, optional) - id needed for all requests. Is unique for application. Generated by server on application registration and can"t be changed.
    Can be kept publicly visible.
    + application_secret (string, optional) - Secret string, generated by server, needed for authorization. Generated by server on application registration, should be kept private.
    Can be regenerated if became public (all application access token become invalid).
    + redirection_endpoint (string, optional) - URI for receiving access token. URI should be given without leading `https://`.
    Leading `www.` also can be omitted. Examples: `aihelps.com/registration_complete`, `aihelps.com/auth`.
    + scope: `full`, `clients_module` (array[string], required) - List of scope tokens (describes needed access to API, will be described later).
    + public: false (boolean, optional) - If application is public (read only). Public applications are visible for all databases and can be easily connected to database through Marketplace. For non-public applications access request is required. If application is public, only `contact_email`, `redirection_endpoint`, `grant_access_url` fields can be changed. To make application public, make request on api@aihelps.com, describe your application. If approved, application will become public in several working days.
    + countries: `` (array[string], optional) - List of countries where application should be shown as public (should be set before making application public). If empty, application is public in all countries.
    + grant_access_url: `www.myapp.com/connected?client={database_code}` (string, optional) - URL template which will be called when someone grants access for application for some database. `{database_code}` will be replaced by 6-digit database code.
    + databases (string, optional) - list of databases to which application can connect, separated by comma. If empty - can connect to any database.
    + created (datetime, optional) - Date/time when application was created (read only)

### Register new application account [POST /applications{?fields}]

Register new application.

**At current moment, due to behaviour of some applications, trying to mimic as well-known, trusted applications and trying to steal users data, we suspended automatic applications registration. All registartions are done in manual mode: please connect AI Helps support by phone or via chat on aihelps.com web site.**

`application_name` is required field to register application. `application_name` even can already exists in applications. 

`redirection_endpoint` and `scope` are also required to make authorization requests (can be changed later).

`application_id` and `application_secret` are generated by server, so they can't be provided by user.

Please pay attention, that you will need at least `application_id` and `application_secret`, thus you need to use `fields` parameter to retrieve information about
application created. It will be impossible to do it later, cause all next queries for this application require Application token, which can be generated using
`application_id` and `application_secret` both.

**Authorization:** AI Helps employee with special rights

+ Parameters
    + fields: `application_name,application_id,application_url,developer_name,contact_email,redirection_endpoint,application_secret,scope,public,countries,grant_access_url,created` (array[string], required) - list of fields to return (separated by comma)

+ Request (application/json)

    + Body

            {
                "application_name": "Test application",
                "application_url": "testapp.com",
                "developer_name": "ABC Solutions",
                "contact_email": "dev@testapp.com",
                "redirection_endpoint": "testapp.com/app",
                "scope": "clients_module",
                "grant_access_url": "testapp.com/access/{database_code}"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "9245f5b4-d66e-483c-b7c4-fd55f9677bb3"
                "application_name": "Test application",
                "application_url": "testapp.com",
                "developer_name": "ABC Solutions",
                "contact_email": "dev@testapp.com",
                "application_id": "5330336c-143a-415a-b220-fc9932bc029d",
                "application_secret": "eb5cb7df-356e-475f-8afb-2eebe63a37bb",
                "redirection_endpoint": "testapp.com/app",
                "scope": "clients_module",
                "public": false,
                "countries": [],
                "grant_access_url": "testapp.com/access/{database_code}"
            }

### Get information about application [GET /applications/{id}{?fields}]

Get information for existing application.

Application token should be retrieved to make this request.
Application information can be received only for application for which token was got.

For your convenience, you can use `me` instead of id of application.

**Authorization:** `Application`

+ Parameters
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - id of application to get. id should match token application or "me" can be used
    + fields: `application_name,application_id,application_url,developer_name,contact_email,redirection_endpoint,application_secret,scope,created` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "9245f5b4-d66e-483c-b7c4-fd55f9677bb3"
                "application_name": "Test application",
                "application_url": "testapp.com",
                "developer_name": "ABC Solutions",
                "contact_email": "dev@testapp.com",
                "application_id": "5330336c-143a-415a-b220-fc9932bc029d",
                "application_secret": "eb5cb7df-356e-475f-8afb-2eebe63a37bb",
                "redirection_endpoint": "testapp.com/app",
                "scope": "clients_module",
                "public": false,
                "countries": [],
                "grant_access_url": "testapp.com/access/{database_code}"
            }

### Change application account information [PUT /applications/{id}{?fields}]

Change information for existing application. Requirements for fields left same as during creation of new application.

Application token should be retrieved to make this request.
Application information can be received only for application for which token was got.

For your convenience, you can use `me` instead of id of application.

**Authorization:** `Application`

+ Parameters
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - id of application to update. id should match token application or "me" can be used
    + fields: `application_name,application_id,application_url,developer_name,contact_email,redirection_endpoint,application_secret,scope,created` (array[string], optional) - list of fields to return (separated by comma)

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "application_name": "Test application",
                "application_url": "testapp.com",
                "developer_name": "ABC Solutions",
                "contact_email": "dev@testapp.com",
                "redirection_endpoint": "testapp.com/app",
                "scope": "clients_module"
            }

+ Response 200 (application/json)

    + Body

            {
                "id": "9245f5b4-d66e-483c-b7c4-fd55f9677bb3"
                "application_name": "Test application",
                "application_url": "testapp.com",
                "developer_name": "ABC Solutions",
                "contact_email": "dev@testapp.com",
                "application_id": "5330336c-143a-415a-b220-fc9932bc029d",
                "application_secret": "4809691c-0c3b-456f-aa3f-f3fca5b67010",
                "redirection_endpoint": "testapp.com/app",
                "scope": "clients_module",
                "public": false,
                "countries": [],
                "grant_access_url": "testapp.com/access/{database_code}"
            }

### Delete application [DELETE /applications/{id}]

Delete existing application.

Application token should be retrieved to make this request.
Application information can be received only for application for which token was got.

For your convenience, you can use `me` instead of id of application.

All access tokens (including Application token used to delete application) for this application will become invalid.

**Authorization:** `Application`

+ Parameters
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - id of application to update. id should match token application or "me" can be used

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Application access [/applications/{id}]

One of the easisest ways for applications to get access to business databases - publish application in Marketplace and/or create access requests.
If application is public, user can easily grant access for application through Marketplace (all public applications are shown by default). If not public, access can be requested via `/auth/database`, applications with access requests are also shown in Marketplace. There in Marketplace user can grant access for specific application in one click. If `grant_access_url` is not empty, request will be iimediately sent to this url. Otherwise application should periodically check `/auth/database` for access granted.

Methods in this section needed for Marketplace internal needs (or you can use them if you want to create custom marketplace). They work with Database token with `full` scope.

### Get applications available for database [GET /applications{?fields}]

Get list of applications that are either public (and available for all countries or country database belongs to) or which requests access to this database.
Information is much shorter than info that can be extracted via Application token and contains only "public" information.

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameter
    + fields: `application_name,application_url,scope,public,status` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Attributes
        + id: `e98d5a05-d439-40ec-8dc2-7cce3f216e35` (identifier, required) - application id
        + application_name (string, required) - application name
        + application_url (string, optional) - link to application site
        + scope: `full`, `clients_module` (array[string], required) - list of scope tokens
        + public: false (boolean, optional) - if application is public
        + status: `none` (enum[string]) - access status
            + Members
                + `none` - no request was made (application is returned because it is public)
                + `pending` - request still pending (user did not take any action)
                + `granted` - request was granted by user
                + `refused` - request was refused by user

    + Body

            [
                {
                    "id": "3286cc03-3f88-4c8e-8d94-0af35b67beaf",
                    "application_name": "Test application",
                    "application_url": "testapp.com",
                    "scope":
                    [
                        "clients_module",
                    ],
                    "public": true,
                    "status": "none"
                },
                {
                    "id": "7c527f8e-2551-40a0-b878-79dc3b16dd18",
                    "application_name": "My first app",
                    "application_url": "",
                    "scope":
                    [
                        "full",
                    ],
                    "public": false,
                    "status": "pending"
                }
            ]

### Grant access for application to database [PUT /applications/{id}/grant]

Grant access for application to current database. If `grant_access_url` was set for application, call to that url will be made with substituted database code.

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameter
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - application id

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204

### Revoke access for application to database [PUT /applications/{id}/revoke]

Revoke access for application to current database. If only request was done, request revokes. If request was already accepted and access tokens was generated, all access tokens become invalid.

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameter
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - application id

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204

## Access tokens [/applications/{id}/tokens]

Each application has list of active tokens, received by authorization process.
Because authorization process can be initiated either by application or by client (inside program), there is need check if application already has token for specific database.
Plus deleting unnecessary tokens available.

+ Attributes
    + access_token: `d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5` (string) - token used for API requests
    + expires_at: `2016-08-12T14:18:27.000Z` (datetime) - access expiration date
    + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string) - token used to update access token
    + type: `Employee` (enum[string]) - defines token type
        + Default: `Employee`
        + Members
            + `Application` - Special token type for managing application and it's tokens. Not connected to any database, thus all non-application/tokens requests will fail.
            + `Database` - Token grants access for location database, according to `scope` field. Only one such token available for one database-application pair.
            + `Employee` - Token created after succesfull employee login (through API). Grants access to database according to `scope` field (this field is copied
            from `Database` token on `Employee` token creation). If you need employee login, employee tokens should be used for all requests after login.
            + `Client` - Token created after succesfull client login (through API). Grants limited access to database: logged client information, client history,
            operations with upcoming appointments. Useful for all kinds of client applications like creating appointments, client feedback etc.
    + scope: `clients`, `sales` (array[string]) - set of rights granted (for `Database`/`Employee` token types)
    + database: `123456` (string) - client database for which access granted (empty for token type `Application`)
    + owner: `f07ed0f2-1f68-4df5-bb90-3b19390f5c7e` (string) - employee/client id for `Employee`/`Client` token types
    + common_owner: `c96a564b-4f8d-41d6-8b8d-e31aa9bd157f` (string) - employee id from common employees list for `Employee` token type

### Get application access tokens [GET /applications/{id}/tokens{?database}]

Get all application active tokens (including expired but with `refresh_token`, thus can be updated).

For your convenience, you can use `me` instead of id of application.

Application token (if active) will be also listed, but with `database` and `scope` null fields.

If needed, can be retrieved only tokens for specific database.

**Authorization:** `Application`

+ Parameters
    + id: `9245f5b4-d66e-483c-b7c4-fd55f9677bb3` (identifier, required) - id of application to get tokens. id should match token application or "me" can be used
    + database: `123456` (string, optional) - if database code is set, filter only tokens for this database

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "tokens":
                    [
                        {
                            "access_token": "d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5",
                            "expires_at": "2017-12-04T23:46:27Z",
                            "refresh_token": "f9e96585-856f-4b48-95ca-9511c0ba5576",
                            "type": "Database",
                            "scope":
                                [
                                    "clients",
                                    "sales"
                                ],
                            "database": "123456",
                            "owner": null,
                            "common_owner": null
                        },
                        {
                            "access_token": "167b88db-9ebc-4ef6-a160-f466bd8f6548",
                            "expires_at": "2017-08-15T17:28:55Z",
                            "refresh_token": "aa742497-a3ae-4f76-9f07-53f93a96248a",
                            "type": "Database",
                            "scope":
                                [
                                    "clients",
                                    "sales"
                                ],
                            "database": "456789",
                            "owner": null,
                            "common_owner": null
                        },
                        {
                            "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
                            "expires_at": "2017-08-15T17:28:55Z",
                            "refresh_token": "13baeeb8-ebe7-4751-86a6-5c960bfe1776",
                            "type": "Application",
                            "scope": null,
                            "database": null,
                            "owner": null,
                            "common_owner": null
                        }
                    ]
            }

# Group Locations

One database represents data either for one beauty salon/gym or for network of beauty salons/gyms.
Let's name each such salon/gym a `location`.
Clients list is usually common, price list of services/products also usually common (but some prices can differ), some employees
can work at several locations at once. Schedule, appointments, sales is separate for each location.

This API provides information about all locations and possibility to change that information. But adding/removing locations is not
available - new locations automatically appeared after payment made for using software with additional location (location status set to active),
locations are not disappeared after payment was not made for specific location, but rather set status `active` to false (so, total number of active locations is defined by number of licenses paid).

## Location [/locations]
<a name="locations"></a>
+ Attributes
    + name (string, required) - Name of location. Max length is 500 characters
    + category (identifier) - Location category (locations can be grouped by city, class or other parameter).
    + code (number) - Unique 6-digit code of location (can't be changed).
    + city (string) - City where location is located. Max length is 100 characters
    + street (string) - Street and building where location is located. Max length is 500 characters
    + phone (string) - Location public phone.
    + geo_position (geoposition) - Geographical position of location. Null if not set.
    + timezone (string) - Location timezone (in TZ database format). Max length is 100 characters
    + active (boolean) - If location is active (has valid license).

### Get all locations [GET /locations{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only name, description, category, code, city, street, phone, web_site, geo_position, timezone, pictures_count, active), `online_store`, `reports`, `services_aggregator` (only name, description, category, code, city, street, phone, web_site, geo_position, timezone, pictures_count, active)

+ Parameters
    + fields: `name,category,code,city,street,phone,geo_position,timezone,active` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "2a0516e3-ddc9-432c-80e4-8d5f5e97cce3",
                    "name": "Salon 123 New York",
                    "category": "e2d21375-e772-4af2-8682-2b9460404b8e",
                    "code": 594365,
                    "city": "New York",
                    "street": "1, Main av.",
                    "phone": "+1 (555) 111 11 11",
                    "timezone": "America/New_York",
                    "active": true
                },
                {
                    "id": "3fe0e240-d779-4a84-9e0c-825d9a6ddcc4",
                    "name": "Salon 123 Los Angelos",
                    "category": "e2d21375-e772-4af2-8682-2b9460404b8e",
                    "code": 866423,
                    "city": "Los Angelos",
                    "street": "1, Main av.",
                    "phone": "+1 (555) 222 22 22",
                    "timezone": "America/Los_Angeles",
                    "active": true
                },
                {
                    "id": "843b8d24-2564-475f-8451-6ef25f2ccd18",
                    "name": "Salon 123 Miami",
                    "category": "ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1",
                    "code": 168544,
                    "city": "Miami",
                    "street": "1, Main av.",
                    "phone": "+1 (555) 333 33 33",
                    "timezone": "America/New_York",
                    "active": true
                }
            ]

### Get location by id [GET /locations/{id}{?fields,active}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only name,city,street,phone,geo_position,timezone,active), `online_store`, `reports`, `services_aggregator` (only name, description, category, code, city, street, phone, web_site, geo_position, timezone, pictures_count, active)

+ Parameters
    + id: `e98d5a05-d439-40ec-8dc2-7cce3f216e35` (identifier, required) - id of location
    + fields: `name,category,code,city,street,phone,geo_position,timezone,active` (array[string], required) - list of fields to return (separated by comma).
    + active: true (boolean, optional) - if to show only active/non-active locations.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "2dde3a05-c916-8d54-8432-7cce3f5e90ec",
                "name": "Salon 123 New York",
                "category": "e2d21375-e772-4af2-8682-2b9460404b8e",
                "code": 594365,
                "city": "New York",
                "street": "1, Main av.",
                "phone": "+1 (555) 111 11 11",
                "timezone": "America/New_York",
                "active": true
            }

### Update location [PUT /locations/{id}{?fields}]

Please note, that `code` field can't be updated.

`pictures` list should be updated one-by-one, see next "Add/Replace/Delete location picture" requests.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `1053a26e-c9dd-4ec0-8324-75cf5e98ce3d` (identifier, required) - id of location
    + fields: `name,category,code,city,street,phone,geo_position,timezone,active` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Salon 123 New York",
                "city": "New York"
            }
            
+ Response 204

## Locations categories [/locations/categories]

Unlike locations, which are bound to paid licenses, location categories can be easily changed. Following methods can be used
to get/update location categories.

### Get all locations categories [GET /locations/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `online_store`, `reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "e2d21375-e772-4af2-8682-2b9460404b8e",
                    "name": "Business class",
                    "parent": null
                },
                {
                    "id": "ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1",
                    "name": "Luxury salons",
                    "parent": null
                }
            ]

### Get location category by id [GET /locations/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `online_store`, `reports`

+ Parameters
    + id: `ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1` (identifier, required) - id of locations category
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1",
                "name": "Luxury salons",
                "parent": null
            }

### Create new locations category [POST /locations/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Nail bars",
                "parent": null
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "966a7cdc-ae92-4d6b-9605-9ab2dd74c776"
            }

### Update locations category [PUT /locations/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1` (identifier, required) - id of locations category
    + fields: `name,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Luxury boutiques",
                "parent": null
            }
            
+ Response 204

### Delete locations category [DELETE /locations/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1` (identifier, required) - id of locations category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get locations and categories tree [GET /locations/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `online_store`, `reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,            
                "categories":
                [
                    {
                        "id": "e2d21375-e772-4af2-8682-2b9460404b8e",
                        "name": "Business class",
                        "parent": null,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "98d52d05-d1c9-40ec-8e62-7c43ce3a3f5e",
                                "name": "Salon 123 New York",
                                "category": "e2d21375-e772-4af2-8682-2b9460404b8e",
                                "code": 594365,
                                "city": "New York",
                                "street": "1, Main av.",
                                "phone": "+1 (555) 111 11 11"
                            },
                            {
                                "id": "3fe0e240-d779-4a84-9e0c-825d9a6ddcc4",
                                "name": "Salon 123 Los Angelos",
                                "category": "e2d21375-e772-4af2-8682-2b9460404b8e",
                                "code": 866423,
                                "city": "Los Angelos",
                                "street": "1, Main av.",
                                "phone": "+1 (555) 222 22 22"
                            }
                        ]
                    },
                    {
                        "id": "ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1",
                        "name": "Luxury salons",
                        "parent": null,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "843b8d24-2564-475f-8451-6ef25f2ccd18",
                                "name": "Salon 123 Miami",
                                "category": "ba41dfa4-4ce1-4328-aa91-3dfa68a4d5c1",
                                "code": 168544,
                                "city": "Miami",
                                "street": "1, Main av.",
                                "phone": "+1 (555) 333 33 33"
                            }
                        ]
                    }
                ],
                "items":
                [
                ]
            }

# Group Clients

Clients in context of Beauty Pro/Fitness Pro are beauty salon/fitness gym visitors (customers).

## Client [/clients]
<a name="clients"></a> 
+ Attributes
    + name: `John Doe` (string) - client full name (read only)
    + firstname: `John` (string) - first name of a person. Max length is 500 characters
    + middlename:  `Grace` (string) - middle name of a person. Max length is 500 characters
    + lastname: `Doe` (string, optional) - last name of a person. Max length is 500 characters
    + title: `good client` (string, optional) - title of a person. Max length is 50 characters
    + gender: `male` (enum[string]) - client gender
        + Default: `male`
        + Members
            + `female`
    + birthday: `2000-01-01T12:00:00.000Z` (datetime) - client birthday. Year 1800 or less means no information, year 1805 means only month and day are valid (birthday year is unknown)
    + location: `1ed08dc9-ba00-478f-adbd-ef7371153fcc` (identifier) - client location [Locations](#locations) (where client was first registered)
        + Default: current location from token
    + balance: 200 (number) - client account balance sum (read only). This value can be changed throgh deposit operations.
        + Default: 0
    + bonus: 100 (number) - client bonus sum. Includes `Bring a friend` bonus sum (read only)
    + card_number: `123456` (string) - client card number or other unique identifier, can be array of identifiers. Max length is 500 characters.
    + phone: `+1 888 206 20 11` (array[string]) - client phone numbers
    + email: `bob@yahoo.com` (array[string]) - client emails
    + photo_exists: true (boolean) - if client has a photo (read only)
    + photo: `https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab990/photo` (string) - URL for getting a photo (readonly)
    + postal_code: `10001` (string) - client postal code. Max length is 50 characters
    + city: `New York` (string) - client city. Max length is 200 characters
    + street: `4-th Avenue` (string) - client street. Max length is 500 characters
    + building: `138` (string) - client building. Max length is 200 characters
    + apartment: `456` (string) - client apartment. Max length is 200 characters
    + categories (array[identifier]) - list of client categories. Only not automatic categories can be updated. See [Client categories](#client-categories)
        + Members
            + `0b646349-e7be-4179-97bd-c7155caab990`
            + `0799e1bc-86eb-4e0f-82e2-98dfe2cd55f9`
    + categories_names: `Often client` (array[string]) - list of client categories names (read only)
    + first_visit: `2018-01-01:13:00:00.000Z` (datetime) - client first visit date and time (read only). `1799-12-31:00:00.000Z` if no visit yet
        + Default: `1799-12-31:00:00.000Z`
    + first_visit_description: `yesterday` (datetime) - client first visit date text description (read only)
    + last_visit: `2018-01-01:13:00:00.000Z` (datetime) - client last visit date and time (read only). `1799-12-31:00:00.000Z` if no visit yet
        + Default: `1799-12-31:00:00.000Z`
    + last_visit_description: `yesterday` (datetime) - client last visit date text description (read only)
    + feedback (object) - information about client feedback (date, text, ratings)
        + date: `2018-01-01:13:00:00.000Z` (datetime) - feedback date
        + text: `cool` (string) - feedback text
        + ratings (object) - client rating for each question
            + id: `98402030-6e00-4235-85f4-9a5799d208f8` (identifier) - question id (questions list can be find in `/options`)
            + value: 5 (number) - client rating (from 1 to 5)
    + additional_fields (object) - additional fields, where additional client data can be stored (default is null). If some data needed to be stored, recommended way to define custom property and assign it's value
    + do_not_send_sms_notification: false (boolean) - do not get appointment notifications via SMS
    + do_not_send_sms_promotion: false (boolean) - do not get promotions and news via sms
    + do_not_send_email: false (boolean) - do not get any emails
    + createDate: '2018-03-01T00:00:00.000Z' (date) - date & time client created
    + deposit_client: '0b646349-e7be-4179-97bd-c7155caab990' (identifier) - if client uses deposit of another client (instead of his own), id of that client
    + referral_source: `0b646349-e7be-4179-97bd-c7155caab990` (identifier) - client referral source (Google, flyers, friends, etc.)
    + referral_source_name: `flyers` (string) - referral source name
    + status: `potential` (enum[string], optional) - status of client. This field is automatically calculated based on client visits (read only)
        + Members
            + `active`
            + `trial`
            + `onetime`
            + `nocard`            
            + `former`
            + `refused`
    + comment: `VIP client` (string) - customer note. Max length is 15728640 characters
    + archive: false (boolean) - if client was archived and can't be used in new sales
        + Default: `false`
    + professional: '0b646349-e7be-4179-97bd-c7155caab990' (identifier) - id of employee chosen for this client

### Get all clients [GET /clients{?fields,location,phone,email,card_number,name,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `online_store` (only name, firstname, middlename, lastname, gender, birthday, card_number, phone, email, comment fields), `reports`, `services_aggregator` (only name, firstname, middlename, lastname, gender, birthday, card_number, email, photo_exists, photo, postal_code, city, street, building, phone fields and only with phone or email filter), `client_access_token` (only with filter client)

+ Parameters
    + fields: `name,firstname,middlename,lastname,title,gender,birthday,location,balance,bonus,card_number,phone,email,photo_exists,photo,postal_code,city,street,building,apartment,categories,categories_names,first_visit,first_visit_description,last_visit,last_visit_description,feedback,additional_fields,do_not_send_sms_notification,do_not_send_sms_promotion,do_not_send_email,createDate,deposit_client,referral_source,referral_source_name,status,comment,archive,professional` (array[string], required) - list of fields to return (separated by comma).
    + location: `0799e1bc-86eb-4e0f-82e2-98dfe2cd55f9` (identifier, optional) - get clients who enabled in location
    + phone: `+355 (55) 255 55 55` (string, optional) - get only client(s) with given phone (several phones can be separated by comma)
    + email: `abc@gmail.com` (string, optional) - get only client(s) with given email (several emails can be separated by comma)
    + card_number: `1234F` (string, optional) - get only client(s) with given card number (several card numbers can be separated by comma)
    + name: `John Brown` (string, optional) get only clients with given name (several name can be separated by comma)
    + archive: false (boolean, optional) - get only archived or non archived clients

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d6248d-e5e8-9c40-5296-bb10043eab27",
                    "name": "John Brown",
                    "firstname": "John",
                    "middlename": "",
                    "lastname": "Brown",
                    "title": "",
                    "gender": "male",
                    "birthday": null,
                    "location": "5382f467-a034-440c-bc7f-3dac932f8b99",
                    "balance": 20.0,
                    "bonus": 0,
                    "card_number": [
                        "127219432"
                    ],
                    "phone": [
                        "+1 800 555 12 34"
                    ],
                    "email": [
                        "john@gmail.com"
                    ],
                    "photo_exists": true,
                    "photo": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab991/photo",
                    "postal_code": "",
                    "city": "New York",
                    "street": "",
                    "building": "",
                    "apartment": "",
                    "first_visit": null,
                    "last_visit": null,
                    "feedback": null,
                    "do_not_send_sms_notification": false,
                    "do_not_send_sms_promotion": false,
                    "do_not_send_email": false,
                    "deposit_client": null,
                    "referral_source": null,
                    "referral_source_name": "",
                    "status": "potential",
                    "comment": "VIP status",
                    "archive": false
                },
                {
                    "id": "6b663d27-90f5-4df0-a8c7-5924b9cd1c39",
                    "name": "James Doe",
                    "firstname": "John",
                    "middlename": "",
                    "lastname": "Brown",
                    "title": "",
                    "gender": "male",
                    "birthday": null,
                    "location": "5382f467-a034-440c-bc7f-3dac932f8b99",
                    "balance": 0,
                    "bonus": 0,
                    "card_number": [],
                    "phone": [
                        "+1 800 555 48 15"
                    ],
                    "email": [],
                    "photo_exists": false,
                    "photo": null,
                    "postal_code": "",
                    "city": "New York",
                    "street": "",
                    "building": "",
                    "apartment": "",
                    "first_visit": null,
                    "last_visit": null,
                    "feedback": null,
                    "do_not_send_sms_notification": false,
                    "do_not_send_sms_promotion": false,
                    "do_not_send_email": false,
                    "deposit_client": null,
                    "referral_source": "5382f467-a034-440c-bc7f-3dac9b2c8191",
                    "referral_source_name": "flyers",
                    "status", "potential",
                    "comment": null,
                    "archive": false,
                    "professional": "5382f467-a034-440c-bc7f-3dac9b2c8191"
                }
            ]

### Get client by id [GET /clients/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`online_store` (only name, firstname, middlename, lastname, gender, birthday, card_number, phone, email, comment fields),`reports`,`client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me`)
    + fields: `name,firstname,middlename,lastname,title,gender,birthday,location,balance,bonus,card_number,phone,email,photo_exists,photo,postal_code,city,street,building,apartment,categories,categories_names,first_visit,first_visit_description,last_visit,last_visit_description,feedback,additional_fields,do_not_send_sms_notification,do_not_send_sms_promotion,do_not_send_email,createDate,deposit_client,referral_source,referral_source_name,status,comment,archive,professional` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d6248d-e5e8-9c40-5296-bb10043eab27",
                "name": "John Brown",
                "firstname": "John",
                "middlename": "",
                "lastname": "Brown",
                "title": "",
                "gender": "male",
                "birthday": null,
                "location": "5382f467-a034-440c-bc7f-3dac932f8b99",
                "balance": 20.0,
                "bonus": 0,
                "card_number": [
                    "127219432"
                ],
                "phone": [
                    "+1 800 555 12 34"
                ],
                "email": [
                    "john@gmail.com"
                ],
                "photo_exists": true,
                "photo": "https://api.aihelps.com/v1/images/696320/6b663d27-90f5-4df0-a8c7-5924b9cd1c48/photo",
                "postal_code": "",
                "city": "New York",
                "street": "",
                "building": "",
                "apartment": "",
                "first_visit": null,
                "last_visit": null,
                "feedback": null,
                "do_not_send_sms_notification": false,
                "do_not_send_sms_promotion": false,
                "do_not_send_email": false,
                "deposit_client": null,
                "referral_source": "5382f467-a034-440c-bc7f-3dac9b2c8191",
                "referral_source_name": "flyers",
                "status", "potential",
                "comment": "VIP status",
                "archive": false,
                "professional": "5382f467-a034-440c-bc7f-3dac9b2c8191"
            }

### Create new client [POST /clients{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `online_store` (only name, firstname, middlename, lastname, gender, birthday, card_number, phone, email, comment fields), `services_aggregator` (only firstname, middlename, lastname, gender, birthday, card_number, phone, email, comment fields)

+ Parameters
    + fields: `name,firstname,middlename,lastname,title,gender,birthday,location,balance,bonus,card_number,phone,email,photo_exists,photo,postal_code,city,street,building,apartment,categories,categories_names,first_visit,first_visit_description,last_visit,last_visit_description,feedback,additional_fields,do_not_send_sms_notification,do_not_send_sms_promotion,do_not_send_email,createDate,deposit_client,referral_source,referral_source_name,status,comment,archive,professional` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "firstname": "John",
                "lastname": " Doe",
                "phone": "+1 555 123 45 67",
                "email": "johndoe@gmail.com"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }

### Update client [PUT /clients/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me`)
    + fields: `name,firstname,middlename,lastname,title,gender,birthday,location,balance,bonus,card_number,phone,email,photo_exists,photo,postal_code,city,street,building,apartment,categories,categories_names,first_visit,first_visit_description,last_visit,last_visit_description,feedback,additional_fields,do_not_send_sms_notification,do_not_send_sms_promotion,do_not_send_email,createDate,deposit_client,referral_source,referral_source_name,status,comment,archive,professional` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "phone": [
                    "+1 555 123 45 67",
                    "+1 555 123 45 68"
                ],
                "gender": "male"
            }
            
+ Response 204

### Delete client [DELETE /clients/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier, required) - id of client

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Client categories [/clientcategories]
<a name="client-categories"></a>

Clients can be splitted into categories. This is quite helpful for filtering clients.

+ Attributes
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier) - id of a client category
    + name: `VIP` (string,required) - name of a category. Max length is 200 characters
    + automatic: true (boolean) - defines, if category was assigned automatically, based on rule (condition), that was set in the application. Read only field

### Get all client categories [GET /clientcategories{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `reports`

+ Parameter
    + fields: `name,automatic` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "3286cc03-3f88-4c8e-8d94-0af35b67beaf",
                    "name": "VIP",
                    "automatic": true
                }
            ]

### Get client category by ID [GET /clientcategories/{id}{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `reports`

+ Parameter
    + id: `bf45f298-3073-413a-876f-0f17f45e374c` (identifier, required) - id of a category
    + fields: `name, automatic` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body

            {
                "id": "bf45f298-3073-413a-876f-0f17f45e374c",
                "name": "VIP",
                "automatic": false
            }

### Create client category [POST /clientcategories{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + fields: `name,automatic` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Gold",
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "d8f244e8-f84d-4c21-b674-90135978216d",
                "name": "Gold",
                "automatic": false
            }
  
### Update client category [PUT /clientcategories/{id}{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `eaf52652-3524-4c76-b6d9-0998bfd50fb9` (identifier, required) - id of a client category
    + fields: `name,automatic` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Gold",
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "d8f244e8-f84d-4c21-b674-90135978216d",
                "name": "Gold",
                "automatic": false
            }

### Remove client categories [DELETE /clientcategories/{id}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `6f300bd6-fa07-44a4-aed2-cca3719e52f5` (identifier, required) - id of a client category, that should be removed

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204 

### Update categories for specific client [POST /clients/{id}:recalculateCategories]

This endpoint provide reculculatig dynamic categories for the client, for example 
in case of changing client infomation or his conditions.

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client.
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Update category for all clients [POST /clients/categories/{id}:recalculateClients]

This endpoint provide reculculatig dynamic categories for all clients, 
checking the customer's compliance with this category.
This can be used in case of changing category or clients conditions.  

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of category.
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Update all category for all clients [POST /clients/categories:recalculateClients]

This endpoint provide reculculatig all dynamic categories for all clients, 
checking the customer's compliance with this categories.
This can be used once daily in case of changing category or clients conditions.  

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Client photo [/clients/id/photo]

### Get photo by id [GET /clients/{id}/photo{?width,height,resize,access_token}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `cbaf2626-0f2b-4cae-a6dd-ca7ddaf2a1db` (identifier, required) - client id. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me`)
    + width: 100 (number, optional) - image width needed
    + height: 100 (number, optional) - image height needed
    + resize: `fit` (enum[string], optional) - type of resize for image
        + Default: `fit`
        + Members
            + `fit`
            + `fit_center_transparent`
            + `stretch`
            + `crop`
    + access_token: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (string, optional) - token used for API requests

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (image/jpeg)

    + Body

### Put photo [PUT /clients/{id}/photo]

Set correct Content-Type header and put image in request body.
Providing no request body means deleting image.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `f90a87cb-ef7f-444f-a05f-bf81ec055627` (identifier, required) - client id. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me`)

+ Request (image/jpeg)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Relations between Clients [/clients/relations]

Relation between clients and employees or client and client or employee and employee

+ Attributes
    + id: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - id of client relation
    + from_client: `6e3a2105-dc9d-40ec-8432-7cce3f5e98d5` (identifier) - id of client
    + to_client: `26e3a105-9cdd-40ec-8432-7c3fec5e98d5` (identifier) - id of client
    + relation_type: `Spouse` (enum[string]) - type of relation between client
        + Members
            + `Spouse` - `to_client` is spouse of `from_client`
            + `Child` - `to_client` is Child of `from_client` 
            + `Sib` - `to_client` is Sibling of `from_client`
            + `Friend` - `to_client` is Friend of `from_client`
            + `Collegua` - `to_client` is Collegua of `from_client`

### Get all clients relations [GET /clients/relations{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `from_client,to_client,relation_type` (array[string], required) - list of fields to return (separated by comma).
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d4b305-e198-f02c-2743-cca9390c6d9b",
                    "from_client": "88d3d8a4-0629-b0ce-5798-55163380117b",
                    "to_client": "88d3d8a4-0629-b0ce-5798-55160e7a6556",
                    "relation_type": "Child"
                },
                {
                    "id": "88d4c84b-9562-db0e-25f9-8879620d3872",
                    "from_client": "88d4c84b-59f2-fb62-25f9-88792de3ea4c",
                    "to_client": "88d4c84b-1b9b-3e06-25f9-88796d89a302",
                    "relation_type": "Sib"
                }
            ]
            
### Get client relation by id [GET /clients/relations/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client relation
    + fields: `from_client,to_client,relation_type` (array[string], required) - list of fields to return (separated by comma).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d4b305-e198-f02c-2743-cca9390c6d9b",
                    "from_client": "88d3d8a4-0629-b0ce-5798-55163380117b",
                    "to_client": "88d3d8a4-0629-b0ce-5798-55160e7a6556",
                    "relation_type": "Child"
                }
            ]

## Client history [/clients/id/history]

Client history based on appointments, group appointments and sales. This endpoint helps to receive client history in one unified view (both sales and appointments) for some client.
But any changes should be done to original appointments or sales. History only for last 2 month are available. If you need to get earlier history, you'll have to contact us individually.

+ Attributes
    + date: `2019-01-01T10:00:00.000Z` (datetime) - appointment date (start date)
    + duration: 60 (number) - appointment duration
    + professional: `5d5c28d1-ff9e-43e8-9df7-f69667e22187` (identifier) - professional id 
    + professional_name: `Bob` (string) - professional name
    + professional_photo_exists: false (boolean) - if professional has photo
    + paid: true (boolean) - if appointment was paid (if true, object was created based on one or several sales, i false - based on one appointment or group appointment)
    + items (array) - info about sales, appointment or group_appointment
        + (object)
            + id: `248afaa1-e69a-4548-82ce-aa7f1712d3ec` (identifier) - item (service/product/group/etc) id
            + name: `Premium` (string) - item (service/product/group/etc) name
            + type: `Service` (enum[string]) - type of item
                + Members
                    + Denture
                    + Card
                    + Group
                    + Product
                    + Certificate
            + picture: `7ff77127-7f43-4533-97ff-22c5be1f46ef` (identifier) - item picture id
            + quantity: 1 (number) - item quantity
            + sum: 500 (number) - paid sum (if paid) or price sum if not paid yet
            + sale_id: `6f7c5afc-9b48-44f4-845b-31cdf04ddf8e` (identifier) - if paid, sale id [Sales](#sales) (each item is separate sale object in sales table)
    + client_notified: true (boolean) - if client was notified about appointment (always true if paid)
    + feedback (object) - feedback information (for paid only, if client left feedback)
        + date: `2019-01-01:00:00:00.000Z` (datetime) - feedback create date
        + text: `Cool stylist!` (string) - feedback text
        + ratings: 5 (number) - client rating for employee

### Get history by id [GET /clients/{id}/history{?fields}]

Get client history for specified client id.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - client id. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me/history`)
    + fields: `date,duration,professional,professional_name,professional_photo_exists,paid,items(id,name,type,picture,quantity,sum,sale_id),client_notified,feedback` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d39450-0c81-9ca8-139a-8eb20176600b",
                    "date": "1799-12-31T00:00:00.000Z",
                    "duration": 60,
                    "professional": null,
                    "professional_name": "",
                    "professional_photo_exists": false,
                    "paid": true,
                    "items": [
                        {
                            "id": "88d37eed-6689-d1d3-6bee-ad104a954fc2",
                            "name": "Premium",
                            "type": "Card",
                            "picture": null,
                            "quantity": 1,
                            "sum": 5000,
                            "sale_id": "88d39450-0c81-9ca8-139a-8eb20176600b"
                        }
                    ],
                    "client_notified": true,
                    "feedback": 
                    {
                        "date": "2018-04-28T21:20:11.000Z",
                        "ratings": 5,
                        "text": "well"
                    }
                },
                {
                    "id": "88d395df-bf56-7e8e-5278-6cf9506aaed4",
                    "date": "2016-06-16T16:00:00.000Z",
                    "duration": 35,
                    "professional": "88d37eed-0e73-b347-6bee-ad103615dee1",
                    "professional_name": "John Doe",
                    "professional_photo_exists": false,
                    "paid": true,
                    "items": [
                        {
                            "id": "88d37ef2-5930-80f8-6bee-ad1021c2fd8b",
                            "name": "Epilation",
                            "type": "Service",
                            "picture": null,
                            "quantity": 1,
                            "sum": 0,
                            "sale_id": "88d395df-bf56-7e8e-5278-6cf9506aaed4"
                        }
                    ],
                    "client_notified": true,
                    "feedback": null
                },
                {
                    "id": "88d39eaf-dc0e-ac59-1be5-be5c762ccc88",
                    "date": "2016-06-27T20:00:00.000Z",
                    "duration": 60,
                    "professional": "88d37eeb-cb47-bedb-6bee-ad1048f62926",
                    "professional_name": "",
                    "professional_photo_exists": false,
                    "paid": false,
                    "items": [],
                    "client_notified": true,
                    "feedback": null
                }
            ]

## Client feedbacks [/clients/feedbacks]

Represents clients feedback details.

+ Attributes
    + firstname: `Bob` (string) - client first name
    + public: true (boolean) - if feedback is available for all (otherwise - hidden by salon manager)
        Default: true
    + date: `2019-01-01:00:00:00.000Z` (datetime, required) - feedback create date
    + text: `Cool stylist!` (string) - feedback text. Max length is 20000 characters
    + ratings (object) - client rating for each question
        + id: `98402030-6e00-4235-85f4-9a5799d208f8` (identifier) - question id (list of questions can be find in `/options`)
        + value: 5 (number) - client rating (from 1 to 5)

### Get all feedbacks [GET /clients/feedbacks{?fields,public}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`,`client_access_token`

+ Parameters
    + fields: `firstname,public,date,text,ratings` (array[string], required) - list of fields to return (separated by comma)
    + public: true (boolean, optional) - return only public feedbacks

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "9802426c-92a8-41f0-976b-5a643e3df6f2",
                    "firstname": "John",
                    "public": true,
                    "date": "2019-01-01T10:00:00.000Z",
                    "text": "cool professional",
                    "ratings":
                    {
                        "a860ecab-e030-48c8-aeba-4c5a419db8c5": 2,
                        "bd603053-a530-4cd0-b67e-db84192a8ea9": 5
                    }
                },
                {
                    "id": "4a87cc85-a4a2-40fa-930c-c0833ef24251",
                    "firstname": "Bob",
                    "public": true,
                    "date": "2019-01-01T10:00:00.000Z",
                    "text": "cool professional",
                    "ratings":
                    {
                        "c680c43d-07ad-4050-959d-279a2c1ba272": 2,
                        "32006adc-a269-43b8-83e7-6fb5ca5bbef3": 5
                    }
                }
            ]
            
### Get feedback by client id [GET /clients/{id}/feedbacks{?fields,public}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`,`client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me/feedbacks`)
    + fields: `firstname,public,date,text,ratings` (array[string], required) - list of fields to return (separated by comma)
    + public: true (boolean, optional) - get only public/non-public feedbacks

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "4a87cc85-a4a2-40fa-930c-c0833ef24253",
                "firstname": "John",
                "public": true,
                "date": "2019-01-01T10:00:00.000Z",
                "text": "cool professional",
                "ratings":
                {
                    "a860ecab-e030-48c8-aeba-4c5a419db8c5": 2,
                    "bd603053-a530-4cd0-b67e-db84192a8ea9": 5
                }
            }

### Update feedback [PUT /clients/{id}/feedbacks{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me/feedbacks`)
    + fields: `firstname,public,date,text,ratings` (array[string], optional) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
        
    + Body

            {
                "public": false,
                "text": "bad"
            }

+ Response 204

### Delete feedback [DELETE /clients/{id}/feedbacks]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`client_access_token` (but id field should match client id, `me` can be used instead of id)

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of client. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/clients/me/feedbacks`)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Referral source [/referralsources]
<a name="referralsources"></a>

This article provides information about referral sources, from which location can attract new clients

+ Attributes
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier) - referral source id
    + name: `advertising` (string,required) - referral source name. Max length is 15000 characters
    + archive: false (boolean) - if referral source was archived and can't be used in create new client
        + Default: `false`

### Get all referral sources [GET /referralsources{?fields,archive}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `client_module`, `reports`

+ Parameter
    + fields: `name,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archive or none archive referral source

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "3286cc03-3f88-4c8e-8d94-0af35b67beaf",
                    "name": "advertising",
                    "archive": false
                },
                {
                    "id": "3286cc03-3f88-4c8e-8d94-0af35b67b1a1",
                    "name": "flyers",
                    "archive": false
                }
            ]

### Get referral source [GET /referralsources/{id}{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `client_module`, `reports`

+ Parameter
    + id: `bf45f298-3073-413a-876f-0f17f45e374c` (identifier, required) - id of referral source
    + fields: `name, archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body

            {
                "id": "3286cc03-3f88-4c8e-8d94-0af35b67b1a1",
                "name": "flyers",
                "archive": false
            }

### Create referral source [POST /referralsources{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + fields: `name,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "flyers"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "d8f244e8-f84d-4c21-b674-90135978216d",
                "name": "flyers"
            }
  
### Update referral source [PUT /referralsources/{id}{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `eaf52652-3524-4c76-b6d9-0998bfd50fb9` (identifier, required) - id of a client category
    + fields: `name,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "flyers"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "d8f244e8-f84d-4c21-b674-90135978216d",
                "name": "flyers"
            }

### Remove referral source [DELETE /referralsources/{id}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameters
    + id: `6f300bd6-fa07-44a4-aed2-cca3719e52f5` (identifier, required) - id of referral source, that should be removed

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204 

# Group Employees

Employees are all persons working in beauty salon/fitness gym: professionals, administrators, directors, cleaning managers, client managers etc.
Some of them can login to program, some not. All employees are grouped into positions, more details about positions, read subsection Position.

## Employee [/employees]
<a name="employees"></a>
+ Attributes
    + name: `John Grace Doe` (string) - client full name (read only)
    + firstname: `John` (string) - first name of a person. Max length is 500 characters
    + middlename:  `Grace` (string) - middle name of a person. Max length is 500 characters
    + lastname: `Doe` (string, optional) - last name of a person. Max length is 500 characters
    + title: `good client` (string, optional) - title of a person. Max length is 20 characters
    + phone: `1 (555) 555 55 55` (array[string]) - employee phone. Employee can have several phones
    + email: `bob@yahoo.com` (array[string]) - employee email. Employee can have several emails
    + gender: `male` (enum[string]) - employee gender
        + Default: `male`
        + Members
            + `female`
    + photo_exists: true (boolean) - if employee has a photo
    + photo: `https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab93b/photo` (string) - URL for getting a photo (readonly)
    + positions: '0b646349-e7be-4179-97bd-c7155caab990' (array[identifier]) - position ids
    + position_names: 'Receptionist' (array[string]) - position names
    + roles: 'professional' (enum[string]) - unique role names
        + Members
            + owner
            + administrator
            + professional
            + technicalStaff
            + assistant
            + professionalReplace
            + clientManager
            + cashier
    + permissions: `PROFESSIONALS.EDIT_APPOINTMENTS_AND_CLIENTS` (array[enum]) - list of permissions for employee. Based on `positions` permissions
        + Members
            + `RIGHT_1`
            + `RIGHT_2`
            + `PROFESSIONALS.EDIT_APPOINTMENTS_AND_CLIENTS`
    + language: 'en' (language) - custom language for certain employee. Empty string means you should use `language` from settings, common for all employees.
    + archive: false (boolean) - if employee was fired and his information now in archive
        + Default: `false`
    + public: true (boolean) - if employee is available for online booking
        Default: `true`
    + address (address) - employee full address
    + schedules (array[object], optional) - all info about employee schedule in different locations
        + `location`: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier) - location id, see [Locations](#locations)
        + `schedule`: `e661288c-3457-429a-ab06-5071ca1708c7` (identifier) - predefined schedule id
        + `start_date`: `2018-08-09T00:00:00.000Z` (datetime) - schedule start date
    + all_locations: true (boolean) - if employee can access all locations information if true
    + default_appointment_duration: 120 (number) - typical duration for employee appointment for last month, if less than 10 appointments - 60 minutes is used (read only). Typical duration means one of [15, 30, 60, 90, 120, 180] minutes that covers 80% of last month durations.
    + feedback_count: 3 (number) - count of public feedbacks
    + average_feedback_rating: 4.25 (number) - average rating of public feedbacks
    + prepaymentRequired: false (boolean) - if booking for this employee requires prepayment
    + comments: `Employee1` (string) - employee description with html tags for font styling. Max length is 65536 characters. Read only
    + commentsPlainText: `Employee1` (string) - employee description as plain text

### Get all employees [GET /employees{?fields,location,position,role,service,free_time,free_time_professionals,free_time_skip_appointments,archive,public,client_gender,archive_service,name}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module` (available fields: photo_exists,photo,gender,default_appointment_duration,name,name_parts,sex,positions,position_names,roles,archive,public,address,schedules,services), `reports`, `services_aggregator` (only photo_exists, gender, default_appointment_duration, name, positions, position_names, roles, archive, public, address, schedules, services, archive)

+ Parameters
    + fields: `name,firstname,middlename,lastname,title,phone,email,gender,photo_exists,photo,positions,position_names,roles,permissions,archive,public,address,schedules,all_locations,feedback_count,average_feedback_rating,prepaymentRequired,comments,commentsPlainText` (array[string], required) - list of fields to return (separated by comma)
    + location: `73a386aa-dfdd-4bba-be50-084c4104c599` (array[identifier], optional) - get only employees which works in given location (several locations can be separated by comma)
    + position: `1bd1288c-3457-429a-ab06-5071ca1704c3` (array[identifier], optional) - get only employees with given position (several positions can be separated by comma)
    + role: `professional` (array[string], optional) - get only employees of given role (several roles can be separated by comma)
    + service: `aad32c8c-1cb7-429a-ab06-5071c4debca3` (array[identifier], optional) - get only employees that can provide all given services (several services can be separated by comma)
    + `free_time`: `2018-11-29T19:30:00.000Z..2018-11-29T20:30:00.000Z` (string, optional) - get only employees who will be free at specified time range. If `location` filter is set, looks only at specified location (otherwise search in all).
    + `free_time_professionals`: `71a98638-ef51-497a-88ae-8b4f1efaaa8b` (array[identifier], optional) - if free_time filter used, specifies professionals (one or many, several appointments can be separated by comma), for which free time will be calculated (if not set - for all)
    + `free_time_skip_appointments`: `bc4311b8-329c-40a9-a784-04cb5f7fe1aa` (array[identifier], optional) - if free_time filter used, specifies which appointments should be skipped checking (several appointments can be separated by comma) - skipped appointments will be treated as free time
    + archive: false (boolean, optional) - get only archive or none archive employees
    + public: false (boolean, optional) - get only public or non public employees
    + client_gender: `male` (enum[string], optional) - get only professionals who can provide services for given gender
        + Members
            + male
            + female
    + `archive_service`: false (boolean, optional) - if `client_gender` filter used, check only archive/non-archive services for professionals
    + name: `John Doe` (string, optional) get only employees with given name (several name can be separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id":"88d4d8c0-3830-5b07-7bfe-b80f0650b9f3",
                    "name": "John Brown",
                    "firstname": "John",
                    "middlename": "",
                    "lastname": "Brown",
                    "title": "",
                    "phone": [ "+1 (800) 555 55 55" ],
                    "email": [ "bob@yahoo.com" ],
                    "gender":"male",
                    "photo_exists":true,
                    "photo": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab923/photo",
                    "positions":
                    [
                        "88d354c4-6ac5-a672-6996-67b22cd78ca6",
                        "88d4d8c4-512f-7d55-7bfe-b80f413625dc"
                    ],
                    "roles": "professional",
                    "permissions": [],
                    "archive": false,
                    "public": true,
                    "services": [],
                    "schedules": 
                    [
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "schedule": null,
                            "start_day": "1800-01-01T00:00:00.000Z"
                        },
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "schedule": null,
                            "start_day": "1799-12-31T00:00:00.000Z"
                        }
                    ],
                    "address": 
                    {
                        "city": "New York",
                        "street": "17 Avenue",
                        "building": "17",
                        "apartment": "",
                        "postal_code": "222204"
                    },
                    "feedback_count": 5,
                    "average_feedback_rating": 4.5,
                    "prepaymentRequired": false,
                    "comments": "Employee1",
                    "commentsPlainText": "Employee1"
                },
                {
                    "id": "72fcfa1d-65a7-404c-8c41-7449265ae489",
                    "name": "John Doe",
                    "firstname": "John",
                    "middlename": "",
                    "lastname": "Doe",
                    "title": "",
                    "phone": [ "+1 (800) 555 55 55" ],
                    "email": [ "John@yahoo.com" ],
                    "gender":"male",
                    "photo_exists":true,
                    "photo": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155cacb21/photo",               
                    "positions":
                    [
                        "ff13c4c4-6ac5-a672-6996-67bc2bd7aca1",
                        "88d4d8c4-512f-7d55-7bfe-b80f4136c5aa"
                    ],
                    "roles": "professional",
                    "permissions": [],
                    "archive": false,
                    "public": true,
                    "services": [],
                    "schedules": 
                    [
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "schedule": null,
                            "start_day": "1800-01-01T00:00:00.000Z"
                        }
                    ],
                    "address": 
                    {
                        "city": "Los Angelos",
                        "street": "West lane",
                        "building": "11",
                        "apartment": "",
                        "postal_code": "03065"
                    },
                    "feedback_count": 5,
                    "average_feedback_rating": 4,
                    "prepaymentRequired": false,
                    "comments": "Employee2",
                    "commentsPlainText": "Employee2"
                }
            ]

### Get employee by id [GET /employees/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module` (available fields: photo_exists,photo,gender,default_appointment_duration,name,name_parts,sex,positions,position_names,roles,archive,public,address,schedules,services), `reports`, `services_aggregator` (only photo_exists,g ender, default_appointment_duration, name, positions, position_names, roles, archive, public, address, schedules, services, archive)

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - employee id (`me` can be used for employee token)
    + fields: `name,firstname,middlename,lastname,title,phone,email,gender,photo_exists,positions,position_names,roles,archive,public,address,schedules,all_locations,feedback_count,average_feedback_rating,photo,prepaymentRequired,comments,commentsPlainText` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "432c41bf-0cd7-4f13-83dd-a0c26ce29143",
                "name": "John Brown",
                "firstname": "John",
                "middlename": "",
                "lastname": "Brown",
                "title": "",
                "phone": [ "+1 (800) 555 55 55" ],
                "email": [ "Doe@yahoo.com" ],
                "gender":"male",
                "photo_exists":true,
                "photo": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab91a/photo",        
                "positions":
                [
                    "ff13c4c4-6ac5-a672-6996-67bc2bd7aca1",
                    "88d4d8c4-512f-7d55-7bfe-b80f4136c5aa"
                ],
                "roles": "professional",
                "permissions": [],
                "archive": false,
                "public": true,
                "services": [],
                "schedules": 
                [
                    {
                        "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                        "schedule": null,
                        "start_day": "1800-01-01T00:00:00.000Z"
                    }
                ],
                "address": 
                {
                    "city": "New York",
                    "street": "17 Avenue",
                    "building": "17",
                    "apartment": "",
                    "postal_code": "222204"
                },
                "feedback_count": 5,
                "average_feedback_rating": 4.5,
                "prepaymentRequired": false,
                "comments": "Employee2",
                "commentsPlainText": "Employee2"
            }

### Create new employee [POST /employees{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,firstname,middlename,lastname,title,phone,email,gender,photo_exists,photo,positions,position_names,roles,archive,public,address,schedules,all_locations,feedback_count,average_feedback_rating,prepaymentRequired,comments,commentsPlainText` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "firstname": "John",
                "middlename": "",
                "lastname": "Brown",
                "phone": [ "+1 555 1234567" ],
                "email": "bobcorn@gmail.com"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "475a20e9-23ef-40ce-b2c4-c7dbcf230ebb"
            }

### Update employee [PUT /employees/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - employee id (`me` can be used for employee token)
    + fields: `name,firstname,middlename,lastname,title,phone,email,gender,photo_exists,photo,positions,position_names,roles,archive,public,address,schedules,all_locations,feedback_count,average_feedback_rating,prepaymentRequired,comments,commentsPlainText` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "phone":
                [
                    "+1 555 1234567",
                    "+1 555 1234568"
                ],
                "gender": "male",
                "archive": false
            }
            
+ Response 204

### Delete employee [DELETE /employees/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - id of employee

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Employee photo [/employees/id/photo]

### Get photo by id [GET /employees/{id}/photo{?width,height,resize,access_token}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `services_aggregator`

+ Parameters
    + id: `de2f90bc-2f19-4f2c-94d7-a735a07c1ae3` (identifier, required) - employee id. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/employees/me`)
    + width: 100 (number, optional) - image width needed
    + height: 100 (number, optional) - image height needed
    + resize: `fit` (enum[string], optional) - type of resize for image
        + Default: `fit`
        + Members
            + `fit`
            + `fit_center_transparent`
            + `stretch`
            + `crop`
    + access_token: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (string, optional) - token used for API requests

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (image/jpeg)

    + Body

### Put photo [PUT /employees/{id}/photo]

Set correct Content-Type header and put image in request body.
Providing no request body means deleting image.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `8f423935-ff1b-4a61-b0a9-8533ce5e683a` (identifier, required) - employee id. If client access token is used, only id of that client can be set here (string identifier `me` can be used instead of id: `/employees/me`) 

+ Request (image/jpeg)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Additional methods [/employees/methods]

### Choose professional from proposed professionals list [GET /employees/pick_professional{?date,professionals}]

In some cases there is no difference, which one of selected list of professionals will provide a service. In that case you can let API to select best-matching employee.
Current strategy is to select employee which is least busy this day (so API tries to make all professionals busy same label).

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `services_aggregator`

+ Parameters
    + date: `2019-09-09` (datetime, required) - date to choose employee
    + professionals: `8f423935-ff1b-4a61-b0a9-8533ce5e683a` (array[identifier], required) - employee ids to select from (several employees are separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "professional": "8f423935-ff1b-4a61-b0a9-8533ce5e683a"
            }

## Position [/positions]

All employees are divided into several categories:

1. owners - owner, director and other persons who have unlimited or big access to program possibilities
2. administrators - persons responsible for managing appointments and tickets
3. professionals - persons who are making services itself
4. assistants - persons, who helps professionals to make services
5. temporary professionals - persons who are not working in salon/gym, but can be invited sometimes to make some services/group lessons
6. client managers - persons responsible for calling potential and active clients and receiving their calls
7. cashiers - separate persons in big salons/gyms responsible only for receiving cash from clients
8. technical stuff - cleaning managers, engineers and other employees (one and only type of stuff who can't login program)

Each category has it's own meaning, details for categories will be described later.

There can be more than one group of persons in each category. For example, professionals can be: stylists, cosmetologists, manicurists, etc.;
administrators can be grouped as top-administrators and normal administrators. Each such group is named a `position`.

+ Attributes
    + name: "Stylists" (string,required) - position name. Max length is 500 characters
    + role: professional (enum[string]) - one of eight predefined employee categories. Default value is administrator
    + permissions: `PROFESSIONALS.EDIT_APPOINTMENTS_AND_CLIENTS` (array[enum]) - list of permissions, allowed to this position employees
        + Members
            + `RIGHT_1`
            + `RIGHT_2`
            + `PROFESSIONALS.EDIT_APPOINTMENTS_AND_CLIENTS`
    + parent: `04c5daec-2c11-4ce9-b00d-03a03d50356f` (identifier) - id of parent category of the position

### Get all positions [GET /positions{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only name field), `reports`

+ Parameters
    + fields: `name,role,permissions,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "04c5daec-2c11-4ce9-b00d-03a03d50356f",
                    "name": "Stylists",
                    "role": "professional",
                    "rights":
                    [
                        "",
                        ""
                    ],
                    "id": "e28ebfd0-f847-4c3f-b2a1-07f4257b0764"
                },
                {
                    "id": "e28ebfd0-f847-4c3f-b2a1-07f4257b0764"
                }
            ]

### Get position by id [GET /positions/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only name field), `reports`

+ Parameters
    + id: `04c5daec-2c11-4ce9-b00d-03a03d50356f` (identifier, required) - id of position
    + fields: `name,role,permissions,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "432c41bf-0cd7-4f13-83dd-a0c26ce29143",
            }

### Create new position [POST /positions{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,role,permissions,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Stylists",
                "role": "professional"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "f1c7f635-dbf7-4e21-a970-4b1fd7514604"
            }

### Update position [PUT /positions/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `04c5daec-2c11-4ce9-b00d-03a03d50356f` (identifier, required) - id of position
    + fields: `name,role,permissions,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Top stylists"
            }
            
+ Response 204

### Delete position [DELETE /positions/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `04c5daec-2c11-4ce9-b00d-03a03d50356f` (identifier, required) - id of position

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Invitation codes [/employees/code]

Running application on a new system requires setting up connection to the database.
To simplify this process, invitation code can be used.

### Send an invitation to an employee [POST /employees/{id}/code]

Generate an invitation and send it to an employee's email.

The e-mail message will contain url, which should be used to download and run the setup. Invitation code will be embedded into the setup.

Generated setup file will be valid for 14 days and can be used only once.

Method returns email on which invitation was sent (first email, if employee has more than one email).

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - id of an employee
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "receiver_email": "example@employeeemail.com"
            }

### Validate invitation code [GET /employees/codes/{code}]

Validate an invitation code. 

When running setup for the first time on a new system, application with embedded invitation code will sent it to the server to validate.

If code is valid, server will send database code and id of an employee, who was invited.

**Authorization:** none

+ Parameters
    + code: `FE6A2BCF` (string, required) - invitation code to validate

+ Response 200 (application/json)

    + Body

            {
                "database_code": "123456",
                "employee_id": "432c41bf-0cd7-4f13-83dd-a0c26ce29143"
            }

## Predefined schedule [/predefinedSchedules]
<a name="predefinedSchedules"></a>

Predefined schedules are designed to create templates for employees workdays.

+ Attributes
    + name: `Everyday` (string,required) - predefined schedule name. Max length is 100 characters.
    + type: `nAfterM` (enum[string]) - predefined schedule type.
        + Members
            + nAfterM
            + evenOdd
            + weekdays
    + byWorkshifts: `true` (boolean,required) - indicates whether the schedule is shift or not.
    + workDaysCount: 2 (number) - number of work days for `N after M` schedule type.
    + freeDaysCount: 2 (number) - number of free days for `N after M` schedule type.
    + workStart: `09:00` (string) - start work time. Required in 24h format.
    + workEnd: `18:00` (string) - end work time. Required in 24h format.
    + workShift: 1 (number) - indicates which shift is working.
    + oddActive: `true` (boolean) - indicates whether working days on odd days or not.
    + evenActive: `true` (boolean) - indicates whether working days on even days or not.
    + oddStart: `09:00` (string) - start work time on odd days. Required in 24h format.
    + oddEnd: `18:00` (string) - end work time on odd days. Required in 24h format.
    + oddShift: 1 (number) - indicates which shift is working on odd days.
    + evenStart: `09:00` (string) - start work time on even days. Required in 24h format.
    + evenEnd: `18:00` (string) - end work time on even days. Required in 24h format.
    + evenShift: 1 (number) - indicates which shift is working on even days.
    + weekSchedule (array) - daily schedule
        + (object)
            + weekDay: `monday` (string) - weekday name. Available names : `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, `sunday`
            + weekDay: `monday` (enum[string]) - weekday name.
                + Members
                    + monday
                    + tuesday
                    + wednesday
                    + thursday
                    + friday
                    + saturday
                    + sunday
            + timeFrom: `09:00` (string) - start work time. Required in 24h format. Can be the same or null with `timeTo` if the day is not working.
            + timeTo: `18:00` (string) - end work time. Required in 24h format. Can be the same or null with `timeFrom` if the day is not working.
    + weekShifts: 1 (array) - indicates which shift is working.
        + (object)
            + weekDay: `monday` (enum[string]) - weekday name.
                + Members
                    + monday
                    + tuesday
                    + wednesday
                    + thursday
                    + friday
                    + saturday
                    + sunday
            + dayShift: 1 (number) - indicates which shift is working on each day.

### Get all predefined schedules [GET /predefinedSchedules/{?fields}]

**Authorization:** `Employee`

**Scope:**  `full`

+ Parameter
    + fields: `name,type,byWorkshifts,workDaysCount,freeDaysCount,workStart,workEnd,workShift,oddActive,evenActive,oddStart,oddEnd,oddShift,evenStart,evenEnd,evenShift,weekSchedule,weekShifts` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body
    
            [
                {
                    "id": "88d896df-3918-481e-67b4-7a05342314b1",
                    "name": "test",
                    "type": "weekdays",
                    "byWorkshifts": false,
                    "workDaysCount": 1,
                    "freeDaysCount": 1,
                    "workStart": "",
                    "workEnd": "",
                    "workShift": 0,
                    "oddActive": false,
                    "evenActive": false,
                    "oddStart": "",
                    "oddEnd": "",
                    "oddShift": 0,
                    "evenStart": "",
                    "evenEnd": "",
                    "evenShift": 0,
                    "weekSchedule": [
                        {
                            "weekDay": "monday",
                            "timeFrom": "09:00",
                            "timeTo": "18:00"
                        },
                        {
                            "weekDay": "tuesday",
                            "timeFrom": "09:00",
                            "timeTo": "18:00"
                        },
                        {
                            "weekDay": "wednesday",
                            "timeFrom": "09:00",
                            "timeTo": "18:00"
                        },
                        {
                            "weekDay": "thursday",
                            "timeFrom": "09:00",
                            "timeTo": "18:00"
                        },
                        {
                            "weekDay": "friday",
                            "timeFrom": "09:00",
                            "timeTo": "18:00"
                        },
                        {
                            "weekDay": "saturday",
                            "timeFrom": "09:00",
                            "timeTo": "09:00"
                        },
                        {
                            "weekDay": "sunday",
                            "timeFrom": "09:00",
                            "timeTo": "09:00"
                        }
                    ],
                    "weekShifts": [
                        {
                            "weekDay": "monday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "tuesday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "wednesday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "thursday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "friday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "saturday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "sunday",
                            "dayShift": 0
                        }
                    ]
                },
                {
                    "id": "88d83a01-d791-95f8-2a0a-8b443dd6ebb3",
                    "name": "Eveeryday",
                    "type": "weekdays",
                    "byWorkshifts": false,
                    "workDaysCount": 1,
                    "freeDaysCount": 1,
                    "workStart": "",
                    "workEnd": "",
                    "workShift": 0,
                    "oddActive": false,
                    "evenActive": false,
                    "oddStart": "",
                    "oddEnd": "",
                    "oddShift": 0,
                    "evenStart": "",
                    "evenEnd": "",
                    "evenShift": 0,
                    "weekSchedule": [
                        {
                            "weekDay": "monday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "tuesday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "wednesday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "thursday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "friday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "saturday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        },
                        {
                            "weekDay": "sunday",
                            "timeFrom": "00:00",
                            "timeTo": "00:00"
                        }
                    ],
                    "weekShifts": [
                        {
                            "weekDay": "monday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "tuesday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "wednesday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "thursday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "friday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "saturday",
                            "dayShift": 0
                        },
                        {
                            "weekDay": "sunday",
                            "dayShift": 0
                        }
                    ]
                }
            ]


### Get predefined schedule by id [GET /predefinedSchedules/{id}{?fields}]

**Authorization:** `Employee`

**Scope:**  `full`

+ Parameter
    + id: `88d83a01-d791-95f8-2a0a-8b443dd6ebb3` (identifier, required) - id of resource
    + fields: `name,type,byWorkshifts,workDaysCount,freeDaysCount,workStart,workEnd,workShift,oddActive,evenActive,oddStart,oddEnd,oddShift,evenStart,evenEnd,evenShift,weekSchedule,weekShifts` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            {
                "id": "88d83a01-d791-95f8-2a0a-8b443dd6ebb3",
                "name": "Everyday",
                "type": "weekdays",
                "byWorkshifts": false,
                "workDaysCount": 1,
                "freeDaysCount": 1,
                "workStart": "",
                "workEnd": "",
                "workShift": 0,
                "oddActive": false,
                "evenActive": false,
                "oddStart": "",
                "oddEnd": "",
                "oddShift": 0,
                "evenStart": "",
                "evenEnd": "",
                "evenShift": 0,
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "friday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "saturday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    }
                ],
                "weekShifts": [
                    {
                        "weekDay": "monday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "tuesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "wednesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "thursday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "friday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "saturday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "sunday",
                        "dayShift": 0
                    }
                ]
            }


### Create new predefined schedule [POST /predefinedSchedule{?fields}]

**Authorization:** `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,type,byWorkshifts,workDaysCount,freeDaysCount,workStart,workEnd,workShift,oddActive,evenActive,oddStart,oddEnd,oddShift,evenStart,evenEnd,evenShift,weekSchedule,weekShifts` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "name": "two by two",
                "type": "nAfterM",
                "byWorkshifts": false,
                "workDaysCount": 2,
                "freeDaysCount": 2,
                "workStart": "09:00",
                "workEnd": "18:00",
                "workShift": 0,
                "oddActive": false,
                "evenActive": false,
                "oddStart": "",
                "oddEnd": "",
                "oddShift": 0,
                "evenStart": "",
                "evenEnd": "",
                "evenShift": 0,
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "friday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "saturday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    }
                ],
                "weekShifts": [
                    {
                        "weekDay": "monday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "tuesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "wednesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "thursday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "friday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "saturday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "sunday",
                        "dayShift": 0
                    }
                ]
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "88d89784-6cc2-7e6a-6668-ee372dd3d84b",
                "name": "two by two",
                "type": "nAfterM",
                "byWorkshifts": false,
                "workDaysCount": 2,
                "freeDaysCount": 2,
                "workStart": "09:00",
                "workEnd": "18:00",
                "workShift": 0,
                "oddActive": false,
                "evenActive": false,
                "oddStart": "08:00",
                "oddEnd": "21:00",
                "oddShift": 0,
                "evenStart": "08:00",
                "evenEnd": "21:00",
                "evenShift": 0,
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "friday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "saturday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    }
                ],
                "weekShifts": [
                    {
                        "weekDay": "monday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "tuesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "wednesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "thursday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "friday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "saturday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "sunday",
                        "dayShift": 0
                    }
                ]
            }

### Update predefined schedule [PUT /predefinedSchedules/{id}{?fields}]

**Authorization:** `Employee`

**Scope:** `full`

+ Parameters
    + id: `88d89784-6cc2-7e6a-6668-ee372dd3d84b` (identifier, required) - id resource
    + fields: `name,type,byWorkshifts,workDaysCount,freeDaysCount,workStart,workEnd,workShift,oddActive,evenActive,oddStart,oddEnd,oddShift,evenStart,evenEnd,evenShift,weekSchedule,weekShifts` (array[string], required) - list of fields to return (separated by comma).
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "two by two 2",
                "type": "nAfterM",
                "byWorkshifts": false,
                "workDaysCount": 2,
                "freeDaysCount": 2,
                "workStart": "09:00",
                "workEnd": "18:00",
                "workShift": 0,
                "oddActive": false,
                "evenActive": false,
                "oddStart": "08:00",
                "oddEnd": "21:00",
                "oddShift": 0,
                "evenStart": "08:00",
                "evenEnd": "21:00",
                "evenShift": 0,
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "friday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "saturday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    }
                ],
                "weekShifts": [
                    {
                        "weekDay": "monday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "tuesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "wednesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "thursday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "friday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "saturday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "sunday",
                        "dayShift": 0
                    }
                ]
            }
            
+ Response 200

    + Body

            {
                "id": "88d89784-6cc2-7e6a-6668-ee372dd3d84b",
                "name": "two by two 2",
                "type": "nAfterM",
                "byWorkshifts": false,
                "workDaysCount": 2,
                "freeDaysCount": 2,
                "workStart": "09:00",
                "workEnd": "18:00",
                "workShift": 0,
                "oddActive": false,
                "evenActive": false,
                "oddStart": "08:00",
                "oddEnd": "21:00",
                "oddShift": 0,
                "evenStart": "08:00",
                "evenEnd": "21:00",
                "evenShift": 0,
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "friday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "saturday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "00:00",
                        "timeTo": "00:00"
                    }
                ],
                "weekShifts": [
                    {
                        "weekDay": "monday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "tuesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "wednesday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "thursday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "friday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "saturday",
                        "dayShift": 0
                    },
                    {
                        "weekDay": "sunday",
                        "dayShift": 0
                    }
                ]
            }


### Delete predefined schedule [DELETE /predefinedSchedules/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `88d89784-6cc2-7e6a-6668-ee372dd3d84b` (identifier, required) - id  resource

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get predefined schedule Name by context [GET /predefinedSchedules:proposeName]

This endpoint get predefined schedule Name by context. Headers Content-Language is required. 

**Authorization:** `Employee`

**Scope:**  `full`

+ Request 

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c
            Content-Language: en

    + Body

            {
                "type": "nAfterM",
                "byWorkshifts": false,
                "workDaysCount": 1,
                "freeDaysCount": 1,
                "workStart": "00:00",
                "workEnd": "00:01"
            },
            {
                "type": "evenOdd",
                "oddActive": true,
                "evenActive": true,
                "oddStart": "00:10",
                "oddEnd": "00:20",
                "evenStart": "01:00",
                "evenEnd": "02:00"
            },
            {
                "type": "weekdays",
                "weekSchedule": [
                    {
                        "weekDay": "monday",
                        "timeFrom": "10:00",
                        "timeTo": "12:00"
                    },
                    {
                        "weekDay": "tuesday",
                        "timeFrom": "10:00",
                        "timeTo": "12:00"
                    },
                    {
                        "weekDay": "wednesday",
                        "timeFrom": "10:00",
                        "timeTo": "12:00"
                    },
                    {
                        "weekDay": "thursday",
                        "timeFrom": "10:00",
                        "timeTo": "14:00"
                    },
                    {
                        "weekDay": "sunday",
                        "timeFrom": "11:00",
                        "timeTo": "12:00"
                    }
                ]
            }

+ Response 200 (application/json)

    + Body

            {            
                "name": "Mo 09:00-18:00, We-Th 09:00-18:00, Sa 09:00-18:00"           
            }

## Free Time [/employees/free_time]

Get free time for employees for given date or time period.

### Get free time [GET /employees/free_time{?step,duration,professionals,from,to,location,services,skip_appointments,skip_sales,skip_group_lessons,exclude,add_now_time,nearest_day_only,client_gender,public_employees,gaps_mode,gaps_positions}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `services_aggregator`

+ Parameter
    + from: `2019-01-01T10:00:00.000Z` (datetime, required) - start date to get free time
    + to: `2019-01-03T10:00:00.000Z` (datetime, required) - end date to get free time
    + duration: 120 (number, required) - number of minutes for planned appointment
    + step: `30m` (enum[string], optional) - time steps in minutes which will be checked and returned if free time. Еach returned step for professional is free time range {returned time;returned time + duration} (only if it is 100% free - even 1 minute of a previous appointment and the entire slot for 3 hours will be busy and will not be returned). If step set `auto` then step is selected automatically depending on provided or calculated `duration` (`step is selected between 15, 30 and 60 minutes depending on duration`).
        + Members
            + 5m
            + 10m
            + 15m
            + 20m
            + 30m
            + 60m
            + 90m
            + 120m
            + 150m
            + 180m
            + auto
    + professionals: `88d4b305-e198-f02c-2743-cca9390c631c` (array[string], optional) - get free time only for given professionals (several professionals are separated by comma) 
    + location: `a7dd57e5-c495-46d4-b54d-753061d355a0` (array[identifier], optional) - get free time only for given location
    + services: `e9968ac9-528c-4fd9-9af8-ef7023420d0a` (array[identifier], optional) - get free time for professionals that can handle all given services (any professional can handle all given services). Also checks if resources are available for given services.
    + skip_appointments: `c8258b28-4499-4d9c-a48d-d14e9c5ac622` (array[identifier], optional) - consider time used by given appointments as free (several appointments are separated by comma)
    + skip_sales: `ef1d31ae-3970-441c-ad91-4aa4b8ec20b4` (array[identifier], optional) - consider time used by given sales as free (several sales are separated by comma)
    + skip_group_lessons: `f80000fe-207e-44ed-b684-effa83430834` (array[identifier], optional) - consider time used by given group lessons as free (several group lessons are separated by comma)
    + exclude: `2019-01-01T10:00:00.000Z..2019-01-02T10:00:00.000Z` (array[rangedate], optional) - consider provided time ranges as busy (several time ranges are separated by comma)
    + add_now_time: 20 (number, optional) - number of minutes from now that should be considered as busy (do not book for upcoming N minutes)
        + Default: 0
    + nearest_day_only: true (boolean, optional) - if true then take free time for nearest day (one day), if false then for all time from attribute `from` to attribute `to`
    + client_gender: `male` (enum[string], optional) - get only professionals who can provide services of given sex (works only if specific services not provided via `services` filter)
        + Members
            + male
            + female
    + public_employees: true (boolean, optional) - get only professionals for which `public` (employee attribute) is either `true` or `false`. `public` true means client can book professional by himself.
        + Default: none
    + `gaps_mode`: `none` (enum[string], optional) - mode used to optimize gaps in professional time (more aggressive node returns less time slots)
        + Default: `none`
        + Members
            + `none` - no optimizations, all available time slots returned
            + `optimal` - ignoring `step`, step calculated based on `duration`
            + `maximal` - allows booking only on start or end of free time slot (for example, booking for 1 hour if free time slots 9:00-12:00 and 14:00-21:00 available will return 9:00, 11:00, 14:00 and 20:00), specific positions allowed depends from `gaps_positions`
            + `maximal_if_has_appointment` - use `maximal` mode if professional has more than one free time slot for specific day (for example, 9:00-12:00 and 14:00-21:00, at least one appointment already assigned), if only one time slot (for example, 9:00-21:00, no appointments yet or appointments at day start/end) use `optimal` mode
    + `gaps_positions`: `day_start` (array[string], optional) - for `maximal` & `maximal_if_has_appointment` `gaps_mode` defines what times are available for booking (several options can be selected). For example, professional works from 9:00 till 21:00, already has bookings 9:00-10:30, 13:00-14:00 and 18:00-18:30, so free time slots are 10:30-13:00, 14:00-18:00 and 18:30-21:00, `duration` is 1 hour. See possible positions in options descriptions.
        + Default: `day_start`
        + Members
            + `day_start` - at day start (or first free time after day start if any time at day start already booked). In example above will return time 10:30.
            + `before_appointment` - time just before appointment in the middle of the day. In example above will return time 12:00 and 17:00.
            + `after_appointment` - time just after appointment in the middle of the day. In example above will return time 14:00 and 18:30.
            + `day_end` - just before day end (or last free time before day end if any time at day end already booked). In example above will return time 20:00.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "88d4b305-e198-f02c-2743-cca9390c631c":
                {
                    "2019-01-01":
                    [
                        "10:00",
                        "10:30",
                        "11:00"
                    
                    ],
                    "2019-01-02":
                    [
                        "10:00",
                        "10:30",
                        "11:00"
                    
                    ]
                },
                "023276a7-7ca9-45c9-9f9c-b536ec20315b":
                {
                    "2019-01-01":
                    [
                        "10:00",
                        "10:30",
                        "11:00"
                    
                    ],
                    "2019-01-02":
                    [
                        "10:00",
                        "10:30",
                        "11:00"
                    
                    ]
                }
            }

## User information [/me]

Each employee has two accounts in system:

1) In each database employee has access information stored about employee, specific to that database (contacts, schedule, salary etc.). If employee has access to several databases, in each database id of that employee would be different.
2) Common employee account - stored not in specific database, but on top level, independently from specific database. Even if employee lost access to last database, such account information still exists. We name this accounts "users" (to differ from employee account in specific database). Such `user` has unique id (different from ids of connected employees accounts), general information, list of contacts (phone numbers and e-mails, either verified or not) and list of connected employees accounts in different databases.

This section describes several endpoints helping to work with user account. All these endpoints need valid employee token.

Do not mess this section `/me/*` endpoints with information about current employee in active database: `/employees/me` endpoint.

New user can easily be created and even contacts be added. But to be able to receive access token, user should verify at least one contact (either phone or e-mail) and
use that contact for login (receiving token). User password can be set after verifying any contact and this password can be used for further logins.

+ Attributes
    + firstname: `John` (string) - user first name
    + lastname: `Doe` (string) - user last name
    + password: true (boolean) - if user password is set (change password providing new password in this field, from 8 to 128 characters, too weak not allowed)
    + databases (array[database_info]) - list of databases and their locations, for which user has access (read-only field)
    + contacts (array[user_contact]) - list of user contacts (read-only field)

### Get user information [GET /me{?fields}]

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + fields: `firstname,lastname,password,databases,contacts` (array[string], required) - list of fields to return (separated by comma).

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "38dc3801-a1aa-40b8-8e58-f00d81bc0019",
                "firstname": "John",
                "lastname": "Doe",
                "password": true,
                "databases":
                [
                    {
                        "id": 123456,
                        "name": "ABC Salon NY",
                        "locations":
                        [
                            {
                                "id": "96e7b6c7-1813-4719-8676-544b1488637e",
                                "name": "ABC Salon NY Brooklyn",
                                "city": "New York",
                                "address": "2818 Foster Ave",
                                "active": true
                            },
                            {
                                "id": "d80e7df4-fa60-4c78-9f27-3f6cca49accb",
                                "name": "ABC Salon NY Manhatten",
                                "city": "New York",
                                "address": "",
                                "active": false
                            },
                            {
                                "id": "7eb575ee-59b1-471d-87c5-ca10ac15712d",
                                "name": "ABC Salon NY Queens",
                                "city": "New York",
                                "address": "",
                                "active": true
                            }
                        ]
                    },
                    {
                        "id": 123457,
                        "name": "ABC Salon LA",
                        "locations":
                        [
                            {
                                "id": "8642faa0-d830-44cc-ac9a-add946edad65",
                                "name": "ABC Salon LA Downtown",
                                "city": "Los Angelos",
                                "address": "",
                                "active": false
                            },
                            {
                                "id": "abc21c49-47b5-4f80-b5e7-f3d61338984e",
                                "name": "ABC Salon LA Hollywood",
                                "city": "Los Angelos",
                                "address": "",
                                "active": true
                            }
                        ]
                    },
                    {
                        "id": 123458,
                        "name": "ABC Salon SF",
                        "locations":
                        [
                            {
                                "id": "de491f53-0969-4d08-9b04-b6d58b44f610",
                                "name": "ABC Salon SF Bay Area",
                                "city": "San Fransisco",
                                "address": "",
                                "active": false
                            }
                        ]
                    }
                ],
                "contacts":
                [
                    {
                        "type": "phone",
                        "contact": "18005550123",
                        "verified": true
                    },
                    {
                        "type": "email",
                        "contact": "a@gmail.com",
                        "verified": false
                    }
                ]
            }

### Update user information [PUT /me{?fields}]

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + fields: `firstname,lastname,password,databases,contacts` (array[string], optional) - list of fields to return (separated by comma)

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "firstname": "Johnathan",
                "lastname": "Brown",
                "password": "VemTVHy9SrRg"
            }

+ Response 204

### Add user contact without verification [POST /me/contacts/{contact}]

Add user contact without verification. Contact can't be used for login until validated and has information purpose only.

You can't add contact without verification if it already exists in someone else contacts list, either verified or not. Use adding contact with verification in this case.

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + contact: `18005550123` (string, required) - contact to add

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Add user contact with verification - send code [POST /me/contacts/{contact}{?send_code}]

Contact verification can be used in several cases:
1) user has contact and contact is not verified yet
2) contact belongs to no one yet - on this call contact will be added to your contacts list (as not verified), next logic is same as for your non-verified contact (combine two steps in one: adding contact + verification)
3) contact belongs to someone else - after verification contact will be stolen from that user or user accounts will be combined (see next endpoint for details)

This endpoint just send sms or e-mail with code (call this endpoint once more if you need to resend message) for your or any other contact.

Method returns error "Your contact already verified" if you have this contact in your contact list and this contact is verified.

Pause should be kept between re-sending message on same contact. If you try to resend sms less than in a minute or e-mail less than in 5 minutes, error will be received with
number of seconds needed to wait before sending message once again will be allowed.

Required header is "Content-Language": it defines language of sms/e-mail text, smth. like "Your login code is: " in english.

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + contact: `18005550123` (string, required) - contact to add
    + send_code (string, required) - pass this parameter without value to send code

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

+ Response 204

### Add user contact with verification - check code [POST /me/contacts/{contact}{?code}]

If you receive code sent by previous endpoint, pass it as `code` parameter. If code is correct, result will be following:

1) If contact already in your list (or was quietly added when code was sent) - it becomes verified
2) If contact in someone else contact list (`user2`), several cases exists:
    1) contact was not verified and user2 has no verified contact - seems that user2 is the same user registered in system on another contact but never made login. User2 account will be merged into current user account (with all non-verified contacts and access to databases).
    2) contact was not verified but user2 has at least one verified contact - user2 already logged into system, this disputable contact can be added by mistake for user2 and merging accounts is not secure - current user and user2 can be two different persons. So system just steals this contact from user2 and makes it verified. If current user and user2 are really one person, current user should add any verified user2 contact or user2 should add any current user verified contact - and we go to case 2.3).
    3) contact was verified - current user has right for this contact and user2 has right for it - possible only when current user and user2 are same person. So it will be safe to merge user2 account into current user account (with all verified & non-verified contacts and access to databases).

If provided code is wrong, error returned. After 5 wrong attempts contact blocks for 10 minutes for verifications and new code will be generated (now always 12-digit for both phone
and e-mail for security reasons). After next 5 wrong attempts contact blocks for another 10 minutes and new code generated.

For user convenience system add dashes for long codes. These dashes are optional and do not influence on verification (you even can set you own separators in code - system strips all non-digits from code before check).

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + contact: `18005550123` (string, required) - contact to add
    + code: `6040-9487-1473` (string, required) - verification code received in sms or e-mail

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Remove user contact [DELETE /me/contacts/{contact}]

Remove user contact. If contact is verified and it is only one user verified contact, it can't be removed.

**Authorization:** `Employee`

**Scope:** any

+ Parameters
    + contact: `18005550123` (string, required) - contact to remove

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Select/change database for employee token [POST /me/select_database]
<a name="select-database"></a> 

When employee access token received, initially it is not connected to any database and location - thus no database specific requests can be made. To select specific database and location (inside database), call this endpoint. If you need to change database or/and location, this method can be called for second time (or more).

On success, token will be re-registered to new database and location (all requests will be made to new database, old database become unavailable). So, only one
database accessible via one access token at once (there should be no situations where you need access to several databases at once).
Access token, expire date and refresh token will be the same.

**Authorization:** `Employee`

**Scope:** any

+ Attributes
    + database: 123456 (number, required) - 6-digit database code
    + location: `1f296585-856f-4b48-95ca-9511c0ba5c71` (identifier, required) - location id (Locations)(#locations)

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "database": 123456,
                "location": "2dabff53-0e22-463f-98e4-d37b501c6545"
            }

+ Response 200 (application/json)

    + Attributes
        + access_token: `d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5` (string) - token used for requests (not changed)
        + expires_at: `2016-08-12T14:18:27.000Z` (datetime) - access expiration date (can be changed)
        + refresh_token: `f9e96585-856f-4b48-95ca-9511c0ba5576` (string) - token used to update access token (not changed)
        + server: 1 (number) - server index where client database is located, preferred for requests
        + database: `123456` (string) - database for which access granted
        + location: `2dabff53-0e22-463f-98e4-d37b501c6545` (identifier) - selected location
        + scope: `clients`, `sales` (array[string]) - set of rights granted
        + possibilities (array[string]) - possibilities according to client license
        + user: `38dc3801-a1aa-40b8-8e58-f00d81bc0019` (identifier) - user id (not changed)
        + employee: `94467e16-13a6-4159-bde8-f78326fa4c52` (identifier) - employee id for user in current database

    + Body

            {
                "access_token": "d075dde6-7c91-4fc0-b1dd-b75b9c99a2f5",
                "expires_at": "2016-08-12T14:18:27.000Z",
                "refresh_token": "f9e96585-856f-4b48-95ca-9511c0ba5576",
                "server": 1,
                "database": 123456,
                "location": "2dabff53-0e22-463f-98e4-d37b501c6545",
                "scope":
                [
                    "clients",
                    "sales"
                ],
                "possibilities":
                [
                    "Clients",
                    "Services",
                    "Products",
                    "Salary",
                    ...
                ],
                "user": "38dc3801-a1aa-40b8-8e58-f00d81bc0019",
                "employee": "94467e16-13a6-4159-bde8-f78326fa4c52"
            }

## Common list of employees - DEPRECATED [/common/employees]

!!! `/common/employees` API is deprecated - use `/me` endpoint instead (described in previous chapter)

Each database has it's own list of employees. But there is also common list of employees. Not all employees from each separate database
automatically appears in common list. Rather application should call methods from this section to add employee to common list or remove them.

All employees in common list are identified by phone number. Thus each employee have one phone number (used for login) and that numbers can't repeat.

Each employee in common list can be connected to several databases. Employee is created on adding new employee-database connection.
If employee was activated, employee is not removed on removing last database connection for given employee (if not activated - remove on removing from last database).
Employee account can be removed manually.

Please pay attention, `id` in this section means `common_employee_id` retrieved from `Employee` token authorized for common employee list.

+ Attributes
    + name: "John Doe" (string) - Employee full name (read only). Content Language header needed to get this field.
    + firstname: `John` (string) - Employee first name.
    + lastname: `Doe` (string) - Employee last name.
    + sex: `Male` (enum[string]) - Employee sex.
        + Default: `Male`
        + Members
            + `Female`
    + birthday: `2018-03-01T00:00:00.000Z` (datetime) - Employee birthday date.
    + phone: `+38 063 666 33 33` (string) - Employee phone (only numbers). Read only, use `/common/employees/change_number` to change phone number.
    + email: `abc@gmail.com` (string) - Employee email (can be used to restore access if phone is unavailable).
    + photo: `77a61323-b591-4490-9e88-3b5f30695b25` (identifier) - Employee photo ID, use /common/employees/photos/id to get photo (read only). No photo if null. If photo changed, new ID will be generated, thus you can use this ID in requests with cache time max age ().
    + databases (array[string]) - List of databases codes employee is connected.
    + databases_details (array[object]) - List of databases information (databases codes and names, locations codes and names) - read only.
    + activated: true (boolean) - If employee account was activated and employee entered (received employee token at least once)

### Get employee information by id [GET /common/employees/{id}{?fields}]

Employee token should be retrieved using common employees list to make this request.
Employee information can be received only for employee for which token was got (id should be same as common_employee_id`).

For your convenience, you can use `me` instead of id of employee.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `2d3ee3e8-0548-4612-b054-ce4aba7baba3` (identifier, required) - id of employee
    + fields: `firstname,lastname,sex,phone` (array[string], optional) - list of fields to return (separated by comma).

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

+ Response 200 (application/json)

    + Body

            {
                "id": "38dc3801-a1aa-40b8-8e58-f00d81bc0019",
                "name": "John Doe",
                "firstname": "John",
                "lastname": "Doe",
                "sex": "Male",
                "birthday": "1980-01-01T00:00:00.000Z",
                "phone": "15551234567",
                "email": "johndoe@gmail.com",
                "photo": "b1afe091-faa3-4d91-89d7-2aef602ace4a",
                "databases":
                [
                    "123456",
                    "123457"
                ],
                "databases_details":
                {
                    "123456":
                    {
                        "name": "ABC Salon NY",
                        "locations":
                        {
                            "96e7b6c7-1813-4719-8676-544b1488637e":
                            {
                                "name": "ABC Salon NY Brooklyn"
                            },
                            "d80e7df4-fa60-4c78-9f27-3f6cca49accb":
                            {
                                "name": "ABC Salon NY Manhatten"
                            },
                            "7eb575ee-59b1-471d-87c5-ca10ac15712d":
                            {
                                "name": "ABC Salon NY Queens"
                            }
                        }
                    },
                    "123457":
                    {
                        "name": "ABC Salon LA",
                        "locations":
                        {
                            "8642faa0-d830-44cc-ac9a-add946edad65":
                            {
                                "name": "ABC Salon LA Downtown"
                            },
                            "abc21c49-47b5-4f80-b5e7-f3d61338984e":
                            {
                                "name": "ABC Salon LA Hollywood"
                            }
                        }
                    },
                    "123458":
                    {
                        "name": "ABC Salon SF",
                        "locations":
                        {
                            "de491f53-0969-4d08-9b04-b6d58b44f610":
                            {
                                "name": "ABC Salon SF Bay Area"
                            }
                        }
                    }
                },
                "activated": true
            }

### Get employee photo [GET /common/employees/photos/{id}{?width,height,resize,access_token}]

`id` should be `photo` field retrieved by GET /common/employees/{id}.

Employee token should be retrieved using common employees list to make this request.
Employee photo can be received only for employee for which token was got (photo for employee with id should be same as common_employee_id`).

Because employee photo id changes on photo change, API will always return same picture for same photo ID. Thus picture can be cached forever, greatly decreasing number of requests.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `b1afe091-faa3-4d91-89d7-2aef602ace4a` (identifier, required) - `id` should be `photo` field retrieved by GET /common/employees/{id}.
    + width: 100 (number, optional) - image width needed
    + height: 100 (number, optional) - image height needed
    + resize: `fit` (enum[string], optional) - type of resize for image
        + Default: `fit`
        + Members
            + `fit`
            + `fit_center_transparent`
            + `stretch`
            + `crop`
    + access_token: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (string, optional) - token used for API requests

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (image/jpeg)

    + Headers
    
            Cache-Control: private, max-age=31536000, immutable
    
    + Body

### Update employee information [PUT /common/employees/{id}{?fields}]

Please note that `name` can't be updated (change `firstname` and `lastname` instead),
`phone` should be updated via `/common/employees/change_number`, `databases` - via `/common/employees/add_database` and
`/common/employees/{phone}/remove_database`, `activated` is set to true after first receiving employee token (can't be updated manually).

Employee token should be retrieved using common employees list to make this request.
Employee information can be updated only for employee for which token was got (id should be same as common_employee_id`).

For your convenience, you can use `me` instead of id of employee.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `2d3ee3e8-0548-4612-b054-ce4aba7baba3` (identifier, required) - id of employee
    + fields: `firstname,lastname,sex,phone` (array[string], optional) - list of fields to return (separated by comma)

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "firstname": "John",
                "lastname": "Doe",
                "sex": "Male",
                "birthday": "1980-01-01T00:00:00.000Z",
                "email": "johndoe@gmail.com"
            }

+ Response 204

### Update employee photo [PUT /common/employees/{id}/photo]

Employee token should be retrieved using common employees list to make this request.
Employee photo can be updated only for employee for which token was got (id should be same as common_employee_id`).

For your convenience, you can use `me` instead of id of employee.

Max image size 300 x 300 pixels.

On successful update you will receive id of added photo (can be retrieved later from employee `photo` field).

If you want to delete photo, just send no content (content length is zero) - photo will be deleted and null returned as photo id. You can also use DELEte method.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `2d3ee3e8-0548-4612-b054-ce4aba7baba3` (identifier, required) - id of employee (`me` can be used for common employee token)

+ Request (image/jpeg)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "photo": "b62cb95f-d235-43b4-9874-0d57175a2f96"
            }

### Delete employee photo [DELETE /common/employees/{id}/photo]

Employee token should be retrieved using common employees list to make this request.
Employee photo can be updated only for employee for which token was got (id should be same as common_employee_id`).

For your convenience, you can use `me` instead of id of employee.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `2d3ee3e8-0548-4612-b054-ce4aba7baba3` (identifier, required) - id of employee (`me` can be used for common employee token)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Change employee phone [POST /common/employees/change_number{?email}]

In case employee phone is inaccessible and employee can't login, this method helps to change employee phone number and login via new phone.
But employee should have e-mail entered and be possible to read notification received on this e-mail.

After calling this method, letter will be sent employee e-mail with link to change phone number. Employee set his new phone number
and can login via sms on new phone number.

**Authorization:** none

+ Parameters
    + email: `johndoe@gmail.com` (string, required) - employee e-mail address

+ Response 204

### Add employee to database [POST /common/employees/add_database]

Common employees list is fulfilled using this method. Using Database or Employee token, send all available information about employee
(firstname, lastname, sex, birthday, phone, email). If employee with such phone not exists, new employee in common list with given requisites
is created (with `activated` = false) and database list containing code of database from token. Crucial parameter is only `phone`, when user
will login first time, he will check all these parameters and update them if needed so better to help them and set correct values if any.

If employee with phone number already exists, only database code is added to databases list (all other parameters ignored).

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Attributes
        + firstname (string) - Employee first name.
        + lastname (string) - Employee last name.
        + sex (enum[string]) - Employee sex.
            + Default: `Male`
            + Members
                + `Male`
                + `Female`
        + birthday (datetime) - Employee birthday date.
        + phone (string) - Employee phone (only numbers). Read only, use `/common/employees/change_number` to change phone number.
        + email (string) - Employee email (can be used to restore access if phone is unavailable).
        + photo (string) - base-64 encoded employee photo (max size 300 x 300 pixels).

    + Body

            {
                "firstname": "John",
                "lastname": "Doe",
                "sex": "Male",
                "birthday": "1980-01-01T00:00:00.000Z",
                "phone": "15551234567",
                "email": "johndoe@gmail.com"
            }

+ Response 204

### Remove employee from database [POST /common/employees/{phone}/remove_database]

Remove information about database for employee in common employee list. If that was last database employee connected with and employee not
activated, employee is removed.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + phone (string) - Employee phone (only numbers)

+ Response 204

# Group Services

Services usually provided by professionals, take some time to be done and usually are planned in schedule.

## Service [/services]
<a name="services"></a>

+ Attributes
    + id: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - service id
    + name: `Compex` (string,required) - service name. Max length is 15000 characters
    + description: `washing head` (string) - service description with html tags for font styling. Max length is 65536 characters
    + descriptionPlaintext: `washing head` (string) - service description as plain text (readonly)
    + duration: 60 (number) - service duration (in minutes) (value greater than zero)
    + gender: `both` (enum[string]) - gender, for whom service is provided
        + Default: `both`
        + Members
            + `male`
            + `female`
    + price_currency: `USD` (string) - currency for service price
    + category: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - service category
    + picture: `f1c36ed0-ee16-4a8a-83df-ad7e4e3d2a03` (identifier) - service picture
    + pictureUrl: `https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab93b/picture` (string) - URL for getting a picture (readonly)
    + public: true (boolean) - if service available for online booking
    + `location_prices` (locationprice) - price list for different locations
    + halls: `88d44832-8c92-01da-6c10-c5991fd7bb08` (array[identifier]) - list of halls, where service can be provided
    + resources: `88d44832-8c92-01da-6c10-c5991fd7bb08` (array[identifier]) - list of resources needed for providing service, see [Resources](#resources)
    + color: `white` (string) - service color in schedule
    + article: `1234` (string) - service article, can be used for quick search. Max length is 500 characters
    + barcode: `12413521235132612` (string) - service barcode, can be used for quick search by barcode reader. Max length is 500 characters
    + with_assistant: true (boolean) - service can be provided with assistant
    + service_by_time: true (boolean) - service duration can be extended, price is multiplied by service duration
    + department: `88d44832-8c92-01da-6c10-c5991fd7bb08` (identifier) - service department
    + archive: false (boolean) - if service was archived and can't be used in new sales
        + Default: `false`
    + prepaymentRequired: false (boolean) - if booking for this service requires prepayment

### Get all services [GET /services{?fields,sex,public,position,has_professional_price,professional,client_gender,free_time,free_time_skip_appointments,location,archive,service_by_time}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only category, name, duration, gender, location_prices, description, descriptionPlainText, sex, price_currency, picture, public, parent, archive, noProfessionalPriceInNativeCurrency, pricesInNativeCurrency fields), `reports`, `services_aggregator` (only category, name, duration, gender, location_prices, description, descriptionPlainText, sex, price_currency, picture, public, parent, archive, noProfessionalPriceInNativeCurrency, pricesInNativeCurrency fields)

+ Parameter
    + fields: `name,description,descriptionPlaintext,duration,gender,price_currency,no_professional_price,category,picture,pictureUrl,public,location_prices,halls,resources,color,article,barcode,with_assistant,service_by_time,department,archive` (array[string], required) - list of fields to return (separated by comma).
    + client_gender: `male` (enum[string], optional) - get only services for given gender
        + Members
            + male
            + female
    + public: true (boolean, optional) - get only public or non public services
    + has_professional_price: true (boolean, optional) - get only services which can be provided by some professional (exclude non professional services if true) or only non professional services if false
    + position: `6ade5e30-9ec9-4975-9a1b-53fdc4b1bb05` (identifier, optional) - get only services, which can be provided by given position professionals (several positions can be separated by comma)
    + professional: `88d44832-8c92-01da-6c10-c5991fd7bb08` (identifier, optional) - get only services, which can be provided by given professionals (several professionals can be separated by comma)
    + `free_time`: `2018-11-29T19:30:00.000Z` (date, optional) - get only services free at specified time start (duration is equal to service duration, custom for each service). If `location` filter is set, looks only at specified location (otherwise search in all).
    + `free_time_skip_appointments`: `bc4311b8-329c-40a9-a784-04cb5f7fe1aa` (array[identifier], optional) - if free_time filter used, specifies which appointments should be skipped checking (several appointments can be separated by comma) - skipped appointments will be treated as free time
    + location: `73a386aa-dfdd-4bba-be50-084c4104c599` (array[identifier], optional) - used in `free_time` filter (several locations can be separated by comma)
    + archive: false (boolean, optional) - get only archived or non archived services
    + service_by_time: false (boolean, optional) - get only services by time (true) or not by time (false)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                    "name": "Compex",
                    "description": "Complex service",
                    "descriptionPlaintext": "Complex service",
                    "duration": 60,
                    "gender": "both",
                    "price_currency": "UAH",
                    "no_professional_price": null,
                    "category": "c867e139-a433-4e4a-885d-ab81489ad488",
                    "picture": null,
                    "pictureUrl": "",
                    "public": true,
                    "location_prices": 
                        [
                            {
                                "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                                "position": "88660d76-a42f-442f-ba18-4378888e430f",
                                "price": 200,
                                "staff_price": 0
                            }
                        ],
                    "halls": null,
                    "resources": null,
                    "color": "White",
                    "article": null,
                    "barcode": null,
                    "with_assistant": false,
                    "service_by_time": false,
                    "department": "b45c58a6-a0ba-4ee2-afda-cb0d6e7102b4",
                    "archive": false
                }
            ]

### Get service by id [GET /services/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only category, name, duration, gender, location_prices, description, descriptionPlainText, sex, price_currency, picture, public, parent, archive, noProfessionalPriceInNativeCurrency, pricesInNativeCurrency), `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - service id
    + fields: `name,description,descriptionPlaintext,duration,gender,price_currency,no_professional_price,category,picture,pictureUrl,public,location_prices,halls,resources,color,article,barcode,with_assistant,service_by_time,department,archive` (array[string], required) - list of fields to return (separated by comma).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                "name": "Compex",
                "description": "Complex service",
                "descriptionPlaintext": "Complex service",
                "duration": 60,
                "gender": "both",
                "price_currency": "UAH",
                "no_professional_price": null,
                "category": "f1c36ed0-ee16-4a8a-83df-ad7e4e3d2a03",
                "picture": "0b646349-e7be-4179-97bd-c7155caab923",
                "pictureUrl": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab923/picture",
                "public": true,
                "location_prices": 
                [
                    {
                        "location": "00df7375-1e7b-44d8-be05-a7777e79b012",
                        "position": "d68cab7f-443f-454c-80f1-8ba8498cb0a4",
                        "price": 200,
                        "staff_price": null
                    }
                ],
                "halls": 
                [
                    "d68cab7f-443f-454c-80f1-8ba8498cb0a4"
                },
                "resources": 
                [
                    "a2f61d84-e659-472b-8a1a-284721c979ae"
                ],
                "color": "#ffffff",
                "article": "234124",
                "barcode": "12351",
                "with_assistant": false,
                "service_by_time": false,
                "department": "88d6133e-95ba-3880-77ef-874262f695de",
                "archive": false
            }

### Create new service [POST /services{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,description,duration,gender,price_currency,no_professional_price,category,picture,pictureUrl,public,location_prices,halls,resources,color,article,barcode,with_assistant,service_by_time,department,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Compex",
                "description": "",
                "duration": 60,
                "gender": "both",
                "price_currency": "UAH",
                "no_professional_price": null,
                "category": "88d60be9-abea-1947-53d9-31154749b7b9",
                "picture": null,
                "public": true,
                "location_prices": 
                [
                    {
                        "location": "8660d976-a42f-442f-ba18-4378888e430f",
                        "position": "b45c58a6-a0ba-4ee2-afda-cb0d6e7102b4",
                        "price": 200,
                        "staff_price": 0
                    }
                ],
                "halls": 
                [
                    "d68cab7f-443f-454c-80f1-8ba8498cb0a4"
                },
                "resources": 
                [
                    "a2f61d84-e659-472b-8a1a-284721c979ae"
                ],
                "color": "White",
                "article": "125122",
                "barcode": "51251",
                "with_assistant": false,
                "service_by_time": false,
                "department": "88d6133e-95ba-3880-77ef-874262f695de"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }
            
### Update service [PUT /services/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `751392d5-a113-44e4-bb30-0d9e190b2f4c` (identifier, required) - service id
    + fields: `name,description,duration,gender,price_currency,no_professional_price,category,picture,pictureUrl,public,location_prices,halls,resources,color,article,barcode,with_assistant,service_by_time,department,archive` (array[string], optional) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c
            
    + Body

            [
                "name": "Compex",
                "description": "",
                "duration": 60
            ]

+ Response 204 (application/json)

### Delete service [DELETE /services/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier, required) - service id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Service categories [/services/categories]

To simplify management, services can be arranged in categories.

+ Attributes
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier) - category id
    + name: `Haircuts` (string,required) - category name. Max length is 500 characters
    + parent: `2310a649-561d-4fa5-9f66-9c33fe8b94b3` (identifier) - parent category
    + picture: `6a765c04-9ed8-4c33-ac64-5ddf5420235c` (identifier) - category picture id
    + archive: false (boolean) - if service category was archived and can't be used in new service
        + Default: `false`

### Get all services categories [GET /services/categories{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent,picture,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived service categories

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                    "name": "Haircuts",
                    "parent": null,
                    "picture": "529e4dca-c512-44ac-aa53-d2470a5ced91",
                    "archive": false
                },
                {
                    "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                    "name": "Manicure",
                    "parent": null,
                    "picture": "893453b0-fd84-450a-bc2f-42083272d792",
                    "archive": false
                }
            ]

### Get service category by id [GET /services/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier, required) - id of services category
    + fields: `name,parent,picture,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                "name": "Haircuts",
                "parent": null,
                "picture": "8bd5bc40-dbfa-4b73-b4a6-18fb4359890a",
                "archive": false,
            }

### Create new service category [POST /services/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent,picture,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Other",
                "parent": null,
                "picture": "8bd5bc40-dbfa-4b73-b4a6-18fb4359890a"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update service category [PUT /services/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of services category
    + fields: `name,parent,picture,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Visage",
                "parent": null,
                "picture": "8bd5bc40-dbfa-4b73-b4a6-18fb4359890a"
            }
            
+ Response 204

### Delete services category [DELETE /services/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of services category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get services and categories tree [GET /services/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "archive": false,
                "categories":
                [
                    {
                        "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                        "name": "Haircuts",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                                "name": "Compex",
                                "description": "Complex service",
                                "duration": 60,
                                "gender": "both",
                                "price_currency": "UAH",
                                "no_professional_price": null,
                                "category": "f1c36ed0-ee16-4a8a-83df-ad7e4e3d2a03",
                                "picture": null,
                                "public": true,
                                "location_prices": 
                                [
                                    {
                                        "location": "00df7375-1e7b-44d8-be05-a7777e79b012",
                                        "position": "d68cab7f-443f-454c-80f1-8ba8498cb0a4",
                                        "price": 200,
                                        "staff_price": 0
                                    }
                                ],
                                "halls": 
                                [
                                    "d68cab7f-443f-454c-80f1-8ba8498cb0a4"
                                },
                                "resources": 
                                [
                                    "a2f61d84-e659-472b-8a1a-284721c979ae"
                                ],
                                "color": "#ffffff",
                                "article": "234124",
                                "barcode": "12351",
                                "with_assistant": false,
                                "service_by_time": false,
                                "department": "88d6133e-95ba-3880-77ef-874262f695de",
                                "archive": false
                            }
                        ]
                    },
                    {
                        "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                        "name": "Manicure",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                        ]
                    }                    
                ],
                "items":
                [
                ]
            }

## Service material [/services/materials]

Information about materials (products) used for providing services

+ Attributes
    + id: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - service material id
    + service: `f6c0fc7d-bcde-4c24-8359-f4f07a7b51c7` (identifier) - service id, see [Services][#services]
    + location: `f637b6a0-7640-4f56-859d-e53bae0e1aa2` (identifier) - location id, see [Locations](#locations)
        + Default: current location from token
    + storage: `9a3cb892-1b74-4e19-9a3d-6ba70f36673d` (identifier,required) - storage where material should be taken from
    + is_professional_storage: true (boolean) - if true, materials will be taken from professional storage rather than storage in `storage` field
    + product: `88d47dd6-d03b-a9ef-7272-d82f68339498` - (identifier) - id of product used
    + quantity: 4 (number) - quantity of products needed for one service (value greater than zero and will be rounded to ten digits after the comma)
    + units: `pcs` (enum[string]) - measurement units for product
        + Default: `pcs`
        + Members
            + `kg`
            + `g`
            + `oz`
            + `l`
            + `ml`
            + `m`
            + `cm`

### Get all service materials [GET /services/materials{?fields,services,locations}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameter
    + fields: `service,location,storage,is_professional_storage,product,quantity,units` (array[string], required) - list of fields to return (separated by comma).
    + locations: `88d4df1a-c2dc-5114-1de2-2be2630ee8bd` (array[identifier], optional) - get service materials with given locations (several locations can be separated by comma)
    + services: `88d3f821-36f8-3abe-7bf8-d11a0599e32f` (array[identifier], optional) - get service materials with given services (several services can be separated by comma)
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d4e876-5c20-00ce-594b-c2161095f936",
                    "service": "88d3f821-36f8-3abe-7bf8-d11a0599e32f",
                    "product": "88d3dbcd-881b-a70a-2213-b3c423431ab4",
                    "location": "88d4df1a-c2dc-5114-1de2-2be2630ee8bd",
                    "storage": "88d3da27-c1d3-01c5-57c4-98ca1b09490d",
                    "is_professional_storage": false,
                    "quantity": 15,
                    "units": "ml"
                },
                {
                    "id": "3993ffb4-c353-453f-ba4c-f128df6d8559",
                    "service": "88d4b941-54d4-17e2-675a-dfa432472027",
                    "product": "9a3cb892-1b74-4e19-9a3d-6ba70f36673d",
                    "location": "88d4df1a-c2dc-5114-1de2-2be2630ee8bd",
                    "storage": "88d3da27-c1d3-01c5-57c4-98ca1b09490d",
                    "is_professional_storage: false,
                    "quantity": 15,
                    "units": "ml"
                }
            ]
            
### Get service material by id [GET /service/materials/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameter
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of service material
    + fields: `service,location,storage,is_professional_storage,product,quantity,units` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            
            {
                "id": "be0b6712-e680-42a7-8b99-b6b2d9fcb1fe",
                "service": "88d4b941-54d4-17e2-675a-dfa432472027",
                "product": "2766e147-c7a0-4aae-a384-25716b5e69e2",
                "location": "88d4df1a-c2dc-5114-1de2-2be2630ee8bd",
                "storage": "898222b4-205b-4d00-8053-138de7f77e8d",
                "is_professional_storage": false,
                "quantity": 15,
                "units": "ml"
            }

### Create new service material [POST /services/materials{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `service,location,storage,is_professional_storage,product,quantity,units` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            [
                {
                    "service": "88d4b941-54d4-17e2-675a-dfa432472027",
                    "product": "8bfa2bd5-33b0-4ff8-8073-8cd817caccc7",
                    "location": "f78e5f86-35d6-4f83-9e39-74a47ec99155",
                    "storage": "88d3da27-c1d3-01c5-57c4-98ca1b09490d",
                    "is_professional_storage": false,
                    "quantity": 15
                }
            ]

+ Response 201 (application/json)

    + Body

            {
                "id": "79dbd056-7f23-4e87-892f-c45e1976e697"
            }
    
### Update service material [PUT /services/materials/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `751392d5-a113-44e4-bb30-0d9e190b2f4c` (identifier, required) - id service materials
    + fields: `service,product,quantity` (array[string], optional) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c
            
    + Body

            [
                "service": "0a125def-907e-4be4-bbdd-4416b61d351c",
                "product": "88d3dbcd-881b-a70a-2213-b3c423431ab4",
                "location": "f78e5f86-35d6-4f83-9e39-74a47ec99155",
                "storage": "88d3da27-c1d3-01c5-57c4-98ca1b09490d",
                "is_professional_storage": false,
                "quantity": 15
            ]

+ Response 204 (application/json)

### Delete service material [DELETE /services/materials/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier, required) - id of service material

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Resource [/resources]
<a name="resources"></a>

Resources are crucial for providing some services. Each service may use a resource while being provided. Thus, resources limit the number of such services that can be provided simultaneously.

+ Attributes
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - resource id
    + name: `Deus ex` (string,required) - resource name. Max length is 100 characters
    + locations (array) - resource availability at different locations
        + (object)
            + location: `f2caf00d-63a0-c9ec-eba0-676dcb4c64` (identifier) - location id, see [Locations](#locations)
            + count: 14 (number) - resource quantity at this location (number of services can be provided simultaneously)

### Get all resources [GET /resources/{?fields}]

**Authorization:** `Database`

**Scope:**  `full`

+ Parameter
    + fields: `name,locations` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d59509-afd8-0b76-3b19-bdda1cfe67f9",
                    "name": "Apparatus 2",
                    "locations": 
                    [
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "count": 2
                        },
                        {
                            "location": "88d525ff-c965-b53b-248f-3ef123c68b02",
                            "count": 2
                        }
                    ]
                },
                {
                    "id": "88d59509-be48-6485-3b19-bdda39d8af18",
                    "name": "Deus ex machina",
                    "locations": 
                    [
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "count": 1
                        },
                        {
                            "location": "88d525ff-c965-b53b-248f-3ef123c68b02",
                            "count": 1
                        }
                    ]
                }
            ]

### Get resource by id [GET /resources/{id}{?fields}]

**Authorization:** `Database`

**Scope:**  `full`

+ Parameter
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - id of resource
    + fields: `name,locations` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
               {
                    "id": "88d59509-afd8-0b76-3b19-bdda1cfe67f9",
                    "name": "Apparatus 2",
                    "locations": 
                    [
                        {
                            "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                            "count": 2
                        },
                        {
                            "location": "88d525ff-c965-b53b-248f-3ef123c68b02",
                            "count": 2
                        }
                    ]
                }   
            ]
    
### Create new resource [POST /resources{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,locations` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "name": "Apparatus 2",
                "location": 
                [
                    {
                        "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                        "count": 2
                    }
                ]
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }

### Update resource [PUT /resources/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - id resource
    + fields: `name,locations` (array[string], optional) - list of fields to return (separated by comma)
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Apparatus 2",
                "location": [
                    {
                        "location": "88d45ef0-6e9a-cfb6-0b47-d56566fe9691",
                        "count": 2
                    }
                ]
            }
            
+ Response 204

### Delete resource [DELETE /resources/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier, required) - id  resource

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

# Group Products

All information about products.

## Product [/products]

+ Attributes
    + name: `Smartgel 3 ml` (string,required) - product name. Max length is 15000 characters
    + description: `This product intended for..` (string) - product description. Max length is 65536 characters
    + category: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - product category
    + picture: `41678090-9109-4750-aa38-6e3444f55fa0` (identifier) - product picture
    + pictureUrl: `https://api.aihelps.com/v1/images/696320/41678090-9109-4750-aa38-6e3444f55fa0/picture` (string) - URL for getting a picture (readonly)
    + vendor_code: `30402` (string) - product vendor code. Max length is 500 characters
    + barcode: `31123` (string) - product barcode. Max length is 500 characters
    + volume: 57.2 (number) - how many units contained in one package (greater than zero)
    + supply_price: 400 (number) - the price supply particular goods 
    + supply_price_currency: `USD` (string) - currency in which the goods arrived
    + supply_price_in_native_currency: 400 (number) have translated currency to country's currency (readonly)
    + units: `ml` (enum[string]) - product measurement units
        + Default: `pcs`
        + Members
            + `items`
            + `kg`
            + `g`
            + `oz`
            + `l`
            + `m`
            + `cm`
    + location_prices (array) - product prices for locations (each object in array defines prices for one location). See [Locations](#locations)
        + (object)
            + location: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - location, for which prices are defined 
            + price: 200 (number) - fixed price for one package. Null means product can't be sold as a package. Price currency stored in `package_price_currency` field.
            + `staff_price`: 180 (number) - fixed price for one package for an employee (if custom prices for employees are turned on in settings, otherwise this setting has no meaning). Null means product can't be sold as a package for an employee. Staff price currency stored in `staff_package_price_currency` field
            + automatic_price: 30 (number) - markup percent from supply price for automatic price calculation (for product package). For example, if supply price is $20, markup 30%, purchase price will be $26.
            + original_automatic_price: 26 (number) - supply price multiply automatic_price (for product package). For example, if supply price is $20, automatic_price 30%, purchase price will be $26 (read_only)
            + `portion_price`: 100 (number) - fixed price for one portion (product field `portion_quantity` should be non-zero). Null means product can't be sold as a portion. Price currency stored in `portion_price_currency` field.
            + staff_portion_price: 70 (number) - fixed price for one portion for an employee (if custom prices for employees are turned on in settings, otherwise this setting has no meaning; product field `portion_quantity` should be non-zero). Null means product can't be sold as a portion for an employee. Staff price currency stored in `staff_portion_price_currency` field
            + `original_automatic_portion_price`: 7 (number) - supply price multiply automatic_portion_price (for product portion). For example, if supply price is $5, automatic_price 40%, purchase price will be $7 (read_only)
            + automatic_portion_price: 40 (number) - markup percent from supply price for automatic price calculation (for product portion). For example, if supply price is $5 per portion, markup 40%, purchase price will be $7.
            + `unit_price`: 30 (number) - fixed price for one unit - g, ml, cm etc. Null means product can't be sold as a unit. Price currency stored in `unit_price_currency` field.
            + staff_unit_price: 20 (number) - fixed price for one unit for an employee (if custom prices for employees are turned on in settings, otherwise this setting has no meaning). Null means product can't be sold as a unit for an employee. Staff price currency stored in `staff_unit_price_currency` field
            + `original_automatic_unit_price`: 0.95 (number) - supply price multiply automatic_unit_price (for product unit). For example, if supply price is $0.5, automatic_price 90%, purchase price will be $0.95 (read_only)
            + automatic_unit_price: 90 (number) - markup percent from supply price for automatic price calculation (for product unit). For example, if supply price is $0.5 per unit (`ml` for example), markup 90%, purchase price will be $0.95.
    + packagePrice: 10 (number) - if not null - price for product package for current location, in `package_price_currency` currency (read only)
    + packagePriceInNativeCurrency: 120 (number) - if not null - price for product package for current location, in database main (default) currency
    + portionPrice: 5 (number) - if not null - price for product portion for current location, in `portion_price_currency` currency (read only)
    + portionPriceInNativeCurrency: 60 (number) - if not null - price for product portion for current location, in database main (default) currency
    + unitPrice: 1 (number) - if not null - price for product unit for current location, in `unit_price_currency` currency (read only)
    + unitPriceInNativeCurrency: 12 (number) - if not null - price for product unit for current location, in database main (default) currency
    + portion_quantity: 12 (number,required) - how many units contained in one portion (greater than zero, if the value set to 0 will be returned as null) 
    + stocks (array) - product stocks at storages (hidden by default)
        + (object)
            + storage (identifier) - storage id
            + quantity (number) - product quantity on this storage
    + package_price_currency: `EUR` (string) - price currency for one package
    + portion_price_currency: `EUR` (string) - price currency for one portion
    + unit_price_currency: `EUR` (string) - price currency for one unit (ml, g, etc.)
    + staff_package_price_currency: `EUR` (string) - price currency for one package for employees as client
    + staff_portion_price_currency: `EUR` (string) - price currency for one portion for employees as client
    + staff_unit_price_currency: `EUR` (string) - price currency for one unit (ml, g, etc.) for employees as client
    + can_sale_package: true (boolean) - can sale by packages
        + Default: true
    + can_sale_portion: true (boolean) - can sale by portions
    + can_sale_units: true (boolean) - can sale by units (ml, g, etc.)
    + tare_weight: 4 (number) - product tare weight (in grams) (value equals or greater than zero and value will be rounded to integer)
    + critical_quantity: 5 (number) - critical quantity for current product (need to order more) (value equals or greater than zero and value will be rounded to number with 5 digits after comma)
    + requisites: `a31daff0-6a03-1c2e-a2e2-418a120bca11` (identifier) - product requisites
    + is_receipt: true (boolean) - is shown has a receipt or not (read only)
    + `product_receipts` (array) - if not null, product is receipt (built from other products), for example, one cup of coffee consists of 100 ml of water and 10 g of coffee
        + (object)
            + product: `8a12e3ff0-1a7e-a24e-8432-7c1c2e5e931d` (identifier) - product in receipt
            + quantity: 4 (number) - product quantity needed
    + count_as_cost_for_salary: true (boolean) - if current product as material is deducted from salasry while salary calculated for professionals
    + tax: 100 (number) - tax sum (value equals or greater than zero and value will be rounded to number with 2 digits after comma)
    + department: `d56e8215-d9dc-0e4c-8324-e3f5e7c3a09c` (identifier) - product department
    + price_for_salary_calculation: 2000 (number,required) - if not null, custom price to be used in salary calculations (value can be returned as null)
    + payed_calculation: true (boolean) - if true, client pays separately for this product as a material in services
    + consignment_supplier: `216e3a05-ddc9-40ec-8432-7cce3f5e98d5` (identifier) - product supplier if consignment model selected for given product
    + archive: false (boolean) - if product was archived and can't be used in new sales
        + Default: `false`

### Get all products [GET /products{?fields,archive}]

**Authorization:** `Database`

**Scope:**  `full`,`clients_module`,`online_store`,`reports`

+ Parameter
    + fields: `name,description,category,picture,pictureUrl,vendor_code,barcode,volume,supply_price,supply_price_currency,supply_price_in_native_currency,units,location_prices,portion_quantity,stocks,package_price_currency,portion_price_currency,unit_price_currency,staff_package_price_currency,staff_portion_price_currency,staff_unit_price_currency,can_sale_package,can_sale_portion,can_sale_units,tare_weight,critical_quantity,requisites,is_receipt,product_receipts,count_as_cost_for_salary,tax,department,price_for_salary_calculation,payed_calculation,consignment_supplier,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived products
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                    "name": "Cotton pads",
                    "description": "120 cotton pads for face",
                    "category": "88d61182-23fc-c6c9-03da-4259788d120a",
                    "picture": null,
                    "pictureUrl": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab923/picture",
                    "vendor_code": "30402",
                    "barcode": "31142",
                    "volume": 120,
                    "supply_price": 300,
                    "supply_price_currency": "USD",
                    "supply_price_in_native_currency": 300,
                    "units": "pcs",
                    "portion_quantity": 10,
                    "package_price_currency": "USD",
                    "portion_price_currency": "USD",
                    "unit_price_currency": "USD",
                    "staff_package_price_currency": "USD",
                    "staff_portion_price_currency": "USD",
                    "staff_unit_price_currency": "USD",
                    "can_sale_package": true,
                    "can_sale_portion": false,
                    "can_sale_units": false,
                    "tare_weight": 1,
                    "critical_quantity": 1,
                    "requisites": "88d4feb7-f4a9-931b-1a0f-5bbe3ee6bc06",
                    "product_receipts": null,
                    "count_as_cost_for_salary": true,
                    "tax": 0,
                    "department": "88d38b8c-2d06-1612-15f9-dd6301fef099",
                    "price_for_salary_calculation": 0,
                    "payed_calculation": false,
                    "consignment_supplier": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                    "location_prices": 
                    [
                        {
                            "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                            "price": 1000,
                            "staff_price": 800,
                            "automatic_price": null,
                            "original_automatic_price": null,
                            "portion_price": 500,
                            "staff_portion_price": 400,
                            "automatic_portion_price": 0,
                            "original_automatic_portion_price": 0,
                            "unit_price": null,
                            "staff_unit_price": 8,
                            "automatic_unit_price": null,
                            "original_automatic_unit_price": null
                        }
                    ],
                    "archive": false
                }
            ]
            
### Get product by id [GET /products/{id}{?fields}]

**Authorization:** `Database`

**Scope:**  `full`,`clients_module`,`online_store`,`reports`

+ Parameter
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - product id
    + fields: `name,description,category,picture,pictureUrl,vendor_code,barcode,volume,supply_price,supply_price_currency,supply_price_in_native_currency,units,location_prices,portion_quantity,stocks,package_price_currency,portion_price_currency,unit_price_currency,staff_package_price_currency,staff_portion_price_currency,staff_unit_price_currency,can_sale_package,can_sale_portion,can_sale_units,tare_weight,critical_quantity,requisites,is_receipt,product_receipts,,count_as_cost_for_salary,tax,department,price_for_salary_calculation,payed_calculation,consignment_supplier,archive` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body 

            {
                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                "name": "Cotton pads",
                "description": "120 cotton pads for face",
                "category": "88d61182-23fc-c6c9-03da-4259788d120a",
                "picture": "0b646349-e7be-4179-97bd-c7155caab923",
                "pictureUrl": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab923/picture","vendor_code": "30402",
                "barcode": "30402",
                "volume": 1,
                "supply_price": 300,
                "supply_price_currency": "USD",
                "supply_price_in_native_currency": 300,
                "units": "pcs",
                "portion_quantity": 10,
                "package_price_currency": "USD",
                "portion_price_currency": "USD",
                "unit_price_currency": "USD",
                "staff_package_price_currency": "USD",
                "staff_portion_price_currency": "USD",
                "staff_unit_price_currency": "USD",
                "can_sale_package": true,
                "can_sale_portion": false,
                "can_sale_units": false,
                "tare_weight": 1,
                "critical_quantity": 0,
                "requisites": "88d4feb7-f4a9-931b-1a0f-5bbe3ee6bc06",
                "product_receipts": null,
                "count_as_cost_for_salary": true,
                "tax": 0,
                "department": "88d38b8c-2d06-1612-15f9-dd6301fef099",
                "price_for_salary_calculation": 0,
                "payed_calculation": false,
                "consignment_supplier": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                "location_prices": 
                [
                    {
                        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                        "price": 1000,
                        "staff_price": 800,
                        "original_automatic_price": 0,
                        "automatic_price": 0,
                        "portion_price": 500,
                        "staff_portion_price": null,
                        "original_automatic_portion_price": 0,
                        "automatic_portion_price": 0,
                        "unit_price": 10,
                        "staff_unit_price": 8,
                        "automatic_unit_price": null,
                        "original_automatic_unit_price": null
                    }
                ],
                "archive": false
            }
            
### Create new product [POST /products{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,description,category,picture,pictureUrl,vendor_code,barcode,volume,supply_price,supply_price_currency,supply_price_in_native_currency,units,location_prices,portion_quantity,stocks,package_price_currency,portion_price_currency,unit_price_currency,staff_package_price_currency,staff_portion_price_currency,staff_unit_price_currency,can_sale_package,can_sale_portion,can_sale_units,tare_weight,critical_quantity,requisites,is_receipt,product_receipts,,count_as_cost_for_salary,tax,department,price_for_salary_calculation,payed_calculation,consignment_supplier,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
                "name": "Smartgel 3 ml",
                "description": "nail polish remover",
                "category": "88d61182-23fc-c6c9-03da-4259788d120a",
                "picture": null,
                "vendor_code": "124512",
                "barcode": "12351",
                "volume": 3,
                "supply_price": 300,
                "supply_price_currency": "USD",
                "supply_price_in_native_currency": 300,
                "units": "ml",
                "portion_quantity": 1,
                "package_price_currency": "USD",
                "portion_price_currency": "USD",
                "unit_price_currency": "USD",
                "staff_package_price_currency": "USD",
                "staff_portion_price_currency": "USD",
                "staff_unit_price_currency": "USD",
                "can_sale_package": true,
                "can_sale_portion": false,
                "can_sale_units": false,
                "tare_weight": 0,
                "critical_quantity": 0,
                "requisites": "88d4feb7-f4a9-931b-1a0f-5bbe3ee6bc06",
                "product_receipts": null,
                "count_as_cost_for_salary": true,
                "tax": 0,
                "department": "88d38b8c-2d06-1612-15f9-dd6301fef099",
                "price_for_salary_calculation": 0,
                "payed_calculation": false,
                "consignment_supplier": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                "location_prices": 
                [
                    {
                        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                        "price": 200,
                        "staff_price": null,
                        "automatic_price": 0,
                        "portion_price": null,
                        "staff_portion_price": 18,
                        "automatic_portion_price": 0,
                        "unit_price": 2,
                        "staff_unit_price": 1.8,
                        "automatic_unit_price": 0
                    }
                ],
                "archive": false
            }
             
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }
            
### Update product [PUT /products/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - product id
    + fields: `name,description,category,picture,pictureUrl,vendor_code,barcode,volume,supply_price,supply_price_currency,supply_price_in_native_currency,units,location_prices,portion_quantity,stocks,package_price_currency,portion_price_currency,unit_price_currency,staff_package_price_currency,staff_portion_price_currency,staff_unit_price_currency,can_sale_package,can_sale_portion,can_sale_units,tare_weight,critical_quantity,requisites,is_receipt,product_receipts,,count_as_cost_for_salary,tax,department,price_for_salary_calculation,payed_calculation,consignment_supplier,archive` (array[string], optional) - list of fields to return (separated by comma)
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    
    + Body
            
            {
                "name": "Smartgel Plus 3 ml"
            }
            
            
+ Response 204

### Delete product [DELETE /products/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - id product
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204

## Product categories [/products/categories]

To simplify products management, products can be arranged in categories.

+ Attributes
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier) - category id
    + name: `Loreal` (string,required) - category name. Max length is 500 characters
    + parent: `2310a649-561d-4fa5-9f66-9c33fe8b94b1` (identifier) - parent category
    + picture: `6a765c04-9ed8-4c33-ac64-5ddf54202353` (identifier) - category picture id
    + archive: false (boolean) - if product category was archived and can't be used in new product
        + Default: `false`

### Get all products categories [GET /products/categories{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`online_store`,`reports`

+ Parameters
    + fields: `name,parent,picture,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived product categories

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                    "name": "Loreal",
                    "parent": null,
                    "picture": "bf78e7aa-f144-418e-b3d9-c6f935d74210",
                    "archive": false
                },
                {
                    "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                    "name": "Wella",
                    "parent": null,
                    "picture": "96e8f3d3-71e4-4641-9d16-76fc191ca1f3",
                    "archive": false
                }
            ]

### Get product category by id [GET /products/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`online_store`,`reports`

+ Parameters
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier, required) - id of products category
    + fields: `name,parent,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                "name": "Loreal",
                "parent": null,
                "picture": "54199865-c6ec-44ec-b047-c7113a4a854f",
                "archive": false
            }

### Create new product category [POST /products/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Other",
                "parent": null,
                "picture": "b8f0bf72-5736-457f-8760-ddbae1906e7f"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update product category [PUT /products/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of products category
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Shwarzkopf",
                "parent": null,
                "picture": "3f692650-e4c6-4175-b167-7be79a51fce6"
            }
            
+ Response 204

### Delete products category [DELETE /products/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of products category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get products and categories tree [GET /products/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`online_store`,`reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "archive": false,
                "categories":
                [
                    {
                        "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                        "name": "Loreal",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                                "name": "Cotton pads",
                                "description": "120 cotton pads for face",
                                "category": "88d61182-23fc-c6c9-03da-4259788d120a",
                                "picture": null,
                                "vendor_code": "30402",
                                "barcode": "31142",
                                "volume": 120,
                                "units": "pcs",
                                "portion_quantity": 10,
                                "package_price_currency": "USD",
                                "portion_price_currency": "USD",
                                "unit_price_currency": "USD",
                                "staff_package_price_currency": "USD",
                                "staff_portion_price_currency": "USD",
                                "staff_unit_price_currency": "USD",
                                "can_sale_package": true,
                                "can_sale_portion": false,
                                "can_sale_units": false,
                                "tare_weight": 1,
                                "critical_quantity": 1,
                                "requisites": "88d4feb7-f4a9-931b-1a0f-5bbe3ee6bc06",
                                "product_receipts": null,
                                "count_as_cost_for_salary": true,
                                "tax": 0,
                                "department": "88d38b8c-2d06-1612-15f9-dd6301fef099",
                                "price_for_salary_calculation": 0,
                                "payed_calculation": false,
                                "automatic_price": false,
                                "automatic_price_markup": 0,
                                "automatic_portion_price_markup": 0,
                                "automatic_unit_price_markup": 0,
                                "consignment_supplier": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                                "location_prices": [
                                    {
                                        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                                        "price": 1000,
                                        "staff_price": 800,
                                        "automatic_price": 0,
                                        "portion_price": 500,
                                        "staff_portion_price": 400,
                                        "automatic_portion_price": 0,
                                        "unit_price": 10,
                                        "staff_unit_price": 8,
                                        "automatic_unit_price": 0
                                    }
                                ],
                                "archive": false
                            }
                        ]
                    },
                    {
                        "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                        "name": "Wella",
                        "parent": null,
                        "archive": false
                        "categories":
                        [
                        ],
                        "items":
                        [
                        ]
                    }                    
                ],
                "items":
                [
                ]
            }

## Storage [/storages]

Storage describes place where products are kept. In each location there can be several storages.

+ Attributes
    + name: "Reception" (string,required) - storage name. Max length is 200 characters
    + sale_unit_types: "Package" (enum[string]) - can sale types from storages
        + Members
            + Portion
            + Units
    + location: `bba7c7ab-3656-489b-b414-c8f872c96cf1` (identifier,required) - storage location. See [Location](#location)

### Get all storages [GET /storages{?fields,location,products_filter}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `online_store`, `reports`

+ Parameters
    + fields: `name,sale_unit_types,location,products_filter` (array[string], required) - list of fields to return (separated by comma).
    + location: `12d4b305-e198-f02c-2743-cca93cb16baa` (array[identifier], optional) - get only storages in given location (several locations can be separated by comma)
    + `products_filter`: `3f69b533-8288-4847-a7f0-ab4d71e598f6,3f69b533-8288-4847-a7f0-ab4d71e59c21;1,0` (filter_info) - Filter that describes which products should be presented.
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                    "name": "Reception",
                    "sale_unit_types": [
                    "Portion"
                    ],
                    "location": "cb1c6ce2-c81f-4d70-ad31-8f627edb41e1",
                    "products_filter": "none"
                },
                {
                    "id": "71668460-255c-491f-b4e6-90cea3f338a9",
                    "name": "Backbar",
                    "sale_unit_types": [
                    "Portion",
                    "Package"
                    ],
                    "location": "cb1c6ce2-c81f-4d70-ad31-8f627edb41e1",
                    "products_filter": "all"
                }
            ]
            
### Get storage by id [GET /storages/{id}{?fields}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `online_store`, `reports`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c741` (identifier, required) - storage id
    + fields: `name,sale_unit_types,location,products_filter` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "bb9114c8-439c-40a9-a784-04885f7fedaf",
                "name": "Reception",
                "sale_unit_types": [
                "Portion",
                "Package"
                ],
                "location": "bb4c68e2-c81f-4d70-ad31-8f627ed9bcec",
                "products_filter": "all"
            }

### Create new storage [POST /storages{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,location,products_filter` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bce1

    + Body

            {
                "name": "Other",
                "location": "bc520f6d-5c0e-4fc9-9768-c4e1af7fbf3c",
                "products_filter": "none"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "fc9114c8-439c-40a9-a784-04885f7fedc1"
            }

### Update storage [PUT /storages/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - storage id
    + fields: `name,sale_unit_types,location,products_filter` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Main storage",
                "location": "bc520f6d-5c0e-4fc9-9768-c4e1af7fbf3c",
                "products_filter": "all"
            }
            
+ Response 204

### Delete storage [DELETE /storages/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - storage id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Supplier [/suppliers]

To simplify supplier management.

+ Attributes
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier) - supplier id
    + name: `Loreal` (string,required) - supplier name. Max length is 200 characters

    
### Get all  suppliers [GET /suppliers{?fields,name}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name` (array[string], required) - list of fields to return (separated by comma).
    + name: `Loreal` (string, optional) - get only suppliers which have name = `Loreal`

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                    "name": "Loreal"
                },
                 {
                    "id": "dd9114c8-439c-40a9-a784-048675f7fedab",
                    "name": "Gonzo"
                }
            ]

## Movement [/movements]

Movements describe such operation types as products order, supply, transfer between storages, replacing one product to another, product waste and return to supplier. Not all fields have meaning for any type. `fromStorage` only valid for 'transfer', 'replace', 'waste' and 'return' types; `toStorage` - for 'order', 'supply', 'transfer' and 'replace' types; `supplier` - for 'supply' and 'return' types; `sum` - for 'supply', 'waste' and 'return' types; `discountSum` and `transportationCosts` - only for 'supply' type. For other movement types these fields have default values - null for identifiers and 0 for numbers.

There is no separate field `location` for movement. Movement is done for specific storage or two storages, each storage belongs to some location.

+ Attributes
    + date: `2019-02-15T00:00:00.000Z` (datetime) - date & time operation was done (moment products was added/removed from storage)
    + invoiceDate: `2019-02-13T00:00:00.000Z` (datetime) - movement date according to invoice (`null` if no invoice)
        + Default: null
    + invoiceNumber: `AQ418` (string) - invoice number (any format appliable), or empty string if no invoice
    + type: `supply` (enum[string]) - movement type
        + Members
            + `order` - products was ordered but not yet received (to storage `toStorage`). `products` will be empty, no changes in storage
            + `supply` - products received from supplier `supplier` to storage `toStorage`, total sum `sum`, including discount `discountSum` and additional `transportationCosts`
            + `transfer` - products moved from storage `fromStorage` to `toStorage` (products and their quantities not changed), storages can be in different locations
            + `replace` - product was replaced to some other product (from storage `fromStorage` to `toStorage`, can be in different locations, can be same storage), quantity and sum could be changed
            + `waste` - products expired, was fully used or for some other reason not more available on storage `fromStorage`. Total sum of wasted products is `sum`
            + `return` - products was returned to supplier `supplier` for some reason from storage `fromStorage` on total sum `sum`. Products can be returned to other supplier and other sum than was supplied.
    + fromStorage: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - storage where products was taken
    + toStorage: `41678090-9109-4750-aa38-6e3444f55fa0` (identifier) - storage where products arrived
    + supplier: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - supplier from which products arrived or returned
    + products (array) - list of product changes in movement
        + (object)
            + product: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - product that was added/removed
            + storage: `a31daff0-6a03-1c2e-a2e2-418a120bca11` (identifier) - storage where product was added/removed
            + quantity: `-2` (number) - quantity change of product (positive - added to storage, negative - removed from storage)
            + sum: `-250` (number) - cost of product change (sign of `sum` is same as sign of `quantity`), sum is calculated base on actual prices on movement `date`
    + sum: 1500.00 (number) - total sum for 'supply', 'waste' and 'return' types
    + discountSum: 175.00 (number) - discount from supplier (for 'supply' type)
    + transportationCosts: 35.00 (number) - additional costs payed for transportation (for 'supply' type)
    + comment: `need paper invoice` (string) - additional comment

### Get all movements [GET /movements{?fields,from,to}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `date,supplier` (array[string], required) - list of fields to return (separated by comma).
    + from: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only movements which was made at `from` or later
    + to: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only movements which was made before `to`

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                    "date": "2019-02-15T00:00:00.000Z",
                    "invoiceDate": "2019-02-13T00:00:00.000Z",
                    "invoiceNumber": "AQ418",
                    "type": "supply",
                    "fromStorage": null,
                    "toStorage": "41678090-9109-4750-aa38-6e3444f55fa0",
                    "supplier": "f6c0daf0-6903-4c9e-a0e2-4381676dc364",
                    "products": [
                        {
                            "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                            "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                            "quantity": 1,
                            "sum": 1200
                        },
                        {
                            "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                            "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                            "quantity": 1,
                            "sum": 300
                        }
                    ],                    
                    "sum": 1500.00,
                    "discountSum": 175.00,
                    "transportationCosts": 35.00,
                    "comment": "need paper invoice"
                },
                {
                    "id": "a0ea1078-eb73-499d-96b2-de3dccff28ba",
                    "date": "2019-02-18T00:00:00.000Z",
                    "invoiceDate": "2019-02-18T00:00:00.000Z",
                    "invoiceNumber": "",
                    "type": "waste",
                    "fromStorage": "41678090-9109-4750-aa38-6e3444f55fa0",
                    "toStorage": null,
                    "supplier": null
                    "products": [
                        {
                            "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                            "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                            "quantity": -1,
                            "sum": -300
                        }
                    ],                    
                    "sum": 300.00,
                    "discountSum": 0.00,
                    "transportationCosts": 0.00,
                    "comment": null
                }
            ]
            
### Get movement by id [GET /movements/{id}{?fields}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c741` (identifier, required) - storage id
    + fields: `date,supplier` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                "date": "2019-02-15T00:00:00.000Z",
                "invoiceDate": "2019-02-13T00:00:00.000Z",
                "invoiceNumber": "AQ418",
                "type": "supply",
                "fromStorage": null,
                "toStorage": "41678090-9109-4750-aa38-6e3444f55fa0",
                "supplier": "f6c0daf0-6903-4c9e-a0e2-4381676dc364",
                "products": [
                    {
                        "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 1200
                    },
                    {
                        "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 300
                    }
                ],                    
                "sum": 1500.00,
                "discountSum": 175.00,
                "transportationCosts": 35.00,
                "comment": "need paper invoice"
            }

## Stocktaking [/stocktakings]

There is no separate field `location` for stocktaking. Stocktaking is done for specific storage, each storage belongs to some location.

+ Attributes
    + date: `2019-02-15T00:00:00.000Z` (datetime) - date & time stocktaking was done (moment products was added/removed from storage)
    + storage: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - storage where stocktaking was done
    + products (array) - list of product for which check ws done
        + (object)
            + product: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - product that was checked
            + correctionQuantity: `-2` (number) - quantity change of product (positive - added to storage, negative - removed from storage, zero - no change)
            + correctionSum: `-250` (number) - cost of product change (sign of `correctionSum` is same as sign of `correctionQuantity`), sum is calculated base on actual prices on stocktaking `date`
    + comment: `needs additional verification` (string) - additional comment

### Get all stocktakings [GET /stocktakings{?fields,from,to}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `date,supplier` (array[string], required) - list of fields to return (separated by comma).
    + from: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only stocktakings which was made at `from` or later
    + to: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only stocktakings which was made before `to`

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                    "date": "2019-02-15T00:00:00.000Z",
                    "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                    "products": [
                        {
                            "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                            "correctionQuantity": 1,
                            "correctionSum": 1200
                        },
                        {
                            "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                            "quantity": 1,
                            "correctionSum": 300
                        }
                    ],                    
                    "comment": "needs additional verification"
                },
                {
                    "id": "a0ea1078-eb73-499d-96b2-de3dccff28ba",
                    "date": "2019-02-18T00:00:00.000Z",
                    "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                    "products": [
                        {
                            "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                            "correctionQuantity": -1,
                            "correctionSum": -300
                        }
                    ],                    
                    "comment": null
                }
            ]
            
### Get stocktaking by id [GET /stocktakings/{id}{?fields}]
    
**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c741` (identifier, required) - storage id
    + fields: `date,supplier` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                "date": "2019-02-15T00:00:00.000Z",
                "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                "products": [
                    {
                        "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                        "correctionQuantity": 1,
                        "correctionSum": 1200
                    },
                    {
                        "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                        "quantity": 1,
                        "correctionSum": 300
                    }
                ],                    
                "comment": "needs additional verification"
            }

# Group Cards

Different types of discount/bonus cards, clips, subscriptions, etc.

## Card [/cards]

+ Attributes
    + name: `Gold` (string,required) - card name. Max length is 200 characters
    + description: `Gold card gives you the biggest possible discount in 10%` (string) - card detailed description. Max length is 15000 characters
    + price: 0 (number) - card price (zero if card is free)
    + price_currency: `USD` (currency) - currency for card price; usually same as main currency for program - in that case price currency can be empty, but also can different
    + category: `b2a72d9a-aefb-4035-8720-8138dbd84c83` (identifier) - card category (cards can be grouped).
    + picture: `180ba3d7-8a74-4f76-8533-bf118a0e2022` (identifier) - ID of picture from `/pictures/` catalog
    + pictureUrl: `https://api.aihelps.com/v1/images/696320/180ba3d7-8a74-4f76-8533-bf118a0e2022/picture` (string) - URL for getting a picture (readonly)
    + visits: 12 (number_or_unlimited) - number of visits by card (used in `discounts` field); represent number of times discount can be & was applied; "unlimited" if not limited
    + public: true (boolean) - if card is visible for clients (otherwise - for internal use)
        + Default: true
    + location_prices (array) - card prices for locations (each object in array defines prices for one location). See [Locations](#locations)
        + (object)
            + location: `a63bc703-c50a-4c6d-8af4-407d8bcf3128` (identifier) - location id, for which prices are defined 
            + price: 250 (number) - price for current location
    + archive: false (boolean) - if card was archived and can't be used in new sales
        + Default: `false`
    + discounts (array) - information about card discounts
        + (object)
            + item_type: `card` (enum[string]) - card discount item type
                + Members
                    + `service`
                    + `product`
                    + `group`
                    + `denture`
            + item: `b2a72d9a-aefb-4035-8720-8138dbd84c81` (identifier) - item/category id. Null means discount for all services/products/groups/dentures/certificates.
            + quantity_type: `unlimited` (enum[string]) - quantity type
                + Members
                    + `visits`
                    + `quantity`
            + quantity: 20.3 (number) - item quantity (valid only for `quantity_type` = 'quantity'), otherwise null
            + discount: 50 (number) - discount percent
    + availableCards: 1 (array[number]) - available numbers range for a card

### Get all cards [GET /cards{?fields,public,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + fields: `name,description,price,price_currency,category,picture,pictureUrl,visits,public,location_prices,archive,discounts` (array[string], required) - list of fields to return (separated by comma).
    + public: true (boolean, optional) - get only public or non public cards
    + archive: false (boolean, optional) - get only archived or non archived cards
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                    "name": "Discount -5%",
                    "description": "Discount card for all services -5%",
                    "price": 0,
                    "category": null,
                    "picture": null,
                    "pictureUrl": "",
                    "visits": 10,
                    "public": true,
                    "discounts": 
                    [
                        {
                            "item_type": "service",
                            "item": "88d4d8c3-81e1-39b5-7bfe-b80f3ff3ab65",
                            "quantity_type": "unlimited",
                            "quantity": null,
                            "discount": 100
                        },
                        {
                            "item_type": "service",
                            "item": "88d4d8c3-81e2-c055-7bfe-b80f0881ee12",
                            "quantity_type": "quantity",
                            "quantity": 10,
                            "discount": 100
                        }
                    ]
                    "location_prices": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "price": 150
                        },
                        {
                            "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                            "price": 100
                        }
                    ],
                    "archive": false,
                    "availableCards":
                    [
                        1,
                        2
                    ]
                },
                {
                    "id": "71668460-255c-491f-b4e6-90cea3f338a9",
                    "name": "10 haircuts",
                    "description": "",
                    "price": 300,
                    "category": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                    "picture": "9b6f7731-69a1-4c9d-844a-90dc75e2acb1",
                    "pictureUrl": "https://api.aihelps.com/v1/images/696320/9b6f7731-69a1-4c9d-844a-90dc75e2acb1/picture",
                    "visits": "unlimited"
                    "public": true,
                    "discounts": 
                    [
                        {
                            "item_type": "Service",
                            "item": "88d4d8c3-81e1-39b5-7bfe-b80f3ff3ab65",
                            "quantity_type": "unlimited",
                            "quantity": null,
                            "discount": 100
                        },
                        {
                            "item_type": "Service",
                            "item": "88d4d8c3-81e2-c055-7bfe-b80f0881ee12",
                            "quantity_type": "quantity",
                            "quantity": 10,
                            "discount": 100
                        }
                    ]
                    "location_prices": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "price": 510
                        },
                        {
                            "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                            "price": 500
                        }
                    ],
                    "archive": false,
                    "availableCards":
                    [
                        1,
                        2
                    ]
                }
            ]

### Get card by id [GET /cards/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id of card
    + fields: `name,description,price,price_currency,category,picture,pictureUrl,visits,public,location_prices,archive,discounts` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                "name": "Discount -5%",
                "description": "Discount card for all services -5%",
                "price": 0,
                "category": null,
                "visits": 10,
                "public": true,
                "discounts": 
                [
                    {
                        "item_type": "Service",
                        "item": "88d4d8c3-81e1-39b5-7bfe-b80f3ff3ab65",
                        "quantity_type": "unlimited",
                        "quantity": null,
                        "discount": 100
                    },
                    {
                        "item_type": "Service",
                        "item": "88d4d8c3-81e2-c055-7bfe-b80f0881ee12",
                        "quantity_type": "quantity",
                        "quantity": 10,
                        "discount": 100
                    }
                ]
                "location_prices": 
                [
                    {
                        "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                        "price": 510
                    }
                ],
                "archive": false,
                "availableCards":
                    [
                        1,
                        2
                    ]
            }

### Create new card [POST /cards{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,description,price,price_currency,category,picture,pictureUrl,visits,public,location_prices,archive,discounts` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "name": "Discount -10%",
                "description": "Discount card for all services -10%"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "dd9114c8-439c-40a9-a784-04885f7fedab"
            }

### Update card [PUT /cards/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id of card
    + fields: `name,description,price,price_currency,category,picture,pictureUrl,visits,public,location_prices,archive,discounts` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "price": 500
            }
            
+ Response 204

### Delete card [DELETE /cards/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id of card

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Card categories [/cards/categories]

To simplify cards management, cards can be arranged in categories.

+ Attributes
    + id: `9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea` (identifier) - category id
    + name: `Special cards` (string,required) - category name. Max length is 500 characters
    + parent: `bb10a649-561d-4fa5-9f66-9c33fe8b94b3` (identifier) - parent category
    + picture: `ac765c04-9ed8-4c33-ac64-5ddf5420235c` (identifier) - category picture id
    + archive: `false` (boolean) - if card category was archived and can't be used in new card
        + Default: `false`

### Get all cards categories [GET /cards/categories{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + fields: `name,parent,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: `false` (boolean, optional) - get only archived or non archived card categories

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                    "name": "Haircuts cards",
                    "parent": null,
                    "archive": false
                },
                {
                    "id": "35dbc6f4-76c6-4cfa-a1f9-a0e8ba91f807",
                    "name": "Special cards",
                    "parent": null,
                    "archive": false
                }
            ]

### Get card category by id [GET /cards/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`,`client_access_token` (but client field should match client id), `reports`

+ Parameters
    + id: `9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea` (identifier, required) - id of cards category
    + fields: `name,parent,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                    "id": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                    "name": "Haircuts cards",
                    "parent": null,
                    "archive": false
            }

### Create new cards category [POST /cards/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Year subscriptions",
                "parent": null
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "2af87771-fb9e-4db6-a5db-c602cb1586c0"
            }

### Update cards category [PUT /cards/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea` (identifier, required) - id of cards category
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Haircuts special offers",
                "parent": null
            }
            
+ Response 204

### Delete cards category [DELETE /cards/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea` (identifier, required) - id of cards category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get cards and categories tree [GET /cards/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "archive": false,
                "categories":
                [
                    {
                        "id": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                        "name": "Haircuts cards",
                        "parent": null,
                        "archive": false
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "71668460-255c-491f-b4e6-90cea3f338a9",
                                "name": "10 haircuts",
                                "description": "",
                                "price": 300,
                                "category": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                                "public": true,
                                "archive": false
                            }
                        ]
                    },
                    {
                        "id": "35dbc6f4-76c6-4cfa-a1f9-a0e8ba91f807",
                        "name": "Special cards",
                        "parent": null,
                        "archive": false
                        "categories":
                        [
                        ],
                        "items":
                        [
                        ]
                    }
                ],
                "items":
                [
                    {
                        "id": "dd9114c8-439c-40a9-a784-04885f7fedab",
                        "name": "Discount -5%",
                        "description": "Discount card for all services -5%",
                        "price": 0,
                        "category": null,
                        "public": true,
                        "archive": false
                    }
                ]
            }

## Client cards [/clientcards]
<a name="client cards"></a>

Describes cards that are sold to clients. Each `card` type can be sold many times, new `clientcard` object is created on each sale.
Each client card has name (equal to `card` name), start and end dates.

+ Attributes
    + name: `Gold` (string,required) - card name. Max length is 500 characters
    + client: `77a61323-b591-4490-9e88-3b5f30695b25` (string) - card owner (client)
    + activated: `true` (boolean) - if card was activated (started)
    + start_date: `2018-03-01T00:00:00.000Z` (datetime, required) - card start date (or null if card was not activated)
    + end_date: `2018-08-01T00:00:00.000Z` (datetime) - card end date, `9999-01-01T00:00:00.000Z` if card is unlimited or null if card was not activated
    + cancelled: `false` (boolean) - if card was cancelled and not active any more
    + sum: `2000.00` (number) - card cost (payed by client)
    + `delayed_payments` (array[delayed_payment]) - list of delayed payments (payments that should be payed by client for card). Max length is 15728640 characters
    + total_visits: `12` (number_or_unlimited) - total number of visits by card (used in `discounts` field); represent number of times discount can be & was applied; "unlimited" if not limited
    + used_visits: `12` (number) - used number of visits by card (used in `discounts` field); represent number of times discount was applied
    + left_visits: `12` (number_or_unlimited) - number of visits by card that can be used, equals to `total_visits` - `used_visits` (used in `discounts` field); represent number of times discount can be applied; "unlimited" if not limited
    + discounts (array) - information about card discounts
        + (object)
            + item_type: `card` (enum[string]) - card discount item type
                + Members
                    + `service`
                    + `product`
                    + `group`
                    + `denture`
            + item: `b2a72d9a-aefb-4035-8720-8138dbd84c81` (identifier) - item/category id. Null means discount for all services/products/groups/dentures/certificates.
            + quantity_type: `unlimited` (enum[string]) - quantity type
                + Members
                    + `visits`
                    + `quantity`
            + total_quantity: 20 (number) - total item quantity (valid only for `quantity_type` = 'quantity'), otherwise null
            + used_quantity: 5 (number) - used item quantity, always show actual number discount was used, for all `quantity_type`
            + left_quantity: 5 (number) - left item quantity (valid only for `quantity_type` = 'quantity'), otherwise null
            + discount: 50 (number) - discount percent
    + `bonuses` (array) - information about card bonuses
        + (object)
            + item_type: `card` (enum[string]) - card discount item type
                + Members
                    + `service`
                    + `product`
                    + `group`
                    + `denture`
            + percent: 10 (number) - bonus percent for item_type
            + filter (filter_info) - information about items current bonus applies to
            + filter_description: `all` (string) - localized description of filter
    + `accumulated_discount` (accumulate_schema) - information about accumulated discount (percent increase on more visits) proposed by card; either accumulated discount or accumulated bonus can be configured
    + `accumulated_bonus` (accumulate_schema) - information about accumulated bonus (percent increase on more visits) proposed by card; either accumulated discount or accumulated bonus can be configured
    + `accumulated_percent` (accumulated_percent) - information about current accumulated discount/bonus
    + `network_card` (boolean) - if a card is valuable in more than one locations in network
    + `all_locations_available` (boolean) - if a card is valuable for all locations in network
    + `availableLocations` (array[identifier]) - list of all locations client card is available in
    + `additionalCard` (boolean) - if a card is additional
    + `timeOfDay` (schedule) - description of time client card is active
    + `description`: `Bonus card` (string) - card description with html tags for font styling. Max length is 65536 characters
    + `descriptionPlainText`: `Bonus card` (string) - card description as plain text
    + `freezeIntervals` (array) - history of freezes of client card
    + freeze_days: 14 (number) - number of days left for card freezing (0 if card can't be freezed). Can't be negative

### Get all client cards [GET /clientcards{?fields,client}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (but client field should match client id and only name, activated, cancelled, start_date, used_visits, left_visits, end_date, sum, delayed_payments, discount, discounts, bonus, bonuses, accumulated_discount, accumulated_bonus, accumulated_percent, freeze_days, client, total_visits, timeOfDay, additionalCard, available_locations, freezeIntervals, description, descriptionPlainText, availableLocations fields), `reports`

+ Parameters
    + fields: `name,client,activated,start_date,end_date,cancelled,sum,delayed_payments,total_visits,used_visits,left_visits,discount,bonus,accumulated_discount,accumulated_bonus,accumulated_percent,freeze_days` (array[string], required) - list of fields to return (separated by comma).
    + client: '88d61987-617d-07b8-2c08-a8d6570241a9' (array[identifier], optional) - get only cards for specified clients.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d61987-93e8-57f1-537e-62c54f9a3355",
                    "name": "*Package 10 author (different groups)",
                    "client": "88d61987-617d-07b8-2c08-a8d6570241a9",
                    "activated": true,
                    "start_date": "2018-09-13T00:00:00.000Z",
                    "end_date": "2018-10-13T00:00:00.000Z",
                    "cancelled": false,
                    "sum": 1000,
                    "delayed_payments": null,
                    "total_visits": 10,
                    "used_visits": 2,
                    "left_visits": 8,
                    "discounts": 
                    [
                        {
                            "item_type": "service",
                            "item": "88d61987-617d-07b8-2c08-a8d657024ba1",
                            "quantity_type": "unlimited",
                            "total_quantity": null,
                            "used_quantity": 4,
                            "left_quantity": null,
                            "discount": 10
                        },
                        {
                            "item_type": "product",
                            "item": null,
                            "quantity_type": "quantity",
                            "total_quantity": 10,
                            "used_quantity": 3,
                            "left_quantity": 7,
                            "discount": 10
                        }
                    ],
                    "bonus": {},
                    "accumulated_discount": null,
                    "accumulated_bonus": null,
                    "accumulated_percent": null,
                    "freeze_days": 0,
                    "network_card": true,
                    "all_locations_available": true,
                    "available_locations": []
                },
                {
                    "id": "88d61990-0b28-65fd-0550-4b62087b4316",
                    "name": "*Package 10 author (different groups)",
                    "client": "88d60d95-ae50-76b3-5016-ff2d301ea21d",
                    "activated": true,
                    "start_date": "2018-09-13T00:00:00.000Z",
                    "end_date": "2018-10-13T00:00:00.000Z",
                    "cancelled": false,
                    "sum": 1000,
                    "delayed_payments": null,
                    "total_visits": "unlimited",
                    "used_visits": 0,
                    "left_visits": "unlimited",
                    "discounts": 
                    [
                        {
                            "item_type": "service",
                            "item": "88d61987-617d-07b8-2c08-a8d657024ba1",
                            "quantity_type": "unlimited",
                            "total_quantity": null,
                            "used_quantity": 4,
                            "left_quantity": null,
                            "discount": 10
                        },
                        {
                            "item_type": "product",
                            "item": null,
                            "quantity_type": "quantity",
                            "total_quantity": 10,
                            "used_quantity": 3,
                            "left_quantity": 7,
                            "discount": 10
                        }
                    ],
                    "bonus": {},
                    "accumulated_discount": null,
                    "accumulated_bonus": null,
                    "accumulated_percent": null,
                    "freeze_days": 0,
                    "network_card": true,
                    "all_locations_available": true,
                    "available_locations": []
                }
            ]

### Get client card by id [GET /clientcards/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (but client field should match client id and only name, activated, cancelled, start_date, used_visits, left_visits, end_date, sum, delayed_payments, discount, discounts, bonus, bonuses, accumulated_discount, accumulated_bonus, accumulated_percent, freeze_days, client, total_visits, timeOfDay, additionalCard, available_locations, freezeIntervals, description, descriptionPlainText, availableLocations fields), `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id of card
    + fields: `name,client,activated,start_date,end_date,cancelled,sum,delayed_payments,total_visits,used_visits,left_visits,discount,bonus,accumulated_discount,accumulated_bonus,accumulated_percent,freeze_days` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d61990-0b28-65fd-0550-4b62087b4316",
                "name": "Package 10 author (different groups)",
                "client": "88d60d95-ae50-76b3-5016-ff2d301ea21d",
                "activated": true,
                "start_date": "2018-09-13T00:00:00.000Z",
                "end_date": "2018-10-13T00:00:00.000Z",
                "cancelled": false,
                "sum": 1000,
                "delayed_payments": null,
                "total_visits": 10,
                "used_visits": 2,
                "left_visits": 8,
                "discounts": 
                    [
                        {
                            "item_type": "service",
                            "item": "88d61987-617d-07b8-2c08-a8d657024ba1",
                            "quantity_type": "quantity",
                            "total_quantity": 20,
                            "used_quantity": 4,
                            "left_quantity": 16,
                            "discount": 10
                        }
                    ],
                "bonus": {},
                "accumulated_discount": null,
                "accumulated_bonus": null,
                "accumulated_percent": null,
                "freeze_days": 0,
                "network_card": true,
                "all_locations_available": true,
                "available_locations": []
            }

            
# Group Certificates

Gift certificates with predefined sum or services list. Certificates can be organized into categories.

## Certificate [/certificates]

+ Attributes
    + name: `Happy Valentines Day $100` (string, required) - certificate name. Max length is 500 characters
    + location: `6f712b8e-1569-4104-833d-50a8f87bb09c` (identifier) - location where certificate was issued
        + Default: current location from token
    + description: `Certificate for Happy Valentines Day contains $100 in deposit` (string) - certificate detailed description. Max length is 15000 characters
    + barcode: `2782843833907` (string) - certificate barcode (for reading with barcode reader). Max length is 200 characters
    + number: 1412 (number, required) - certificate unique number (integer greater than zero)
    + one_time: false (boolean) - certificate can be used only once; not spent money or services will burns out
        + Default: false
    + price: 100 (number) - price, for which certificate can be sold/was sold certificate (zero if certificate is free); can't be chaged for sold certificate
    + total_sum: 100 (number) - all money available for certificate (value equals or greater than zero)
    + used_sum: 50 (number) - used sum by certificate (read only)
    + left_sum: 50 (number) - left sum at certificate (read only)
    + services (array) - services provided for free by certificate
        + (object)
            + service (identifier) - service id
            + total_quantity (number) - max available quantity for visit of services
            + used_quantity (number) - used quantity for service (read only)
            + left_quantity (number) - left quantity for service (read only)
    + expire_date: `2019-02-15T00:00:00.000Z` (datetime) - date when certificate become expired (or `null` if certificate is unlimited)
        + Default: null
    + client: `fc3c5f2c-3dee-4b63-b555-1912482b3b69` (identifier) - client who use certificate (if certificate sold), not the same person who bought it, set to null on certificate sale
    + status (enum[string]) - certificate status for current moment
        + Default: `new`
        + Members
            + `new` - certificate was issued, but not sold yet
            + `sold` - certificate was sold and valid now (not expired), but was not used yet
            + `used` - certificate was sold and used at least once (but not fully utilized yet and not expired); certificate surely not one time
            + `fully_utilized` - certificate was sold and was either fully utilized or used once for one-time certificate and can't be used any more
            + `expired` - certificate was expired (can be new or already sold or even fully utilized before)
    + archive: false (boolean) - if certificate was archived and can't be used in new sales
        + Default: `false`

### Get all certificates [GET /certificates{?fields,name,location,barcode,number,client,status,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `name,location,description,barcode,number,one_time,price,total_sum,used_sum,left_sum,expire_date,services,status,client,archive` (array[string], required) - list of fields to return (separated by comma).
    + name: `Valentine` (string, optional) - get only certificates where name contains given text (several phrases can be separated by comma)
    + location: `0be9ab15-2569-47e1-96e3-ae3211a7a7ec` (identifier, optional) - get only certificates from given location (several locations can be separated by comma)
    + barcode: `2782843833907` (string, optional) - get only certificate(s) with given barcode (several barcodes can be separated by comma)
    + number: 1412 (number, optional) - get only certificate with given number (several numbers can be separated by comma)
    + client: `fc3c5f2c-3dee-4b63-b555-1912482b3b69` (identifier, optional) - get only certificates for given client (several clients can be separated by comma)
    + status: `fully_utilized` (string, optional) - get only certificates of given status (several statuses can be separated by comma)
    + archive: false (boolean, optional) -  get only archived or non archived certificates

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "5227a78f-fe20-4c1b-8136-a43c7d26f3d5",
                    "name": "Happy Valentines Day $100",
                    "location": "6f712b8e-1569-4104-833d-50a8f87bb09c",
                    "description": "Certificate for Happy Valentines Day contains $100 in deposit",
                    "barcode": "2782843833907",
                    "number": 1412,
                    "one_time": false,
                    "price": 100,
                    "total_sum": 100,
                    "used_sum": 50,
                    "left_sum":50,
                    "services": [],
                    "expire_date": "2019-02-15T00:00:00.000Z",
                    "client": "fc3c5f2c-3dee-4b63-b555-1912482b3b69",
                    "status": "sold",
                    "archive": false
                },
                {
                    "id": "eed48dd4-3371-4e9d-bb06-ad8bc2b2bd3c",
                    "name": "End of Winter Massages",
                    "location": "6f712b8e-1569-4104-833d-50a8f87bb09c",
                    "description": "Certificate for several massages",
                    "barcode": "2782843833435",
                    "number": 1753,
                    "one_time": false,
                    "price": 300,
                    "total_sum": 1000,
                    "used_sum": 100,
                    "left_sum": 900,
                    "services": 
                    [
                        {
                            "service": "eb2de366-7de6-4712-85ce-0e6d242770f5",
                            "total_quantity": 10,
                            "used_quantity": 0,
                            "left_quantity": 10
                        },
                        {
                            "service": "bd44104d-c2e4-44d3-ba9f-e5a38d1fb5aa",
                            "total_quantity": 100,
                            "used_quantity": 2,
                            "left_quantity": 98
                        }
                    ],
                    "expire_date": "2019-03-01T00:00:00.000Z",
                    "client": "fc3c5f2c-3dee-4b63-b555-1912482b3b69",
                    "status": "used",
                    "archive": false
                }
            ]

### Get certificate by id [GET /certificates/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + id: `5227a78f-fe20-4c1b-8136-a43c7d26f3d5` (identifier, required) - id of certificate
    + fields: `name,location,description,barcode,number,one_time,price,total_sum,used_sum,left_sum,expire_date,services,status,client,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "5227a78f-fe20-4c1b-8136-a43c7d26f3d5",
                "name": "Happy Valentines Day $100",
                "location": "6f712b8e-1569-4104-833d-50a8f87bb09c",
                "description": "Certificate for Happy Valentines Day contains $100 in deposit",
                "barcode": "2782843833907",
                "number": 1412,
                "one_time": false,
                "price": 100,
                "total_sum": 100,
                "used_sum": 50,
                "left_sum": 50,
                "services": [],
                "expire_date": "2019-02-15T00:00:00.000Z",
                "client": "fc3c5f2c-3dee-4b63-b555-1912482b3b69",
                "status": "sold",
                "archive": false
            }

### Create new certificate [POST /certificates{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,location,description,barcode,number,one_time,price,total_sum,used_sum,left_sum,expire_date,services,status,client,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "name": "$200 Gift certificate",
                "number": 735,
                "price": 200,
                "total_sum": 100
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "eecf8043-6789-48a0-afb7-660ab8786aea"
            }

### Update certificate [PUT /certificates/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5227a78f-fe20-4c1b-8136-a43c7d26f3d5` (identifier, required) - id of certificate
    + fields: `name,location,description,barcode,number,one_time,price,total_sum,used_sum,left_sum,expire_date,services,status,client,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "client": "e4deddc0-ecf6-4399-831c-90f43932aa81"
            }
            
+ Response 204

### Delete certificate [DELETE /certificates/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id of certificate

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Certificate categories [/certificates/categories]

To simplify certificates management, certificates can be arranged in categories.

+ Attributes
    + id: `748337fe-ac89-44a8-8f78-2046241d3271` (identifier) - category id
    + name: `Deposit certificates` (string,required) - category name. Max length is 500 characters
    + parent: `fa10a649-561d-4fa5-9f66-9c33fe8b94c1` (identifier) - parent category
    + picture: `cb765c04-9ed8-4c33-ac64-5ddf5420b3ac` (identifier) - category picture id
    + archive: false (boolean) - if certificate category was archived and can't be used in new certificates
        + Default: `false`

### Get all certificates categories [GET /certificates/categories{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `name,parent,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived certificate categories

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "748337fe-ac89-44a8-8f78-2046241d3271",
                    "name": "Deposit certificates",
                    "parent": null,
                    "archive": false
                },
                {
                    "id": "64c0b701-f79a-4dd6-97bc-9c5f9e7c8ebc",
                    "name": "Special certificates",
                    "parent": null,
                    "archive": false
                }
            ]

### Get certificate category by id [GET /certificates/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + id: `748337fe-ac89-44a8-8f78-2046241d3271` (identifier, required) - id of certificates category
    + fields: `name,parent,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                    "id": "9b6f7731-69a1-4c9d-844a-90d2f1e2a8ea",
                    "name": "Deposit certificates",
                    "parent": null,
                    "archive": false
            }

### Create new certificates category [POST /certificates/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "For men",
                "parent": null
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "8bc889be-1cfb-455c-a965-a98c4fbe9bb8"
            }

### Update certificates category [PUT /certificates/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `8bc889be-1cfb-455c-a965-a98c4fbe9bb8` (identifier, required) - id of certificates category
    + fields: `name,parent,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Offers for men",
                "parent": null
            }
            
+ Response 204

### Delete certificates category [DELETE /certificates/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `8bc889be-1cfb-455c-a965-a98c4fbe9bb8` (identifier, required) - id of certificates category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get certificates and categories tree [GET /certificates/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "archive": false,
                "categories":
                [
                    {
                        "id": "748337fe-ac89-44a8-8f78-2046241d3271",
                        "name": "Deposit certificates",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "5227a78f-fe20-4c1b-8136-a43c7d26f3d5",
                                "name": "Happy Valentines Day $100",
                                "location": "6f712b8e-1569-4104-833d-50a8f87bb09c",
                                "description": "Certificate for Happy Valentines Day contains $100 in deposit",
                                "barcode": "2782843833907",
                                "number": 1412,
                                "one_time": false,
                                "price": 100,
                                "total_sum": 100,
                                "used_sum": 50,
                                "left_sum": 50,
                                "services": [],
                                "expire_date": "2019-02-15T00:00:00.000Z",
                                "client": "fc3c5f2c-3dee-4b63-b555-1912482b3b69",
                                "status": "sold",
                                "archive": false
                            }
                        ]
                    },
                    {
                        "id": "64c0b701-f79a-4dd6-97bc-9c5f9e7c8ebc",
                        "name": "Special certificates",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "eed48dd4-3371-4e9d-bb06-ad8bc2b2bd3c",
                                "name": "End of Winter Massages",
                                "location": "6f712b8e-1569-4104-833d-50a8f87bb09c",
                                "description": "Certificate for several massages",
                                "barcode": "2782843833435",
                                "number": 1753,
                                "one_time": false,
                                "price": 300,
                                "total_sum": 100,
                                "used_sum": 50,
                                "left_sum": 50,
                                "services": [
                                    {
                                        "service": "eb2de366-7de6-4712-85ce-0e6d242770f5",
                                        "total_quantity": 3,
                                        "used_quantity": 2,
                                        "left_quantity": 1
                                    },
                                    {
                                        "service": "bd44104d-c2e4-44d3-ba9f-e5a38d1fb5aa",
                                        "total_quantity": 3,
                                        "used_quantity": 2,
                                        "left_quantity": 1
                                    }
                                ],
                                "expire_date": "2019-03-01T00:00:00.000Z",
                                "client": "fc3c5f2c-3dee-4b63-b555-1912482b3b69",
                                "status": "used",
                                "archive": false
                            }
                        ]
                    }                    
                ],
                "items":
                [
                ]
            }

# Group Groups

Groups keep information about group, such as name, group schedule this group, location prices. To simplify groups management, groups can be arranged in categories.

## Group [/groups]
<a name="groups"></a>

+ Attributes
    + name: `7.00 Classik Boxing (ELOHIN)` (string,required) - name of group. Max length is 200 characters
    + category: `3510c4ea-3451-4c77-a391-f4677135fc7f` (identifier) - id category
    + picture: `3510c4ea-3451-4c77-a391-f4677135fc7f` (identifier) - id picture
    + pictureUrl: `https://api.aihelps.com/v1/images/696320/3510c4ea-3451-4c77-a391-f4677135fc7f/picture` (string) - URL for getting a picture (readonly)
    + schedules (object) - schedule work professional for current group (field read only)
        + location: `d978322e-055c-43c7-bca3-beed1a07961c` (identifier) - id location, see [Locations](#locations)
        + professional: `6a261cc2-2646-4b95-aa79-9d527085cfdc` (identifier) - id employee [Employee](#employees)
    + public: true (boolean) - if group is available for online booking 
        Default: true
    + location_prices (array) - group prices for locations (each object in array defines prices for one location). See [Locations](#locations)
        + (object)
            + location: `a63bc703-c50a-4c6d-8af4-407d8bcf3128` (identifier) - location id, for which prices are defined 
            + price: 250 (number) - price for current location
    + archive: false (boolean) - if group was archived and can't be used in new sales
        + Default: `false`

### Get all groups [GET /groups{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + fields: `name,category,picture,pictureUrl,schedules,public,location_prices,archive` (array[string], required) - list of fields to return (separated by comma)
    + archive: false (boolean, optional) - get only archived or non archived groups

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
               {
                    "id": "88d492f3-a3d2-a47b-5727-32c726a49eb0",
                    "name": "20.30 Classic Boxing(LITVINENKO)",
                    "category": null,
                    "picture": "0b646349-e7be-4179-97bd-c7155caab923",
                    "pictureUrl": "https://api.aihelps.com/v1/images/696320/0b646349-e7be-4179-97bd-c7155caab923/picture",
                    "schedules": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "professional": 
                            [
                                "88d5933d-5d10-3650-74d8-4f87020a31eb"
                            ]
                        }
                    ],
                    "location_prices": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "price": 150
                        },
                        {
                            "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                            "price": 300
                        },
                        {
                            "location": "88d670d2-42c1-0942-2d52-3c107ce4ad0b",
                            "price": 250
                        }
                    ],
                    "archive": false
                },
                {
                    "id": "88d48be2-f6e6-e7a8-5bed-d7e145fd0eff",
                    "name": "7.00 Classik Boxing (ELOHIN)",
                    "category": null,
                    "picture": null,
                    "pictureUrl": "",
                    "schedules": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "professional": 
                            [
                                "88d48be2-4a37-13d9-5bed-d7e15bd486ee"
                            ]
                        }
                    ],
                    "location_prices": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "price": 150
                        },
                        {
                            "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                            "price": 200
                        },
                        {
                            "location": "88d670d2-42c1-0942-2d52-3c107ce4ad0b",
                            "price": 250
                        }
                    ],
                    "archive": false
                }
            ]    
            
### Get group by id [GET /groups{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `reports`

+ Parameters
    + fields: `name,category,picture,pictureUrl,schedules,public,location_prices,archive` (array[string], required) - list of fields to return (separated by comma)
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - id of group

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

               {
                    "id": "432c41bf-0cd7-4f13-83dd-a0c26ce29143",
                    "name": "20.30 Classic Boxing(LITVINENKO)",
                    "category": null,
                    "picture": "674f53c0-c623-464a-addc-173a35997f3e",
                    "pictureUrl": "https://api.aihelps.com/v1/images/696320/674f53c0-c623-464a-addc-173a35997f3e/picture",
                    "schedules": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "professional": 
                            [
                                "88d5933d-5d10-3650-74d8-4f87020a31eb"
                            ]
                        }
                    ],
                    "location_prices": 
                    [
                        {
                            "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                            "price": 150
                        },
                        {
                            "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                            "price": 200
                        },
                        {
                            "location": "88d670d2-42c1-0942-2d52-3c107ce4ad0b",
                            "price": 250
                        }
                    ],
                    "archive": false
                }            

### Create new group [POST /groups{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,category,picture,pictureUrl,schedules,public,location_prices,archive` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "20.30 Classic Boxing(LITVINENKO)",
                "category": null,
                "picture": "674f53c0-c623-464a-addc-173a35997f3e",
                "location_prices": 
                [
                    {
                        "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                        "price": 150
                    },
                    {
                        "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                        "price": 200
                    },
                    {
                        "location": "88d670d2-42c1-0942-2d52-3c107ce4ad0b",
                        "price": 250
                    }
                ],
                "archive": false
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "475a20e9-23ef-40ce-b2c4-c7dbcf230ebb"
            }

### Update group [PUT /groups/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `a1a9b38e-1627-4218-8beb-3c72230124bc` (identifier, required) - id group
    + fields: `name,category,picture,pictureUrl,schedules,public,location_prices,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
        
    + Body

            {
                "name": "20.30 Classic Boxing(LITVINENKO)",
                "category": null,
                "picture": "674f53c0-c623-464a-addc-173a35997f3e",
                "location_prices": 
                [
                    {
                        "location": "88d48bd0-ac0d-ae7f-42ac-dddc77408d38",
                        "price": 150
                    },
                    {
                        "location": "88d670d2-4294-2ce2-2d52-3c106bb0deca",
                        "price": 200
                    },
                    {
                        "location": "88d670d2-42c1-0942-2d52-3c107ce4ad0b",
                        "price": 250
                    }
                ],
                "archive": false
            }

+ Response 204 (application/json)

### Delete group [DELETE /groups/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - id of groups

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Group schedule [/groupschedules]

Each group can have different schedules in different locations. One `groupschedule` represent schedule for one specific group for one specific location.
Each such `groupschedule` represents information about lessons start times, duration and professional (or several).

+ Attributes
    + location: `5cbd3343-9bf4-47a6-a77a-0efe01d1f87d` (identifier) - location id, see [Locations](#locations)
        + Default: current location from token
    + group: `1a21565a-65a6-419d-94d1-7cea7255de03` (identifier) - group id, see [Groups](#groups)
    + group_name: `7.00 Classic Boxing` (string) - group name (read only)
    + professionals: `9adf5363-ed6e-4550-97a6-72e2144650dd` (array[identifier]) - professionals ids (separated by comma)
    + professionals_names: `Bob` (array[string]) - professionals names (read only)

### Get all group schedules [GET /groupschedules{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + fields: `location,group,group_name,professionals,professionals_names` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "location": "ee2eaacc-4775-40c1-99bc-78451ef66f54",
                    "group": "75a236ad-a957-4573-aac1-769e9bb65eb2",
                    "professionals": 
                    [
                        "490a1856-8889-4ec9-a03c-cf09ebea4f3d",
                        "e29bc115-3a65-4f91-81d0-fdc5ca9a8c67"
                    ],
                    "professionals_names": 
                    [
                        "Bob",
                        "John"
                    ]
                },
                {
                    "location": "15116d0b-ef50-4704-a08f-e2110355e6b0",
                    "group": "e5e23b9f-7d92-4518-8714-782edee848f3",
                    "professionals": 
                    [
                        "9e45d349-ee5c-4c37-8d80-866efe72dd8e",
                        "0e968a65-eb76-4966-90c7-d4744c4af333"
                    ],
                    "professional_names": 
                    [
                        "Kevin",
                        "David"
                    ]
                }
            ]

### Get group schedule by id [GET /groupschedules/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + fields: `location,group,group_name,professionals,professionals_names` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "location": "ee2eaacc-4775-40c1-99bc-78451ef66f54",
                "group": "75a236ad-a957-4573-aac1-769e9bb65eb2",
                "professionals": 
                [
                    "490a1856-8889-4ec9-a03c-cf09ebea4f3d",
                    "e29bc115-3a65-4f91-81d0-fdc5ca9a8c67"
                ],
                "professionals_names": 
                [
                    "Bob",
                    "John"
                ]
            }
            
### Create new group schedule [POST /groupschedules{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `location,group,group_name,professionals,professionals_names` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "location": "ee2eaacc-4775-40c1-99bc-78451ef66f54",
                "group": "75a236ad-a957-4573-aac1-769e9bb65eb1",
                "professionals": 
                [
                    "490a1856-8889-4ec9-a03c-cf09ebea4f3a",
                    "e29bc115-3a65-4f91-81d0-fdc5ca9a8c6c"
                ]
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c741"
            }

### Update group schedule [PUT /groupschedules/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - groups schedule id
    + fields: `location,group,group_name,professionals,professionals_names` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "location": "ee2eaacc-4775-40c1-99bc-78451ef66f54",
                "group": "75a236ad-a957-4573-aac1-769e9bb65eb1"
            }
            
+ Response 204

### Delete groups schedule [DELETE /groupschedules/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - group schedule id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Group categories [/groups/categories]

To simplify groups management, groups can be arranged in categories.

+ Attributes
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier) - category id
    + name: `TRX` (string, required) - category name. Max length is 500 characters
    + parent: `21664d40-a5a4-46cf-b728-a54b8c65af0b` (identifier) - parent category
    + picture: `baede13b-e983-4fed-b976-c11f14207afa` (identifier) - category picture id
    + archive: false (boolean) - if group category was archived and can't be used in new group
        + Default: `false`

### Get all groups categories [GET /groups/categories{?fields,archive}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent,picture,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived group categories

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                    "name": "TRX",
                    "parent": null,
                    "picture": "a9e3b96e-8a1d-4070-96f5-0c1b15349f00",
                    "archive": false
                },
                {
                    "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                    "name": "Yoga",
                    "parent": null,
                    "picture": "e8ae17d0-60f5-46b2-b761-8417367737be",
                    "archive": false
                }
            ]

### Get group category by id [GET /groups/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier, required) - id of groups category
    + fields: `name,parent,picture,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                "name": "TRX",
                "parent": null,
                "picture": "2b604ce8-d035-4d10-8af0-18fc7123b29d",
                "archive": false
            }

### Create new group category [POST /groups/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent,picture,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Other",
                "parent": null,
                "picture": "1ea5b057-5a79-4180-a557-bafbfb37cd70",
                "archive": false
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update group category [PUT /groups/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of groups category
    + fields: `name,parent,picture,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Box",
                "parent": null,
                "picture": "1ce07b1d-5a1e-411e-b9e9-3f1915c1eade"
            }
            
+ Response 204

### Delete groups category [DELETE /groups/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of groups category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get groups and categories tree [GET /groups/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "archive": false,
                "categories":
                [
                    {
                        "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                        "name": "TRX",
                        "parent": null,
                        "archive": false
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "88d51648-c8c9-23bc-701d-2a562dade0f9",
                                "name": "TRX Day",
                                "category": "88d6719a-1e3b-ddd0-4a7d-aadf0d39db2b",
                                "picture": null,
                                "archive": false
                            },
                            {
                                "id": "88d5a478-ae15-8084-3b46-d5184883e3c5",
                                "name": "TRX 19:00",
                                "category": "88d6719a-1e3b-ddd0-4a7d-aadf0d39db2b",
                                "picture": null,
                                "archive": false
                            },
                            {
                                "id": "88d51648-fc5b-6266-701d-2a5641936e3d",
                                "name": "TRX for Mam's 16:00",
                                "category": "88d6719a-1e3b-ddd0-4a7d-aadf0d39db2b",
                                "picture": null,
                                "archive": false
                            }
                        ]
                    },
                    {
                        "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                        "name": "Yoga",
                        "parent": null,
                        "archive": false,
                        "categories":
                        [
                        ],
                        "items":
                        [
                        ]
                    }                    
                ],
                "items":
                [
                ]
            }

## Group lesson [/grouplessons]
<a name="grouplessons"></a>

Group lesson represents one lesson of specific group. Info about group lesson include lesson start date, duration, group and professional.

+ Attributes
    + date: `2019-01-01T10:10:10.000Z` (datetime, required) - group lesson start date
    + duration: 60 (number) - lesson duration (greater than zero)
        + Default: 60
    + location: `e77673a6-1f77-4e22-b19a-9d5fadfc83c6` (identifier) - location id [Locations](#locations)
        + Default: current location from token
    + group: `ac9868df-5c23-4bf3-b422-998135201051` (identifier) - group id [Groups](#groups)
    + group_name: `Fitness` (string) - group name (read only)
    + group_price: 500 (number) - price for visiting one lesson
    + group_picture: `baa01c14-ede4-4e05-aa58-06344ec15030` (identifier) - group picture id (read only) [Pictures](#pictures)
    + professional: `d0f85061-e980-4444-a77a-3295b32b7d27` (identifier) - professional who conduct lesson
    + professional_name: `Bob` (string) - professional name (read only)    
    + assistant: `d0f85061-e980-42b4-a77a-3295b32bcd1c` (identifier) - assistant who conduct lesson
    + assistant_name: `John` (string) - assistant name (read only)
    + hall: `d0f85061-e980-42b4-a77a-3295b32bbd2f` (identifier) - hall id [Halls](#halls)
    + filled_completely: false (boolean) - if group lesson is filled completely - max number of clients reached (read only)
    + descriptionPlaintext: `the best group ever` (string) - description of a group as plain text without html tags

### Get all group lessons [GET /grouplessons{?fields,from,to,location,group,professional,public,has_location_prices,publicProfessional}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only group, group_name, group_picture, location, group_price, date, duration, professional, professional_name, filled_completely, descriptionPlainText fields), `client_access_token` (only group, group_name, group_picture, location, group_price, date, duration, professional, professional_name, filled_completely, descriptionPlainText fields), `reports`


+ Parameters
    + fields: `date,duration,location,group,group_name,group_price,group_picture,professional,professional_name,assistant,assistant_name,hall,filled_completely,descriptionPlaintext` (array[string], required) - list of fields to return (separated by comma).
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - return only lessons started at date or later
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - return only lessons started at date or earlier
    + location: `12d4b305-e198-f02c-2743-cca93cb16baa` (array[identifier], optional) - get only lessons in given location (several locations can be separated by comma)
    + group: `8a6b32dd-53bb-4181-b7e5-1afb4e74f0f6` (array[identifier], optional) - get only given groups lessons (several groups can be separated by comma)
    + professional: `5829ad23-0b78-41ed-a3ef-240725e31262` (array[identifier], optional) - get only lessons with given professionals (several professionals can be separated by comma)
    + public: true (boolean) - if group lesson is available for online booking
    + has_location_prices: true (boolean) - get only group lesson where has price
    + publicProfessional: true (boolean) - get group lessons with no professionals or with at least one active professional

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c1efb8e7-00ec-4513-8745-739169e7b251",
                    "date": "2019-01-01T10:00:00.000Z",
                    "duration": 60,
                    "location": "23c32b29-f871-448a-8646-d235a9b40d22",
                    "group": "c20d2ff7-379b-4f02-b69d-c0c3aae52a02",
                    "group_name": "Pilates",
                    "group_price": 600,
                    "group_picture": "90044752-ebb8-4de6-8202-25533dcdb7a5",
                    "professional": "a64742ef-f39f-4828-a688-6efc44041b06",
                    "professional_name": "Kevin",
                    "assistant": "b1cac8b2-ebb8-fdef-b212-fcf13dcb1ccc",
                    "assistant_name": "Bob",
                    "hall": "b1cac8b2-ebb8-fdef-b212-ccfd2dcb1b1a",
                    "filled_completely": false,
                    "descriptionPlaintext": "the best group ever"
                },
                {
                    "id": "c1efb8e7-00ec-4513-8745-739169e7b452",
                    "date": "2019-01-01T10:00:00.000Z",
                    "duration": 60,
                    "location": "e9291108-f440-4822-a0d7-880fb5d09849",
                    "group": "efcba4e6-c2c2-4a18-9ffb-7288caa91de2",
                    "group_name": "Pilates",
                    "group_price": 600,
                    "group_picture": "de570138-91fe-4427-8ee6-c950b6d2bb17",
                    "professional": "cb201d83-c4e2-44fb-9cac-5bfa145d699d",
                    "professional_name": "Bob",
                    "assistant": "b1cac8b2-ebb8-fdef-b212-fcf13dcb1ccc",
                    "assistant_name": "Bob",
                    "hall": "b1cac8b2-ebb8-fdef-b212-ccfd2dcb1b1a",
                    "filled_completely": true,
                    "descriptionPlaintext": "the best group ever"
                }
            ]

### Get group lesson by id [GET /grouplessons/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module` (only date, duration, group, group_name, group_price, group_picture, professional, professional_name fields), `reports`

+ Parameters
    + id: `adac3979-cf71-47fa-9a39-5f366fd1378e` (identifier, required) - group lesson id
    + fields: `date,duration,location,group,group_name,group_price,group_picture,professional,professional_name,assistant,assistant_name,hall,filled_completely,descriptionPlaintext` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "adac3979-cf71-47fa-9a39-5f366fd1378e",
                "date": "2019-01-01T10:00:00.000Z",
                "duration": 60,
                "location": "23c32b29-f871-448a-8646-d235a9b40d22",
                "group": "c20d2ff7-379b-4f02-b69d-c0c3aae52a02",
                "group_name": "Pilates",
                "group_price": 600,
                "group_picture": "90044752-ebb8-4de6-8202-25533dcdb7a5",
                "professional": "1234c7b2-ebb8-fdef-b212-fccc3dc9bca1",
                "professional_name: "John",
                "assistant": "b1cac8b2-ebb8-fdef-b212-fcf13dcb1ccc",
                "assistant_name": "Bob",
                "hall": null,
                "filled_completely": true,
                "descriptionPlaintext": "the best group ever"
            }

### Create new group lesson [POST /grouplessons/{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

**Custom errors:**
1. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time

+ Parameters
    + fields: `date,duration,location,group,group_name,group_price,group_picture,professional,professional_name,assistant,assistant_name,hall,filled_completely,descriptionPlaintext` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "date": "2019-01-01T10:00:00.000Z",
                "duration": 60,
                "location": "23c32b29-f871-448a-8646-d235a9b40d22",
                "group": "c20d2ff7-379b-4f02-b69d-c0c3aae52a02",
                "group_price": 600,
                "professional": "a64742ef-f39f-4828-a688-6efc44041b06",
                "filled_completely": false
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update group lesson [PUT /grouplessons/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

**Custom errors:**
1. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - group lesson id
    + fields: `date,duration,location,group,group_name,group_price,group_picture,professional,professional_name,assistant,assistant_name,hall,descriptionPlaintext ` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "duration": 60,
                "location": "23c32b29-f871-448a-8646-d235a9b40d22",
                "group": "c20d2ff7-379b-4f02-b69d-c0c3aae52a02",
                "group_price": 600
            }
            
+ Response 204

### Delete group lesson [DELETE /grouplessons/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - groups lesson id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Group appointment [/group_appointment]

Represents preliminary booking for group lesson for some client.

+ Attributes
    + id: `8a14c75f-a0b4-4acd-afa1-cb2efff93d2b` (identifier) - group appointment id
    + client: `601f3b0f-8637-49ff-b699-a34dd1f378a2` (identifier, optional) - client id [Clients](#clients). Íf client token is used, `client` should not be passed, client id will be taken from client token.
    + group_lesson: `33147ead-b003-4465-97e9-bb320a5f03b1` (identifier, required) - group lesson id [Groups](#grouplessons)

### Get all group appointments [GET /group_appointments{?fields,client}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id)

+ Parameters
    + fields: `client,group_lesson` (array[string], required) - list of fields to return (separated by comma).
    + client: `f4b1a3b8-83c3-4bda-aaf4-8589c945dcc3` (string, optional) - get appointments only for given client (several clients can be separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "8a14c75f-a0b4-4acd-afa1-cb2efff93d2b",
                    "client": "601f3b0f-8637-49ff-b699-a34dd1f378a2",
                    "group_lesson": "33147ead-b003-4465-97e9-bb320a5f03b1"
                },
                {
                    "id": "40d89d84-ac9a-4965-8121-086ef1f2dd7c",
                    "client": "985d5a2d-819e-4ae0-984f-d14569637ac4",
                    "group_lesson": "c2eb6f56-713c-4d3b-b98c-3ecabf013082"
                },  
            ]

### Get group appointment by id [GET /group_appointments/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id)

+ Parameters
    + id: `adac3979-cf71-47fa-9a39-5f366fd1378e` (identifier, required) - group lesson id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "adac3979-cf71-47fa-9a39-5f366fd1378e",
                "client": "601f3b0f-8637-49ff-b699-a34dd1f378a2",
                "group_lesson": "33147ead-b003-4465-97e9-bb320a5f03b1"
            }

### Create new group appointment [POST /group_appointments]

**Authorization:** `Database`, `Employee`, `Client`

**Scope:** `full`, `client_access_token` (`client` field should match client id), `clients_module`

**Custom errors:**
1. [ALREADY_EXISTS](#error-ALREADY_EXISTS) - client already booked on group lesson or max clients in group reached (see error details)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Language: en

    + Body

            {
                "client": "60444aac-8c98-4725-bef3-4e6868b95833",
                "group_lesson":"881f83ca-987d-4c58-97ae-a5796c4cfe27"
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bb765afbc321",
            }

### Delete group appointment [DELETE /group_appointment/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id)

+ Parameters
    + id: `d291df13-0782-44fc-a409-18df9dbca888` (identifier) - group appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

# Group Pictures

All price pictures (services, products, cards, etc.) are stored in pictures. Each item contains picture informationand picture itself.

## Pictures [/pictures]
<a name="pictures"></a>

+ Attributes
    + name: `Personal training` (string,required) - picture name. Max length is 500 characters
    + category: `d32bfd16-2d63-4937-acfe-fada1a064fe9` (identifier) - picture category id

### Get all pictures information [GET /pictures{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + fields: `name,category` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "7d11956f-64e8-454e-af32-736dbcb9cb31",
                    "name": "Fitness",
                    "category": "ebb2776c-14d3-4409-9bfc-594cc6722a1c"
                },
                {
                    "id": "239e1f2a-5fee-4b9d-9efd-481fd813f942",
                    "name": "Training program",
                    "category": "0a27b0a8-1ce3-45f6-b0b8-6e6f6ce86f8b"
                }
            ]

### Get picture information by id [GET /pictures/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + id: `3ed3c526-3b6b-46da-9976-6fc3d52400a3` (identifier, required) - picture id
    + fields: `name,category` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "51fbd5c0-6b29-456e-8d9f-779da67e3cdf",
                "name": "Hall rental",
                "category": "ebb2776c-14d3-4409-9bfc-594cc6722a1c"
            }

### Get picture image by id [GET /pictures/{id}/image{?width,height,resize,access_token}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + id: `3ed3c526-3b6b-46da-9976-6fc3d52400a3` (identifier, required) - id picture.
    + width: 100 (number, optional) - image width needed
    + height: 100 (number, optional) - image height needed
    + resize: `fit` (enum[string], optional) - type of resize for image
        + Default: `fit`
        + Members
            + `fit`
            + `fit_center_transparent`
            + `stretch`
            + `crop`
    + access_token: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (string, optional) - token used for API requests

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (image/jpeg)

    + Body

## Picture categories [/pictures/categories]

To simplify pictures management, pictures can be arranged in categories.

+ Attributes
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier) - category id
    + name: `Groups` (string,required) - category name . Max length is 500 characters
    + parent: `2c11a649-561d-1fd5-9f6d-9c35f18194cc` (identifier) - parent category

### Get all pictures categories [GET /pictures/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                    "name": "Groups",
                    "parent": null
                },
                {
                    "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                    "name": "Loreal products",
                    "parent": null
                }
            ]

### Get picture category by id [GET /pictures/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier, required) - id of pictures category
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                "name": "Groups",
                "parent": null
            }

### Create new picture category [POST /pictures/categories{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Other",
                "parent": null
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update picture category [PUT /pictures/categories/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of pictures category
    + fields: `name,parent` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Stylists services",
                "parent": null
            }
            
+ Response 204

### Delete pictures category [DELETE /pictures/categories/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of pictures category

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Get pictures and categories tree [GET /pictures/tree{?fields,categories_fields,empty_categories}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,parent` (array[string], required) - list of fields to return (separated by comma)
    + categories_fields: `name,category` (array[string], required) - list of categories fields to return (separated by comma)
    + empty_categories: `` (string, optional) - if empty categories should be returned (otherwise empty categories are not returned)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": null,
                "name": "",
                "parent": null,
                "categories":
                [
                    {
                        "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                        "name": "Groups",
                        "parent": null
                        "categories":
                        [
                        ],
                        "items":
                        [
                            {
                                "id": "7d11956f-64e8-454e-af32-736dbcb9cb31",
                                "name": "Fitness",
                                "category": "c9edc51c-a95a-4f9c-84fd-67022b957d79"
                            },
                            {
                                "id": "239e1f2a-5fee-4b9d-9efd-481fd813f942",
                                "name": "Training program",
                                "category": "c9edc51c-a95a-4f9c-84fd-67022b957d79"
                            }
                        ]
                    },
                    {
                        "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                        "name": "Loreal products",
                        "parent": null
                        "categories":
                        [
                        ],
                        "items":
                        [
                        ]
                    }                    
                ],
                "items":
                [
                ]
            }

# Group Appointments

Appointment is planned visit for specified client (employee or guest also can be clients) on specified date into specified location.

Appointments are one of the most complex objects in API: they have complex validation, several unique checks and sending messages on different events.
While updating appointment, try to call update appointment endpoint and pass there all services, products, etc., not updating each appointment sub-item one-by-one,
cause in such situation all these operations and validations will be called each time.

## Appointment [/appointments]
<a name="appointments"></a>

Description of appointment states:
- `reserved` - temporary state used for editing appointments. Main task of such appointments - reserve professionals/halls/resources time while editing appointment and simplifies process of editing appointment. Important moment: time reservation is valid only 15 minutes after `created_date`. If more needed, update `created_date`. Appointment state can't be changed to this state - only new appointment with this state can be created.
- `planned` - default appointment state after creation. Appointment is valid.
- `confirmed` - same as `planned`, but user confirmed his visit.
- `cancelled` - appointment was cancelled and saved only for history reasons (also state can be changed to other states).

Appointment items can be payed separately, so there is no appointment `payed` state. Only appointments in `planned` or `confirmed` states can contain payed items (paying for item automatically switch appointment state from `planned` to `confirmed`). Payed items are items with `sale` field not equals to null.
Appointments and appointment items only for last 2 month are available. If you need to get earlier data, you'll have to contact us individually.

**Validations**

+ For specified client (or employee) on specified date and specified location only one appointment can exists in states `planned` or `confirmed`. If you try to add appointment with client/employee + date + location and state `planned` or `confirmed` or change appointment fields so there is another appointments with same fields already exists - that another appointment will be instantly merged with specified appointment (so always only one appointment with location+date+client/employee triple exists). Fields for these appointments will be merged (values from merged appointment have higher priority) and appointment items will be combined. To simplify process and make merge recoverable you can use `reserved` appointment for editing existing one - read details below in this section. If no `client`/`employee` set for appointment, check will not be done - there can be many appointments with no client/employee for one date and location - all these "guests" are treated as different ones. Also, there can be many appointments in `cancelled` or `reserved` state for one client/employee + day + location - they all will be OK until trying to change status to `planned` or `confirmed`.
+ Valid appointments are all appointments in state `planned`, `confirmed` and appointments in state `reserved` if `createDate` (UTC) is less than 15 minutes from now (or `createDate` is any date in future, if manually set). Invalid appointments are all appointments in `cancelled` state and appointments in state `reserved` if `createDate` (UTC) is more than 15 minutes ago from now. All valid appointments occupy professional/assistant/hall/resources time and attempt to set new valid appointment or change existing (changing appointment/appointment service `date`, `start`, `duration`, `professional` etc. or making invalid appointment valid via changing `state` or `createDate`) that intersect with existing one will cause 409 "Time conflict" error, providing detailed description on conflicted items. If user really sure he wants to ignore conflicts, you can pass `force=true` parameter and corresponding check will be skipped.
+ For all types of appointments there is another check: each professional is checked if appointment services are outside professionals work time (same check for assistants). If appointment service time is outside professional work time (even partly), 409 "Time conflict" error raised, providing detailed description on conflicted items. If user really sure he wants to ignore conflicts, you can pass `force=true` parameter and corresponding check will be skipped. If you set `force=true`, employee work time will be expanded to cover all appointment services.
Two last validations (time conflict): mention that checks are executed only if at least one of described parameters changed, so if appointment was saved with `force` parameter and now you changed, for example, `notifySms` fields, no checks will be done and no errors happen. If you set `force=true` parameter, 409 error will be returned, but result will conflict `timeConflicts` field with list of conflicts.

+ All payed appointment items can't be changed or deleted.
+ Only valid appointments can contain payed items, so you can't change state for `confirmed` appointment with payed items. Also, you can't change next appointment fields, if it has at least one payed item: `date`, `location`, `client`, `employee` and `state` (as mentioned above). And such appointements can't be removed.

+ For each appointment only one connected reserve appointment can exist (see Appointment Edit section). Because reserve appointments usually used for editing appointment, this really means that one appointment can be edited only by one person at once. Otherwise 409 "Already exists" error raised. This error can happen in 3 situations: you create new reserve appointment from existing one (`copy` parameter), where existing one already have another reserve; while insert or update reserve appointment you provide new date + location + client/employee, for which appointment already exists and has attached reserve.

**Sending messages**

Adding/updating/deleting appointments or changing appointment services also triggers some messages sent to client/employee or professional:
+ "New appointment created" message is sent on creating new appointment in `planned` or `confirmed` states or changing state of existing appointment from `reserved` or `cancelled` to `planned` or `confirmed` states.
+ "Appointment changed" message is sent on changing appointment `date`, `location`, appointment service `first_start` or `first_professional` fields (other fields like `service` are ignored and mo message sent even in case they can appear in message text) while appointment is in `planned` or `confirmed` states
+ "Appointment cancelled" message is sent on deleting appointment in `planned` or `confirmed` states or changing it state to `reserved` or `cancelled`

`first_start` is smallest field `start` from all appointment services, `first_professional` - `professional` field from corresponding appointment service. These 2 values also will be used in message text.

These messages are sent only if client/employee set, his number provided and `do_not_send_sms_notification` option not set for client. Changing `client`/`employee` is treated like deleting appointment for old client and creating new one for new client, thus 2 messages will be sent: "Appointment cancelled" for old client and "New appointment created" for new one.

Depending from program settings and `notifySms` client can receive one or several notification messages "Reminder about appointment" at specified number of hours or days before `first_start`. Changing appointment services can cause `first_start` change - in this case all reminder messages not sent yet will be cancelled and new messages planned to send (only if it is still enough time for that message - if options set to send one message in 1 day and another in 2 hours, but appointment changed 4 hours before `first_start`, only second notification will be planned to send). Along with "Appointment cancelled" message all reminder messages not sent yet will be cancelled.

There are some other messages (like request for feedback or periodical services reminder), but they are triggered by other endpoints (like purchase) and will be described there.

If `NotifyProfessionalsSMS` option is set, `professional` and `assistant` are also notified about appointment via message. They receive notification "You have new appointment" on creating new appointment in `planned` or `confirmed` states or changing state of existing appointment from `reserved` or `cancelled` to `planned` or `confirmed` states (for appointments where appointment service exists for current professional or assistant); adding appointment service with professional or assistant to existing appointment in `planned` or `confirmed` states or changing appointment services so that first appointment service start or total duration (last appointment service end) for professional or assistant changed.

**Appointment edit**

Typical process of creating new/edit existing appointment dialog should be following:
1) If edit existing appointment, make local copy of current appointment with all items (or create empty appointement info for new appointment).
2) On any services items change: if not yet created, create separate `reserve` appointment and save all new service items to it. Any change to service items should be reflected in this reserve appointement and saved to API. If you reserve error about conflict, highlight conflicted items in interface with detailed description. To skip conflicts with original appointment (while editing existing appointment), in reserve appointment save only time not existing in original appointment.
3) Do not forget to update reserve `createDate` each 10-14 minutes (in 15 minutes you reserved time will become free and someone else can occupy it)
4) All payed items in appointment can't be changed. Also, you can't change date, location, client and employee if at least one payed item exists. And can't remove appointment.
5) If user change date, location or client/employee, you should check for valid appointment with these new date, location and client/employee. If exists, show combined information for both your new/existing appointment with changes you made + information from appointment with new requisites. If user change one of these 3 parameters once more, all changes to appointment with new requisites should be cancelled and check if other appointment with your new requisites exists. And process repeated.
6) If user wants to exit without saving changes, you need to remove reserve appointment, if created.
7) If user wants to exit and save changes, you save all changes to one appointment and try to save it. If conflicts found, show all conflicts to user. User can either refuse to save (see previous item) or force save with conflicts.

This process is pretty complex. To make it easier, we implemented several helpers based on reserve appointments, making edit appointment simplier. So, previous process now looks so:
1) If you want new appointment, create new one with `reserved` state. If you want to edit existing appointment, create new one with `reserved` state and pass `copy={id}` parameter in query string with id of appointment you want to edit. Do not forget to pass `"state": "reserved"` in body in both cases. You will receive id of newly created reserved appointment.
2) Any change to appointment or appointment items - just save changes to your reserve appointment (except setting `state` field to some other than `reserve` - this field usage is reserved; save state somewhere else). Pass `force=true`, so conflicts will be returned, but will not cause an error. If any, highlight conflicted items in interface with detailed description.
3) Do not forget to update reserve `createDate` each 10-14 minutes (in 15 minutes you reserved time will become free and someone else can occupy it). Updating `createDate` left - user can still just close page with appointment edited. But If he reopens it even in several hours, you can update `createDate` and receive back all conflicts if found.
4) Still the same: all payed items in appointment can't be changed. Also, you can't change date, location, client and employee if at least one payed item exists. And can't remove appointment.
5) If user change date, location or client/employee, just update corresponding fields of reserve appointment and save to API. All complex logic of finding existing appointment, merging with existing data (and un-merging from from previous appointment) will be done in API. After changing one of these fields you need to reload all appointment fields and items. Merging appointments with different states can lead to situation you don't know what state to show in appointment edit dialog: use `actualState` field instead of `state` (remember, state always be `reserved` for reserved appointment). If you try to add appointment with client/employee + date + location and state `planned` or `confirmed` or change appointment these fields so there is another appointments with same fields - current appointment will be merged with already existing one and `id` of existing appointment will be used as `id` of combined appointments. All fields will be merged (for example, string fields combined, bool fields - non-default value will be taken and so on). Please mention: in case of updating existing appointment and changing `client`/`employee`, `date`, `location` or `state`, `id` of your appointment can change (you will receive other `id` in responce than you provided for update, `id` field is always returned for updating appointments). If no `client`/`employee` set for appointment, no checks and merging will happen - there can be many appointments with no client/employee for one date and location - all these "guests" are treated as different ones. Also, there can be many appointments in `cancelled` or `reserved` state for one client/employee + day + location - they all will be OK until trying to change status to `planned` or `confirmed`.
6) If user wants to exit without saving changes, just remove reserve appointment.
7) If user wants to exit and save changes, just set `state` field to state you want. If conflicts found, show all conflicts to user. User can either refuse to save (see previous item) or force save with conflicts. If it was creating new appointment, appointment state will be changed to provided and merged appointment will be deleted. If it was editing existing appointment, all reserved appointment fields and items will be copied to edited appointment (full overwrite) and both reserved and merged appointment will be deleted.
8) We strictly recommend not to delete appointments, but cancel them (so, it remains as cancelled in history with information about cancellation). Deleting this cancelled appointment can be done somewhere (for example, in client history), but not in edit appointment dialog (in this case you need to delete reserve, edit and merged appointment-  but, once more, we do not recommend to do so).

**Additional validations**

Except validations described earlier, there are some more complex validations, usually taking into account two or more objects and hard to predict and fix forehead. That are either depends from settings or relations between objects in different tables. For example: provided product is not more available on provided storage, provided professional can't handle provided service anymore, card activation date can be only month start according to settings etc. There are two problems here:
1) smaller one: these dependencies between objects can be changed between data loaded and changes to appointment made. Requires subscribing to changes + additional checks on client but feasible to prevent in many cases (not all though, you can't subscribe to settings changes).
2) bigger one: you can create a valid appointment with valid appointment items, and save it to database. Then someone can change services, products, settings, etc. in a way that your appointment becomes invalid. So next time you open it (in a week or two, for example) it is already invalid.
If `force` parameter is not provided, such validation raise 400 Bad Request error with type "APPOINTMENT_VALIDATION" and list of validation errors on such complex validations. If `force=true`, list of such failures is returned on each insert/update/get appointment call if "validations" field declared in `fields` list. So, such appointments can be stored in database. If any such validation fails found, API returns you list of fails in `validations` error (additional field in result, absent if no fails but always present if any fails found - you can't and don't need to set it in `fields` parameter). But item can't be sold if some validations failed - you will receive 400 Bad Request error if you try to pay for such invalid item (`/sales/purchase` endpoint).
Each validation fail structure is described in `validation` field.

We propose to show "Description" (localized or not) with substituted fields (service/product name etc.) to client and do not allow paying for items if any validation fails exists.

Please take into account that these validations can be returned for items you didn't changed in request in case already stored in database items already fail validations. For example, you update appointment changing `services` field, but validations fails for `products` will also be returned.
Also, important moment: if you insert/update/get appointment item and requested "validations" field in `fields`, validations will be returned for all items, not only for requested one. This is needed because sometimes items can depend from another items and change in one can raise validation result in others.

**Update appointment items**

Appointment services, products, cards, certificates and dentures can be updated (for existing appointment) in 3 different ways:

1) using specific endpoints POST '/appointments/items', PUT/DELETE '/appointments/items/{item id}' (where `items=services/products/cards/certificates/dentures`) - in this way you can add, update or delete item. Benefit of this method - you can update/delete item without knowing appointment id, otherwise we recommend use method 3.
2) full rewrite all items of specific type: PUT '/appointments/{appointment id}', corresponding array ("services"/"products"/"cards"/"certificates"/"dentures") is array of items. You can't provide ids in this case - they will be automatically generated.
3) update items of specific type: PUT '/appointments/{appointment id}', corresponding array ("services"/"products"/"cards"/"certificates"/"dentures") is array of items, each having property `action=insert/update/delete`. Specifying `action=insert` item will be added (id will be generated), `action=update` - item will be updated (id is required), `action=delete` - item will be removed (id is required, all other properties ignored). This method has big advantage over previous - it does not change items you did not specify, thus more safe for concurrent changes.

+ Attributes
    + state: `planned` (enum[string]) - appointment current state, see details in each state description
        + Default: `planned`
        + Members
            + `reserved` - client or employee is in process of creating new appointment. Time for specific professional is marked as busy (so no one else can book this time), but final decision not yet submtted. May be comments, some services will be added or time changed. Appontments in this state are valid only for 15 minutes after `createDate`, after that become old and booked time can be used by other appointment. If you want to extend validity time of reserve appointment, just update `createDate`.
            + `planned` - default state, appointment is valid, waiting client to confirmed
            + `confirmed` - client was succesfully notied about upcoming visit
            + `cancelled` - appointment was cancelled by client or salon, not valid any more
    + actualState: `planned` (enum[string]) - state of original appointment while editing reserved appointment
        + Default: `planned`
        + Members
            + `reserved` - client or employee is in process of creating new appointment. Time for specific professional is marked as busy (so no one else can book this time), but final decision not yet submtted. May be comments, some services will be added or time changed. Appontments in this state are valid only for 15 minutes after `createDate`, after that become old and booked time can be used by other appointment. If you want to extend validity time of reserve appointment, just update `createDate`.
            + `planned` - default state, appointment is valid, waiting client to confirmed
            + `confirmed` - client was succesfully notied about upcoming visit
            + `cancelled` - appointment was cancelled by client or salon, not valid any more
    + date: `2020-01-01` (datetime) - appointment date (should not contain time part - error if hours, minutes or seconds are not zeros)
    + location: `e77673a6-1f77-4e22-b19a-9d5fadfc83c6` (identifier) - location id [Locations](#locations)
        + Default: current location from token
    + client: `f9c254bd-069a-4d4c-87b6-e981cbb966d1` (identifier) - client id (null means guest or employee). [Clients](#clients) If client token is used, default value is client for whom token was issued. Only one of properties `client` and `employee` can be set.
    + clientName: `John Michael Doe` (string) - full client name
    + clientFirstName: `John` (string) - client first name
    + clientMiddleName: `Michael` (string) - client middle name
    + clientLastName: `Doe` (string) - client last name
    + employee: `6e37a5ad-8809-4256-b46c-d8daaae7fd6e` (identifier) - employee id (null means client or guest) [Employees](#employees). Only one of properties `client` and `employee` can be set.
    + employeeName: `John Michel Brown` (string) - full employee name 
    + employeeFirstName: `John` (string) - employee first name
    + employeeMiddleName: `Michael` (string) - employee middle name
    + employeeLastName: `Brown` (string) - employee last name
    + receptionist: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - receptionist id (employee who booked client)
    + clientsModule: true (boolean) - if appointment was booked via client module
    + clientsModuleConfirmed: true (boolean) - if appointment, booked via client module was confirmed by receptionist (if confirmation needed)
    + notifySms: `default` (enum[string]) - number of minutes before appointment start when notification sms should be sent to client
        + Members
            + `default` - according to common settings for all appointments
            + `none` - no notifications
            + 60
            + 120
            + 180
            + 240
            + 1440
    + cancelDate: `2019-01-01T11:00:00.000Z` (datetime) - appointment cancel date in UTC, read only (if `state` is `cancelled` or null otherwise) - this field is set automatically for `cancelled` state and cleared on changing to any other state.
    + cancelReason: `Plans changed` (string) - appointment cancel reason (if `state` is `cancelled`) - this field can be set only for `cancelled` state, on changing to any other state field automaically clears. Max length is 15000 characters
    + cancelReceptionist: `8039d005-f519-425f-8c09-11fe7cb25738` (identifier) - employee, who cancelled appointment (if `state` is `cancelled`) - this field can be set only for `cancelled` state, on changing to any other state field automaically clears.
    + receptionist: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - receptionist id (employee who booked client)
    + controlManager: `f627c0d9-904b-448a-9174-26ca88f3f353` (identifier) - control manager id (manager that controls client visit, usually in big locations)
    + createDate: `2019-01-01T10:00:00.000Z` (datetime) - date and time appointment was created (in UTC)
    + important: false (boolean) - visit is important (informational field)
    + color: '#ff0000' (color) - visit color in schedule
    + `services` (array) - services for appointment. See [Appointment Service](#appointments_services)
        + (object)
            + start: `10:10:10.000Z` (datetime) - appointment service start time (date getting from appointment)
            + duration: 60 (number) - appointment duration (greater than zero)
                + Default: 60
            + professional: `5acc652b-c762-4eda-983b-59cc86ea4481` (identifier) - profesional id (who provides services) (can be null)
            + assistant: `4357be34-aa4b-4711-8a4d-34b3059b5b43` (identifier) - assistant id (helps professional to provide services)
            + service: `4b244443-90d3-4182-918a-fe3ac4a03688` (identifier) - service id (can be null)
            + quantity: 2 (number) - how much quantity service will be provided (on default equals 1. Value equals or greater than zero)
            + hall: `4b244443-90d3-4182-918a-fe3ac4abc681` (identifier) - hall id (can be null)  
            + customMaterials: false (boolean) - materials can be taken from the service or from professional who provided the service
            + recommendedBy: `4b244443-90d3-4182-918a-e73c491bf6f2` (identifier) - who recommended service
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cab11` (identifier,required) - appointment id
    + `products` (array) - product for services. See [Appointment Service Product](#appointments_products)
        + (object)
            + product: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - product id.
            + storage: `13adf8fb-6304-4e1b-a036-1c092b9e82e2` (identifier,required) - storage id.
            + quantity:  3 (number) - quantity's product (on default 1. Value equals or greater than zero).
            + recommendedBy: `4b244443-90d3-4182-918a-e73c491bf6f2` (identifier) - who recommended product.
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cfb11` (identifier,required) - appointment id.
            + sale: `13adf8fb-6304-4e1b-a036-1c092b91cfb11` (identifier) - sale id.
    + `certificates` (array) -  items service for services.
        + (object)
            + certificate: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - certificate id.
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cb11` (identifier,required) - appointment id.
            + sale: `13adf8fb-6304-4e1b-a036-1c092b91cfb11` (identifier) - sale id.
    + `dentures` (array) -  item dentures for appointment. See [DentureCard](#denturecards)
        + (object)
            + denture: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - card id
            + laboratory: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - company id
            + technican: `some text` (string) - string some
            + comments: `some text for comment` (string) - string some
            + teeth: `6` (string, required) - teeth name
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91ca11` (identifier,required) - appointment id
            + sale: `13adf8fb-6304-4e1b-a036-1c092b91cfb11` (identifier) - sale id.
    + `cards` (array) -  item cards for appointment. See [Appointment Card](#appointments_cards)
        + (object)
            + card: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - card id
            + cardNumber: 20 (number) - card number
            + activationDate: `2019-01-01T10:10:10.000Z` (datetime) - activation date
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cv b11` (identifier,required) - appointment id
    + `feedback` (array) - client feedback on appointment. For one professional one feedback can be made.
        + (object)
            + id: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - feedback id. Can be used to update/delete feedback via `feedbacks/{id}` endpoint.
            + professional: `13adf8fb-6304-4e1b-a036-1c092b9e82e2` (identifier,required) - professional, for whom feedback was left
            + date: `2019-01-01T11:27:14.000Z` (datetime) - date feedback was left
            + rating:  5 (number) - client feedback from 1 to 5 or 0 if not rated
            + text: `Good job!` (string) - client feedback message
            + published: `true` (boolean,required) - if feedback is visible for all (default true)
    + hasAnySales: true (boolean) - if appointment has at least one appointment item with a sale
    + clientInside: false (boolean) - if client of the appointment is present on the location of the appointments. false if appointment is for a guest.
    + original: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - id of original appointment
    + merged: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - id of merged appointment
    + `requiredPrepayment` (array) - prepayment, needed to be made for services with required prepayment, in format appointment service id - amount. Null if prepayments not activated or empty object if no services to prepay
        + (object)
            + id: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - appointment service id
            + amount: 200.00 (number,required) - required prepayment for this appointment service
    + `optionalPrepayment` (array) - prepayment, needed to be made for services with optional prepayment, in format appointment service id - amount. Null if prepayments not activated or empty object if no services to prepay
        + (object)
            + id: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - appointment service id
            + amount: 180.00 (number,required) - required prepayment for this appointment service
            + discount: 20.00 (number,required) - bonus or discount that will be received for optional prepayment
    + `latePrepayment` (array) - prepayment, needed to be made for services when it is close to appointment start, in format appointment service id - amount. Null if prepayments not activated or empty object if no services to prepay
        + (object)
            + id: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - appointment service id
            + amount: 200.00 (number,required) - required prepayment for this appointment service
    + `timeConflicts` (array) -  list of time conflicts for appointment services items. Read only, can be retrieved only for specific appointment. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error.
        + (object)
            + `id` (identifier) - id of object which changes raised time conflict (id of group lesson or aappointment service when changing itself, id of specific appointment service when changing appointment)
            + `itemType` (enum[string]) - type of conflicted resource
                + Members
                    + `professional` - professional has another work in same time
                    + `hall` - hall is used for another activirty in same time
                    + `resource` - resource exhausted in same time
                    + `appointment` - apointment service overlapped with another appointment service (`object` always `appointmentService`)
            + `itemId` (identifier) - id of conflicted resource
            + `objectType` (enum[string]) - another object, that use `itemId`
                + Members
                    + `appointmentService` - conflicted object is another appointment service
                    + `groupLesson` - conflicted object is another group lesson
                    + `nonWorkingTime` - professional does not work in specified time (for professional only)
            + `objectId` (identifier) - appointment service id or group lesson id
            + `objectDescriptionId` (identifier) - object that describes conflicted object: appointment client or employee id for `appointmentService`, group id for `groupLesson`
            + `objectStart` (datetime) - start of conflicted period (date and time)
            + `objectDuration` (number) - duration of conflicted period in minutes
    + `validations` (array) -  list of validations errors for appointment items. Read only, can be retrieved only for specific appointment. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)
            + `id`: `c8a7e4ce-c70a-413d-989a-ff8db3293798` (identifier) - appointment item id, can be used to find item in case appointment has more than one item. If `type` starts with "SERVICE_", `id` is appointment service id, `type` starts "PRODUCT_" - `id` is appointment product id and so on.
            + `type`: `SERVICE_PROFESSIONAL_CANT_PROVIDE_SERVICE` (enum[string]) - list of possible types described below
                + Members
                    + `CARD_CARD_NUMBER_REQUIRED` - Card number should be provided if card stocks is on in settings (in case card stocks is on but card number not provided). Additional fields: none.
                    + `CARD_CARD_NUMBER_IS_UNAVAILABLE` - Card number `cardNumber` is unavailable (either not exists or already sold). Additional field: `cardNumber`.
                    + `CARD_CARD_NUMBER_ONLY_IF_CARD_STOCK` - Card number can be provided only if card stocks is on in settings (in case card number is provided but card stocks is off, provide "0" as card number if card stocks is off). Additional fields: none.
                    + `CARD_ACTIVATION_DATE_ONLY_MONTH_START` - According to card settings, this card type can be activated only on first day of month (date of month should be "1"). Additional fields: none.
                    + `CERTIFICATE_ALREADY_EXPIRED` - Certificate `certificate` already expired. Additional field: `certificate`.
                    + `CERTIFICATE_ALREADY_SOLD` - Certificate `certificate` already sold. Additional field: `certificate`.
                    + `PRODUCT_PRODUCT_IS_UNAVAILABLE_ON_STORAGE` - Product `product` is unavailable on storage `storage`. Additional fields: `product` and `storage`.
                    + `PRODUCT_SALES_DISABLED_FOR_STORAGE` - Sales are disabled from storage `storage`. Additional field: `storage`.
                    + `PRODUCT_QUANTITY_TYPE_SALE_FORBIDDEN_FOR_STORAGE` - Sale by quantity type `productQuantityType` is forbidden from storage `storage`. Additional fields: `storage` and `productQuantityType`.
                    + `SERVICE_PROFESSIONAL_CANT_PROVIDE_SERVICE` - Professional `professional` can't provide service `service`. Additional fields: `service` and `professional`.
                    + `SERVICE_SERVICE_CANT_BE_PROVIDED_WITHOUT_PROFESSIONAL` - Service `service` can't be provided without professional. Additional field: `service`.
                    + `SERVICE_ASSISTANT_NOT_ALLOWED_FOR_SERVICE` - Assistant not allowed for service `service`. Additional field: `service`.
                    + `SERVICE_EMPLOYEE_IS_NOT_ASSISTANT` - Employee `employee` is not an assistant. Additional field: `employee`.
                    + `SERVICE_SERVICE_DOES_NOT_SUPPORT_HALLS` - Service `service` does not support selecting hall. Additional field: `service`.
                    + `SERVICE_SERVICE_WRONG_HALL` - Service `service` can't be provided in hall `hall`. Additional fields: `service` and `hall`.
                    + `SERVICE_SERVICE_DOES_SUPPORT_HALLS` - Service `service` supports selecting hall, hall must be set. Additional field: `service`.
                    + `SERVICE_TEETH_SHOULD_BE_SET_FOR_SERVICE` - Teeth should be set for service `service`. Additional field: `service`.
                    + `SERVICE_ONLY_ONE_TOOTH_SHOULD_BE_SET_FOR_SERVICE` - Only one tooth should be set for service `service`. Additional field: `service`.
                    + `SERVICE_TEETH_CANT_BE_SET_FOR_SERVICE` - Teeth can't be set for service `service`. Additional field: `service`.
                    + `SERVICE_MATERIAL_PRODUCT_IS_UNAVAILABLE_ON_STORAGE` - Product `product` is unavailable on storage `storage`. Additional fields: `product` and `storage`.
            + `cardNumber`: 125 (number) - card number
            + `certificate`: `83554967-74fe-49d8-be00-fb55eee308c5` (identifier) - certificate id
            + `product`: `03f0697a-a81a-4666-9fce-fb9db52396ea` (identifier) - product id
            + `storage`: `c1644821-fb17-4de0-85b2-d64b9f4e7b8b` (identifier) - storage id
            + `productQuantityType`: `Package` (enum[string]) - type of product sale
                + Members
                    + Portion
                    + Units
            + `service`: `b56007d6-42e7-491e-8648-704470340c91` (identifier) - service id
            + `professional`: `72537154-5e62-4f80-aff6-647f1e0d9672` (identifier) - employee id
            + `employee`: `679b72f6-aa42-4770-81b8-ed9b7f878abb` (identifier) - employee id
            + `hall`: `e1462348-3bda-45fe-b9bc-a2ee8df8b60e` (identifier) - hall id
    + `newItemsIds` (object) - for insert/update appointment only - returns id's of added items (services, products, cards, certificates, dentures). Order of ids is same as order of new items in request.
        + `services` (array[identifier]) - ids of new services
        + `products` (array[identifier]) - ids of new products
        + `cards` (array[identifier]) - ids of new cards
        + `certificates` (array[identifier]) - ids of new certificates
        + `dentures` (array[identifier]) - ids of new dentures
    + `groupComments` (array) - comments for blocks in appointment. Comment is characterized with professional and start of the block. If professional is empty, comment is displayed in section with recommendation and non-service appointment items. Otherwise the most relevan block in the appointment will be chosen.
        + (object)
            + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cab11` (identifier,required) - appointment id
            + start: `2020-01-01T10:10:10.000Z` (datetime) - start of the block
            + professional: `5acc652b-c762-4eda-983b-59cc86ea4481` (identifier) - profesional, that block is linked to (can be null)
            + text: `Comment text` (string) - text  
            

### Get all appointments [GET /appointments{?fields,from,to,client,professional,location,state}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + fields: `date,location,client,clientName,clientFirstName,clientMiddleName,clientLastName,employee,employeeName,employeeFirstName,employeeMiddleName,employeeLastName,receptionist,notifySms,services,products,certificates,dentures,cards,cancelDate,cancelReason,cancelReceptionist,createDate,feedback,groupComments(id,appointment,professional,start,text)` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointments which starts at or after specified date
    + to: `2019-01-02T12:00:00.000Z` (datetime, optional) - get appointments which starts before specified date
    + location: `12d4b305-e198-f02c-2743-cca93cb16baa` (identifier, optional) - get appointments only for given location (several locations can be separated by comma)
    + state: reserved (enum[string], optional) - get only appointments of specified state (or several states)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "e132e1c1-0450-4e50-973e-0d59007331ca",
                    "state": "planned",
                    "date": "2020-01-01",
                    "location": "e77673a6-1f77-4e22-b19a-9d5fadfc83c6",
                    "client": "f9c254bd-069a-4d4c-87b6-e981cbb966d1",
                    "clientName": "John Michael Brown",
                    "clientFirstName": "John",
                    "clientMiddleName" "Michael",
                    "clientLastName: "Brown",
                    "employee": null,
                    "employeeName": "",
                    "employeeFirstName": "",
                    "employeeMiddleName": "",
                    "employeeLastName: "",
                    "receptionist": "13adf8fb-6304-4e1b-a036-1c092b9e82e0",
                    "notifySms": "default",
                    "services": [
                        {
                            "service": "13adf8fb-6304-4e1b-b0b6-1cf32b1e82e1",
                            "professional": "13adf8fb-6304-4e1b-a136-1c792bfc8ae1",
                            "start": "10:00:00",
                            "duration": 60,
                            "appointment": "13adf8fb-63ae-4d1b-ad36-8c092bf48aee" 
                        }
                    ],
                    "products": [
                        {                           
                            "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                            "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                            "quantity": 2,
                            "type": "package"
                        }
                    ],
                    "certificates": null,
                    "cards": null,
                    "dentures": null,
                    "clientsModule": true,
                    "cancelDate": null,
                    "cancelReason": null,
                    "cancelReceptionist": null,
                    "createDate": "2019-01-01T10:00:00.000Z",
                    "feedback":
                    [
                        {
                            "id": "13adf8fb-6304-4e1b-a036-1c092b9e82e0",
                            "professional": "13adf8fb-6304-4e1b-a036-1c092b9e82e2",
                            "date": "2019-01-01T11:27:14.000Z",
                            "rating": 5,
                            "text": "Good job!",
                            "published": true
                        }
                    ],
                    "hasAnySales": true,
                    "clientInside": false,
                    "groupComments" : []
                },
                {
                    "id": "5024efc3-4cf1-4359-a3fc-f94eb4c4e3d2",
                    "state": "cancelled",
                    "date": "2020-01-01",
                    "location": "9b21186c-950d-4f1d-8ad5-0927472ee7c3",
                    "client": null,
                    "clientName": "",
                    "clientName": "",
                    "clientFirstName": "",
                    "clientMiddleName": "",
                    "clientLastName: "",
                    "employee": "d4c38e7f-519c-4964-9d0e-8bf91a74146f",
                    "employeeName": "John Michael Brown",
                    "employeeFirstName": "John",
                    "employeeMiddleName": "Michael",
                    "employeeLastName: "Brown",
                    "receptionist": "13adf8fb-6304-4e1b-a036-1c092b9e82e0",
                    "notifySms": "default",
                    "clients_module": true,
                    "services": [
                        {
                            "service": "13adf8fb-6304-4e1b-b0b6-1cf32b1e82e1",
                            "professional": "13adf8fb-6304-4e1b-a136-1c792bfc8ae1",
                            "start": "10:00:00",
                            "duration": 60,
                            "appointment": "13adf8fb-63ae-4d1b-ad36-8c092bf48aee" 
                        },
                        {
                            "service": "13adf8f2-6304-4e1b-b0b6-1cf3db5eefec",
                            "professional": "13adf8fb-6304-4e1b-a136-1c792bfc8ae1",
                            "start": "11:00:00",
                            "duration": 60,
                            "appointment": "13adf8fb-63ae-4d1b-ad36-8c092bf48aee" 
                        },
                    ],
                    "products": null,
                    "certificates": null,
                    "cards": null,
                    "dentures": null,
                    "cancelDate": "2019-01-01T11:00:00.000Z",
                    "cancelReason": "Plans changed",
                    "cancelReceptionist": "8039d005-f519-425f-8c09-11fe7cb25738",
                    "createDate": "2019-01-01T10:00:00.000Z",
                    "feedback": null,
                    "hasAnySales": true,
                    "clientInside": false,
                    "original": null,
                    "merged": null,
                    "groupComments" : [
                         {
                              "appointment":"88db3f7d-5845-6dee-2bb3-bfbf3ad6e3bf",
                              "professional": null,
                              "start":"2023-04-17T10:00:00",
                              "text": "Text1"
                          }
                      ]
                }
            ]

### Get appointment by id [GET /appointments/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`, `services_aggregator` (only state, date, location, client, comments, cancelReason, clientsModule and services fields - services only with start, duration, service, professional subfields)

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,location,client,clientName,clientFirstName,clientMiddleName,clientLastName,employee,employeeName,employeeFirstName,employeeMiddleName,employeeLastName,receptionist,notifySms,services,products,certificates,dentures,cards,cancelDate,cancelReason,cancelReceptionist,createDate,feedback,groupComments(id,appointment,professional,start,text)` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "5c05c7a4-74da-42e5-ac5d-dbc489748143",
                "state": "planned",
                "date": "2019-01-01",
                "location": "e77673a6-1f77-4e22-b19a-9d5fadfc83c6",
                "client": "f9c254bd-069a-4d4c-87b6-e981cbb966d1",
                "clientName": "John Michael Brown",
                "clientFirstName": "John",
                "clientMiddleName" "Michael",
                "clientLastName: "Brown",
                "employee": null,
                "employeeName": "",
                "employeeFirstName": "",
                "employeeMiddleName": "",
                "employeeLastName: "",
                "receptionist": "13adf8fb-6304-4e1b-a036-1c092b9e82e0",
                "notifySms": "default",
                "clientsModule": true,
                "services": [
                    {
                        "service": "13adf8fb-6304-4e1b-b0b6-1cf32b1e82e1",
                        "professional": "13adf8fb-6304-4e1b-a136-1c792bfc8ae1",
                        "start": "10:00:00",
                        "duration": 60,
                        "appointment": "13adf8fb-63ae-4d1b-ad36-8c092bf48aee" 
                    }
                ]
                "products": null,
                "certificates": null,
                "cards": null,
                "dentures": null,
                "cancelDate": null,
                "cancelReason": null,
                "cancelReceptionist": null,
                "comments": "some comment",
                "createDate": "2019-01-01T10:00:00.000Z",
                "feedback":
                [
                    {
                        "id": "13adf8fb-6304-4e1b-a036-1c092b9e82e0",
                        "professional": "13adf8fb-6304-4e1b-a036-1c092b9e82e2",
                        "date": "2019-01-01T11:27:14.000Z",
                        "rating": 5,
                        "text": "Good job!",
                        "published": true
                    }
                ],
                "hasAnySales": true,
                "clientInside": false,
                "original": null,
                "merged": null,
                "groupComments" : []
            }

### Create new appointment [POST /appointments{?fields,force,copy,skipRequiredPrepayment}]

By default, if `professional` is set, method checks if proposed appointment do not intersects with another appointments, reserves or sales.
If found, error 409002 returned (list of intersected items also returned as `another_items` property). If you want to skip time intersection checks and save appointment anyway, add `force` parameter to request.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `services_aggregator` (only state, date, location, client, comments, cancelReason, clientsModule and services fields - services only with start, duration, service, professional subfields)

**Custom errors:**
1. [ALREADY_EXISTS](#error-ALREADY_EXISTS) - appointment you trying to edit already have reserve attached.
2. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time

As sending notification smses is not crucial for creating new appointment, error not returned if smses was not sent. Sending these smses is last step after saving new appointment (sending notification smses to client and professional). If failed, additional field returned in response:
```
{
    "smsError":
    {
        "type": "SMS_BALANCE",
        "message": "Not enough money for sending sms"
    }
}
```
Where type can be [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) or [SMS_BALANCE](#error-SMS_BALANCE) - use `type` for checks in code and show corresponding info for user. `message` describes additional details and can help developer to understood problem.

**More details:**

Remember, that `date` & `location` (can be omitted in request and will be taken from access token) fields are required except case `copy` parameter provided.

In most cases new appointment will be created with provided properties. But there are several special cases:
1) if you provide `client` or `employee` fields and `state` = 'planned'/'confirmed'/(not provided equals to 'planned') and appointment with provided `date`+`location`+`client`/`employee` and `state` = 'planned'/'confirmed' already exists, new appointment will not be created, but rather existing appointment will be updated (and endpoint will return `id` of existing appointment). Updated fields are: `services`/`products`/`cards`/`certificates`/`dentures` - new items will be appended; `comments` - updated if not already contains proposed text (to prevent duplicate text in comments); `clientsModule` - can be set to true if original appointment not set yet; `clientsModuleConfirmed` - can be set to 'false' if `clientsModule` set to true; `important` - can be set to true if original appointment not set yet; `notifySms` - can be set to non 'default' value if original appointment value is 'default'; `receptionist` - can be set to provided value if original appointment value was null; `controlManager` - can be set to provided value if original appointment value was null. So, this case is extremely useful if you want to add one item (for example, service) on specific date+location+client/employee, but don't know if such appointment exists and id of this appointment if exists. Instead of searching and loading appointment by date+location+client/employee, appending item and sving again or inserting new appointment if not found - you can make just one simple call. Also, please take into account, that in case someone edit appointment with separate reserved appointment, all changes except items with `sale` not null field will be lost.
2) if you provide `client` or `employee` fields and `state` = 'reserved' and appointment with provided `date`+`location`+`client`/`employee` and `state` = 'planned'/'confirmed' already exists, new reserved appointment will be created and found appointment will be marked to merge on edit end. Appointments will be merged in new reserved appointment on same rules as in previous case (info from found appointment will be copied to existing one), but found appointment will left unchanged. 409 Conflict "ALREADY_EXISTS" will be raised if found appointment already has attached valid reserve appointment (both ids of found and altready attched reserved appointment will be returned, so you can delete reserved appointment if needed). Also, same error with `id=null` will be raised if you try to create reserve but other reserve already exists (in this case `id` filed will be empty but `reserve` field will contain id of reserve appointment).
3) if you provide `copy` parameter (remember, you can't provide `date`, `location`, `client`, `employee` fields and should set `state` = 'reserved' in this case), new reserved appointment will be created, that will be a full copy of `copy` appointment except new `id` field and `state` = 'reserved'. If you want, you can provide new field values for any fields except `date`, `location`, `client`, `employee`, `state`, they will be overwritten (`services`/`products`/`cards`/`certificates`/`dentures` will replace existing one if provided). 409 Conflict "ALREADY_EXISTS" will be raised if copied appointment already has attached valid reserve appointment (both ids of found and altready attched reserved appointment will be returned, so you can delete reserved appointment if needed).

+ Parameters
    + fields: `date,location,client,clientName,clientFirstName,clientMiddleName,clientLastName,employee,employeeName,employeeFirstName,employeeMiddleName,employeeLastName,receptionist,notifySms,services,products,certificates,dentures,cards,cancelDate,cancelReason,cancelReceptionist,createDate,groupComments(id,appointment,professional,start,text)` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.
    + copy: 'e94eb710-dc90-4af1-b836-3cc6b2de3126' (identifier, optional) - if set, new appointment will be filled with data from proposed appointment (only `id` field will be new). If you set some fields in body, they will override fields from copied appointment. This parameter is useful for creating reserve appointment when editing existing appointment: create reserve as copy from existing, edit it and replace existing on save. Prerequisites: copy appointement should have planned/confirmed state (not reserve or cancelled), you should pass `state` = 'reserved' so result appointment will be reserved and do not pass new `date`, `location`, `client` or `employee` parameters.
    + skipRequiredPrepayment: `false` (boolean, optional) - if true, prepayment is not required for services (in case required prepayment is configured).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "date": "2018-01-01T10:00:00.000Z",
                "location": "8fe29e48-d5a5-4dcc-800e-80eb6e9cac5c",
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment [PUT /appointments/{id}{?fields,force,skipRequiredPrepayment}]

By default, if `professional` is set, method checks if changed appointment do not intersects with another appointments, reserves or sales.
If found, error 409002 returned (list of intersected items also returned as `another_items` property). If you want to skip time intersection checks and save appointment anyway, add `force` parameter to request.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id), `services_aggregator` (only state, date, location, client, comments, cancelReason, clientsModule and services fields - services only with start, duration, service, professional subfields)

**Custom errors:**
1. [ALREADY_EXISTS](#error-ALREADY_EXISTS) - appointment you trying to edit already have reserve attached.
2. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time

As sending notification smses is not crucial for creating updated appointment, error not returned if smses was not sent. Sending these smses is last step after saving updated appointment (sending notification smses to client and professional). If failed, additional field returned in response:
```
{
    "smsError":
    {
        "type": "SMS_BALANCE",
        "message": "Not enough money for sending sms"
    }
}
```
Where type can be [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) or [SMS_BALANCE](#error-SMS_BALANCE) - use `type` for checks in code and show corresponding info for user. `message` describes additional details and can help developer to understood problem.

**More details:**

Appointment has one of four states, but not all state changes can be done: reserved appointments can only be created, but changing other states to reserved is illegal.

When changing `date`, `location`, `client` or `employee` and appointment with new `date`, `location`, `client`/`employee` and `state`='planned'/'confirmed' exists, update operation differs a bit for different appointment states:
1) New state is cancelled - no merging with found appointment at all
2) Both old and new state are not reserved (planned/confirmed/cancelled) and new state is not cancelled - permanently merge with new appointment, found appointment is deleted. Moreover, 409 Conflict "ALREADY_EXISTS" will be raised if current appointment has attached valid reserve appointment (both ids of found and altready attched reserved appointment will be returned, so you can delete reserved appointment if needed) - you can't change `date`, `location`, `client` or `employee` fields if someone edit appointment. 409 Conflict "ALREADY_EXISTS" will be raised if found appointment already has attached valid reserve appointment (both ids of found and altready attched reserved appointment will be returned, so you can delete reserved appointment if needed).
3) Old state is reserved state, new state is any except cancelled - unmerge from previous appointment and merge with new one. Merges in reserved state are not permanent (all merged appointment data is copied into reserved appointment and is commited on reserve appointment change state to planned or confirmed).

If reserved appointment state changes to planned or confirmed, editing process finishes.
1) If reserve was editing existing appointment, all changes from reserved appointment are applied to original appointment, including data from merged appointment. Reserved and merged (if exist) appointments are deleted.
2) If reserve was editing new appointment, appointment state changes from from reserved to proposed one, including data from merged appointment, merged (if exists) appointment is deleted.
As appointment items with `sale` not null can't be edited, they are copied to final appointment from original and merged appointments. All other items in final appointment are taken from reserved appointment, losing changes to original/merged appointments done after start editing/merging respectively.

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,location,client,clientName,clientFirstName,clientMiddleName,clientLastName,employee,employeeName,employeeFirstName,employeeMiddleName,employeeLastName,receptionist,notifySms,services,products,certificates,dentures,cards,cancelDate,cancelReason,cancelReceptionist,createDate,groupComments(id,appointment,professional,start,text)` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales should be skipped.
    + skipRequiredPrepayment: `false` (boolean, optional) - if true, prepayment is not required for services (in case required prepayment is configured).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "date": "2018-01-01T00:00:00.000Z",
                "location": "8fe29e48-d5a5-4dcc-800e-80eb6e9cac5c"
            }

+ Response 204 (application/json)

### Delete appointment [DELETE /appointments/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (`client` field should match client id)

As sending notification smses is not crucial for deleting appointment, error not returned if smses was not sent. Sending these smses is last step after deleting appointment (sending notification sms to client about cancelled visit). If failed, additional field returned in response:
```
{
    "smsError":
    {
        "type": "SMS_BALANCE",
        "message": "Not enough money for sending sms"
    }
}
```
Where type can be [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) or [SMS_BALANCE](#error-SMS_BALANCE) - use `type` for checks in code and show corresponding info for user. `message` describes additional details and can help developer to understood problem.

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

### Send additional notification [POST /appointments/{id}:sendNotification] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

API automatically sends notifications to client 1 or 2 hours before appointment start (or other time according to appointment and general settings).
But additional notification can be sent manually if needed by this endpoint.

Next prerequisites are required:
1) Appointment is in 'planned' or 'confirmed' state
2) At least one appointment service exists in appointment
3) Appointment has a client (not a guest) and client had not refused to receive notification smses
4) Sms service and sms notification text was configured

In these cases 400 Bad Request will be returned.

IF sms succesfully sent, sms send result will be returned:

```
{
    "status": "Accepted"
}
```

If error while sending sms happened, next result will be returned:
```
{
    "smsError":
    {
        "type": "SMS_BALANCE",
        "message": "Not enough money for sending sms"
    }
}
```
Where type can be [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) or [SMS_BALANCE](#error-SMS_BALANCE) - use `type` for checks in code and show corresponding info for user. `message` describes additional details and can help developer to understood problem.

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "status": "Accepted"
            }

## Appointment Service [/appointments/services]
<a name="appointments_services"></a>

Appointment is planned visit of a client for some date for one or several services.

Two moments to remember:
+ Valid appointments are all appointments in state `planned`, `confirmed` and appointments in state `reserved` if `createDate` (UTC) is less than 15 minutes from now (or `createDate` is any date in future, if manually set). Invalid appointments are all appointments in `cancelled` state and appointments in state `reserved` if `createDate` (UTC) is more than 15 minutes ago from now. All valid appointments book time for professional and attempt to set new valid appointment or change existing (changing appointment `date`, `duration` or `professional` or making invalid appointment valid via changing `state` or `createDate`) that intersect with existing one will cause 409001 conflict error.

+ Attributes
    + start: `10:10:10.000Z` (datetime) - appointment service start time (date getting from appointment)
    + duration: 60 (number) - appointment duration (greater than zero)
        + Default: 60
    + professional: `5acc652b-c762-4eda-983b-59cc86ea4481` (identifier) - profesional id (who provides services) (can be null)
    + professionalName: `3b0070ed-d302-4765-90f9-ce67a848d370` (string) - professional name (read only)
    + professionalPhone: `+38(063)1234567` (string) - professional phone (read only)
    + professionalPhotoExists: true (boolean) -  if professional has a photo (read only)
    + assistant: `4357be34-aa4b-4711-8a4d-34b3059b5b43` (identifier) - assistant id (helps professional to provide services)
    + service: `4b244443-90d3-4182-918a-fe3ac4a03688` (identifier) - service id (can be null)
    + serviceName: `Service1` (string) - service name (read only)
    + predictedSum: 100 (number) - service price, includes discounts for the client (read only)
    + quantity: 2 (number) - how much quantity service will be provided (on default equals 1. Value equals or greater than zero)
    + teeth: `6` (string, required) - teeth name. Max length is 100 characters
    + hall: `4b244443-90d3-4182-918a-fe3ac4abc681` (identifier) - hall id (can be null)  
    + hallName: `Hall1` (string) - hall name
    + customMaterials: false (boolean) - materials can be taken from the service or from professional who provided the service
    + recommendedBy: `4b244443-90d3-4182-918a-e73c491bf6f2` (identifier) - who recommended service
    + createDate: `2019-01-01T10:00:00.000Z` (datetime) - date and time appointment was created (in UTC)
    + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cab11` (identifier,required) - appointment id. See [Appointments](#appointments)
    + cancelReason: `Cancelled by admin` (string) - cancel reason forappointment (if it is cancelled), can be null, read only
    + state: `planned` (enum[string]) - state from appointment (read only). See [Appointments](#appointments)
    + client: `13adf8fb-6304-4e1b-a036-1c092b91cab11` (identifier) - client from appointment (read only). See [Clients](#clients)
    + clientName: `John Doe` (string) - client name (read only)
    + clientPhone: `+38(063)1234567` (string) - client name (read only)
    + location: `c2adf8fb-6304-4e1b-a036-1c092b91c1b2b` (identifier) - location from appointment (read only). See [Location](#locations)
    + sale: `1ca6fdfb-6304-4e1b-a036-1cfb2b915accb` (identifier) - sale id (read only). See [Sales](#sales)
    + `timeConflicts` (array) -  list of time conflicts for all appointment services items. Read only. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error. See more details in appointment description.
        + (object)
    + `validations` (array) -  list of validations errors for appointment items. Read only. See more details in appointment description. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)

### Get all appointments services [GET /appointments/services{?fields,from,to,client,professional,location,state}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `start,duration,assistant,professional,professionalName,professionalPhone,professionalPhotoExists,service,price,serviceName,quantity,teeth,hall, hallName,customMaterials,recommendedBy,createDate,appointment,state,cancelReason,client,clientName,clientPhone,location,sale` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment services which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment services which starts before specified date
    + professional: `f43d9001-fb13-4511-8ca6-d75481e811d4` (identifier, optional) - get appointments only for given professional (several professionals can be separated by comma)
    + location: `f43d9001-fb13-4511-8ca6-d75481ad1cdb` (identifier, optional) - get appointment service only for given location. (several locations can be separated by comma)
    + client: `f43d9001-fb13-4511-8ca6-d75481ad322b` (identifier, optional) - get appointment service only for given client. (several clients can be separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "e132e1c1-0450-4e50-973e-0d59007331ca",
                    "start": "10:10:10.000Z",
                    "duration": 60,
                    "assistant": "4357be34-aa4b-4711-8a4d-34b3059b5b43",
                    "professional": "5acc652b-c762-4eda-983b-59cc86ea4481",
                    "professionalName": "Bob",
                    "professionalPhone": "+38(063)1234567",
                    "professionalPhotoExists": true,
                    "service": "4b244443-90d3-4182-918a-fe3ac4a03688",
                    "price": 100,
                    "quantity": 2,
                    "hall": null,
                    "hallName": null,
                    "createDate": "2019-01-01T10:00:00.000Z",
                    "appointment": "4b244443-90d3-4182-918a-fe3ac4a0331b",
                    "cancelReason": "",
                    "state": "planned",
                    "client": "4b244443-90d3-4182-918a-fe3ac4a0dc11",
                    "clientName": "John Doe".
                    "clientPhone": "+38(063)1234567",
                    "location": "4b244443-90d3-4182-918a-fe3ac4a01311",
                    "sale": null
                },
                {
                    "id": "5024efc3-4cf1-4359-a3fc-f94eb4c4e3d2",
                    "start": "10:10:10.000Z",
                    "duration": 60 ,
                    "assistant": "4357be34-aa4b-4711-8a4d-34b3059b5b43",
                    "professional": "5acc652b-c762-4eda-983b-59cc86ea4481",
                    "professionalName": "3b0070ed-d302-4765-90f9-ce67a848d370",
                    "professionalPhone": "+38(063)1234567",
                    "professionalPhotoExists": true,
                    "service": "3d2f87de-61fb-4f02-9d6c-93dc6a86fb6a",
                    "price": 100,
                    "quantity": 2,
                    "hall": null,
                    "hallName": null,
                    "createDate": "2019-01-01T10:00:00.000Z",
                    "state": "confirmed",
                    "client": "4b244443-90d3-4182-918a-fe3ac4a0dc11",
                    "clientName": "John Doe".
                    "clientPhone": "+38(063)1234567",
                    "location": "cb244443-90d3-4182-918a-fe3ac4a01311",
                    "sale": "1b244443-90d3-4182-918a-fe3afdabaa1a"
                }
            ]

### Get appointment services by id [GET /appointments/services/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `start,duration,assistant,professional,professionalName,professionalPhone,professionalPhotoExists,service,price,serviceName,quantity,teeth,hall, hallName,customMaterials,recommendedBy,createDate,appointment,state,cancelReason,client,clientName,clientPhone,location,sale,timeConflicts,validations` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "5c05c7a4-74da-42e5-ac5d-dbc489748143",
                "start": "10:10:10.000Z",
                "duration": 60,
                "assistant": "4357be34-aa4b-4711-8a4d-34b3059b5b43",
                "professional": "5acc652b-c762-4eda-983b-59cc86ea4481",
                "professionalName": "Bob",
                "professionalPhotoExists": true,
                "service": "4b244443-90d3-4182-918a-fe3ac4a03688",
                "quantity": 2,
                "hall": null,
                "createDate": "2019-01-01T10:00:00.000Z",
                "state": "planned",
                "client": "4b244443-90d3-4182-918a-fe3ac4a0dc11",
                "location": "cb244443-90d3-4182-918a-fe3ac4a01311",
                "sale": null
            }

### Create new appointment services [POST /appointments/services{?fields,force,skipRequiredPrepayment}]

By default, if `professional` is set, method checks if proposed appointment do not intersects with another appointments, reserves or sales.
If found, error 409002 returned (list of intersected items also returned as `another_items` property). If you want to skip time intersection checks and save appointment anyway, add `force` parameter to request.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id)

**Custom errors:**
1. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time
2. [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) and [SMS_BALANCE](#error-SMS_BALANCE) - last step after saving updated appointment is sending notification smses to client and professional. Receiving these errors means save appointment operation successfully finished and stored in database, but notification smses was not sent (not so crucial to rollback appointment, but you should be notified).

+ Parameters
    + fields: `start,duration,assistant,professional,professionalName,professionalPhone,professionalPhotoExists,service,price,serviceName,quantity,teeth,hall, hallName,customMaterials,recommendedBy,createDate,appointment,state,cancelReason,client,clientName,clientPhone,location,sale,timeConflicts,validations` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.
    + skipRequiredPrepayment: `false` (boolean, optional) - if true, prepayment is not required for services (in case required prepayment is configured).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "start": "10:00:00.000Z",
                "duration": 150,
                "professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment services [PUT /appointments/services/{id}{?fields,force,skipRequiredPrepayment}]

By default, if `professional` is set, method checks if changed appointment do not intersects with another appointments, reserves or sales.
If found, error 409002 returned (list of intersected items also returned as `anotherItems` property). If you want to skip time intersection checks and save appointment anyway, add `force` parameter to request.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id)

**Custom errors:**
1. [TIME_CONFLICT](#error-TIME_CONFLICT) - one or several appointment services conflicts with other appointments services, group lessons or non working time
2. [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) and [SMS_BALANCE](#error-SMS_BALANCE) - last step after saving updated appointment is sending notification smses to client and professional. Receiving these errors means save appointment operation successfully finished and stored in database, but notification smses was not sent (not so crucial to rollback appointment, but you should be notified).

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `start,duration,assistant,professional,professionalName,professionalPhone,professionalPhotoExists,service,price,serviceName,quantity,teeth,hall, hallName,customMaterials,recommendedBy,createDate,appointment,state,cancelReason,client,clientName,clientPhone,location,sale,timeConflicts,validations` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.
    + skipRequiredPrepayment: `false` (boolean, optional) - if true, prepayment is not required for services (in case required prepayment is configured).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "start": "2018-01-01T10:00:00.000Z",
                "duration": 150,
                "professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"
            }

+ Response 204 (application/json)

### Delete appointment service [DELETE /appointments/services/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id)

**Custom errors:**
1. [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) and [SMS_BALANCE](#error-SMS_BALANCE) - last step after saving updated appointment is sending notification smses to client and professional. Receiving these errors means save appointment operation successfully finished and stored in database, but notification smses was not sent (not so crucial to rollback appointment, but you should be notified).

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Service Product [/appointments/services/materials]
<a name="appointment_service_products"></a>

Appointment is planned visit of a client for some date for one or several services.

+ Attributes
    + date: `2020-05-05` (datetime) - date of appointment (read only). See [Appointments](#appointments)
    + product: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - product id. See [Products](#products)
    + storage: `13adf8fb-6304-4e1b-a036-1c092b9e82e2` (identifier,required) - storage id. See [Storages](#storages)
    + quantity: 3 (number) - quantity's product (on default 1. Value equals or greater than zero)
    + appointment_service: `13adf8fb-6304-4e1b-a036-1c092b91cfb11` (identifier,required) - appointment service id. See [Appointment services](#appointment_services)
    
### Get all appointments service product [GET /appointments/services/materials{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `product,storage,quantity,service,appointment` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment service products which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment service products which starts before specified date
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "date": "2020-01-01",
                    "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                    "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                    "quantity": 2,
                    "appointment_service": "f2c7ed19-2b49-4c8e-8592-9b26c8ab9f1a"
                },
                {
                    "date": "2019-01-02",
                    "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1bbb",
                    "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1c1c",
                    "quantity": 2,
                    "appointment_service": "cc827fb4-40e9-4a61-bad8-d24a1081c81c"
                }
            ]

### Get appointment service product by id [GET /appointments/services/materials/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,product,storage,quantity,appointment_service` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "date": "2020-01-01",
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "appointment_service": "f2c7ed19-2b49-4c8e-8592-9b26c8ab9f1a"
            }

### Create new appointment service product [POST /appointments/services/materials{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,product,storage,quantity,appointment_service` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "appointment_service": "f2c7ed19-2b49-4c8e-8592-9b26c8ab9f1a"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment service product [PUT /appointments/services/materials/{id}{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,product,storage,quantity,appointment_service` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales should be skipped.
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "appointment_service": "f2c7ed19-2b49-4c8e-8592-9b26c8ab9f1a"
            }

+ Response 204 (application/json)

### Delete appointment service product [DELETE /appointments/services/materials/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Products [/appointments/products]
<a name="appointments_products"></a>

Appointment is planned visit of a client for some date for one or several services.

+ Attributes
    + date: `2020-01-01` (datetime) - appointment start date (read only). See [Appointments](#appointments)
    + product: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - product id
    + storage: `13adf8fb-6304-4e1b-a036-1c092b9e82e2` (identifier,required) - storage id
    + quantity: 2 (number) - quantity's product
    + type: `Package` (enum[string]) - type of product sale
        + Members
            + Portion
            + Units
    + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cv b11` (identifier,required) - appointment id. See [Appointments](#appointments)
    + sale: `1ca6fdfb-6304-4e1b-a036-1cfb2b915accb` (identifier) - sale id (read only). See [Sales](#sales)
    + `timeConflicts` (array) -  list of time conflicts for all appointment services items. Read only. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error. See more details in appointment description.
        + (object)
    + `validations` (array) -  list of validations errors for appointment items. Read only. See more details in appointment description. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)

### Get all appointment products [GET /appointments/products{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `date,product,storage,quantity,type,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment products which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment products which starts before specified date
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "date": "2020-01-01",
                    "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                    "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                    "quantity": 2,
                    "type": "package",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                },
                {
                    "date": "2020-01-01",
                    "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1bbb",
                    "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1c1c",
                    "quantity": 2,
                    "type": "package",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c81c",
                    "sale": null
                }
            ]

### Get appointment products by id [GET /appointments/products/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,product,storage,quantity,type,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "date": "2020-01-01",
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "type": "package",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                "sale": null
            }

### Create new appointment products [POST /appointments/products{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,product,storage,quantity,type,appointment,sale` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "storage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "type": "package",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment products [PUT /appointments/products/{id}{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,product,storage,quantity,type,appointment,sale` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "product": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "strage": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea16aa",
                "quantity": 2,
                "type": "package",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 204 (application/json)

### Delete appointment products [DELETE /appointments/products/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Certificates [/appointments/certificates]
<a name="appointments_certificates"></a>

Appointment is planned visit of a client for some date for one or several services.

+ Attributes
    + date: `2020-01-01` (datetime) - appointment start date (read only)
    + certificate: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - certificate id. See [Certificates](#certificates)
    + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cv b11` (identifier,required) - appointment id. See [Appointments](#appointments)
    + sale: `1ca6fdfb-6304-4e1b-a036-1cfb2b915accb` (identifier) - sale id (read only). See [Sales](#sales)
    + `timeConflicts` (array) -  list of time conflicts for all appointment services items. Read only. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error. See more details in appointment description.
        + (object)
    + `validations` (array) -  list of validations errors for appointment items. Read only. See more details in appointment description. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)

### Get all appointment certificate [GET /appointments/certificates{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`
                      
+ Parameters
    + fields: `date,certificate,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment certificates which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment certificates which starts before specified date
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "date": "2020-01-01T10:10:10.000Z",
                    "certificate": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                },
                {
                    "date": "2020-01-02T10:10:10.000Z",
                    "certificate": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1bbb",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                }
            ]

### Get appointment certificate by id [GET /appointments/certificates/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,certificate,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "date": "2020-01-01",
                "certificate": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                "sale": null
            }

### Create new appointment certificate [POST /appointments/certificates{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,certificate,appointment` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "certifcate": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment certificate [PUT /appointments/certificates/{id}{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,certificate,appointment` (array[string], optional) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "certificate": "d6d3d3d7-f6c6-46eb-a820-5a68f7ea1611",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 204 (application/json)

### Delete appointment certificate [DELETE /appointments/certificates/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Cards [/appointments/cards]
<a name="appointments_cards"></a>

+ Attributes
    + date: `2019-01-01T10:10:10.000Z` (datetime) - appointment start date (read only). See [Appointments](#appointments)
    + card: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - card id
    + cardNumber: 20 (number) - card number
    + activationDate: `2019-01-01T10:10:10.000Z` (datetime) - activation date
    + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cv b11` (identifier,required) - appointment id
    + sale: `1ca6fdfb-6304-4e1b-a036-1cfb2b915accb` (identifier) - sale id (read only). See [Sales](#sales)
    + `timeConflicts` (array) -  list of time conflicts for all appointment services items. Read only. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error. See more details in appointment description.
        + (object)
    + `validations` (array) -  list of validations errors for appointment items. Read only. See more details in appointment description. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)

### Get all appointment card [GET /appointments/cards{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `date,card,cardNumber,activationDate,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment cards which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment cards which starts before specified date
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "date": "2020-01-01T10:10:10.000Z",
                    "card": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                    "activationDate": "2019-01-20T10:10:10.000Z",
                    "cardNumber": 2,
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                },
                {
                    "date": "2019-01-02T10:10:10.000Z",
                    "card": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                    "activationDate": "2019-12-12T10:10:10.000Z",
                    "cardNumber": 2,
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                    "sale": null
                }
            ]

### Get appointment cards by id [GET /appointments/cards/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,card,cardNumber,activationDate,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "date": "2019-01-02T10:10:10.000Z",
                "card": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                "activationDate": "2019-12-12T10:10:10.000Z",
                "cardNumber": 2,
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                "sale": null
            }

### Create new appointment card [POST /appointments/card{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,card,cardNumber,activationDate,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "date": "2019-01-02T10:10:10.000Z",
                "card": "cc827fb4-40e9-4a61-bad8-d24a1081c891",
                "activationDate": "2019-12-12T10:10:10.000Z",
                "cardNumber": 2,
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c891"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment card [PUT /appointments/cards/{id}{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment card id
    + fields: `date,card,cardNumber,activationDate,appointment` (array[string], required) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "date": "2019-01-02T10:10:10.000Z",
                "activationDate": "2019-12-12T10:10:10.000Z",
                "cardNumber": 2,
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c891"
            }

+ Response 204 (application/json)

### Delete appointment card [DELETE /appointments/cards/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204 (application/json)


## Dentures [/appointments/dentures]
<a name="appointments_dentures"></a>

+ Attributes
    + date: `2020-01-01` (datetime) - appointment start date and time
    + denture: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier,required) - card id
    + laboratory: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - company id
    + technican: `some text` (string) - technican information. Max length is 100 characters.
    + professional: `13adf8fb-6304-4e1b-a036-1c092b9e82e0` (identifier) - installs dentures for clients
    + comments: `some text for comment` (string) - card comment. Max length is 65536 characters.
    + teeth: `6` (string, required) - teeth name. Max length is 100 characters.
    + appointment: `13adf8fb-6304-4e1b-a036-1c092b91cv b11` (identifier,required) - appointment id
    + sale: `1ca6fdfb-6304-4e1b-a036-1cfb2b915accb` (identifier) - sale id (read only). See [Sales](#sales)
    + `timeConflicts` (array) -  list of time conflicts for all appointment services items. Read only. To use this field, declare `force=true`, otherwise any item in this list will raise 409 error. See more details in appointment description.
        + (object)
    + `validations` (array) -  list of validations errors for appointment items. Read only. See more details in appointment description. To use this field, declare `force=true`, otherwise any item in this list will raise 400 error.
        + (object)

### Get all appointment dentures [GET /appointments/dentures{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + fields: `date,denture,laboratory,technican,professional,comments,teeth,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + from: `2019-01-01T10:00:00.000Z` (datetime, optional) - get appointment denture cards which starts at or after specified date
    + to: `2019-01-01T12:00:00.000Z` (datetime, optional) - get appointment denture cards which starts before specified date
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "date": "2020-01-01",
                    "denture": "cc827fb4-40e9-4a61-bad8-d24ac0c1c191",
                    "laboratory": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1c2",
                    "technican": "some text",
                    "professional": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1v4",
                    "comments": "some text comments",
                    "teeth": "some teeth",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                },
                {
                    "date": "2020-01-01T10:10:10.000Z",
                    "denture": "cc827fb4-40e9-4a61-bad8-d24ac0c1cc92",
                    "laboratory": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1c2",
                    "technican": "some text",
                    "professional": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1v5",
                    "comments": "some text comments",
                    "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                    "sale": null
                }
            ]

### Get appointment denture by id [GET /appointments/dentures/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (`client` field should match client id), `reports`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment id
    + fields: `date,denture,laboratory,technican,professional,comments,teeth,appointment,sale` (array[string], required) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "date": "2020-01-01T10:10:10.000Z",
                "denture": "cc827fb4-40e9-4a61-bad8-d24ac0c1cc92",
                "laboratory": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1c2",
                "technican": "some text",
                "professional": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1v1",
                "comments": "some text comments",
                "teeth": "some teeth",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b",
                "sale": null
            }

### Create new appointment denture [POST /appointments/dentures{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,denture,laboratory,technican,professional,comments,teeth,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "denture": "cc827fb4-40e9-4a61-bad8-d24ac0c1cc92",
                "laboratory": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1c2",
                "technican": "some text",
                "professional": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1v2",
                "comments": "some text comments",
                "teeth": "some teeth",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update appointment denture [PUT /appointments/dentures/{id}{?fields,force}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - appointment card id
    + fields: `date,denture,laboratory,technican,professional,comments,teeth,appointment,sale` (array[string], required) - list of fields to return (separated by comma)
    + force: `true` (boolean, optional) - Indicates that checking for time conflicts with another appointments, reserves and sales and validations should be skipped.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "denture": "cc827fb4-40e9-4a61-bad8-d24ac0c1cc92",
                "laboratory": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1c2",
                "technican": "some text",
                "professional": "cc827fb4-40e9-4a61-bad8-d24ac0c2b1v3",
                "comments": "some text comments",
                "teeth": "some teeth",
                "appointment": "cc827fb4-40e9-4a61-bad8-d24a1081c89b"
            }

+ Response 204 (application/json)

### Delete appointment denture [DELETE /appointments/dentures/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204 (application/json)

## Schedule [/schedule]

This endpoint need for take schedule professionals
Endpoints to work with employees schedule

### Get schedule by professionals [GET /schedule{?from,to,professional,location}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameter
    + from: `2019-01-01T01:01:01.000Z` (datetime, required) - schedule period start
    + to: `2019-01-02T01:01:01.000Z` (datetime, required) - schedule period end
    + professional: `0997d37b-49fc-4bd6-8c24-0fbcacb141a7` (identifier, optional) - get schedule only for given professional
    + location: `3635cbcf-54e0-4d23-aeb5-6ecb58abb129` (identifier, optional) - get schedule only for given location, see [Locations](#locations)
    
+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body

            {
                "columns":
                [
                    {
                        "professional": "dff82ed3-a2a4-46e0-b17f-53c51ed8d4bd",
                        "date": "",
                        "worktime":
                        [
                            {
                                "start": "2017-12-31T10:00:00.000Z",
                                "end": "2017-12-31T20:00:00.000Z"
                            }
                        ],
                        "reserves": 
                        [
                            {
                                "id": "e27d6e58-f33a-4749-8756-4d7fd091747b",
                                "start": "2018-01-01T10:00:00.000Z",
                                "duration": 60
                            },
                            {
                                "id": "fc17223b-cb46-4827-ab1f-972a1d536f0e",
                                "start": "2018-01-01T10:00:00.000Z",
                                "duration": 60
                            }
                        ],
                        "appointments":
                        [
                            "b0cb755a-f03d-43ff-91fd-051888bb0de1",
                            "4e650711-a2b0-4eed-bf4e-29dc3cd79be4"
                        ]
                    }
                ],
                "appointments": 
                {
                    "b0cb755a-f03d-43ff-91fd-051888bb0de1":
                    {
                        {
                            "id": "a7251983-9e0c-463d-81a6-b7afe8a1d7d9",
                            "date": "2019-01-03T12:00:00.000Z",
                            "duration": 30,
                            "location": "871cf34d-e490-430f-a0d5-21833230cb46",
                            "client": "d8bc691e-4907-4f6f-8b9f-cefb52c7eeaa",
                            "employee": null,
                            "professional": "dff82ed3-a2a4-46e0-b17f-53c51ed8d4bd",
                            "assistant": "dff82ed3-a2a4-46e0-b17f-53c5aef8c2b1",
                            "services": 
                            [
                                "af9714fe-bae4-44b2-ac45-ef5afa97f147",
                                "95310d51-89dd-4cae-9908-b65856e87392"
                            ],
                            "clients_module": "Desktop",
                            "comments": "nice",
                            "notifySms": "",
                            "cancel": false,
                            "cancelDate": null,
                            "cancelReason": "",
                            "client_notified": true,
                            "createDate": "2019-01-01T00:00:00.000Z",
                            "paid": false
                        },
                        {
                            "id": "4bf92104-9fba-431c-aa13-d7f2305fdd7a",
                            "date": "2019-01-03T11:00:00.000Z",
                            "duration": 30,
                            "location": "871cf34d-e490-430f-a0d5-21833230cb46",
                            "client": "d8bc691e-4907-4f6f-8b9f-cefb52c7eeaa",
                            "employee": null,
                            "professional": "dff82ed3-a2a4-46e0-b17f-53c51ed8d4bd",
                            "assistant": "dff82ed3-a2a4-46e0-b17f-53c5aef8c2b1",
                            "services": 
                            [
                                "af9714fe-bae4-44b2-ac45-ef5afa97f147",
                                "95310d51-89dd-4cae-9908-b65856e87392"
                            ],
                            "clients_module": "Desktop",
                            "comments": "nice",
                            "notifySms": "",
                            "cancel": false,
                            "cancelDate": null,
                            "cancelReason": "",
                            "client_notified": true,
                            "createDate": "2019-01-01T00:00:00.000Z",
                            "paid": false
                        }
                    }
                }
            }

## Prepayments [/appointments/prepayments]
<a name="prepayments"></a>

Severl requests for appointments prepayments.

### Send sms to client with link for prepayment [POST /appointments/{id}:sendPrepaymentRequest] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

Send sms to client with link for appointment prepayment.

Next prerequisites are required:
1) Appointment is in 'planned' or 'confirmed' state
2) At least one appointment service exists in appointment
3) Appointment has a client (not a guest) and client had not refused to receive notification smses
4) Sms service and sms prepayment text was configured

In these cases 400 Bad Request will be returned.

IF sms succesfully sent, sms send result will be returned:

```
{
    "status": "Accepted"
}
```

If error while sending sms happened, next result will be returned:
```
{
    "smsError":
    {
        "type": "SMS_BALANCE",
        "message": "Not enough money for sending sms"
    }
}
```
Where type can be [SMS](#error-SMS), [SMS_AUTH](#error-SMS_AUTH) or [SMS_BALANCE](#error-SMS_BALANCE) - use `type` for checks in code and show corresponding info for user. `message` describes additional details and can help developer to understood problem.

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "status": "Accepted"
            }

### Generate parameters for appointment prepayment [POST /appointments/{id}/prepaymentRequest{?format}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

Generate parameters or url for payment processor to make prepayment for appointment services.

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - appointment id
    + format: `parameters` (enum[string], required) - response format - JSON object with parameters or prepared URL
        + Members
            + `parameters`
            + `url`

+ Request

    + Attributes
        + type: `required` (enum[string],required) - type of prepayment
            + Members
                + `required`
                + `optional`
                + `late`
        + language: `en-US` (string) - preffered user language
        + appointmentServices (object,required) - information about appointment services. Depending from `type`, should be received from appointment `requiredPrepayment`, `optionalPrepayment` or `latePrepayment` fields

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "type": "required",
                "language": "en",
                "appointmentServices":
                [
                    {
                        "id": "3f90adea-01a2-42c3-b346-b15a9b6b365e",
                        "amount": 100
                    },
                    {
                        "id": "4026b1d9-6dbd-4921-9205-0b25537eb0f0",
                        "amount": 250
                    }
                ]
            }

+ Response 200 (application/json)

    + Body

            {
                "merchantAccount": "123456789",
                "amount": "100"
            }

+ Request

    + Attributes
        + type: `required` (enum[string],required) - type of prepayment
            + Members
                + `required`
                + `optional`
                + `late`
        + language: `en-US` (string) - preffered user language
        + appointmentServices (object,required) - information about appointment services. Depending from `type`, should be received from appointment `requiredPrepayment`, `optionalPrepayment` or `latePrepayment` fields

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "type": "required",
                "language": "en",
                "appointmentServices":
                [
                    {
                        "id": "3f90adea-01a2-42c3-b346-b15a9b6b365e",
                        "amount": 100
                    },
                    {
                        "id": "4026b1d9-6dbd-4921-9205-0b25537eb0f0",
                        "amount": 250
                    }
                ]
            }

+ Response 200 (application/json)

    + Body

            {
                "url": "https://secure.wayforpay.com/pay/get?merchantAccount=123456789&amount=100"
            }

## Feedbacks [/feedbacks]

For each appointment for each professional where appointment service item exists, client can leave a feedback.

+ Attributes
    + date: `2018-09-01T10:10:10Z000` (datetime, required) - feedback create date
    + appointment: `32b4ac32-da44-465d-a12a-915f50d1f84f` (identifier) - appointment, for which feedback was left
    + firstName: `Denis` (string) - feedback client name (read only, taken from appointment)
    + professional: `88d606ab-c596-353b-6676-79bc3fd265d2` (identifier) - professional, for whom feedback was left, see [Employees](#employees)
    + professional_name: `Herold` (string) - professinal name (read only, taken from professional)
    + professional_photo_exists: `true` (boolean) - if professional has photo (read only, taken from professional)
    + rating: 5 (number) - feedback rating (from 0 to 5)
    + text: `feedback` (string) - feedback message left by client. Max length is 65536 characters

### Get all feedbacks [GET /feedbacks{?fields,appointment,professional}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameter
    + fields: `firstName,appointment,date,professional,professionalName,professionalPhotoExists,rating,text` (array[string], required) - list of fields to return (separated by comma).
    + appointment: `32b4ac32-da44-465d-a12a-915f50d1f84f` (identifier, optional) - get feedback only for given appointments (several appointments can be separated by comma)
    + professional: `80fc1a75-f859-426d-b1b3-c89f478150b3` (identifier, optional) - get feedback only for given professionals (several professionals can be separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "806ab8d6-9c5a-5c3b-ba76-2279bc365dfd",
                    "appointment": "32b4ac32-da44-465d-a12a-915f50d1f84f",
                    "firstname": "Denis",
                    "date": "2018-09-01T10:10:10Z000",
                    "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                    "rating": 5,
                    "text": "Good job"
                },
                {
                    "id": "6860a8db-c9c5-15bb-6ca1-a3b3a57d6dc1",
                    "appointment": "32b4ac32-da44-465d-a12a-915f50d1f84f",
                    "firstname": "Denis",
                    "date": "2018-09-01T10:10:10Z000",
                    "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                    "rating": 5,
                    "text": "Cool!"
                }
            ]
            
### Get feedback by id [GET /feedbacks/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + id: `d608b86a-59c6-3b5c-623a-7965d2bc3fd2` (identifier, required) - feedback id
    + fields: `appointment,firstName,date,professional,professionalName,professionalPhotoExists,rating,text` (array[string], required) - list of fields to return (separated by comma).
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "d608b86a-59c6-3b5c-623a-7965d2bc3fd2",
                "appointment": "32b4ac32-da44-465d-a12a-915f50d1f84f",
                "firstname": "Denis",
                "date": "2018-09-01T10:10:10Z000",
                "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                "rating": 100,
                "text": "Good job"
            }

### Create feedback [POST /feedbacks]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c
            
    + Body

            {
                "firstname": "Denis",
                "appointment": "32b4ac32-da44-465d-a12a-915f50d1f84f",
                "date": "2018-09-01T10:10:10Z000",
                "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                "rating": 5,
                "text": "Nice work"
            }

+ Response 204

### Update feedback [PUT /feedbacks/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id feedback

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c
            
    + Body

            {
                "firstname": "Denis",
                "date": "2018-09-01T10:10:10Z000",
                "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                "rating": 5,
                "text": "Nice work"
            }

+ Response 204

### Delete feedback [DELETE /feedbacks/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `client_access_token` (but `client` field should match client id), `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - feedback id
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 204

## Halls [/halls]
<a name="halls"></a>

Halls where services or group lessons are provided.

+ Attributes
    + name: `massage room` (string,required) - hall name. Max length is 200 characters
    + color: `#ff0000` (string) - hall color (in schedule)
    + location: `d32bfd16-2d63-4937-acfe-fada1a064fe9` (identifier,required) - location id [Locations](#locations)
        + Default: current location from token
    + zone: `d32bfd16-2d63-4937-acfe-fada1a064fe1` (identifier) - zone id
    + forbid_simultaneous_visits: true (boolean) - can more than one person walk to the hall
    + daily_pay: 100 (number) - salary for professional duty in this hall per day. Can't be negative.
    + hourly_pay: 20 (number) - salary for professional duty in this hall per hour. Can't be negative.

### Get all halls [GET /halls{?fields,name,location}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + fields: `name,color,location,zone,daily_pay,hourly_pay` (array[string], required) - list of fields to return (separated by comma).
    + name: `massage room` (string, optional) get only hall with given name (several hall can be separated by comma)
    + location: `d32bfd16-2d63-4937-acfe-fada1a064fe1` (identifier, optional) get only hall with given location (several hall can be separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                    "name": "massage room",
                    "location": "d32bfd16-2d63-4937-acfe-fada1a064fe9",
                    "zone": null,
                    "daily_pay": 1000,
                    "hourly_pay": 100
                },
                {
                    "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
                    "name": "gym room",
                    "location": "d32bfd16-2d63-4937-acfe-fada1a064fe9",
                    "zone": null,
                    "daily_pay": 1000,
                    "hourly_pay": 100
                }
            ]

### Get hall by id [GET /halls/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`,`clients_module`,`reports`

+ Parameters
    + id: `c9edc51c-a95a-4f9c-84fd-67022b957d79` (identifier, required) - id of hall
    + fields: `name,color,location,zone,daily_pay,hourly_pay` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
                "name": "solarium room",
                "location": "c9edc51c-a95a-4f9c-84fd-67022b957d32"
            }

### Create new hall [POST /halls{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,color,location,zone,daily_pay,hourly_pay` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "solarium room",
                "location": "c9edc51c-a95a-4f9c-84fd-67022b957d32",
                "daily_pay": 500,
                "hourly_pay": 100
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "bd98b869-2e5c-4e0d-be16-ce715a35c742"
            }

### Update hall [PUT /halls/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of hall
    + fields: `name,color,location,zone,daily_pay,hourly_pay` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "gym room"
            }
            
+ Response 204

### Delete hall [DELETE /halls/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `bd98b869-2e5c-4e0d-be16-ce715a35c742` (identifier, required) - id of hall

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

# Group Sales

This section provides information about sold items and closed tickets

## Account [/accounts]
<a name="accounts"></a>
This section provides information about accounts. Account is place where money is kept. Each account has it's name and money sum currently stored. Examples of accounts: cashdesk, bank account, safe (do not includes client deposits or company deposits).

+ Attributes
    + id: `88d4b305-e198-f02c-2743-cca9390c6d92` (identifier) - account id
    + name: `Cash` (string, required) account name. Max length is 200 characters
    + location: `88d4b305-e198-f02c-2743-cca9390c6d91` (identifier) - location id, see [Locations](#locations) for which account belongs to
        + Default: current location from token
    + type: `cashdesk` (enum) - type of account
        + Members
            + bank
            + other
    + sum: 0 (number) - amount in the account (read only), sum on the account is automatically calculated from sales and can be changed only by creating new sales via /sales/purchase or /sales/payment
    + archive: false (boolean) - if account was archived and can't be used in new sales

### Get all accounts [GET /accounts{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `online_store`, `reports`

+ Parameter
    + fields: `name,location,type,sum,archive` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d4d8bf-a9f1-8063-7bfe-b80f2d3d9b58",
                    "name": "Cash",
                    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
                    "type": "cashdesk",
                    "sum": 1000,
                    "archive": false
                },
                {
                    "id": "88d4d8bf-a9f4-8da3-7bfe-b80f437c45a6",
                    "name": "Account in JPMorgan",
                    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
                    "type": "bank",
                    "sum": 2000,
                    "archive": false
                },
                {
                    "id": "88d4d8bf-a9f4-8da3-7bfe-b80f437c45a6",
                    "name": "Account in Citygroup",
                    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
                    "type": "bank",
                    "sum": 500,
                    "archive": false
                }
            ]

### Get account by id [GET /accounts/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `online_store`, `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - account id
    + fields: `name,location,type,sum,archive` (array[string], required) - list of fields to return (separated by comma).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d4d8bf-a9f4-8da3-7bfe-b80f437c45a6",
                "name": "Account in JPMorgan",
                "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
                "type": "bank",
                "sum": 2000,
                "archive": false
            }

### Create new account [POST /accounts{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,location,type,sum,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "Owner's safe",
                "type": "other",
                "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
            }

### Update account [PUT /accounts/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `5c05c7a4-74da-42e5-ac5d-dbc489748143` (identifier, required) - account id 
    + fields: `name,location,type,sum,archive` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "type": "bank"
            }

+ Response 204 (application/json)

### Delete account [DELETE /accounts/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - account id

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Sale [/sales]

Sales only for last 2 month are available. If you need to get earlier sales, you'll have to contact us individually.

+ Attributes
    + id: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier, required) - sale id
    + sale_date: `2018-09-16T20:02:00.000Z` (datetime) - date when sale was made
    + calendar_date: `2018-09-16T21:02:00.000Z` (datetime) - booked time for providing service (or group lesson) by this sale
    + duration: 120 (number) - booked time duration for service or group lesson
    + location: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - location id, see [Locations](#locations)
        + Default: current location from token
    + location2: `19401352-c6da-45f9-8d15-0caae0aeabf1` (identifier) - for some rare cases, one sale belongs to two locations. At current moment only one such case exists: transfer money from account of one location to some account in other location. In such case `location2` will match id of second location, in all other situations it will be null [Locations](#locations)
    + name: `Gold card` (string) - service/product/group/card/certificate name
    + type (enum) - type sold 
        + Members
            + service - service was sold
            + product - product was sold
            + card - card was sold
            + certificate - certificate was sold
            + group - one time group visit was sold
            + tips - tips left for professional
            + salary - salary paid to employee
            + supplier - payment to supplier for products supply
            + client_deposit - client put money on deposit or get from deposit
            + company_deposit - company put money on deposit or get from deposit
            + refund - refund money for client
            + transfer - move money between accounts
            + payment - some other expences/profits made
    + product_id: `c3cb7a99-22f2-4290-b33d-5f10cc5992d1` (identifier) - id of sold service, product, certificate, group or card.
    + client: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - client id, see [Clients](#clients)
    + professional: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - employee who provides service or group lesson, see [Employees](#employees)
    + professional_name: `John Doe` (string) - employee name (read only)
    + professional_photo_exists: true (boolean) - if employee has photo (read only)
    + sum: 1000 (number) - amount paid by client
    + quantity: 2 (number) - sale quantity
    + product_quantity_type: `package` (enum) - type of quantity
        + Members
            + portion
            + units
    + receptionist: `4c99e293-6755-425d-b7aa-c5d125e65657` (identifier) - receptionist id
    + item_id: `4c99e293-6755-425d-b7aa-c5d125e65651` (identifier) - lesson id for group visit [Group lesson](#grouplessons)
    + deposit_sum: 1000 (number) - sum withdrawn or put on client deposit
    + deposit_client: `e39ab383-8b89-4744-8b75-0ddc1372ddbe` (identifier) - client, whom deposit was used
    + discount_sum: 100 (number) - one-time discount sum for client
    + `discount_reason`: `new year` (string) - reason why one-time discount was given for client
    + `client_card`: `4c99e293-cbda-3cf1-c115-fa2a31aeb0c3` (identifier) - client card used for discounting current sale [Client cards](#client cards)
    + `client_card_discount`: `4c99e293-cbda-3cf1-c115-fa2a31aebc85` (identifier) - client card discount used for discounting current sale
    + recommended: `19401352-c6da-45f9-8d15-0caae0ae7085` (identifier) - who recommended to buy service/product
    + appointment: `f9b01352-cbda-3cf1-c115-fa2a3cae2081` (identifier) - appointment id [Appointments](#appointments), which describes visit details
    + cancel: false (boolean) - if sale was cancelled
    + cancelReason: `sick leave` (string) - the reason why receptionist cancelled the sale
    + cancelDate: `2018-09-17T20:02:00.000Z` (datetime) - date when sale was cancelled
    + payments (array) - information about all payments
        + (object)
            + account: `4c99e293-cbda-3cf1-c115-fa2f31ceb681` (identifier) - account id [Account](#accounts)
            + sum: 250 (number) - amount in the account
            + commissionSum: 10 (number) - commission amount
    + products (array) - list of product changes in sale
        + (object)
            + product: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier) - product that was added/removed
            + storage: `a31daff0-6a03-1c2e-a2e2-418a120bca11` (identifier) - storage where product was added/removed
            + quantity: `-2` (number) - quantity change of product (positive - added to storage, negative - removed from storage)
            + sum: `-250` (number) - cost of product change (sign of `sum` is same as sign of `quantity`), sum is calculated base on actual prices on sale `date`
    + commonTicket: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - id of ticket, all items sold in one ticket will have the same value of this field

### Get all sales [GET /sales{?fields,sale_date_from,sale_date_to,calendar_date_from,calendar_date_to,client,professional,location,type}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameter
    + fields: `sale_date,calendar_date,duration,location,location2,type,name,client,professional,sum,quantity,product_quantity_type,product_id,product_name,receptionist,item_id,deposit_sum,deposit_client,discount_sum,discount_reason,client_card,client_card_discount,recommended,appointment,cancel,cancelReason,cancelDate,products` (array[string], required) - list of fields to return (separated by comma).
    + sale_date_from: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only sales which was made at `sale_date_from` or later
    + sale_date_to: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only sales which was made before `sale_date_from`
    + calendar_date_from: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only sales which was scheduled to time `sale_date_from` or later
    + calendar_date_to: `2018-09-16T21:02:00.000Z` (datetime, optional) - get only sales which was scheduled to time earlier than `sale_date_from`
    + client: `88d4b305-e198-f02c-2743-cca9390c6d9b` (array[identifier], optional) - get only sales for given client (several clients can be separated by comma)
    + professional: `88d4b305-e198-f02c-2743-cca9390c6d91` (array[identifier], optional) - get only sales for given professional (several professionals can be separated by comma)
    + location: `0be9ab15-2569-47e1-96e3-ae3211a7a7ec` (array[identifier], optional) - get only sales from given location (several locations can be separated by comma)
    + type: `Service` (enum) - get only sales from given type (several types can be separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d61289-34c6-0b1e-3b5f-838036bfebfd",
                "sale_date": "2018-09-04T20:09:42.000Z",
                "calendar_date": "2018-09-04T21:00:00.000Z",
                "duration": 60,
                "location": "88d60c1b-f953-b89b-0312-ba0502b8bc35",
                "location2": null,
                "name": "Personal training 3 category",
                "type": "service",
                "product_id": "88d60c1b-f953-b89b-0312-ba0502b82c31",
                "client": "88d60d95-adf7-1cb4-5016-ff2d57b87fce",
                "professional": "88d606ab-c596-48d2-6676-79bc63a6e813",
                "professional_name": "John Doe",
                "professional_photo_exists": true,
                "sum": 500,
                "quantity": 2,
                "product_quantity_type": "units",
                "receptionist": null,
                "item_id": null,
                "deposit_sum": 0,
                "deposit_client": "88d60c1b-f953-b89b-0312-ba0502bfcb22",
                "discount_sum": 100,
                "discount_reason": "birthday",
                "client_card": null,
                "client_card_discount": null,
                "recommended": null,
                "appointment": "88d61289-34c6-0b1e-3b5f-a3f0bc1ce321",
                "cancel": false,
                "cancelReason": "",
                "cancelDate": null,
                "payments": [],
                "products":
                [
                    {
                        "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 1200
                    },
                    {
                        "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 300
                    }
                ]
            },
            {
                "id": "88d61289-70a2-19e0-3b5f-83800e4e4512",
                "sale_date": "2018-09-04T20:11:22.000Z",
                "calendar_date": "2018-09-04T21:00:00.000Z",
                "duration": 120,
                "location": "88d60c1b-f953-b89b-0312-ba0502b8bc35",
                "location2": null,
                "name": "Personal training 2 category",
                "type": "service",
                "product_id": "88d60c1b-f953-b89b-0312-ba0502b82c31",
                "client": "88d60d95-adf0-3eaa-5016-ff2d41c8a8c7",
                "professional": "88d606ab-c596-353b-6676-79bc3fd265d2",
                "professional_name": "John Doe",
                "professional_photo_exists": true,
                "sum": 1000,
                "quantity": 1,
                "product_quantity_type": "portion",
                "receptionist": "88d60c1b-f953-b89b-0312-ba0c02ba2cf2",
                "item_id": null,
                "deposit_sum": 0,
                "deposit_client": "88d60c1b-f953-b89b-0312-ba0502bfcb22",
                "discount_sum": 100,
                "discount_reason": "birthday",
                "client_card": "88d60c1b-f953-b89b-0312-ba1c12bfcbcf",
                "client_card_discount": "88d60c1b-fee3-b8fb-0d12-cab542b2c1cb",
                "recommended": null,
                "appointment": "88d61289-34c6-0b1e-3b5f-a3f0bc1ce322",
                "cancel": true,
                "cancelReason": "client didn't come",
                "cancelDate": "2018-09-04T20:11:22.000Z",
                "payments": [
                    {
                        "account": "88d60c1b-fee3-b8fb-0d12-cab54abbcfc1",
                        "sum": 200
                    }
                ],
                "products":
                [
                ]
            }

### Get sale by id [GET /sales/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`, `client_access_token` (but `client` field should match client id), `reports`

+ Parameters
    + id: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier, required) - id sale
    + fields: `sale_date,calendar_date,duration,location,location2,type,name,client,professional,sum,quantity,product_quantity_type,product_id,product_name,receptionist,item_id,deposit_sum,deposit_client,discount_sum,discount_reason,client_card,client_card_discount,recommended,appointment,cancel,cancelReason,cancelDate,products` (array[string], required) - list of fields to return (separated by comma).
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "88d61289-34c6-0b1e-3b5f-838036bfebfd",
                "sale_date": "2018-09-04T20:09:42.000Z",
                "calendar_date": "2018-09-04T21:00:00.000Z",
                "duration": 60,
                "location": "88d60c1b-f953-b89b-0312-ba0502b8bc35",
                "location2": null,
                "name": "Personal training 3 category",
                "type": "service",
                "product_id": "88d60c1b-f953-b89b-0312-ba0502b82c31",
                "client": "88d60d95-adf7-1cb4-5016-ff2d57b87fce",
                "professional": "88d606ab-c596-48d2-6676-79bc63a6e813",
                "professional_name": "John Doe",
                "professional_photo_exists": true,
                "sum": 500,
                "quantity": 1,
                "product_quantity_type": "units",
                "receptionist": "88d60c1b-f953-b89b-0312-ba0c02ba2cf2",
                "item_id": null,
                "deposit_sum": 0,
                "deposit_client": "88d60c1b-f953-b89b-0312-ba0502bfcb22",
                "discount_sum": 100,
                "discount_reason": "birthday",
                "client_card": "88d60c1b-f953-b89b-0312-ba1c12bfcbcf",
                "client_card_discount": "88d60c1b-fee3-b8fb-0d12-cab542b2c1cb",
                "recommended": null,
                "appointment": "88d61289-34c6-0b1e-3b5f-a3f0bc1ce322",
                "cancel": false,
                "cancelReason": "",
                "cancelDate": null,
                "payments": 
                [
                    {
                        "account": "88d60c1b-fee3-b8fb-0d12-cab54abbcfc1",
                        "sum": 200
                    },
                    {
                        "account": "88d60c1b-61c3-b8fb-0f12-cfb54abb8fc2",
                        "sum": 300
                    }
                ],
                "products":
                [
                    {
                        "product": "983eb289-1e66-413c-8052-7cbcc3e806bc",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 1200
                    },
                    {
                        "product": "adfb426a-f467-4f20-8af7-b723c9cb0ed0",
                        "storage": "41678090-9109-4750-aa38-6e3444f55fa0",
                        "quantity": 1,
                        "sum": 300
                    }
                ]    
            }

## Cancel sales [/sales:]

Program supports sale cancellation. Here they are.


### Check if sales can be cancelled [POST /sales:canCancel]

This method verifies if provided sales can be cancelled and returns list of sales that can't be cancelled with localized reason messages.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Attributes
        + sales (array[identifier], required) 
                 - list of sales to  be verify (separated by comma).
       
    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
               "sales":
                [
                    "ae9134c8-439c-40a9-a784-04885275caa1",
                    "dd439cc8-9114-aa79-8404-04feda885f7b",
                    "50e07101-6a5d-458e-af39-0e7b6b204014"
                ]
            }

+ Response 200 (application/json)

    + Body

            [
                {
                    "sale": "ae9134c8-439c-40a9-a784-04885275caa1",
                    "message": "This card can't be canceled, had related sales",
                    "relatedSales":
                    [
                      "ae9134c8-439c-40a9-a784-04885275caa1",
                      "dd439cc8-9114-aa79-8404-04feda885f7b",
                      "50e07101-6a5d-458e-af39-0e7b6b204014"
                    ]
                },
                {
                    "sale": "ae9134c8-439c-40a9-a784-04885275caa1",
                    "message": "This card can't be canceled",
                }
            ]

### Make refund for online payment order [POST /sales:refund]

If prepayment was payed via some payment processor, refund for such payment can be done via this endpoint. Only full refund is supported (not partial one). New sale object will be created with type 'onlineRefund' and money will be returned to client card via payment processor. Please keep in mind that too many refunds can lower your merchant rating and can be a reason to suspend your merchant account.
Refund can also be initiated from client bank or payment processor side.

On refund, system will try to cancel all prepayments done with original payment. If some prepaid service was paid, prepayment will not be cancelled. In such case and in case prepayment already cancelled, corresponding refunded sum will be subtracted from client deposit.

Method fails if refund already done.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Attributes
        + `order`: 'dd439cc8-9114-aa79-8404-04feda885f7b' (identifier, required) - payment processor order
        + `reason`: 'Service temporary not be provided' (string, optional) - reason to cancel sale
      
    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
                "order": "dd439cc8-9114-aa79-8404-04feda885f7b",
                "reason": "Service temporary not be provided"
            }

+ Response 204

### Cancel sales [POST /sales:cancel]

Cancel provided list of sales.
If some sales can't be cancelled, returns 409 Conflict error with same result as `/sales:canCancel` method

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Attributes
        + cancelReason (string) - reason to cancel sale
        + sales (array[identifier], required)
                 - list of sales to be canceled (separated by comma).
      
    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
                "cancelReason": "office error" 
                "sales":
                [
                    "ae9134c8-439c-40a9-a784-04885275caa1",
                    "dd439cc8-9114-aa79-8404-04feda885f7b",
                    "50e07101-6a5d-458e-af39-0e7b6b204014"
                ]
            }

+ Response 204

## Promotions [/promotions]

Promotions allows to give discount to client based on set of rules.

+ Attributes
    + id: `5382f467-a034-440c-bc7f-3dac932f8b99` (identifier, required) - Promotion id.
    + name: `New Year Promotion` (string, required) - Promotion name.
    + category: `5382f467-a034-440c-bc7f-3dac932f8b99` (identifier, optional) - Promotion category id.
    + discount: 10 (number, optional) - Discount percent for promotion.
    + sum: 100 (number, optional) - Discount sum for promotion.
    + unlimitedTime: `true` (boolean, optional) - If promotion doesn't have time limits.
    + condition: `Promotion condition` (string, optional) - Promotion condition.
    + description: `Promotion description` (string, optional) - Promotion description.
    + descriptionOnReceipt: `true` (boolean, optional) - If promotion description is placed on a receipt.
    + discountType: `Percent` (enum[string], optional) - Type of discount for promotion.
        + Members
            + FixedSum
            + FixedSumPlusPercent
            + FixedPriceSum
    + startDate: `2017-01-01T13:51:55.000Z` (datetime, optional) - Start date of promotion.
    + expireDate: `2018-01-01T13:51:55.000Z` (datetime, optional) - End date of promotion.
    + archive: false (boolean, optional) - if promotion is archived.

### Get all promotions [GET /promotions{?fields,archive}]

Get information about all promotions.

**Authorization:** TODO

**Scope:** TODO

+ Parameters
    + fields: `name,category,discount,sum,unlimitedTime,condition,description,descriptionOnReceipt,discountType,startDate,expireDate,archive` (array[string], required) - list of fields to return (separated by comma).
    + archive: false (boolean, optional) - get only archived or non archived promotions

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d6248d-e5e8-9c40-5296-bb10043eab27",
                    "name": "New Year Promotion",
                    "category": "5382f467-a034-440c-bc7f-3dac932f8b99",
                    "discount": 10,
                    "sum": 50,
                    "unlimitedTime": true,
                    "condition": "Promotion condition",
                    "description": "Description",
                    "descriptionOnReceipt": false,
                    "discountType": "Percent",
                    "startDate": "2017-01-01T13:51:55.000Z",
                    "expireDate": "",
                    "archive": false
                }
            ]

### Get promotion by id [GET /promotions/{id}{?fields}]

Get information about promotion with specified id.

**Authorization:** TODO

**Scope:** TODO

+ Parameters
    + fields: `name,category,discount,sum,unlimitedTime,condition,description,descriptionOnReceipt,discountType,startDate,expireDate,archive` (array[string], required) - list of fields to return (separated by comma).
    + id: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (identifier, required) - promotion id to get information for.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body
            
            {
                "id": "88d6248d-e5e8-9c40-5296-bb10043eab27",
                "name": "New Year Promotion",
                "category": "5382f467-a034-440c-bc7f-3dac932f8b99",
                "discount": 10,
                "sum": 50,
                "unlimitedTime": true,
                "condition": "Promotion condition",
                "description": "Description",
                "descriptionOnReceipt": false,
                "discountType": "Percent",
                "startDate": "2017-01-01T13:51:55.000Z",
                "expireDate": "",
                "archive": false
            }

### Create new promotion [POST /promotions{?fields}]

**Authorization:** TODO

**Scope:** TODO

+ Parameters
    + fields: `name,category,discount,sum,unlimitedTime,condition,description,descriptionOnReceipt,discountType,startDate,expireDate,archive` (array[string], optional) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "name": "New Year Promotion",
                "category": "5382f467-a034-440c-bc7f-3dac932f8b99",
                "discount": 10,
                "sum": 50,
                "unlimitedTime": false,
                "condition": "Promotion condition",
                "description": "Description",
                "descriptionOnReceipt": false,
                "discountType": "Percent",
                "startDate": "2017-01-01T13:51:55.000Z",
                "expireDate": "2018-01-01T13:51:55.000Z",
                "archive": false
            }
            
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }

### Update promotion [PUT /promotions/{id}{?fields}]

**Authorization:** TODO

**Scope:** TODO

+ Parameters
    + id: `be0b6712-e680-42a7-8b99-b6b2d9fcb1fe` (identifier, required) - id of promotion to update.
    + fields: `name,category,discount,sum,unlimitedTime,condition,description,descriptionOnReceipt,discountType,startDate,expireDate,archive` (array[string], optional) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "unlimitedTime": true,
                "startDate": "2019-01-01T13:51:55.000Z",
                "expireDate": ""
            }
            
+ Response 204

### Delete promotion [DELETE /promotions/{id}]

**Authorization:** TODO

**Scope:** TODO

+ Parameters
    + id: `cc903b03-ecaf-46d4-a030-bbfd1882f490` (identifier, required) - id of promotion to delete.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204         

## Crosssale success reports [/sales/crossale/report_success]

Internal methods for recommendation system

+ Attributes
    + sex: `Male` (enum[string]) - client sex 
        + Members
            + Female
    + selected_product: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - product that was selected
    + proposed_product: `88d4b305-e198-f02c-2743-cca9390c6d9b` (identifier) - product that was recommended
    + success: `true` (boolean) - if client accepted proposal

### Report about new success [POST /sales/crossale/report_success]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request

    + Headers

            Authorization: Bearer 9a068ce2-4dcf-7d30-a811-8f62ced7ed9b

    + Body

            {
                "sex": "Male",
                "selected_product": "9c4068e2-c81f-4d70-ad31-8f627ed9bced",
                "proposed_product" : "8f627e91-a23a-dc48-15b3-c40bce68e2d9",
                "success" : true
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "79dbd056-7f23-4e87-892f-c45e1976e694"
            }

## Crosssale recommendations [/sales/crosssale/recommendations]

Represents crosssale recommendation details.

+ Attributes
    + selected_product: `daf6c0f0-6903-4c9e-a0e2-4381676cd641` (identifier) - product that was selected
    + proposed_product: `abf2c1f2-6903-4c9e-ab11-23f1cba1dc4c` (identifier) - product that was recommended
    + certainty: 0.5 (number) - rating from 0 to 1

### Get recomendation [GET /sales/crosssale/recommendations{?current_items,client,sex,number,proposed_product_type}]

**Authorization** `Database`, `Employee`

**Scope** `full`

+ Parameter
    + current_items: `e1f6c0b0-afc3-419e-74e2-43a1c7663654, daf6c0f0-6903-4c9e-a0e2-4381676c3641` (array[identifier], required) - get only this items,  (separated by comma)
    + client: `eac6c2b0-afc3-ab9e-cbe2-13a1c7663651` (identifier, required) - id client
    + sex: `Male` (enum[string], optional) - client sex, can be empty
        + Default: `Male`
        + Members
            + `Male`
            + `Female`
    + number: 10 (number) - range from 1 to 100
        + Default 4
    + proposed_product_type: `services` (enum[string], optional) - get only items with this type 
        + Default: `all`
            + Members
                + `services`
                + `products`
                + `all`

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body
    
            [
                {
                    "selected_product": "9c4068e2-c81f-4d70-ad31-8f627ed9bced",
                    "proposed_product": "8f627e91-a23a-dc48-15b3-c40bce68e2d9",
                    "certainty": 0.1
                },
                {
                    "selected_product": "1c3c6be2-c81f-4d70-ad31-8f627ed9bced",
                    "proposed_product": "25637ea1-a23a-dc48-15b3-c40bce68e2d9",
                    "certainty": 0.2
                }
            ]

# Group Callbacks

Represents callback details

## Callbacks [/callbacks]

+ Attributes
    + date: `2019-01-01T00:00:00.000Z` (datetime, required) - date of callback
    + phone: `+380 (00) 555 55 55` (string) - phone for callback

### Get all callbacks [GET /callbacks{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,phone` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                   "id": "dfb93597-7e01-4d5e-b563-208625ca7f51",
                   "date": "2019-01-01T00:00:00.000Z",
                   "phone": "+380 (00) 555 55 55"
                },
                {
                   "id": "a6c7571a-038e-4d15-a733-29928b0195c1",
                   "date": "2019-01-02T00:00:00.000Z",
                   "phone": "+380 (00) 556 56 56"
                }
            ]    
    
### Get callback by id [GET /callbacks/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `79fa7afb-c567-4a6a-a4b7-3186190c728d` (identifier, required) - id callback
    + fields: `date,phone` (array[string], required) - list of fields to return (separated by comma)
    
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
               "id": "dfb93597-7e01-4d5e-b563-208625ca7f51",
               "date": "2019-01-01T00:00:00.000Z",
               "phone": "+380 (00) 555 55 55"
            }
    
### Create new callback [POST /callbacks{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `clients_module`

+ Parameters
    + fields: `date,phone` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
        
    + Body

            {
                "date": "2019-01-01T00:00:00.000Z",
                "phone": "+380 (00) 655 65 65"
            }

+ Response 201 (application/json)

    + Body

            {
                "id": "475a20e9-23ef-40ce-b2c4-c7dbcf230ebb"
            }

### Update callback [PUT /callbacks/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `79fa7afb-c567-4a6a-a4b7-3186190c728d` (identifier, required) - id callbacks
    + fields: `date,phone` (array[string], optional) - list of fields to return (separated by comma)

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
        
    + Body

            {
                "date": "2019-01-01T00:00:00.000Z",
                "phone": "+380 (00) 555 55 55"
            }

+ Response 204 (application/json)

### Delete callback [DELETE /callbacks/{id}] 

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `432c41bf-0cd7-4f13-83dd-a0c26ce29143` (identifier, required) - id of callback

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

# Group Messaging

Describes text (sms) and e-mail single messages and campaigns.

## SMSes [/smses]

Clients can receive text messages (smses). This section helps to get information about sent messages.

+ Attributes
    + id: `daf6c0f0-6903-4c9e-a0e2-4381676c364d` (identifier, required) - id of sms
    + date: `2018-03-01T00:00:00.000Z` (datetime, required) - date when sms was sent (not received)
    + client: `77a61323-b591-4490-9e88-3b5f30695b25` (identifier) - client for whom sms was sent
    + text: `Hello!` (string) - sms message. Max length is 15000 characters
    + appointment_notification: true (boolean) - if sms is notification for appointment

### Get all smses [GET /smses{?fields,client,appointment_notification}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `client_access_token` (only with filter client and id must match token owner)

+ Parameter
    + fields: `date,client,text,appointment_notification` (array[string], required) - list of fields to return (separated by comma).
    + client: `fc3c5f2c-3dee-4b63-b555-1912482b3b69` (identifier, optional) - get only smses for given client (several clients can be separated by comma)
    + `appointment_notification`: true (boolean, optional) -  get only appointment notification smses (if true) or others (if false)

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body
    
            [
                {
                    "id": "f6c0daf0-6903-4c9e-a0e2-4381676dc364",
                    "date": "2018-03-01T00:00:00.000Z",
                    "client": "77a61323-b591-4490-9e88-3b5f30695b25",
                    "text": "Hello!",
                    "appointment_notification": true
                }
            ]

### Get sms by ID [GET /smses/{id}{?fields}]

**Authorization** `Database`, `Employee`

**Scope** `full`, `client_access_token` (only with filter client and id must match token owner)

+ Parameter
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - id of sms
    + fields: `date,client,text,appointment_notification` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 200 (application/json)

    + Body
    
            {
                "id": "f6c0daf0-6903-4c9e-a0e2-4381676dc364",
                "date": "2018-03-01T00:00:00.000Z",
                "client": "77a61323-b591-4490-9e88-3b5f30695b25",
                "text": "Hello!",
                "appointment_notification": true
            }

# Group Settings

This section describes common settings for database, location settings, hardware settings and user preferences and some other.

At current stage hardware settings not implemented.

## Common settings [/settings]

Settings, common for whole database (several locations). Like country, default language, currency, etc.
These settings describes the way program should look and behave. They are usually set up at program first use and rarely changed in future.

All settings are identified by name, they belongs to predefined list shown below. You can't set any setting not in list,
only change settings from predefined list.

All settings are grouped in sections.

+ Attributes
    + `common` (object) - section `common`
        + `type`: beauty (string) - program name ("beauty" or "fitness"), read only
        + `licenseType`: Ultimate (string) - license type for a program (Light, Standard or Ultimate), read only
        + `isDemo` : false(boolean) - if database is in demo period, read only
        + `databaseStatus` : Active_Work (string) - status of client's database, read only
        + `country`: USA (country) - country where business is located
        + `currency`: USD (currency) - currency for all money operations
        + `language`: en (language) - default language for all users
        + `logo` (image) - business logo (max size 2048x2048)
        + `client_feedback_ratings` (array[object]) - list of rating items for client feedback about place
    + `information` (object) - section `information`
        + `description` (text_in_different_languages) - html-formatted description of this location in several languages. If needed language not found, "en" is default
        + `web_site`: `https://beautysalon.com` (url) - business web site
        + `instagram`: `https://instagram.com/beautysalon` (url) - business Instagram page
        + `facebook`: `https://facebook.com/beautysalon` (url) - business Facebook page
        + `viber`: `+1 800 555 0123` (phone) - business Viber page phone number
        + `telegram`: `https://t.me/beautysalon` (url) - business telegram page
    + `statistics` (object) - section `statistics`
        + `default_appointment_duration`: 120 (number) - typical duration for appointment for last month, if less than 10 appointments - 60 minutes is used (read only). Typical duration means one of [15, 30, 60, 90, 120, 180] minutes that covers 80% of last month durations.
    + `client_module` (object) - section `client_module`
        + `name`: beautysalon (string) - name for beauty salon online module. Module will be available at `https://beautyprosoftware.com/b/[name]` (if name not set, database code can be used as name). `Name` should have 3 symbols or longer, consists of symbols 'A'-'Z', 'a'-'z' (case matters), '0'-'9', '_' and '-' and contains at least one letter. If one salon already use some name, other salon can't set same name as his own (first always wins). Can't be set for clients with Light software version. Setting this option to null means that clinet module should be accessed via database code not name.
        + `enabled`: true (boolean) - if client module is enabled
        + `several_services`: true (boolean) - if client can book several services for one appointment
        + `can_cancel_in_48_hours`: true (boolean) - if appointment can be cancelled by client if more than 48 hours left
        + `services_gender_filter`: `both` (enum[string]) - filter for services (all, only male, only female)
            + Default: both
            + Members
                + male
                + female
        + `nearest_booking_minutes`: 0 (number) - appointments can be made for "now + `nearest_booking_minutes` minutes" or later
            + Default: 0
        + `time_step`: `auto` (enum[string]) - size of time block (in minutes), professional work time is splitted by
            + Default: `auto`
            + Members
                + 5m
                + 10m
                + 15m
                + 20m
                + 30m
                + 60m
                + 90m
                + 120m
                + 150m
                + 180m
        + `calendar`: `week` (enum[string]) - type of calendar, for 5 days of for a month
            + Default: `week`
            + Members
                + week
                + month
        + `routes` (array[enum[string]]) - routes available for clients for booking
            + Members
                + timeServiceProfessional
                + serviceProfessionalTime
                + professionalServiceTime
        + `color`: `#6f3bf5` (color) - color used as theme color (if not set, theme default color will be used)
        + `theme`: `soft` (enum[string]) - client module theme
            + Default: `soft`
            + Members
                + normal
                + strong
        + `language`: null (enum[string]) - if set, default language (loaded at module start). If not set - selected according user browser language
            + Default: null
            + Members
                + cs
                + en
                + lv
                + ru
                + uk
        + `logo` (image) - business logo in client module (max size 640x160)
        + `supported_languages` (array[string]) - returns array of supported languages (read only)
        + `gaps_mode`: `none` (enum[string], optional) - mode used to optimize gaps in professional time (more aggressive node returns less time slots)
            + Default: `none`
            + Members
                + `none` - no optimizations, all available time slots returned
                + `optimal` - ignoring `step`, step calculated based on `duration`
                + `maximal` - allows booking only on start or end of free time slot (for example, booking for 1 hour if free time slots 9:00-12:00 and 14:00-21:00 available will return 9:00, 11:00, 14:00 and 20:00), specific positions allowed depends from `gaps_positions`
                + `maximal_if_has_appointment` - use `maximal` mode if professional has more than one free time slot for specific day (for example, 9:00-12:00 and 14:00-21:00, at least one appointment already assigned), if only one time slot (for example, 9:00-21:00, no appointments yet or appointments at day start/end) use `optimal` mode
        + `gaps_positions`: `day_start` (array[string], optional) - for `maximal` & `maximal_if_has_appointment` `gaps_mode` defines what times are available for booking (several options can be selected). For example, professional works from 9:00 till 21:00, already has bookings 9:00-10:30, 13:00-14:00 and 18:00-18:30, so free time slots are 10:30-13:00, 14:00-18:00 and 18:30-21:00, `duration` is 1 hour. See possible positions in options descriptions.
            + Default: `day_start`
            + Members
                + `day_start` - at day start (or first free time after day start if any time at day start already booked). In example above will return time 10:30.
                + `before_appointment` - time just before appointment in the middle of the day. In example above will return time 12:00 and 17:00.
                + `after_appointment` - time just after appointment in the middle of the day. In example above will return time 14:00 and 18:30.
                + `day_end` - just before day end (or last free time before day end if any time at day end already booked). In example above will return time 20:00.
        + `confirm_services`: false (boolean) - if appointment should be verified by receptionist after booking
            + Default: false
        + `button_text`: `online record` (string) - name of client module button
        + `button_color`: '#6f3bf5' (color) - color button at HEX
        + `button_position`: 'bottom right' (enum) - position of module button
            + Members
                + `top right`
                + `bottom left`
        + `element_id`: 'element id' (string) - element id to attach on click
        + `services_from_filter`: `3f69b533-8288-4847-a7f0-ab4d71e598f6,3f69b533-8288-4847-a7f0-ab4d71e59c21;1,0` (filter_info) - Filter that describes which services should be presented with word "from" before price.
        + `googleAnalyticsCode`: 'US-1234567-89' (string) - Google Analytics code for tracking clients behaviour in client module 
        + `showServicesAndGroupsDescriptions`: false (boolean) - if services and groups descriptions should be shown
    + `prepayments` (object) - section `prepayments`
        + `active`: true (boolean) - if prepayments activated
            + Default: false
        + `moreThanMinutesToStart`: 120 (number) - minimum number of minutes to appointment start from now for prepayments to be available on appointment
        + `requiredActive`: true (boolean) - if required prepayments activated
            + Default: false
        + `requiredType`: 'full' (enum[string]) - method used to calculate required payment amount
            + Members
                + `full` - full sum for all services should be payed (same as `percent` with `requiredPercent` = 100)
                + `fixedForAppointment` - fixed amount for whole appointment, amount is stored in `requiredAmount`
                + `fixedForService` - fixed amount for each ordered service, amount is stored in `requiredAmount`
                + `percent` - percent of total appointment sum, percent is stored in `requiredPercent`
                + `time` - fixed amount `requiredAmount` for every `requiredTime` minutes (number of booked minutes rounded up to integer number of `requiredTime` minutes blocks)
        + `requiredPercent`: 100 (number) - percent of total appointment sum (form 0 to 100), 100 means full prepayment, percent value is stored in `requiredPercent` (if `requiredType` = 'percent')
        + `requiredAmount`: 25 (number) - fixed amount for one service (if `requiredType` = 'fixed') or for `requiredTime` minutes (if `requiredType` = 'time')
        + `requiredTime`: 30 (enum[number]) - size of time block for calculating prepayment total amount (time will be rounded up to size of such block), in minutes
            + Members
                + 1
                + 15
                + 30
                + 60
        + `optionalRewardType`: 'discount' (enum[string]) - type of reward for optional prepayment
            + Members
                + `discount` - client receives instant discount on prepayment sum
                + `bonus` - client earns bonuses he can spend on later purchases
        + `optionalRewardPercent`: 10 (number) - percent of discount/bonus client receives for prepayment (from 0 to 100)
        + `smsTemplate`: '' (string) - sms template text used to create smses for clients with proposal to make a prepayment

### Get all settings [GET /settings{?fields}]

Get settings. You must provide all settings you want to receive in `fields` parameter (not only sections, but all settings you needed).

**Authorization:** `Database`, `Employee`

**Scope:** `clients_module` (only `common` section), `full`

+ Parameters
    + fields: `common(type,country,currency,client_feedback_ratings),information(description,web_site,instagram,facebook,viber,telegram),client_module(name,enabled,several_services,can_cancel_in_48_hours,services_gender_filter,nearest_booking_minutes,time_step,calendar,color,theme,language,supported_languages,button_text,button_color,element_id)` (array[string], required) - settings to get

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "common":
                {
                    "type": "beauty",
                    "country": "USA",
                    "currency": "USD",
                    "licenseType": "Ultimate",
                    "isDemo": false,
                    "databaseStatus": "Active_Work",
                    "client_feedback_ratings":
                    {
                        "3f69b533-8288-4847-a7f0-ab4d71e598f6": {
                            "names": {
                                "en": "Service quality",
                                "fr": "Qualité de service",
                            }
                        },
                        "3b82c804-ce68-4f31-999c-ca9002d3fcbe": {
                            "names": {
                                "en": "Customer care",
                                "fr": "Service à la clientèle",
                            }
                        },
                        "6c9a8313-72b0-4087-9968-00a293ec6152": {
                            "names": {
                                "en": "Interior",
                                "fr": "Intérieur",
                            }
                        }
                    }                    
                },
                "information":
                {
                    "description":
                    {
                        "en": "Some description <span style='color: #FF0000;'>with color</span>",
                        "fr": "Quelques description <span style='color: #FF0000;'>avec couleur</span>"
                    },
                    "web_site": "salon123.com",
                    "instagram": "instagram.com/salon123",
                    "facebook": "facebook.com/salon123.com",
                    "viber": "+1 800 555 0123",
                    "telegram": "t.me/salon123.com"
                },
                "client_module":
                {
                    "name": "beautysalon",
                    "enabled": true,
                    "several_services": true,
                    "can_cancel_in_48_hours": true,
                    "services_gender_filter": "both",
                    "nearest_booking_minutes": 0,
                    "time_step": "auto",
                    "calendar": "week",
                    "color": "#000000",
                    "theme": "soft",
                    "language": null,
                    "supported_languages":
                    [
                        "en",
                        "fr"
                    ],
                    "minimize_gaps_mode": "maximal",
                    "minimize_gaps_position":
                    [
                        "day_start",
                        "before_appointment",
                        "after_appointment"
                    ],
                    "confirm_services": false,
                    "button_text": "online record",
                    "button_color": "#6f3bf5",
                    "element_id": "element",
                    "services_from_filter": "all"
                }
            }

### Get settings picture [GET /settings/{section}/{setting}{?width,height,resize,access_token}]

Get settings picture by setting section and name.

**Authorization:** `Database`, `Employee`

**Scope:** `clients_module` (only `common` section), `full`

+ Parameters
    + section: `common` (string, required) - section name to get picture
    + setting: `logo` (string, required) - setting name to get picture
    + width: 100 (number, optional) - image width needed
    + height: 100 (number, optional) - image height needed
    + resize: `fit` (enum[string], optional) - type of resize for image
        + Default: `fit`
        + Members
            + `fit`
            + `fit_center_transparent`
            + `stretch`
            + `crop`
    + access_token: `9c4068e2-c81f-4d70-ad31-8f627ed9bced` (string, optional) - token used for API requests

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (image/jpeg)

    + Body

### Update settings [PUT]

In some cases settings should be updated. This can be done specifing settings new values.
Not all values can be changed. List of values that can be changed will be added later.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "common":
                {
                    "country": "USA",
                    "currency": "USD"
                }
            }

+ Response 204

### Set settings picture [PUT /settings/{section}/{setting}]

Set settings picture by setting section and name.

If additional header `Test` is set, input values are validating, but not saved.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + section: `common` (string, required) - section name to set picture
    + setting: `Logo` (string, required) - setting name to set picture

+ Request (image/jpeg)

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Test: true

+ Response 204

## Required prepayment weekdays [/requiredPrepaymentWeekdays]

+ Attributes
    + weekday: `sunday` (enum[string],required) - day of week with required prepayment
        + Members
            + `sunday`
            + `monday`
            + `tuesday`
            + `wednesday`
            + `thursday`
            + `friday`
            + `saturday`
    + start: `00:00` (string,required) - time from which prepayment is required
    + end: `24:00` (string,required) - time till which prepayment is required

### Get all required prepayment days [GET /requiredPrepaymentWeekdays{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:**  `full`

+ Parameter
    + fields: `weekday,start,end` (array[string], required) - list of fields to return (separated by comma).
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                    "weekday": "sunday",
                    "start": "00:00",
                    "end": "24:00"
                }
            ]
            
### Get required prepayment day by id [GET /requiredPrepaymentWeekdays/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameter
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - required prepayment day id
    + fields: `weekday,start,end` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body 

            {
                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                "weekday": "sunday",
                "start": "00:00",
                "end": "24:00"
            }
            
### Create new required prepayment day [POST /requiredPrepaymentWeekdays{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `weekday,start,end` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                "weekday": "sunday",
                "start": "00:00",
                "end": "24:00"
            }
             
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }
            
### Update required prepayment day [PUT /requiredPrepaymentWeekdays/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - required prepayment day id
    + fields: `weekday,start,end` (array[string], required) - list of fields to return (separated by comma).
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    
    + Body
            
            {
                "start": "12:00"
            }
            
+ Response 204

### Delete required prepayment day [DELETE /requiredPrepaymentWeekdays/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - required prepayment day id
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204

## Required prepayment days [/requiredPrepaymentDays]

+ Attributes
    + date: `2019-02-15T00:00:00.000Z` (datetime,required) - date with required prepayment
    + start: `00:00` (string,required) - time from which prepayment is required
    + end: `24:00` (string,required) - time till which prepayment is required
    + employee: `41678090-9109-4750-aa38-6e3444f55fa0` (identifier) - employee for which prepayment is required (null - for all employees)

### Get all required prepayment days [GET /requiredPrepaymentDays{?fields,from,to}]

**Authorization:** `Database`, `Employee`

**Scope:**  `full`

+ Parameter
    + fields: `date,start,end,employee` (array[string], required) - list of fields to return (separated by comma).
    + from: `2019-01-01T00:00:00.000Z` (datetime, required) - start date to get required prepayment days
    + to: `2019-01-03T00:00:00.000Z` (datetime, required) - end date to get required prepayment days
    
+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                    "date": "2019-02-15T00:00:00.000Z",
                    "start": "00:00",
                    "end": "24:00",
                    "employee": "88d61182-23fc-c6c9-03da-4259788d120a"
                }
            ]
            
### Get required prepayment day by id [GET /requiredPrepaymentDays/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameter
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifier, required) - required prepayment day id
    + fields: `date,start,end,employee` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 751392d5-a113-44e4-bb30-0d9e190b2f4c

+ Response 200 (application/json)

    + Body 

            {
                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                "date": "2019-02-15T00:00:00.000Z",
                "start": "00:00",
                "end": "24:00",
                "employee": "88d61182-23fc-c6c9-03da-4259788d120a"
            }
            
### Create new required prepayment day [POST /requiredPrepaymentDays{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `date,start,end,employee` (array[string], required) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
    + Body

            {
                "id": "88d60d95-aa05-283f-5016-ff2d5d4dc20a",
                "date": "2019-02-15T00:00:00.000Z",
                "start": "00:00",
                "end": "24:00",
                "employee": "88d61182-23fc-c6c9-03da-4259788d120a"
            }
             
+ Response 201 (application/json)

    + Body

            {
                "id": "cc903b03-ecaf-46d4-a030-bbfd1882f490"
            }
            
### Update required prepayment day [PUT /requiredPrepaymentDays/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - required prepayment day id
    + fields: `date,start,end,employee` (array[string], required) - list of fields to return (separated by comma).
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
    
    + Body
            
            {
                "employee": null
            }
            
+ Response 204

### Delete required prepayment day [DELETE /requiredPrepaymentDays/{id}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `f6c0daf0-6903-4c9e-a0e2-4381676dc364` (identifer, required) - required prepayment day id
 
+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            
+ Response 204

# Group Additional materials

AI Helps provides some materials to make clients life easier: e-mail templates, sms templates, default services list etc. Application can access and use these
materials as "predefined".

These materials are common for all databases. API allows to get materials but not to change them.

## E-mail templates [/materials/email_templates]

E-mail templates that can be used by clients to make e-mail campaigns or sending single messages.

Please mention that `content` field contains big HTML pages, so good strategy will be to load all templates without `content` field and then
load needed template content by id.

+ Attributes
    + name (string) - Template name. Max length is 200 characters
    + program (enum[string]) - Program template is made for.
        + Default: `Beauty`
        + Members
            + `Beauty`
            + `Fitness`
    + language (string) - Template language code (ISO 639-1).
    + content (string) - Template content (HTML). Max length is 15728640 characters
    + category (identifier) - Template category.
    + category_name (string) - Template category name (read only).

### Get all e-mail templates [GET /materials/email_templates{?fields,program,language}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `name,program,language` (array[string], optional) - list of fields to return (separated by comma).
    + program: `Beauty` (array[enum[string]], optional) - Filter only templates for specific program(s).
        + Members
            + `Beauty`
            + `Fitness`
    + language: `en` (array[string], optional) - Filter only templates on specific language code(s) (ISO 639-1).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "24187db7-05e1-40d2-bd1f-f3ddbb2517b8",
                    "name": "Simple template",
                    "program": "Beauty",
                    "language": "en",
                    "content": "This is <b>simple</b> template",
                    "category": null,
                    "category_name": ""
                },
                {
                    "id": "60dc068b-5660-4f1a-bc6a-8fc1be253b85",
                    "name": "Modèle simple",
                    "program": "Beauty",
                    "language": "fr",
                    "content": "C'est un modèle <b>simple</b>",
                    "category": "f7ddfdfd-51ba-475a-923c-bd615545543d",
                    "category_name": "French version"
                }
            ]

### Get e-mail template by id [GET /materials/email_templates/{id}{?fields}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + id: `24187db7-05e1-40d2-bd1f-f3ddbb2517b8` (identifier, required) - id of client
    + fields: `name,program,language` (array[string], optional) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "id": "24187db7-05e1-40d2-bd1f-f3ddbb2517b8",
                "name": "Simple template",
                "program": "Beauty",
                "language": "en",
                "content": "This is <b>simple</b> template",
                "category": null,
                "category_name": ""
            }

### Save email-template [POST /materials/email_templates/save_template/{code}]

Editing e-mail template requires callback from an editable web-page. Using this request, web-page will send changes to the application, that awaits changes.

**Authorization:** none

+ Parameters
    + code: `24187db7-05e1-40d2-bd1f-f3ddbb2517b8` (identifier, required) - one-time e-mail template identifier, generated by the application

+ Request

    + Body

            `base64 encoded json object with changes`

+ Response 204

### Save image [POST /materials/email_templates/save_image]

Editing e-mail template provides possibility to change images. To make this images available for everyone, we need to save them. 
Furthermore, we need to save this images in such a way, so anyone from any part of the world could load them as fast as possible.

Saving image will use AIHelps CDN infrastructure to make images more accessible.

To save the image, use one of the method, described in the [Update image](#updating-image) section

Response will contain an url, which will substitute existing image inside the template.

**Authorization:** `Database`, `Employee`

+ Request

    + Header

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Type: text/plain

    + Body

            https://image.url.example.com/background.jpg

+ Response 200 (application/json)

    + Body
    
            {
                "url": "https://cdn.aihelps.com/emails/ZjLG93ZWlq.jpg"
            }

# Group Updates

This section describes how to get realtime notifications about the changes in
the database.

Main idea is following:
after any table changed, API generates event, which contains information, what table has changed, type of operation (row inserted, updated or deleted),
row id and new row state. You can subscribe to these events and be sure your local copy of data is always valid.

To handle updates tracking, each update has incremental integer update id (starting from 1 for new database). Each change (row insert, update or delete)
in any table inside one database increment this global counter and save update information with new number as id. All subscribed users will be notified
about that event or you can request batch of updates in case your server was down or connection to our server was lost for some time.

Several moments to keep in mind:
1) If your database is located on some server, you should subscribe for updates via that server (for example, if database located on server 2, make
requests to api2.aihelps.com), servers does not share updates information and you will receive nothing (API will warn you). AFter subscribed, you will receive
events instantly (no matter changes was done via API or direct database access).
2) If your database is located not on one of AIHelps servers but on your custom external server, all mechanisms still works, you can subscribe to any API server, but
you will receive updates once in 15 seconds, not instantly: API has no possibility for deep integration with database and just polling external server every 15 seconds.
Despite internal mechanisms are different, for sake of easyness, public API is same for both cases.
3) When you register webhook or web socket remember that if you send parameter `fields` in the body as empty string waiting for that will be returned only id without items 

There is a way to receive updates via web hooks.

## Updates via web hooks [/updates]

This section describes receiving updates via web hooks.

### Register url [PUT /updates/webhooks/register]

If the client sends an `register` message, server starts sending updates
about the provided table.

**Authorization:** `Database`, `Employee`

**Scope:**  `full`,`clients_module`
  
+ Attributes
    + url: `https://myapp.com/integrations/webhooks/aihelps/updates` (string) - url to which updates will be sent (many different tables updates can be sent to one url)
    + table: `clients` (string) - a table for which client wants to receive updates
    + fields: `name,phone` (string) - all the fields for which client wants to receive updates, separated by comma

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bce1

    + Body

            {
                "url": "https://myapp.com/integrations/webhooks/aihelps/updates",
                "table": "appointments",
                "fields": "date,professional,client"
            }

+ Response 204

### Unregister url [PUT /updates/webhooks/unregister]

If the client sends an `unregister` message, the server stops sending updates
about the provided table.

**Authorization:** `Database`, `Employee`

**Scope:**  `full`,`clients_module`

+ Attributes
    + url: `https://myapp.com/integrations/webhooks/aihelps/updates` (string) - url to which sending updates should be stopped
    + table: `clients` (string) - table for which sending updates should be stopped

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bce1

    + Body

            {
                "https://url": "myapp.com/integrations/webhooks/aihelps/updates",
                "table": "appointments"
            }

+ Response 204

### Batch updates [PUT /updates/webhooks/batch] 

If client sends a `batch` message, server get all updates starting from `from_update` and send updates to all urls for all tables (with same access token)

**Authorization:** `Database`, `Employee`

**Scope:**  `full`,`clients_module`

+ Attributes
    + `from_update`: 3000 (number) - id from which all updates should be sent

+ Request

    + Header

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bce1

    + Body

            {
                "from_update": 3000
            }

+ Response 204

# Group Reports

Retrieving structured reports about sales, appointments, etc.

## Dashboard [/reports/dashboard]
  
Overall summary for a specific location for certain period of time. Contains general information about new clients, sales, appointments etc.

In most dashboard reports you provide report period and numbers for previous period returned for comparison.

Data only for the last 2 month is available. If you need older data you have to contact us individually.

### Get cashdesk [GET /reports/dashboard/cashdesk{?location,employee}]

Returns accounts with non-zero balance, available for provided employee (only accounts provided employee can view balance).

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + employee: `514441ad-ebb4-4da5-80be-745f3bda7173` (identifier, required) - employee, for whom account balances should be shown

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "id": "Cash",
                    "sum": 67893.5
                },
                {
                    "id": "Bank",
                    "sum": 57872,4
                }
            ]

### Get incomes and losses [GET /reports/dashboard/incomeLosses{?location,start,end,employee}]

Returns income and losses for certain period of time and previous period.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated
    + employee: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of an employee, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "income": 8240,3,
                "losses": -6112,43,
                "previousIncome": 7925.94,
                "previousLosses": -6278.2
            }

### Get future appointments [GET /reports/dashboard/futureAppointments{?location,start,end}]

Get information about future appointments:
1) Number of non-cancelled appointments in certain period
2) Sum of non-cancelled appointments in certain period
3) Workload (percent of busy time) for each day in certain period - calculates as total busy time for all professionals divided by total work time for all professionals. Null if no one works in specific day

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "appointments": 57,
                "appointmentsSum": 49,
                "workload":
                [
                    0.57,
                    1.0,
                    0.89,
                    0.78,
                    0.63,
                    0.84,
                    null
                ]
            }

### Get appointments history [GET /reports/dashboard/appointments{?location,start,end,onlyOnlineModule}]

Returns visits (appointments) statistics:
1) `count` - number of appointments for certain period
2) `previousCount` - number of appointments for previous period
3) `payedCount` - number of appointments for certain period where at least one item is payed
4) `payedCountAmount` - some of all payed items

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated
    + onlyOnlineModule: false (boolean, optional) - if true, filter only visits from online module

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "count": 57,
                "previousCount": 49,
                "payedCount": 23,
                "payedCountAmount": 8765.9
            }

### Get products report [GET /reports/dashboard/products{?location,start,end}]

Returns products statistics:
1) Sum of products sold in certain period
2) Sum of products sold in previous period (for comparison)
3) Number of visits with products sold in certain period
4) Total number of visits in certain period

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "amount": 32567.8,
                "previousAmount": 27542.5,
                "visitsWithProducts": 41,
                "totalVisits": 261
            }

### Get clients report [GET /reports/dashboard/clients{?location,start,end}]

Returns clients statistics:
1) Number of clients for certain period (each guest check is separate client, employees are skipped)
2) Number of new clients between clients in item 1
3) Number of clients as in item 1 but for previous period (for comparison)
4) Number of clients in db with "active" status and not archive
5) Total number of clients in database except archive

Returns how many clients visits location, how many new clients and how many those, who visited location more than one time for certain period of time

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "clients": 278,
                "newClients": 57,
                "previousClients": 256,
                "activeInDb": 1521,
                "totalInDb": 3624
            }

### Get clients retention [GET /reports/dashboard/clientsRetention{?location,start,end}]

Returns information about how many clients was retentioned comparing certain period of time.

Let's assume 3 periods exists:
1) current: (start, end)
2) previous: (prevStart, prevEnd), where prevEnd = start and previous duration equals to current duration
2) pre-previous: (prevPrevStart, prevPrevEnd), where prevPrevEnd = prevStart and pre-previous duration equals to current (and previous) duration

So, `clientsRetention` is part of clients from previous period that also come in current period. For example, in previous period was 100 different clients, in current only 85 of that 100 come at least once, so `clientsRetention` will be 0.85
`newClientsRetention` is same as `clientsRetention` but only for client that come in previous period first time (never was in location before).
`oldClientsRetention` is same as `clientsRetention` but only for client that already come to location before previous period.
`previousRetention` is part of clients from pre-previous period that also come in previous period. Can be used to show progress.

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "clientsRetention": 0.61,
                "newClientsRetention": 0.36,
                "oldClientsRetention": 0.84,
                "previousRetention": 0.37
            }

### Get cards report [GET /reports/dashboard/cards{?location,start,end}]

Returns how many cards was sold and amount of sold cards for certain period of time

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "count": 29,
                "amount": 5799.9
            }

### Get services rating [GET /reports/dashboard/servicesRating{?location,start,end,maxCount}]

Returns services rated by turnover for certain period of time

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated
    + maxCount: 6 (integer, required) - how many services will be in a list

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "service": "1ad4e8bb-08c6-44fe-b1c9-85d91a8d7d75",
                    "amount": 27956.7
                },
                {
                    "service": "7cd5b0f3-e50d-475c-9e18-486fbbdf9d52",
                    "amount": 25958.3
                },
                {
                    "service": "8926e418-e85f-4eb4-9dea-47035806746f",
                    "amount": 20887.2
                },
                {
                    "service": "ca2f3d24-073f-4828-b2c8-aee59c70e3d2",
                    "amount": 16656.8
                },
                {
                    "service": "fa3850c7-eed1-4cfd-b71b-fd30f0d1d9d1",
                    "amount": 13966.6
                },
                {
                    "service": "7bdfb8bf-fb06-4fba-be93-1d6d175ac05e",
                    "amount": 7956.1
                }
            ]

### Get professionals rating [GET /reports/dashboard/professionalsRating{?location,start,end,maxCount}]

Returns professionals rated by turnover for certain period of time

**Authorization:** `Database`, `Employee`

**Scope:** `full`, `reports`

+ Parameters
    + location: `3ec0a275-2276-4178-a0fc-002ff46b81fe` (identifier, required) - id of a location, for which dashboard information will be generated
    + start: `2019-08-27T00:00:00.000Z` (datetime, required) - start of a period, for which dashboard information will be generated
    + end: `2019-08-28T00:00:00.000Z` (datetime, required) - end of a period, for which dashboard information will be generated
    + maxCount: 5 (integer, required) - how many professionals will be in a list

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "professional": "1ad4e8bb-08c6-44fe-b1c9-85d91a8d7d75",
                    "amount": 27956.7
                },
                {
                    "professional": "7cd5b0f3-e50d-475c-9e18-486fbbdf9d52",
                    "amount": 25958.3
                },
                {
                    "professional": "8926e418-e85f-4eb4-9dea-47035806746f",
                    "amount": 20887.2
                },
                {
                    "professional": "ca2f3d24-073f-4828-b2c8-aee59c70e3d2",
                    "amount": 16656.8
                },
                {
                    "professional": "fa3850c7-eed1-4cfd-b71b-fd30f0d1d9d1",
                    "amount": 13966.6
                }
            ]

# Group NPS

These endpoints helps to track users Net Promoter Score.

## Show NPS dialog [/nps]

These endponts helps to show NPS dialog for user with access token. Thus these endpoints works only with employee access token.

### Check if to show NPS dialog [POST /nps/start]

Check if NPS dialog should be shown (usually enough time from last time shown). If endpoint returns `true`, it also creates record for NPS measurement with status "Not submitted".
To submit result, call next endpoint.

Please mention: as this endpoint starts new record (if returns `true`), next calls to this endpoint for a long period of time (for at least several months) will return false (do not show
too often even if user did not submit result). So, do not call it several times.

**Authorization:** `Employee`

**Scope:** `full`

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            {
                "start": true
            }

### Submit result [POST /nps/submit]

Submit result, if user selected score. Should be called during 24 hours after `/nps/start`.
If score <= 8, task for customer success added to check if everything is OK.

**Authorization:** `Employee`

**Scope:** `full`

+ Request 200 (application/json)

    + Attributes
        + score: 10 (number, required) - score from 0 to 10
        + comments: `` (string, optional) - user comments

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "score": 10,
                "comments": "I love your product!"
            }
            
+ Response 204

# Group Tasks and Long requests

## Tasks [/tasks]

AI Helps API provides service for executing planned tasks (something similar to `cron`). You can setup tasks for one-time call at specified time
or execute task minutely, hourly, daily, monthly or each year at specified time.

Tasks are executed on server in protected environment and user can't affect how tasks are executed. Planned tasks can be removed but if tasks started
it can't be stopped until fully executed.

Types of tasks executed are limited and will be added later.

+ Attributes
    + type (enum[string]) - Type of task to execute.
        + Members
            + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
            + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
            + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.
    + id (string) - Task id. Each task is identified by `type`+`id` pair (thus for each type you can add only one task with same `id`). This `id` should be provided (not like other objects where `id` is autogenerated).
    + arguments (object) - JSON object with details about task to be executed.
    + date (datetime) - Date and time of task to be executed (with precision up to 1 second). If task is periodic - date and time of first execution. If date/time already past, task will be executed immediately.
    + period (enum[string]) - how often task should be executed.
        + Default: `once`
        + Members
            + `once` - task ewill be executed once and removed after execution
            + `day` - task will be executed each day at specified time
            + `month` - task will be executed each month at specified date and time
            + `year` - task will be executed each year at specified date and time
    + period_multiplier (number) - multiplies time for `period`. For example, if `period` = 'day' and `period_multiplier` = 3, task will be executed once in three days.
        + Default: 1

### Get all planned tasks [GET /tasks{?fields,type}]

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + fields: `type,id,arguments` (array[string], optional) - list of fields to return (separated by comma).
    + type (enum[string], optional) - Filter only tasks of specified type.
        + Members
            + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
            + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
            + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Body

            [
                {
                    "type": "http_request",
                    "id": "report_regeneration",
                    "arguments":
                    {
                        "url": "https://site.com/regenerate_report.php",
                        "method": "POST"
                    },
                    "date": "2017-01-01T13:51:55.000Z",
                    "period": "day",
                    "period_multiplier": 1
                },
                {
                    "type": "send_sms_campaign",
                    "id": "f070c39e-2c0c-4f2c-83bd-eabb7d86bcbd",
                    "arguments": null,
                    "date": "2017-01-01T12:00:00.000Z",
                    "period": "once",
                    "period_multiplier": 1
                }
            ]

### Get information about planned task [GET /tasks/{type}/{id}{?fields}]

Get information about planned task. If task not exists and was never executed, returns 404 error.
If task was removed, but executed at least once, task parameters will be retrieved from last execution.

Anyway, if task exists or not if task was executed at least once, `last_result` returns information about last execution (otherwise `null`).
Some task types can provide `last_result` during task execution, you can query `last_result` for task before task finished.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + type (enum[string], required) - Type of task to execute.
        + Members
            + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
            + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
            + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.
    + id (string, required) - Task id. Each task is identified by `type`+`id` pair (thus for each type you can add only one task with same `id`).
    + fields: `type,id,arguments` (array[string], optional) - list of fields to return (separated by comma).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 200 (application/json)

    + Attributes

        + type (enum[string], required) - Type of task to execute.
            + Members
                + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
                + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
                + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.
        + id (string, required) - Task id. Each task is identified by `type`+`id` pair (thus for each type you can add only one task with same `id`).
        + arguments (object, optional) - JSON object with details about task to be executed.
        + date (datetime, required) - Date and time of task to be executed (with precision up to 1 second). If task is periodic - date and time of first execution. If date/time already past, task will be executed immediately.
        + period (enum[string], optional) - how often task should be executed.
            + Default: `once`
            + Members
                + `once` - task ewill be executed once and removed after execution
                + `day` - task will be executed each day at specified time
                + `month` - task will be executed each month at specified date and time
                + `year` - task will be executed each year at specified date and time
        + period_multiplier (number) - multiplies time for `period`. For example, if `period` = 'day' and `period_multiplier` = 3, task will be executed once in three days.
            + Default: 1
        + active (boolean) - if task is active (not removed)
        + last_result (object) - result of task last run (JSON object, fields depends from task type, in case of error field `error` always present)

    + Body

            {
                "type": "http_request",
                "id": "report_regeneration",
                "arguments":
                {
                    "url": "https://site.com/regenerate_report.php",
                    "method": "POST"
                },
                "date": "2017-01-01T13:51:55.000Z",
                "period": "day",
                "period_multiplier": 1,
                "active": true,
                "last_result":
                {
                    "status": 200,
                    "response": "Done"
                }
            }

### Add new task [PUT /tasks]

If task with given `type` and `id` already exists, task will be updated (second task will not be created).

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Request (application/json)

    + Attributes

        + type (enum[string], required) - Type of task to execute.
            + Members
                + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
                + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
                + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.
        + id (string, required) - Task id. Each task is identified by `type`+`id` pair (thus for each type you can add only one task with same `id`). This `id` should be provided (not like other objects where `id` is autogenerated).
        + arguments (object, optional) - JSON object with details about task to be executed.
        + date (datetime, required) - Date and time of task to be executed (with precision up to 1 second). If task is periodic - date and time of first execution. If date/time already past, task will be executed immediately.
        + period (enum[string], optional) - how often task should be executed.
            + Default: `once`
            + Members
                + `once` - task ewill be executed once and removed after execution
                + `day` - task will be executed each day at specified time
                + `month` - task will be executed each month at specified date and time
                + `year` - task will be executed each year at specified date and time
        + period_multiplier (number) - multiplies time for `period`. For example, if `period` = 'day' and `period_multiplier` = 3, task will be executed once in three days.
            + Default: 1

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

    + Body

            {
                "type": "http_request",
                "id": "report_regeneration",
                "arguments":
                {
                    "url": "https://site.com/regenerate_report.php",
                    "method": "POST"
                },
                "date": "2017-01-01T13:51:55.000Z",
                "period": "day",
                "period_multiplier": 1
            }

+ Response 204

### Remove task [DELETE /tasks/{type}/{id}]

If task is already executing, task will be deleted, but execution continue.

**Authorization:** `Database`, `Employee`

**Scope:** `full`

+ Parameters
    + type (enum[string], required) - Type of task to execute.
        + Members
            + `http_request` - execute request to some external url. Required argument `url` (address to call), optional `method` (default `GET`), `id` field ignored
            + `send_sms` - send sms saved in database with specified `id`. SMS should not be either sent yet or cancelled.
            + `send_sms_campaign` - send sms feed saved in database with specified sms feed `id`. SMS feed should not be cancelled, only sending not sent yet and not cancelled smses.
    + id (string, required) - Task id. Each task is identified by `type`+`id` pair (thus for each type you can add only one task with same `id`).

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced

+ Response 204

## Long running requests queue [/queue]

Some requests in API can take long time. For such requests there is no reason to keep connection. In this case server response with 202 Accepted. In location header server return url to check task staus or cancel task (if available).

### Get task status [GET /queue/{task}]

Each task could be in one of following statuses:

- `progress` - task still executing. Request status once more in several seconds. Task in this status can be tried to cancel (if supported by this task type).
- `success` - task was finished successfully, task result is provided in `result` field (if there any meaningful result, depends from task type).
- `error` - unexpected error happened during task execution, error message is provided in `error` field.
- `cancelling` - task was succesfully called to cancel, but not stopped yet.
- `cancelled` - task was succesfully called to cancel and cancel process is finished, task result (what was done before `cancel` called) is provided in `result` field (if there any meaningful result, depends from task type).

After you received status `success`, `error` or `cancelled` (final statuses, they can't change), information about task will be deleted and next calls with same task id will return 404.

Please mention, that tasks from regular API and internal API are different, thus providing task id from regular API to this endpoint will return 404 Not Found.

+ Parameter
    + task: `2653-245` (string, required) - task id for which get status (received in response with 202 status for long running operations)

+ Response 200 (application/json)

    + Body

            {
                "status": "progress"
            }

+ Response 200 (application/json)

    + Body

            {
                "status": "success",
                "result":
                {
                    "files": 37
                }
            }

+ Response 200 (application/json)

    + Body

            {
                "status": "error",
                "error": "Access denied to /tmp folder"
            }

+ Response 200 (application/json)

    + Body

            {
                "status": "cancelling"
            }

+ Response 200 (application/json)

    + Body

            {
                "status": "cancelled",
                "result":
                {
                    "files": 12
                }
            }

### Cancel task [DELETE /queue/{task}]

If task supports cancellation, tries to cancel execution of current task. Depending from task already made changes may be reverted (if possible) or process just stopped without reverting.
If task is on stage where next step is complete (no intermediate steps left), cancellation failed (returs false).
Each task could be in one of following statuses:

If task do not support cancellation, 501 Not Implemented will be returned.

+ Parameter
    + task: `2653-245` (string, required) - task id to cancel (received in response with 202 status for long running operations)

+ Response 200 (application/json)

    + Body

            {
                "success": true
            }

# Group Web push notifications

Web push notifications is a technology, that allows users to receive messages, sent by a web site.
Messages can be sent at any time and they will be shown as soon as user will open his browser.

## Push notifications [/push_notifications]

### Subscribe user to notifications [POST /push_notifications/subscribe]

**Authorization:** Employee, Database

To send notifications to the user, user has to be subscribed.
Browser will ask user to subscribe, and user should allow browser to receive push notifications from web site.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Type: application/json

    + Body

            {
                "client_id": "7e744e1e-bf96-4f08-9eaf-7a94a78185a7"
                "endpoint": "https://example.enpoint.url.com/",
                "keys": {
                    "p256dh": "zj0CAQYIKoZIzj0DAQcDQgAE8xOUetsCa8EfOlDEBAfREhJqspDoyEh6Szz2in47Tv5n52m9dLYyPCbqZkOB5nTSqtscpkQD/HpykCggvx09iQ=",
                    "auth": "PmiurYh939ir7ROgeA9usm=="
                },
                "device_name": "Chrome 59"
            }

+ Response 200 (application/json)

    + Body

            {
                "success": true
            }

### Unsubscribe user from notification [POST /push_notifications/unsubscribe]

**Authorization:** Employee, Database

To stop sending notifications to the client, this endpoint should be used.

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Type: application/json

    + Body

            {
                "endpoint": "https://example.enpoint.url.com/"
            }

+ Response 200 (application/json)

    + Body

            {
                "success": true
            }

### Send push notification [POST /push_notifications/send/{client_id}]

**Authorization:** Employee, Database

Sending message to all devices, that is subscribed to notifications.

+ Parameters
    + client_id: `7e744e1e-bf96-4f08-9eaf-7a94a78185a7` (identifier, required) - id of a client, who should stop receiving notifications

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Type: application/json

    + Body

            {
                "ttl": 86400,
                "subject": "mailto:mail@example.com",
                "payload": "Message to be sent"
            }

+ Response 200 (application/json)

    + Body

            {
                "total": 3,
                "success": 2
            }

### Send batch push notifications [POST /push_notifications/send/batch]

**Authorization:** Employee, Database

Sending message to many clients

+ Request

    + Headers

            Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced
            Content-Type: application/json

    + Body

            {
                "ttl": 86400,
                "send_info": [
                    {
                        "client_id": "7e744e1e-bf96-4f08-9eaf-7a94a78185a7",
                        "subject": "mailto:mail@example.com",
                        "payload": "Message to be sent"
                    }
                ]
            }

+ Response 200 (application/json)

    + Body

            {
                "total": 3,
                "success": 2
            }

# Group WebHooks

The application may use some external services. Currently, such services are email and SMS sending services.

Webhooks is a technology, that allows such services to alert application about some event happened with a custom callback.
So the information will be sent to the application as soon as event occurred without awaiting.
No request is required for a webhook, it just sends the data when it’s available.

For example, if e-mail was sent via external service, this service can provide notification about such events as
* Delivered
* Opened
* Link clicked
* Unsubscribed
* etc.

To make it work, we need to provide an endpoint url to this services, an url, on which events will be sent.

For now, url format is:
`https://api.aihelps.com/v1/webhooks/{database_code}/{implementation_type}/{implementation_name}`

## Webhooks [/webhooks]

### Process webhook request [POST /webhooks/{database}/{type}/{name}]

**Authorizathion:** none

Processing webhook request, that contains list of events

+ Parameters
    + database: `123456` (int, required) - database code of a client, who should receive event notification
    + type: `email_senders` (enum[string], required) - type of an service
        + Members
            + `email_senders` - for receiving events from e-mail services
            + `sms_sender` - for receiving events from sms services
    + name: `send_pulse` (string, required) - name of a service, depends on `type`
        + Members
            + `send_pulse` - for SendPulse (`type=email_sender`)
            + `mail_chimp` - for MailChimp (`type=email_sender`)
            + `send_grid` - for SendGrid (`type=email_sender`)

+ Request

    + Body

            `webhook request body, sent by a service`

+ Response 204

# Group Helper methods

This section contains helper methods that can be helpful.

All methods in this section works without authorization.

## Person names [/helpers/names]

These methods helps processing person names.

Person names (in most common case) can contain next parts:
1) Title - "Mr.", "Ms." etc. Not used in most cases, still used for backward compability
2) Firstname - person name
3) Middlename - optional field for some countries
4) Lastname - person surname

Order of these parts differs in different cultures. For example, in USA format is "Firstname Lastname" (we try to ignore Title),
in eastern Europe common format is "Lastname Firstname Middlename" and so on.

### Generate person full name from parts [GET /helpers/names/format{?title,firstname,lastname,middlename}]

Generate full name from name parts. Additionally, method tries to predict person sex (find name in list of male/female names, works for different countries, not only US).

Content language header should be set for this method (because format of full name and sex prediction depend from language).

**Authorization:** none

+ Parameters
    + title: Mr. (string, optional) - Person title
    + firstname: John (string, optional) - Person firstname
    + middlename: `` (string, optional) - Person middlename
    + lastname: Doe (string, optional) - Person lastname

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Attributes
        + fullname (string) - Person full name
        + sex (enum[string]) - Predicted person sex
            + Default: null
            + Members
                + `Male`
                + `Female`
                + null
        
    + Body

            {
                "fullname": "John Doe",
                "sex": "Male"
            }

### Parse person name into separate parts [GET /helpers/names/parse{?fullname}]

Try to parse full person name. Method tries to find firstnames, titles or middlenames in string using big database for different cultures, separating lastname.
If failed, try to predict parts using default order for language. So, both "John Doe", "Mr. Doe John" ("John" found in firstnames, Mr. in titles, list, another
part is surname) and "JohnA Doe" (JohnA not in names list but parts order is default).

Additionally, if firstname is recognized, method tries to predict person sex (find name in list of male/female names, works for different countries, not only US).

Content language header should be set for this method (because format of full name and sex prediction depend from language).

**Authorization:** none

+ Parameters
    + fullname: John Doe (string, required) - Person full name

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Attributes
        + title: "Mr." (string) - Person title
        + firstname: "John" (string) - Person firstname
        + middlename: "" (string) - Person middlename
        + lastname: "Doe" (string) - Person lastname
        + sex (enum[string]) - Predicted person sex
            + Default: null
            + Members
                + `Male`
                + `Female`
                + null
        
    + Body

            {
                "fullname": "John Doe",
                "sex": "Male"
            }

### Order in which name parts is shown in full name [GET /helpers/names/order]

In different cultures name parts in full name is shown in different order (separated by spaces). This function just returns order of these parts.

Content language header should be set for this method (because format of full name depends from language).

Elements that can be returned: `title`, `firstname`, `middlename`, `lastname`.

**Authorization:** none

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Body

            [
                "title",
                "firstname",
                "lastname"
            ]

### Get list of firstnames/middlenames for some culture [GET /helpers/names/list{?sex,type}]

For some cultures lists of firstnames or middlenames is available. These lists exists separately for males, females or combined versions.
You can get these lists using current method.

Content language header should be set for this method (because format of full name and sex prediction depend from language).

**Authorization:** none

+ Parameters
    + sex: male (enum[string], required) - male, female firstnames (middlenames) in list or combined
        + Members
            + `male`
            + `female`
            + `both`
    + type: firstname (enum[string], required) - type of items in list: either firstnames or middlenames
        + Members
            + `firstname`
            + `middlename`

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Body

            [
                "Aaron",
                "Abbott",
                "Abel",
                "Abner",
                "Abraham",
                "...",
                "Zebediah"
            ]

## Countries [/helpers/countries]

These methods represents actual information about countries.

### Get list of countries [GET /helpers/countries{?fields}]

This method returns detailed information about all countries. You can specify fields needed to return using `fields` parameter.

Content language header should be set for this method - countries names will be returned in specified language. This header should be set even if you do not
need `name` field - list of countries are sorted by localized name, so language is needed anyway.

**Authorization:** none

+ Parameters
    + fields: `code3,name,flag` (array[string], optional) - list of fields to return (separated by comma).

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Attributes
        + code2: "US" (string) - two-letter country code (ISO 3166-1 alpha-2)
        + code3: "USA" (string) - three-letter country code (ISO 3166-1 alpha-3)
        + name: "United States" (string) - country name in specified language
        + currency: "USD" (string) - country currency code (ISO 4217)
        + domain: "us" (string) - country national top-level domain name
        + languages: "en", "es" (array[string]) - list of popular languages in country, sorted by usage (most used first)
        + phone: "+1" (string) - international phone number prefix
        + flag: "" (string) - country flag 16x10 pixels in base64 encoding (not shown by default, specify in `fields` to use)

    + Body

            [
                {
                    "code2": "US",
                    "code3": "USA",
                    "name": "United States",
                    "currency": "USD",
                    "domain": "us",
                    "languages": ["en","es","zh-CHT","fr","de","fil","it","vi","ko","ru","nv","yi","haw","chr","lkt","ik"],
                    "phone": "+1",
                    "flag": ""
                }
            ]

### Get country and city by ip [GET /helpers/countries/by_ip{?ip}]

Get country (two- and three-letter country code, ISO 3166-1 alpha-2/3) and city for provided IP address.
If not detected, country and city can be empty string.

**Authorization:** none

+ Parameters
    + ip: `8.8.8.8` (string, optional) - ip to get country and city. If parameter not set, client IP address is used.

+ Response 200 (application/json)

    + Attributes
        + country2: "US" (string) - three-letter country code (ISO 3166-1 alpha-2)
        + country3: "USA" (string) - three-letter country code (ISO 3166-1 alpha-3)
        + city: "New York" (string) - city name

    + Body

            {
                "country2": "US",
                "country3": "USA",
                "city": "New York"
            }

## Phone numbers [/helpers/phones]

These methods represents detailed information about phone numbers.

### Get phone number information [GET /helpers/phones/{phone}{?country}]

This method returns detailed information about phone number.

Phone should contain only digits in international or national format (in second case country needed).

If phone is invalid (wrong number of digits or such country/region code not exists), all response fields will be empty.

**Authorization:** none

+ Parameters
    + phone: `07312345678` (string, required) - phone number in international or national format (in second case country needed), only digits
    + country: `GB` (string, optional) - two-letter or three-letter country code (ISO 3166-1)

+ Response 200 (application/json)

    + Attributes
        + phone: `+44 (312) 345678` (string) - formatted phone number
        + country: GBR (string) - three-letter country code (ISO 3166-1 alpha-3)
        + country_phone_code: +44 (string) - country international phone number prefix
        + is_mobile: true (string) - predicts if phone number is mobile

    + Body

            {
                "phone": "+44 (7312) 345678",
                "country": "GBR",
                "country_phone_code": "+44",
                "is_mobile": true
            }

## Emails [/helpers/emails]

These methods represents detailed information about email address and address validation.

### Validate email address [GET /helpers/emails/{email}]

Validate if email adress is correct (in correct format).

**Authorization:** none

+ Parameters
    + email: info@aihelps.com (string, required) - email address to validate

+ Response 200 (application/json)

    + Attributes
        + email: `info@aihelps.com` (string) - original email address
        + valid: true (string) - if address is in correct format

    + Body

            {
                "email": "info@aihelps.com",
                "valid": true
            }

## Passwords [/helpers/passwords]

These methods helps estimate password quality.

### Estimate password quality [POST /helpers/passwords]

Estimate password and returns information about password quality and suggestions to improve it.

**Authorization:** none

+ Request

    + Attributes
        + password: `qwerty` (string, required) - password to estimate

    + Body

            {
                "password": "qwerty"
            }

+ Response 200 (application/json)

    + Attributes
        + score: 0 (number) - from 0 (very bad) to 4 (very good). Usually 2 and more are enough
        + warning: `This is a top-10 common password` (string) - english language description, why password is bad or not so good
        + suggestions (array[string]) - english language descriptions how to make your password better

    + Body

            {
                "score": 0,
                "warning": "This is a top-10 common password",
                "suggestions":
                [
                    "Add another word or two. Uncommon words are better."
                ]
            }

## Dates [/helpers/dates]

These methods represents detailed information about dates.

### Get date text representation [GET /helpers/dates/{date}]

This method returns date in text representation.

Content language header should be set for this method - text representation will be returned in specified language.

**Authorization:** none

+ Parameters
    + date: `2017-01-01T13:51:55.000Z` (datetime, required) - date in ISO format

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Attributes
        + date: `2017-01-01T13:51:55.000Z` (datetime) - date itself
        + description: two days ago (string) - date text representation

    + Body

            {
                "date": "2017-01-01T13:51:55.000Z",
                "description": "two days ago"
            }

### Get day periods descriptions [GET /helpers/dates/day_periods]

This method returns day periods names and their start/end for different languages. For example, for english language:

```
[
    ["night", "00:00/06:00"],
    ["morning", "06:00/12:00"],
    ["afternoon", "12:00/18:00"],
    ["evening", "18:00/21:00"],
    ["night", "21:00/24:00"]
]
```

Day periods do not overlap and fill all 24-hour period. List of periods is sorted by period start. First period always starts at "00:00" and last one always end at "24:00" (thus name of first and last period sometimes can be same).

Content language header should be set for this method - text representation will be returned in specified language.

**Authorization:** none

+ Request

    + Headers
    
            Content-Language: en

+ Response 200 (application/json)

    + Body

            [
                ["night", "00:00/06:00"],
                ["morning", "06:00/12:00"],
                ["afternoon", "12:00/18:00"],
                ["evening", "18:00/21:00"],
                ["night", "21:00/24:00"]
            ]

# Data Structures

## `identifier` (string)

## `url` (string)

## `phone` (string)

## `country` (string)

## `currency` (string)

## `language` (string)

## `datetime` (string)

## `color` (string)

## `section` (string)

## `rangedate` (string)

## `image` (object)

## `address` (object)
- city (string) - city
- street (string) - street 
- building (number) - building number
- apartment (string) - apartment number
- postal_code (number) - postal code (ZIP)

## `item` (object) - information about one sale item in ticket to be purchased
- `SUM FIELDS:` (section)
- original_price (number) - original price for sale item according to location price (read only)
- price (number) - if receptionist can raise price for client, `price` can be bigger than `original_price`
- quantity (number) - item quantity (default 1), can be changed if `can_change_quantity=true` and new value greater than zero and not bigger then `max_quantity`
- can_change_quantity (boolean) - if `quantity` can be changed (read only)
- max_quantity (number) - if `quantity` can be changed, defines what max value can be set (read only)
- sum (number) - final sum for item which is calculated as `price`*`quantity` minus discount, if appliable (read only)
- `DISCOUNT FIELDS:` (section)
- card_discount (identifier) - identifier of client card, if discount is appliable
- one_time_discount (one_time_discount) - information about one-time discount, as input parameter can be just number with discount sum
- promotion (identifier) - promotion id (if match promotion with condition, will be set automatically, otherwise id of any promotion can be provided)
- free_of_charge (boolean) - this sale item will be sold free of charge (discount 100%)
- `DESCRIPTIVE FIELDS:` (section)
- name (string) - sale item name, callculated automatically depending on sale item type (read only)
- comments (string) - additional comments about sale item
- clients_module (boolean) - if client booked via clients module/app by himself (true) or was booked by receptionist (false)
- receptionist (identifier) - id of receptionist, who processing the ticket
- appointment_receptionist (identifier) - id of receptionist, who booked client
- appointment_create_date (datetime) - booking date/time
- `SERVICE FIELDS:` (section)
- service (identifier) - service to purchase (required field for purchasing service)
- professinal (identifier) - professinal which will provide service (can be null if service can be provided without professional)
- assistant (identifier) - if service can be provided with assistant; otherwise field ignored
- recommended_professional (identifier) - in case one professional recommends visit another professional; this fields represents who recommended, not to whom
- calendar_date (datetime) - date/time when service will be provided
- duration (number) - time in minutes will be spent to provide service
- hall (identifier) - hall where service will be provided
- materials (material) - materials will be used; if not provided, default materials will be used
- appointment (identifier) - appointment id [Appointments](#appointments)
- by_certificate (identifier) - certificate id, if service will be paid by certificate
- `PRODUCT FIELDS:` (section)
- product (identifier) - product to purchase (required field for purchasing product)
- storage (identifier) - storage where product will be taken (required field for purchasing product)
- quantity_type (quantitytype) - in which units `quantity` field is calculated (please mention product should have possibility to be sold in such units)
- `GROUP FIELDS:` (section)
- lesson (identifier) - lesson client is going to (required field for purchasing group) 
- trial (boolean) - if client visit to lesson is trial one (possibility depends on program options but can be done only once)
- `CARD FIELDS:` (section)
- card (identifier) - card to purchase (required field for purchasing card)
- activation (datetime) - date of card activation (card period start) or null if card should be not activated
- `delayed_payments` (array[delayed_payment]) - if not full card sum should be payed at once, contains list of dates and sums to pay; sum of all payments should match `sum` field; otherwise null
- professional (identifier) - if card requires selecting professional on card sale, one should be provided
- group (identifier) - if card requires selecting group on card sale, one should be provided
- card_number (number) - if card stock activated, for each card type there is a list of available cards and on purchase operation card number that will be sold should be provided; this number should be available and not sold yet
- `CERTIFICATE FIELDS:` (section)
- certificate (identifier) - certificate to purchase (required field for purchasing certificate)
- `TIPS FIELDS:` (section)
- tips (number) - tips to leave for professional (required field for tips sale item type)
- professional - professional who will receive tips (required field for tips sale item type)
- `ROUNDING FIELDS:` (section)
- rounding (number) - ticket rounding sum, calculated automatically if rounding is on, provided value ignored (required field for rounding sale item type)
- `SALE FIELDS:` (section)
- id (identifier) - id of created sale on succesfull sale operation (read only)

## `delayed_payment` (object) - information about card delayed payment
- date: `2017-01-01T00:00:00.000Z` (datetime) - date of delayed payment
- sum: 1000.00 (number) - payment sum

## `quantitytype` (enum) - how product is sold (what means `quantity` field)
- package - `quantity` means number of packages (default value)
- portion - `quantity` means number of portions
- units - `quantity` means number of units (ml, g, etc.)

## `one_time_discount` (object) - information about one time discount
- sum (number) - discount sum
- max_percent (number) - discount max percent
- reason (string) - discount reason

## `material` (object)
- product (identifier) - product id
- storage (identifier) - id of storage where product should be taken
- quantity (number) - product quantity

## `payment` (object) - information about payment type and sum; only one of fields `account`, `deposit`, `bonus`, `company`, `certificate` should be declared, if `account`, `company` or `certificate` field is set, field `sum` represents amount to be payed
- account (identifier) - client will pay money to specified account (bank, cashdesk etc.), field contains account id, amount should be in `sum` field
- deposit (number)  - client will withdraw money from his deposit, field contains money to be withdrawn
- bonus (number) - client will use accumulated bonus money, field contains money to be used
- company (identifier) - client will pay money from company account (company deposit), field contains company id, amount should be in `sum` field
- certificate (identifier) - client will use certificate, field contains certificate id, amount should be in `sum` field
- sum (number) - sum to be paid (for `account`, `company` or `certificate` types)

## `text_in_different_languages` (object)
- en (string) - text in English
- fr (string) - text in French

## `geoposition` (object)
- latitude (string) - latitude of specific position
- longitude (string) - longitude of specific position

## `filter` (object)
- all (boolean, optional) - if filter applied for all items (default is `true`)
- *87d63709-c3dd-4469-991a-0e8d7994e07e (identifier)* (boolean) - if filter applied for item or category, represented by item or category id

## `filter_info` (enum)
- none (string) - filter applied for all items
- all (string) - filter applied for none items
- (filter) - filter applied to part of items, described in this field

## `filtered_list` (enum)
- false (string) - filter applied for none items
- true (string) - filter applied for all items
- `4da9ccd9-5a6b-49b5-97c6-ddad1ba03d57`, `894de0a7-dff1-43c9-b2d0-cd4590fd69dc` (array[identifier]) - filter applied to specific enumerated items

## `number_or_unlimited` (enum)
- unlimited (string) - represents unlimited number
- *100* (number) - specific limited number

## `discount_quantity` (enum)
- unlimited (string) - number of discounted visits is unlimited
- visits (string) - number of discounted visits is limited by client card `visits` field
- *100* (number) - number of discounted visits left

## `discount_details` (object)
- total_quantity: `visits` (discount_quantity) - total number of discounted visits
- left_quantity: `visits` (discount_quantity) - number of discounted visits left
- discount: 100 (number) - discount percent, from 0 to 100 (100 means totally free)

## `discount_details_info` (enum)
- (discount_details) - discount details, long form
- *100* (number) - discount percent, from 0 to 100 (100 means totally free), short form, in case quantity is unlimited

## `discount_details_list` (object)
- all (discount_details_info, optional) - discount details for all items of specified type
- *87d63709-c3dd-4469-991a-0e8d7994e07e (identifier)* (discount_details_info) - discount details for specific item or category, represented by item or category id

## `card_discounts_info` (object)
- services (discount_details_list) - discounts for services proposed by card
- products (discount_details_list) - discounts for products proposed by card
- groups (discount_details_list) - discounts for groups proposed by card
- cards (discount_details_list) - discounts for other cards proposed by card

## `bonus_details` (object)
- percent: 5 (number) - bonus percent, from 0 to 100
- filter (filter) - filter on items which are applied for bonus

## `bonus_details_info` (enum)
- (bonus_details) - bonus details, long form
- *5* (number) - bonus percent, from 0 to 100, short form, in case all items are applied for bonus

## `card_bonus_info` (object)
- services (bonus_details_info) - bonus for services proposed by card
- products (bonus_details_info) - bonus for products proposed by card
- groups (bonus_details_info) - bonus for groups proposed by card
- cards (bonus_details_info) - bonus for other cards proposed by card
- certificates (bonus_details_info) - bonus for other cards proposed by card

## `accumulated_level` (object)
- sum: 1000.00 (number) - level end sum (if no `sum` field - no next level)
- percent: 5 (number) - percent at current level, from 0 to 100

## `accumulate_items` (object)
- services (filter_info) - filter applied for services
- products (filter_info) - filter applied for products
- groups (filter_info) - filter applied for groups
- cards (filter_info) - filter applied for cards
- certificates (filter_info) - filter applied for certificates

## `locationprice` (object)
- location: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier) - location id
- position: `dd9114c8-439c-40a9-a784-04885f7fedab` (identifier) - position id
- price: 100 (number) - price on this location 
- staff_price: 100 (number) - staff price

## `accumulate_schema` (object)
- schema:   (array[accumulated_level]) - list of accumulation levels
- accumulate_from:   (accumulate_items) - items from which sum is accumulated
- accumulate_from_detailed (string) - localized description of filter accumulate_from
- apply_to:   (accumulate_items) - items to which discount/bonus can be applied
- apply_to_detailed (string) - localized description of filter apply_to

## `accumulated_percent` (object)
- accumulated_sum: 1567.20 (number) - sum already accumulated
- percent: 4 (number) - current discount/bonus percent (according to `accumulated_sum`)
- next_level: 2000.00 (number) - accumulated sum for reaching next level
- next_percent: 5 (number) - discount/bonus percent on next level

## `user_contact_type` (enum)
- phone (string) - phone number contact
- email (string) - e-mail contact

## `location_info` (object)
- id: `96e7b6c7-1813-4719-8676-544b1488637e` (identifier) - location id
- name: `ABC Salon NY Brooklyn` (string) - location name
- city: `New York` (string) - location city
- address: `2818 Foster Ave` (string) - location address

## `database_info` (object)
- id: 123456 (number) - database code
- name: `ABC Salon NY` (string) - database name
- locations (array[location_info]) - list of locations in database

## `user_contact` (object)
- type: `phone` (user_contact_type) - contact type
- contact: `18005550123` (string) - contact number/e-mail
- verified: true (boolean) - if contact is verified

## `schedule` (object) - information about time ranges (from, to) grouped by weekday