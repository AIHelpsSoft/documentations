# Integration with Beauty/Dental Services Aggregators (Tutorial)

There are some requests to AI Helps for integrating Beauty Pro/Denta Pro products with services aggregators. We made this tutorial to simplify the connecting process and collect all corresponding information in one place.

If you have any questions, feel free to contact the group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.

# Show Online Module on Your Site

The easiest way to allow the booking to Beauty Pro/Denta Pro from your site: just show our online module on your site page.

You can see the module https://beautyprosoftware.com/b/295974 as an example.

Pros:
+ takes 5 minutes to implement
+ no setup work with AI Helps needed, you can do it by yourselves
+ minimal efforts needed from salon/clinic

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
        elementId: "bookButton"   // html element id, click on which should show online module
      });
    </script>
```

Ask the salon/clinic for client id. It can be found in Beauty Pro/Denta Pro on Settings page => Location settings => Location code (6 digits, big letters).
If needed, AI Helps technical support can help the client to get the client id.
Try to avoid situations when you found client id in some way and connect his online module on your aggregator without his agreement - clients don't like someone running booking to their salons/clinics on external resources without their confirmation.

There are many options to interact with the module: open/close it manually (do not attach to some element) or track open/close events, control module position and color theme, specify certain services/professionals to book. You can see detailed information about AI Helps init script setup here: https://github.com/AIHelpsSoft/documentations/blob/master/ModuleInitialScript.md

# Integration via API <a name="API"></a>

This method is much harder and can take days to implement but gives you all data needed for booking and provides full control over all booking process.

You communicate with AI Helps API (full API documentation is available at https://api.aihelps.com).

First, you register your application in AI Helps application registry, providing information about your company, contacts and receive application ID & secret needed for the next steps. Registration is one simple API call and response is automatical (no verification process). This step should be made only once.

Next step: for each client database (either salon or clinic) you want to get access, you need to receive an access token. Access grant request should be sent and the client either grants access or refuse inside Beauty Pro/Denta Pro application. If granted, you will receive access token for access to a specific client database. This step should be done for each of your clients.

After you have an access token, you can make API requests to get a list of services, professionals, free time and make a booking. Possible scenarios will be described later in detail.

## Register Your Application

First, you need to do is to register your application. To do this, make the following request:

```http
POST https://api.aihelps.com/v1/applications?fields=application_id,application_secret
```

```json
{
  "application_name": "Best Salons",
  "application_url": "bestsalons.com",
  "developer_name": "Best Salons Solutions",
  "contact_email": "dev@bestsalons.com",
  "scope":
  [
    "clients_module",
    "services_aggregator"
  ],
  "grant_access_url": "bestsalons.com/access/{database_code}"
}
```

Most fields are self-explanatory.
Application name and URL, the developer name will be shown to the salon/clinic when access will be asked. A contact email will not be shown to clients and needed to communicate with the application owner/developer.

The scope defines rights that application will have in clients database. `clients_module` was made to fit all needs for online booking, connecting aggregator and similar tasks (get services, professionals, free time), `services_aggregator` - find clients and make appointment without logging in as client. There are other available scopes but they probably not needed in our case (you can still read about them in documentation).

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

## Request Access Token for Client Database

On this step, we assume you already have `application_id` & `application_secret` (if not, see the previous chapter) and now want to receive access to some client database. This step should be repeated for each salon/clinic you want to receive access.

To make an access request to some clients database, you need to know AI Helps client code - 6-digit number, representing salon/clinic.
Ask the salon/clinic for client id. It can be found in Beauty Pro/Denta Pro on Settings page => Location settings => Location code (6 digits, big letters).
If needed, AI Helps technical support can help the client to get the client id.
Try to avoid situations when you found client id in some way and connect his online module on your aggregator without his agreement - clients don't like someone running booking to their salons/clinics on external resources without their confirmation.

So, in this scenario, unlike just placing the online module on your page, client code (not so private information indeed), the client should confirm access by action. This is crucial for clients as they stay confident about access to their data.

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

```json
{
  "server": 1,
  "access_token": "a71923bd-84c5-49a3-a7fb-a0825e26bbe4",
  "database": "123456",
  "scope":
  [
    "clients_module",
    "services_aggregator"
  ],
  "expires_at": "2039-06-01T00:00:00.000Z"
}
```

Access token, that will be needed for next requests is `access_token` (each client database will need its own access token). Save it in a secure place.
You will receive a token for 20 years, so there is no need to worry about token expiration.

The next important parameter is server - it contains a server number where the client database is stored. Servers are distributed around the globe, you can make requests to any server (and they will respond correctly), but requesting database through the different server will take much more time. So, accessing the clients database, make requests to the next API server:
* https://api.aihelps.com if `server` is 1
* https://api{server}.aihelps.com (for example https://api3.aihelps.com ) if `server` is greater than 1

## Request Access Token for Client Database after Application Published

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

## Get Services List

First, you need to receive a list of services, provided by the salon/clinic.

Request `/services` endpoint, providing access token in Authorization header for the database you want to get data:

```http
GET https://api.aihelps.com/v1/services/tree?fields=name,description,duration,gender,location_prices&categories_fields=name&public=true&has_professional_price=true&archive=false

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

In this example, we requested non-archive (archive services can't be sold any more) public services (allowed to booked by clients themself, usually all services) that have a price for at least one professional (skipping services like tanning, where no professionals needed).

```json
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

All services are grouped into categories. Categories are objects with next fields:
- `id` - category id (not needed in our scenario)
- `name` - category name
- `categories` - child categories for current category
- `items` - services located in this category
Categories and services are presented in a hierarchical view. The root is a base category with null `id` and without a name.

Each service has the next fields:
- `id` - service id (will be used for booking)
- `name` - services name
- `description` - optional service description (with basic HTML formatting)
- `duration` - service duration, in minutes
- `gender` (`both`/`male`/`female`) - client gender for current service
- `location_prices` - prices for current service for different professionals and employees (service price can differ in different locations and for different professionals)
  + `location` - location, for which price is given, see section [Working with chains (networks)](#chains) for details
  + `position` - professionals position for which price is given (price sets not for specific professional but for professionals position instead)
  + `price` - service price
  + `staff_price` - service price for staff under some conditions, not needed in our scenario

There is no such thing as a common catalog of services, each client in his database creates its own services and services structure, you should connect them to some sort of your services catalog manually or with help of some sort of AI - if you need a common list of services between all your salons/clinics.

## Get Professionals List

The next thing you need is professionals list with their positions.

Get all possible professional positions:

```http
GET https://api.aihelps.com/v1/positions?fields=name&role=professional

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

The result will be something like this (you need a name for each position id, received in services or professionals lists):

```json
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

Get the list of all non-archive (archive professionals can't provide any services) public professionals (allowed for booking by clients themself):

```http
GET https://api.aihelps.com/v1/employees?fields=name,positions&role=professional&archive=false&public=true

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

The result will be something like this:

```json
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

Please take into account that one professional can belong to several positions. So certain professionals can provide certain services if they have at least one common position (service can have several positions, professional can have several positions).

## Get Professionals Free Time

On this step, you already know services and professionals.

One more thing you need to know before booking client - time available for booking, equal to employee free time (API forbids booking one employee with two overlapping appointments).

So, this endpoint returns free time for all professionals for certain time range:

```http
GET https://api.aihelps.com/v1/employees/free_time?from=2019-01-01T10:00:00.000Z&to=2019-01-03T14:30:00.000Z&duration=120&step=30m

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

`duration` and `step` needs to be explained:
the result will be returned as a list of dates/times for each day for each professional. Each date/time is free for booking for certain professional for at least `duration` minutes (pass total selected services duration in this field to be sure endpoint returns date/time where enough time for all selected services). `duration` is any number greater than 0. `step` represent frequency of date/times available, in minutes, possible values: 10m, 15m, 20m, 30m, 60m, 90m, 120m, 150m, 180m, auto.

So, if `duration` = 90 and `step` = 30m and professional has free time from "10:00" till "13:00", endpoint will return: "10:00", "10:30", "11:00" and "11:30".

You will receive free time in the next format:

```json
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

You also can optionally specify professionals for which you want to get free time (filter out all other professionals; this can be quicker than free time for all professionals):

```http
GET https://api.aihelps.com/v1/employees/free_time?from=2019-01-01T10:00:00.000Z&to=2019-01-03T14:30:00.000Z&duration=120&step=30m&professionals=88d4b305-e198-f02c-2743-cca9390c631c&location=a7dd57e5-c495-46d4-b54d-753061d355a0

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

## Working with Chains (Networks) <a name="chains"></a>

Several words should be said about databases where a chain of several locations works. For example, the beauty salon chain has 5 different locations with common clients list, but different employees and different prices for services in different locations (sometimes happens that some service has a price and can be sold not in every location).

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

The client selects a certain location and then you get service prices and employees' free time just for that location.

## Timezones

While getting employee free time and creating an appointment requires working with date/time, so the question with time zones rises. In some cases it's obvious: you aggregate clients inside one country with one time zone so for all your salons/clinics you always know the correct timezone and time is common for all your aggregator. In more complex cases you need to know timezone for each database and even for each location (yes, locations with different timezones can be stored in one database). Timezones are represented in the TZData format (all browsers are using), so no issues with receiving the current time zone or working with time zones should happen. For each location you can get actual timezone:

```http
GET https://api.aihelps.com/v1/locations?fields=name,timezone

Authorization: Bearer 9c4068e2-c81f-4d70-ad31-8f627ed9bced // your received access token is here
```

You will receive something like:

```json
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

## Selecting Client

Now you have all the information about location: selected services, professional, appointment start time and duration. Now we can select the client and save an appointment.

So, as result of this step you should receive client id.

You usually have client phone and we propose to match clients via phone. So, first you need to do is check if client with some phone number exists:

```http
GET https://api.aihelps.com/v1/clients?phone=18005550123&fields=

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

Please note that you should pass empty fields list - you do not have access to information about clients. You will receive one or several client ids (if found). If more than one client id returned for provided phone - take any (first one), this mean several clients has same number and you can't select client more precise.

If no client was found, you should create new one. You have possibility to set client first name, middle name, last name, gender, birthday, phone, e-mail and comment.
But you will not be able to get this client fields later, change them or delete client (for security reasons). So pass all fields at once:

```http
POST https://api.aihelps.com/v1/clients

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "firstname": "Peter",
  "middlename": "Ivanovych",
  "lastname": "Brown",
  "gender": "male",
  "birthday": "1990-01-01",
  "phone": "+1 800 555 0123",
  "email": "peter@gmail.com",
  "comment": "some comments"
}
```

You will receive id, that can be used to book an appointment.

We do not recommend you store client ids between sessions - sometimes they can be changed (for example, merging two client cards with same contacts), you should better check client by phone on each booking.

## Book an Appointment

So, you have all appointment details and client id, you can make an appointment:

```http
POST https://api.aihelps.com/v1/appointments

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "state": "planned",
  "date": "2018-01-01T10:00:00.000Z",
  "location": "8fe29e48-d5a5-4dcc-800e-80eb6e9cac5c",
  "client": "c07210d0-d956-4067-b9bd-ba1401a11df3",
  "comments": "",
  "clientsModule": true,
  "services":
  [
	{
		"start": "12:00",
		"duration": 90,
		"service": "4b244443-90d3-4182-918a-fe3ac4a03688",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	},
	{
		"start": "13:30",
		"duration": 60,
		"service": "5b1a5043-8fbc-4ed0-aa01-3b9ded3ac77a",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	}
  ]
}
```

All fields are self explanatory enough.

While creating an appointment, you should check for HTTP Status code 409 (Conflict). This can happen in two situations:
1) appointment for that client & date in that location already exists (`type` = "ALREADY_EXISTS") - only one appointment can exists for specific client & date & location:
```json
{
    "status": 409,
    "type": "ALREADY_EXISTS",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
    "message": "Only one planned or confirmed appointment should exists for specified date + location + client + employee",
    "details": "https://aihelps.docs.apiary.io/#introduction/errors/409-conflict-already_exists",
    "type": "appointment",
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
Take existing appointment `id`, get all services:

```http
GET https://api.aihelps.com/v1/appointments/74e85faf-de73-4fe6-9530-3af1dd7c1b34?fields=services(start,duration,service,professional)

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "id": "74e85faf-de73-4fe6-9530-3af1dd7c1b34",
  "services":
  [
	{
		"id": "32491b07-e932-4db2-b763-b9fddf1dd1df",
		"start": "19:00",
		"duration": 60,
		"service": "0fa92303-7dc1-464b-ae77-5c0955422cdb",
		"professional": "14643b4c-0d1d-4ac0-968b-cf8644b3bf3a"		
	}
  ]
}
```

Add yours and update services:

```http
PUT https://api.aihelps.com/v1/appointments/74e85faf-de73-4fe6-9530-3af1dd7c1b34

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "services":
  [
	{
		"id": "32491b07-e932-4db2-b763-b9fddf1dd1df",
		"start": "19:00",
		"duration": 60,
		"service": "0fa92303-7dc1-464b-ae77-5c0955422cdb",
		"professional": "14643b4c-0d1d-4ac0-968b-cf8644b3bf3a"		
	},  
	{
		"start": "13:00",
		"duration": 90,
		"service": "4b244443-90d3-4182-918a-fe3ac4a03688",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	},
	{
		"start": "14:30",
		"duration": 60,
		"service": "5b1a5043-8fbc-4ed0-aa01-3b9ded3ac77a",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	}
  ]
}
```

2) second situation - professional is busy at the specified time (even one-minute overlap, `type` = "TIME_CONFLICT") - either overlaps with other appointments or your appointment is outside employee work time. This can happens either because of problems in your code or sometimes another client book that time just between you select a free time and made an appointment. No problems - show "Excuse me" dialog and ask the client to select a new appointment time.

```json
{
    "status": 409,
    "type": "TIME_CONFLICT",
    "request": "a7dfb9a8-0383-4087-a1e6-606fc2bc714c",
	...
}
```

After succesfull booking you will receive appointment id:

```json
{
  "id": "3ab52942-da15-4a20-b14c-b8fca7e4bf1a"
}
```

According to program settings, the client will receive SMSes about successful booking and later notification several hours before the visit start.

If you need to change some details, call request with fields that should be changed:

```http
PUT https://api.aihelps.com/v1/appointments/3ab52942-da15-4a20-b14c-b8fca7e4bf1a

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "comments": "some changes",
  "services":
  [
	{
		"start": "13:00",
		"duration": 90,
		"service": "4b244443-90d3-4182-918a-fe3ac4a03688",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	},
	{
		"start": "14:30",
		"duration": 60,
		"service": "5b1a5043-8fbc-4ed0-aa01-3b9ded3ac77a",
		"professional": "2783c237-214d-4f13-bae2-ac4eee79ba03"		
	}
  ]
}
```

If you need to cancel the appointment:

```http
PUT https://api.aihelps.com/v1/appointments/3ab52942-da15-4a20-b14c-b8fca7e4bf1a

Authorization: f4e6bdc1-58a9-4b2e-a06c-98ca194b2bf4
```

```json
{
  "state": "cancelled",
  "cancel_reason": "Client cancelled visit"
}
```

That's all!

If you have some questions, feel free to contact the group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.
