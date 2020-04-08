# Integration with online stores (Tutorial)

There are some requests to AI Helps for integrating Beauty Pro/Denta Pro products with online stores. We made this tutorial to simplify the connecting process and collect all corresponding information in one place.

If you have any questions, feel free to contact the group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.

# Overview

All work integrating your online store with Beauty Pro/Denta Pro is done via AI Helps API (full API documentation is available at https://api.aihelps.com).

First, you register new application in AI Helps application registry, providing short information about your company, contacts and receive application ID & secret needed for the next steps. Registration is one simple API call and response is automatical (no verification process).

Next step: for client database (either salon or clinic) you want to get access, you need to receive an access token. Access grant request should be sent and the client either grants access or refuse inside Beauty Pro/Denta Pro application. If granted, you will receive access token for access to a specific client database. This step should be done for each of your beauty salons/dental clinics.

After you have an access token, you can make API requests to get a list of products and stocks and make a purchase. Possible scenarios will be described later in detail.

## Register your application

First, you need to do is to register your application. To do this, make the following request:

```http
POST https://api.aihelps.com/v1/applications?fields=application_id,application_secret
```

```json
{
  "application_name": "Beauty Pro Shopify Connector",
  "application_url": "beautypro-shopify.com",
  "developer_name": "IT Guys",
  "contact_email": "shopify@it-guys.com",
  "scope":
  [
    "online_store"
  ],
  "grant_access_url": "beautypro-shopify.com/access/{database_code}"
}
```

Most fields are self-explanatory.
Application name and URL, the developer name will be shown to the salon/clinic when access will be asked. A contact email will not be shown to clients and needed to communicate with the application owner/developer.

The scope defines rights that application will have in client's database. `online_store` was made to fit all needs for online stores, providing products and stocks, making a purchase and similar tasks. This scope is needed for our scenario. Other available scopes: `reports` - read-only access for all client's data and `full` - read/change all clients database; gives much more information than needed and probably most clients will refuse to give you access to all their data.

`grant_access_url` - URL on your site which will receive notifications about clients granted access to their data. Will be described in the next chapter.

Server response would be the following:
```json
{
    "id": "9245f5b4-d66e-483c-b7c4-fd55f9677bb3",
    "application_id": "5330336c-143a-415a-b220-fc9932bc029d",
    "application_secret": "eb5cb7df-356e-475f-8afb-2eebe63a37bb"
}
```

Save `application_id` & `application_secret` in a secure place - they will be needed to request an access token for the client database.

More details about application registration can be found at https://aihelps.docs.apiary.io/#reference/applications/application/register-new-application-account

## Request access token for client database

On this step, we assume you already have `application_id` & `application_secret` (if not, see the previous chapter) and now want to receive access to some client database. This step should be repeated for each salon/clinic you want to receive access.

To make an access request to some clients database, you need to know AI Helps client code - 6-digit number, representing salon/clinic.
Ask the salon/clinic for client id. It can be found in Beauty Pro/Denta Pro on Settings page => Location settings => Location code (6 digits, big letters).
If needed, AI Helps technical support can help the client to get the client id.

Pay attention: client code is not so private information indeed, but the client should confirm access by action. This is crucial for clients as they stay confident about access to their data.

When you know client code, access request can be made:

```http
GET https://api.aihelps.com/v1/auth/database?application_id=a47cbc05-5ce6-4551-9456-2855b52a17f0&application_secret=a3db644a-907a-4da2-bcdd-bc9f7068237d&database_code=123456
```

Normally you should receive an answer:

```json
{
  "status": "pending"
}
```

meaning that we are waiting for confirmation from someone of salon/clinic persons, who have `owner` (full) access to Beauty Pro/Denta Pro.

Such owner needs to login to a desktop application, go to Settings page => General Settings => Marketplace, select your application and press Grant access. After person grants access, you will receive a notification on `grant_access_url` URL (GET request without body, the access token is not sent straight to this URL for security reasons), described earlier in application settings. When you receive such notification, repeat grant access request (all parameters same):

```http
GET https://api.aihelps.com/v1/auth/database?application_id=a47cbc05-5ce6-4551-9456-2855b52a17f0&application_secret=a3db644a-907a-4da2-bcdd-bc9f7068237d&database_code=123456
```

You will receive access token:

{
  "server": 1,
  "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
  "database": "123456",
  "scope":
  [
    "online_store"
  ],
  "expires_at": "2039-06-01T00:00:00.000Z"
}

Access token, that will be needed for next requests is `access_token` (each client database will need its access token). Save it in a secure place.
You will receive a token for 20 years, so there is no need to worry about token expiration.

The next important parameter is server - it contains a server number where the client database is stored. Servers are distributed around the globe, you can make requests to any server (and they will respond correctly), but requesting database through the different server will take much more time. So, accessing the client's database, make requests to the next API server:
* https://api.aihelps.com if `server` is 1
* https://api{server}.aihelps.com (for example https://api3.aihelps.com ) if `server` is greater than 1

## Request access token for client database after application published

If all things going OK, you already connected at least 10 clients and keep going, we can make your application public. We check your application details once more (application details can't be changed for published application), select countries for which application will be published (or worldwide) and publish it. After that, anyone using the application can see your application in the Marketplace and grant access without your access request. You receive notification and can request to get access token.

In this case no need to search for client code - you just ask the client to enter Marketplace and grant access to your application - and access token will be yours.
If you receive many notifications with different client codes at once and get lost which code belongs to which client, you can get location name (or names of all locations in the chain for networks) request locations endpoint:

```http
GET https://api.aihelps.com/v1/locations?fields=name

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive something like:

```json
[
  {
    "id": "2a0516e3-ddc9-432c-80e4-8d5f5e97cce3",
    "name": "Salon 123 New York"
  },
  {
    "id": "3fe0e240-d779-4a84-9e0c-825d9a6ddcc4",
    "name": "Salon 123 Los Angelos"
  },
  {
    "id": "843b8d24-2564-475f-8451-6ef25f2ccd18",
    "name": "Salon 123 Miami"
  }
]
```

## Get products list

First, you need to receive a list of products, provided by the salon/clinic.

Request `/v1/products` endpoint, providing access token in Authorization header for the database you want to get data:

```http
GET https://api.aihelps.com/v1/products/tree?fields=name,description,picture,location_prices(location,price),stocks(storage,quantity),package_price_currency,can_sale_package&categories_fields=name,picture&archive=false

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

In this example, we requested non-archive (archive products can't be sold any more) products.

```json
{
    "id": null,
    "name": "",
    "categories":
    [
        {
            "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
            "name": "Shampoos",
            "categories":
            [
            ],
            "items":
            [
                {
                    "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                    "name": "Shampoo X",
                    "description": "Best for all hair types",
                    "location_prices": [
                      {
                        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                        "price": 35
                      }
                    ],
                    "stocks": [
                        {
                            "storage": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                            "quantity": 12,
                        }
                    ],
                    "package_price_currency": "USD",
                    "can_sale_package": true
                }
            ]
        },
        {
            "id": "ddc757ab-3656-489b-b414-c8f872c967fe",
            "name": "Manicure",
            "categories":
            [
            ],
            "items":
            [
                {
                    "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                    "name": "Gel X",
                    "description": "",
                    "location_prices": [
                      {
                        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
                        "price": 50
                      }
                    ],
                    "stocks": [
                        {
                            "storage": "88d419f7-17f3-d48b-00ca-aea77f0775df",
                            "quantity": 4,
                        }
                    ],
                    "package_price_currency": "USD",
                    "can_sale_package": true
                }
            ]
        }                    
    ],
    "items":
    [
    ]
}
```

All products are grouped into categories. Categories are objects with next fields:
- `id` - category id
- `name` - category name
- `categories` - child categories for current category
- `items` - products located in this category

Categories and products are presented in a hierarchical view. The root is a base category with null `id` and without a name.

Each product has the next fields:
- `id` - product id (will be used for purchase)
- `name` - product name
- `description` - optional product description (with basic HTML formatting)
- `location_prices` - prices for the current product for different locations (product price can differ in different locations in one chain)
  + `location` - location, for which price is given, see section [Working with chains (networks)](#chains) for details
  + `price` - product price. If you sure there is only one location, you can take `price` from the first `location_prices` item
- `stocks` - products quantity left in different storages at the current moment (products are stored in storages, each location has one or several storages)
  + `storage` - storage id, see section [Working with storages](#storages) for details
  + `price` - products left quantity
- `package_price_currency` - currency for which price is shown for the current product. Usually the same as default country currency, but in some cases can differ and price in local currency can get multiplying price by corresponding currency rate
- `can_sale_package` - if the current product can be sold. Just skip products that can't be sold

## Working with storages <a name="storages"></a>

When purchasing products, you need to specify storage from which product will be taken. You can get a list of all storages using endpoint `/v1/storages/`:

```http
GET https://api.aihelps.com/v1/storages?fields=name,location,can_sale

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

The result will be something like this:

```json
[
    {
        "id": "04c5daec-2c11-4ce9-b00d-03a03d50356f",
        "name": "Stylists storage",
        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
        "can_sale": false
    },
    {
        "id": "e28ebfd0-f847-4c3f-b2a1-07f4257b0764",
        "name": "Showcase",
        "location": "88d60c1b-5ddd-a723-0312-ba0507cf5e92",
        "can_sale": true
    }
]
```

The crucial moment here - you only can sell products from storage where `can_sale` = true.

## Working with chains (networks) <a name="chains"></a>

Several words should be said about databases where a chain of several locations works. For example, the beauty salon chain has 5 different locations with common clients list, but different employees and different prices for products in different locations (sometimes happens that some product has a price and can be sold not in every location).

So, in this case, you should get all the locations:

```http
GET https://api.aihelps.com/v1/locations?fields=name

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive something like:

```json
[
  {
    "id": "2a0516e3-ddc9-432c-80e4-8d5f5e97cce3",
    "name": "Salon 123 New York"
  },
  {
    "id": "3fe0e240-d779-4a84-9e0c-825d9a6ddcc4",
    "name": "Salon 123 Los Angelos"
  },
  {
    "id": "843b8d24-2564-475f-8451-6ef25f2ccd18",
    "name": "Salon 123 Miami"
  }
]
```

The client selects a certain location (or you specify exact location in code) and then you use product prices just for that location.

## Working with clients <a name="clients"></a>

Before making a purchase you must decide, if the purchase will be connected to some client or no client will be attached to purchase.

If you want to attach a purchase to the client, you need to find the corresponding client in the database by phone, e-mail, card number or other requisites.

To get all clients, call `/v1/clients/`:

```http
GET https://api.aihelps.com/v1/clients?fields=name,firstname,middlename,lastname,gender,birthday,card_number,phone,email,comment

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive a full list of clients:

```json
[
    {
        "id": "88d6248d-e5e8-9c40-5296-bb10043eab27",
        "name": "John Brown",
        "firstname": "John",
        "middlename": "",
        "lastname": "Brown",
        "gender": "male",
        "birthday": null,
        "card_number": [
            "127219432"
        ],
        "phone": [
            "+1 800 555 12 34"
        ],
        "email": [
            "john@gmail.com"
        ],
        "comment": "VIP status"
    },
    {
        "id": "6b663d27-90f5-4df0-a8c7-5924b9cd1c39",
        "name": "James Doe",
        "firstname": "James",
        "middlename": "",
        "lastname": "Doe",
        "gender": "male",
        "birthday": null,
        "card_number": [],
        "phone": [
            "+1 800 555 48 15"
        ],
        "email": [],
        "comment": null
    }
]
```

You also can search clients by phone, e-mail or card number:

`/v1/clients?fields=name,firstname,lastname?phone=18005550123`
`/v1/clients?fields=name,firstname,lastname?email=a@gmail.com`
`/v1/clients?fields=name,firstname,lastname?card_number=1234567890`

If the client not found, you can create a new client:

```http
POST https://api.aihelps.com/v1/clients

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```
```json
{
    "firstname": "Bob",
    "lastname": "Black",
    "gender": "male",
    "phone": [
        "+1 800 555 12 00"
    ]
}
```

Id of the new client will be returned:

```json
{
    "id": "f90a87cb-ef7f-444f-a05f-bf81ec055627"
}
```

## Purchasing a product

So, now you have all the details and can make a purchase:

```http
POST https://api.aihelps.com/v1/sales/purchase

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "location": "d114d9c8-40ac-9439-a784-0fedab4885f7",
  "client": "dd9114c8-439c-40a9-a784-04885f7fedab",
  "date": "2018-08-09T00:00:00.000Z",
  "items": [
    {
      "product": "ac9114c8-439c-40a9-a784-048852s4eea1",
      "storage": "d377d0dc-5b6c-4595-b8d4-5761048baae3",
      "quantity": 1
    }
  ],
  "payments": [
    {
      "deposit": 200
    },
    {
      "account": "24cd2118-439c-40aa-a784-048cacd85275",
      "sum": 320
    }
  ]
}
```

- `location` field is required, describes the location where the purchase is made.
- `client` (optional) - client id
- `date` - purchase date. Now if omitted
- `items` - one or several products purchased
    + `product` (required) - product id
    + `storage` (required) - storage where product will be taken from
    + `quantity` (optional, default 1) - product quantity purchased
- `payments` - one or several payment options:
    + `deposit` - sum taken from deposit
    + `account` - account id where money will be stored. Can be bank account, cashdesk etc. See section [Accounts](#accounts) for details
    + `sum` - sum put on the account
    
In the example above client purchase one product for 520, paid with deposit and card payment.

## Accounts <a name="accounts"></a>

Each location has several accounts:

```http
GET https://api.aihelps.com/v1/accounts?fields=name,location,type

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```
```json
[
  {
    "id": "88d4d8bf-a9f1-8063-7bfe-b80f2d3d9b58",
    "name": "Cash",
    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
    "type": "cashdesk"
  },
  {
    "id": "88d4d8bf-a9f4-8da3-7bfe-b80f437c45a6",
    "name": "Account in JPMorgan",
    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
    "type": "bank"
  },
  {
    "id": "88d4d8bf-a9f4-8da3-7bfe-b80f437c45a6",
    "name": "Account in Citygroup",
    "location": "88d4542a-bb12-03b1-3c0b-41fa26f1aaf1",
    "type": "bank"
  }
]
```

Only accounts with types `bank` and `cashdesk` are allowed for purchase operations (there are several other types).

## The End

That's all!

If you have some questions, feel free to contact the group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.