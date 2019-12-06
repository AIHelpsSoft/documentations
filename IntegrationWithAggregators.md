# Integration with beauty/dental services aggregators (Tutorial)

TODO: check `Integration via API` after finish with Grammarly
TODO: set http formatting

There are some requests to AI Helps for integrating Beauty Pro/Denta Pro products with services aggregators. We made this tutorial to simplify the connecting process and collect all corresponding information in one place.

If you have any questions, feel free to contact the group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.

# Show Online module on your site

The easiest way to allow the booking to Beauty Pro/Denta Pro from your site: just show our online module on your site page.

You can see example module: https://beautyprosoftware.com/b/503953 (**TODO: get demo database from sales**)

Pros:
+ takes 5 minutes to implement
+ no setup work with AI Helps needed, you can do it by yourselves
+ no additional agreements needed from salon/clinic

Cons:
- client select service, professional, free time inside the module, you can't make common services list for all salons/clinics in this way
- also, you do not influence the client booking process and even don't know if the client successfully booked
- the module has own design, different from your site and has limited design customization possibilities, it will anyway be "alien" element on your site

So, this method is recommended for salon/clinic catalogs, which don't have a common services list, do not check free time to propose only locations with time available and have no or small integration with salons/clinics.
For aggregators, which have its own services list and propose a salon/clinic based on service needed and free time available and wants to control all booking process, we propose a method, described in [Integration via API](#API).

## Implementation

To show the online module on your page, just insert the next script in your page:
```html
    <script type="module">
      import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";
      init({
        database: 503953,       // client id, you can get it from client or client online module
        elementId: bookButton   // html element id, click on which should show online module
      });
    </script>
```

Client id can be received from the client or can be taken from the client module, running on the client's site, Facebook or Instagram page.

There are many options to interact with the module: open/close it manually (do not attach to some element) or track open/close events, control module position and color theme, specify certain services/professionals to book. You can see detailed information about AI Helps init script setup here: https://github.com/AIHelpsSoft/documentations/blob/master/ModuleInitialScript.md

# Integration via API <a name="API"></a>

This method is much harder and can take days to implement but gives you all data needed for booking and provides full control over all booking process.

You communicate with AI Helps API (full API documentation is available at https://api.aihelps.com).

First, you register your application in AI Helps application registry, providing information information about your company and contacts and receive application ID & secret needed for next steps. Registration is one simple API call and response is automatical (no verification process). This step should be made only once.

Next step: for each client database (either salon or clinic) you want to get access, you need to receive access token. Access grant request should be sent and client either grants access or refuse from Beauty Pro/Denta Pro application. If granted, you will receive access token for access to specific client daabase. This step should be done for each your client.

After you have access token, you can make API requests to get list of services, professionals, free time and make booking. Possible scenarios will be described later in details.

## Register your application

First you need to do is to register your application. To do this, make following request:

```http
POST https://api.aihelps.com/v1/applications?fields=application_id,application_secret
```

```json
{
  "application_name": "Best Salons",
  "application_url": "bestsalons.com",
  "developer_name": "Best Salons Solutions",
  "contact_email": "dev@bestsalons.com",
  "scope": "clients_module",
  "grant_access_url": "bestsalons.com/access/{database_code}"
}
```

Most fields are self-explanatory.
Application name and URL, developer name will be shown to salon/clinic when access will be asked. Contact email will not be shown to clients and needed to communicate with application owner/developer.

Scope defines rights application will have in clients database. `clients_module` was made to fit all needs for online booking, connecting aggregator and similar tasks (get services, professionals, free time and client login - used to make an appointment). Other available scopes: `reports` - read only access for all clients data and `full` - read/change all clients database gives much more information than needed and probably most clients will refuse to give you access to all their data.

`grant_access_url` - URL on your site which will receive notifications about clients granted access to their data. Will described in next chapter.

Server response would be following:
```
{
    "id": "9245f5b4-d66e-483c-b7c4-fd55f9677bb3"
    "application_id": "5330336c-143a-415a-b220-fc9932bc029d",
    "application_secret": "eb5cb7df-356e-475f-8afb-2eebe63a37bb"
}
```

Save `application_id` & `application_id` in secure place - they will be needed to request access token for client database.

More details about application registration can be found at: https://aihelps.docs.apiary.io/#reference/applications/application/register-new-application-account

## Request access token for client database

On this step we assume you already has `application_id` & `application_id` (if not, see previous chapter) and now want to receive access to some clients database. This step should be repeated for each salon/clinic you want to receive access.

To make access request to some clients database, you need to know AI Helps client code - 6-digit number, representing salon/clinic. This number can be found:
* in Beauty Pro/Denta Pro software as Settings -> Location settings -> Location code 
* on salon/clinic site, Facebook or Instagram page, where AI Helps online module is installed - in page source in module script client code is used
* in AI Helps support - we can provide client code by salon/clinic name and location, phone, site or some requisites (requisites to uniquely identify location between others)

When you know client code, access request can be made:

```
GET https://api.aihelps.com/v1/auth/database?application_id=a47cbc05-5ce6-4551-9456-2855b52a17f0&application_secret=a3db644a-907a-4da2-bcdd-bc9f7068237d&database_code=123456
```

Normally you should receive answer:

```
{
  "status": "pending"
}
```

meaning that we are waiting confirmation from someone of salon/clinic persons, who have `owner` (full) access to Beauty Pro/Denta Pro.

Such owner needs to login into desktop or application, go to Settings->Marketplace, select your application and press Grant access. After person grants access, you will receive notification on `grant_access_url` url (GET request without body, access token not sent straight to this url for security reasons), described earlier in application settings. When you receive such notification, repeat grant access request (all parameters same):

```
GET https://api.aihelps.com/v1/auth/database?application_id=a47cbc05-5ce6-4551-9456-2855b52a17f0&application_secret=a3db644a-907a-4da2-bcdd-bc9f7068237d&database_code=123456
```

You will receive access token:

{
  "server": 1,
  "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
  "database": "123456",
  "scope": [
    "clients_module"
  ],
  "expires_at": "2039-06-01T00:00:00.000Z"
}

Access token, that will be needed for next requests is `access_token` (each clients database will need its own access token). Save it in secure place.
You will receive token for 20 years so there is no need to worry about token expiration.

Next important parameter is server - it contains server number where client database is stored. Servers are distributed around the globe, you can make requests to any server (and they will respond correctly), but requesting database through different server will take much more time. So, accessing clients database, make requests to next API server:
* https://api.aihelps.com if `server` is 1
* https://api{server}.aihelps.com (for example https://api3.aihelps.com) if `server` is greater than 1

## Request access token for client database after application published

If all things going OK, you already connected at least 10 clients and keep going, we can make your application public. We check your application details once more (application details can't be changed for published application), select countries for which application will be published (or worldwide) and publish it. After that anyone using application can see your application in the Marketplace and grant access without your access request. You receive notification and can make request to get access token.

In this case no need to search for client code - you just ask client to grant access in Marketplace and receive access token.
If you receive many notifications with different client codes at once and get lost which code belongs to which client, you can get location name (or names of all locations in chain for networks) making request to locations endpoint:

```
GET https://api.aihelps.com/v1/locations?fields=name

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive smth. like:

```
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

## Get services list

First you need to receive is list of services provided by salon/clinic.

Make request to `/services` endpoint, providing access token in Authorization header for database you want to get data:

```
GET https://api.aihelps.com/v1/services/tree?fields=name,description,duration,gender,location_prices&categories_fields=name&public=true&has_professional_price=true&archive=false

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

In this example we requested non-archive (archive services can't be sold any more) public services (allowed to booked by clients themself, usually all services) that have price for at least one professional (and skipping services like tanning, where no professional needed).

```
{
    "id": null,
    "name": "",
    "categories":
    [
        {
            "id": "c9edc51c-a95a-4f9c-84fd-67022b957d79",
            "name": "Haircuts",
            "categories":
            [
            ],
            "items":
            [
                {
                    "id": "88d60be6-fba7-49c0-3d28-066a5cb5f1b9",
                    "name": "Complex",
                    "description": "Complex service",
                    "duration": 60,
                    "gender": "both",
                    "location_prices": 
                    [
                        {
                            "location": "00df7375-1e7b-44d8-be05-a7777e79b012",
                            "position": "d68cab7f-443f-454c-80f1-8ba8498cb0a4",
                            "price": 200,
                            "staff_price": 0
                        }
                    ]
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
                    "name": "Manicure",
                    "description": "",
                    "duration": 30,
                    "gender": "female",
                    "location_prices": 
                    [
                        {
                            "location": "00df7375-1e7b-44d8-be05-a7777e79b012",
                            "position": "d68cab7f-443f-454c-80f1-8ba8498cb0a4",
                            "price": 100,
                            "staff_price": 0
                        }
                    ]
                }
			]
        }                    
    ],
    "items":
    [
    ]
}
```

All services are grouped in categories. Categories are objects with next fields:
- `id` - category id (not needed in our scenario)
- `name` - category name
- `categories` - child categories for current category
- `items` - services located in this category
Categories and services are presented in hierarchical view. Root is base category with null `id` and without name.

Each service has next fields:
- `id` - service id (will be used for booking)
- `name` - services name
- `description` - optional service description (with basic HTML formatting)
- `duration` - service duration in minutes
- `gender` (`both`/`male`/`female`) - client gender for current service
- `location_prices` - prices for current service for different professionals and employees (service price can differ in different locations and for different professionals)
  + `location` - location, for which price is given, see section [Working with chains (networks)](#chains) for details
  + `position` - professionals position for which price is given (price sets not for specific professional but for professionals position instead)
  + `price` - service price
  + `staff_price` - service price for staff under some conditions, not needed in our scenario

There is no such thing as common catalog of services, each client in his database creates it's own services and services structure, you should connect them to some sort of your services catalog manually or with help of some sort of AI - if you need common list of services between all your salons/clinics.

## Get professionals list

Next thing you need is professionals list with their positions.

Get all possible professional positions:

```
GET https://api.aihelps.com/v1/positions?fields=name&role=professional

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

Result will be something like this (you need name for each position id received in services or professionals lists):

```
[
    {
        "id": "04c5daec-2c11-4ce9-b00d-03a03d50356f",
        "name": "Stylists"
    },
    {
        "id": "e28ebfd0-f847-4c3f-b2a1-07f4257b0764",
        "name": "Hairdressers"
    }
]
```

Get list of all non-archive (archive professionals can't provide any services) public professionals (allowed to booked by clients themself):

```
GET https://api.aihelps.com/v1/employees?fields=name,positions&role=professional&archive=false&public=true

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

Result will be something like this:

```
[
  {
    "id": "88d4d8c0-3830-5b07-7bfe-b80f0650b9f3",
    "name": "John Doe",
    "positions": [
      "04c5daec-2c11-4ce9-b00d-03a03d50356f"
    ]
  },
  {
    "id": "72fcfa1d-65a7-404c-8c41-7449265ae489",
    "name": "Kevin Deriman",
    "positions": [
      "04c5daec-2c11-4ce9-b00d-03a03d50356f",
      "e28ebfd0-f847-4c3f-b2a1-07f4257b0764"
    ]
  }
]
```

Please take into account that one professional can belong to several positions. So certain professional can provide certain service if they have at least one common position (service can has several positions, professional has several positions).

## Get professionals free time

On this step you already know services and professionals.

One more thing you need to know before booking client - time which is available for booking, equal to employee free time (API forbids booking one time block for one employee with two overlapping appointments).

So, this endpoint returns you free time for all professionals for certain time range:

```
GET https://api.aihelps.com/v1/employees/free_time?from=2019-01-01T10:00:00.000Z&to=2019-01-03T14:30:00.000Z&duration=120&step=30m

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

`duration` and `step` needs to be explained:
result will be returned as list of date/time for each day for each professional. Each date/time is free for booking for certain professional for at least `duration` minutes (pass total selected services duration in this fied to be sure endpoint returns date/time where enough time for all selected services). `duration` is any number greater than 0. `step` represent frequency of date/times available, in minutes, possible values: 10m, 15m, 20m, 30m, 60m, 90m, 120m, 150m, 180m, auto.

So, if `duration` = 90 and `step` = 30m and professional has free time from 10:00 till 13:00, endpoint will return: "10:00", "10:30", "11:00" and "11:30".

You will receive free time in next format:

```
{
  "88d4b305-e198-f02c-2743-cca9390c631c": {			// employee id
    "2019-01-01": [									// day
      "10:00",
      "10:30",
      "11:00"
    ],
    "2019-01-02": [
      "10:00",
      "10:30",
      "11:00"
    ]
  },
  "023276a7-7ca9-45c9-9f9c-b536ec20315b": {
    "2019-01-01": [
      "10:00",
      "10:30",
      "11:00"
    ],
    "2019-01-02": [
      "10:00",
      "10:30",
      "11:00"
    ]
  }
}
```

You also can optionally specify professionals for which you want to get free time (filter out all other professionals + can be quicker than free time for all professionals):

```
https://api.aihelps.com/v1/employees/free_time?from=2019-01-01T10:00:00.000Z&to=2019-01-03T14:30:00.000Z&duration=120&step=30m&professionals=88d4b305-e198-f02c-2743-cca9390c631c&location=a7dd57e5-c495-46d4-b54d-753061d355a0

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

## Working with chains (networks) <a name="chains"></a>

Several words should be said about databases where chain of several locations works. For example, beauty salon chain have 5 different locations with common clients list, but different employees and different prices for services in different locations (sometimes happens that some service has price and can be sold not in every location).

So in this case you should get all locations 

```
GET https://api.aihelps.com/v1/locations?fields=name

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive smth. like:

```
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

Client selects certain location and then you get service prices and employees free time just for that location.

## Timezones

While getting employee free time and creating appointment requires working with date/time, so question with time zones rises. For some cases it's obvious: you aggregate clients inside one country with one time zone so for all your salons/clinics you always know correct timezone and time is common for all your aggregator. In more complex cases you need to know timezone for each database and even for each location (yes, locations with different timezones can be stored in one database). Timezones are represented in TZData format (all browsers are using), so no issues with receiving current time zone or working with time zones should happen. For each location you can get actual timezone:

```
GET https://api.aihelps.com/v1/locations?fields=name,timezone

Authorization:Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive smth. like:

```
[
  {
    "id": "2a0516e3-ddc9-432c-80e4-8d5f5e97cce3",
    "name": "Salon 123 New York",
    "timezone": "America/New_York"
  },
  {
    "id": "3fe0e240-d779-4a84-9e0c-825d9a6ddcc4",
    "name": "Salon 123 Los Angeles",
    "timezone": "America/Los_Angeles"
  },
  {
    "id": "843b8d24-2564-475f-8451-6ef25f2ccd18",
    "name": "Salon 123 Miami",
    "timezone": "America/New_York"
  }
]
```

## Client login

Now you have all information: selected services, professional, appointment start time and duration. Now we can select client and save appointment.

There are two ways:
1) *not recommended* - if you have `full` scope, you can get all clients list search them by name, phone or other requisites, add new if needed. But salons/clinics do not eager to share their clients databases with aggregator services, thus chances you'll have `full` scope for your application is close to zero. So, second way...
2) with `client_login` scope, you can simulate client log in by phone: you provide client phone number, sms with one-time code is sent to this code and you have limited number of attempts to enter this code. After succesfull code enter you will receive client access token for existing client (if phone number is found) or new client will be created and his token will be returned. With that token you can change client details and manage his appointments. So, client should enter code for booking but all sides are confident about security.

So, let's describe second case. You have phone number and want to receive client access token.

Receiving client's token is 2-step process (or even 3-step for new clients). On first step client provides his phone number and you send him sms with random-generated code using this method (code is 4 digits, generated by server and not visible via API for security reasons):

```
GET https://api.aihelps.com/v1/auth/client?application_id=a47cbc05-5ce6-4551-9456-2855b52a17f0&database_code=123456&phone=18005551234
```

Client phone number should be in international format (digits only), do net set database access token in Authorization header for this request. Response:

```
{
  "auth_id": "91d14822-9150-4ba7-8adc-a20d9507a811"
}
```

If sms sent succesfully, method will return `auth_id`, that will be needed to finish receiving client's token. After client receives code in sms you can make next step:

```
GET https://api.aihelps.com/v1/auth/client?auth_id=91d14822-9150-4ba7-8adc-a20d9507a811&code=8664
```

Several restrictions exists on this step:
+ Time between steps 1 and 2 should be less than 2 hours (auth_id expires in two hours).
+ After 20 wrong codes, auth_id becomes invalid.

There are 3 possible outcomes here:
+ code is wrong - 401 error will be returned with description (please note, that number of tries is limited)
+ code is correct - client token (and refresh token) will be generated and returned (along with client id)
+ code is correct but phone not exists in database - error 404 returned

In last case you should ask client for more details - client firstname, lastname, e-mail, gender and location and pass them as parameters once again (properties can be empty but all should be passed) + pass `Content-Language` header:

```
GET https://api.aihelps.com/v1/auth/client?auth_id=91d14822-9150-4ba7-8adc-a20d9507a811&code=8664&firstname=John&lastname=Brown&sex=Male&email=john%40gmail.com&location=dbc97829-9392-451c-a00d-331d4f8c5d91

Content-Language: en
```

In both cases you will receive access token smth. like this:

```
{
    "server": 1,
    "access_token": "f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4",
    "expires_at": "2017-05-15T14:00:00.000Z",
    "refresh_token": "67d8f73c-52f8-45ad-a242-f3cb72ff79c2",
    "client_id": "d81dc7a7-879b-47a6-b98f-033558d5b074",
}
```

This access token has same principles as database access token described earlier but two moments:
1) `client_id` represents client id for existing or newly created client in database
2) token will expire in 24 hours. Usually it's enough. But if you want to provide smooth user experience, you can store `refresh_token` and refresh it after expiration. Process is simple enough, details can be found at https://aihelps.docs.apiary.io/#reference/authorization/token-refresh/refresing-expired-token

Changing client requisites (name, phone, e-mail, etc.) is outside scope of this tutorial, if needed, check https://aihelps.docs.apiary.io/#reference/clients/client/update-client

## Book an appointment

So, you have all appointment details and client access token, you can make an appointment:

```
POST https://api.aihelps.com/v1/appointments

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4

{
  "state": "planned",
  "date": "2018-01-01T10:00:00.000Z",
  "duration": 90,
  "location": "8fe29e48-d5a5-4dcc-800e-80eb6e9cac5c",
  "professional": "2783c237-214d-4f13-bae2-ac4eee79ba03",
  "services":
  [
	"4b244443-90d3-4182-918a-fe3ac4a03688"
  ],
  "clients_module": true,
  "comments": ""
}
```

Client field can be omitted, client automatically will be taken from client access token `client_id` field. `"clients_module": true` describes that appointment was made by client itself, not by receiptionist or other staff. `comments` can be any additional text.

While creating appointment you should check for HTTP Status code 409 (Conflict) - this means that professional is busy at specified time (even for one second) - either overlapse with other appointments or appointment is outside employee work time. It can shows either problems in your code or sometimes other client book that time just between you select free time and made an appointment. No problems - show "Excuse me" dialog and ask client to select new appointment time.

You will receive appointment id:

```
{
  "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
}
```

According to program settings, client will receive smses about succesfull booking and later notification several hours before visit start.

If you need to change some details, call request with fields that should be changed:

```
PUT https://api.aihelps.com/v1/appointments/3ab52942-da15-4a20-b14c-b8fca7e4bf1a

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4

{
  "duration": 120,
  "services":
  [
	"4b244443-90d3-4182-918a-fe3ac4a03688",
	"daf6c0f0-6903-4c9e-a0e2-4381676c364d"
  ]
}
```

If you need to cancel appointment:

```
PUT https://api.aihelps.com/v1/appointments/3ab52942-da15-4a20-b14c-b8fca7e4bf1a

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4

{
  "state": "cancelled",
  "cancel_date": "2018-01-01T10:00:00.000Z",
  "cancel_reason": "Client cancelled visit"
}
```

You can delete appointment (but better to set appointment as cancelled so client history can be checked):

```
DELETE https://api.aihelps.com/v1/appointments/3ab52942-da15-4a20-b14c-b8fca7e4bf1a

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```