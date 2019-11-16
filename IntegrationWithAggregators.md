# Integration with beauty/dental services aggregators (Tutorial)

TODO: check all document after finish with Grammarly

There are some requests to AI Helps for integrating Beauty Pro/Denta Pro products with services aggregators. We made this tutorial to simplify connecting process and collect all corresponding information in one place.

If you have any questions, feel free to contact group **[API AIHELPS](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)** in Telegram for more info.

# Show Online module on your site

The easiest way to allow booking to Beauty Pro/Denta Pro from your site: just show our online module on your site page.

You can see example module: https://beautyprosoftware.com/b/503953 (**TODO: get demo database from sales**)

Pros:
+ takes 5 minutes to implement
+ no setup work with AI Helps needed, you can do it by yourselves
+ no additional agreements needed from salon/clinic

Cons:
- client select service, professional, free time inside module, you can't make common services list for all salons/clinics in this way
- also you have no influence on client booking process and even don't know if client succesfully booked
- module has own design, different from your site and have limited design customization possibilities, it will anyway be "alien" element on your site

So, this method is recomended for salon/clinic catalogues, which don't have common services list, do not check free time to propose only locations with time available and have no or small integration with salons/clinics.
For aggregators, which have it's own services list and propose salon/clinic based on service needed and free time available and wants to control all booking process, we propose method, described in [Integration via API](#API).

## Implementation

To show online module on your page, just insert next script in your page:
```html
    <script type="module">
      import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";
      init({
        database: 503953,       // client id, you can get it from client or client online module
        elementId: bookButton   // html element id, click on which should show online module
      });
    </script>
```

Client id can be received from client or can be taken from client module, running on client's site, Facebook or Instagram page.

There are many options to interact with module: open/close it manually (do not attach to some element) or track open/close events, control module position and color theme, specify certain services/professionals to book. You can see detailed information about AI Helps init script setup here: https://github.com/AIHelpsSoft/documentations/blob/master/ModuleInitialScript.md

# Integration via API <a name="API"></a>

This method is much harder and can take days to implement but gives you all data needed fro booking and provides full control over all booking process.

You communicate with AI Helps API (full API documentation is available at https://api.aihelps.com).

First, you register your application in AI Helps application registry, providing information information about your company and contacts and receive application ID & secret needed for next steps. Registration is one simple API call and response is automatical (no verification process). This tep should be made only once.

Next step: for each client database (either salon or clinic) you want to get access, you need to receive access token. Access grant request should be sent and client either grants access or refuse from Beauty Pro/Denta Pro application. If granted, you will receive access token for access to specific client daabase. This step should be done for each your client.

After you have access token, you can make API requests to get list of services, professionals, free time and make booking. Possible scenarios will be described later in details.

## Register your application

// how to register, example, link to API documentation
// scope should be online_module, otherwise clients normally would not agree to give full access to data

## Request access token for client database


## Get services list


## Get professionals list

## Get professionals free time

## Client login

## Book an appointment

// separately say about possibility to make reserve while bboking process
// different errors should be explained here