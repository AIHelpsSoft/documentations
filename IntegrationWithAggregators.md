# Integration with beauty/dental services aggregators (Tutorial)

TODO: check all document after finish with Grammarly

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

```
POST https://api.aihelps.com/v1/applications?fields=application_id,application_secret

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

## Get professionals list

## Get professionals free time

## Working with chains (networks)


## Client login

## Book an appointment

// separately say about possibility to make reserve while bboking process
// different errors should be explained here