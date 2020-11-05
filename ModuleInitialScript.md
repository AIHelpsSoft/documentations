# AIHelps init script

## Prewords

This documentation describes actions needed to install AIHelps online module on your site.
If you have any questions, feel free to contact **https://t.me/online_mod_AIHelps (_telegram_)** for more info.

## Get Started

1. Include script `<script type="module">...</script>` don't forget set `type="module"`. All scripts are inserted in HTML between tags `<head> </head>` or between
   `<body> </body>` tags
2. The first you need to import `init` method from ES module like this
   ```js
   import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";
   ```
3. Call init method like `init()` (on page start) and send init settings, see
   [Property Settings](#settings).
4. Module will create its own button or will be attached to some element by id, so will automatically be opened/closed. You can
   manually open or close module from your code, calling `openModule()` or `closeModule()` respectively. for more information about methods, see
   [AIHelpsModule methods](#AIHelpsModule).

For example:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- ... snip ...  -->
    <title>Online Booking</title>
    <!-- ... snip ...  -->
  </head>
  <body>
     <!-- ... snip ...  -->
     <script type="module">
      import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";

      init({
        database: 503953
      });
    </script>
    <!-- ... snip ...  -->
  </body>
</html>
```

## AIHelpsModule methods <a name="AIHelpsModule"></a>

### init()

This method is needed to initialize module [settings](#settings). You can
send [settings](#settings) as the argument in this method. Other methods will
not work until this method called.

Example usage:

```js
import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  database: 503953 /** your database code, this value is required */,
  color: "#a7a7d7",
  position: "bottom right",
  text: "Click me!",
  designTheme: "normal",
  onOpen: () => console.log("module open"),
  onClose: () => console.log("module close")
});
```

### openModule()

This function can be called to show module manually (call after init method).

Example use:

```js
import {
  init,
  openModule,
} from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  /** ... */
});

openModule();
```

### closeModule()

This function can be called to hide module manually (call after init method).

Example use:

```js
import {
  init,
  closeModule,
} from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  /** ... */
});

closeModule();
```

## Settings <a name="settings"></a>

Settings is an object with settings, provided to `init` method. List of all possible settings provided in table below.

Only parameter `database` is required, all other parameters are optional.

| Property       |         Type         |                                              Values                                              |          Default value         |Description                                                                                                                                                                                        |
| -------------- | :------------------: | :----------------------------------------------------------------------------------------------: | :----------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| database       |        string        |                                        database code                                        | no default value, property is **required** |Your database code. This parameter is **required**.|
| elementId      |        string        |                                         element id or class name                                |  null        | Id of element or class of elements, click on which should open module. If parameter not set, script will display own "Book now!" button.                                                                               |
| position       |        string        | `${top \| bottom \| right \| left} ${top \| bottom \| right \| left}` for example: `"top right"`| `bottom right` | "Book now!" button position on page. Have no sense if you set elementId.                                                                                                                               |
| text           |        string        |                                            any string                                 | `Онлайн-запись`           | "Book now!" button text. Have no sense if you set elementId.                                                                                                                                            |
| color          |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`        |  `#6F3BF5`       | "Book now!" button color. Have no sense if you set elementId.                                                                                                                                                    |
| onOpen         |       function       |                                           any function                                | null           | Callback function to call on module open.                                                                                                                                                    |
| onClose        |       function       |                                           any function                                | null           | Callback function to call on module close.                                                                                                                                                   |
| professional   |        string        |                                         professional id                               | null           | Provide default professional for booking (will be automatically preselected in online module)                                                                                                                                                                        |
| services       | `string \| [string]` |                                           services id                                 | empty array    | Provide default list of services for booking (will be automatically preselected in online module)                                                                                                                                                                       |
| location       |        string        |                                           location id                                 | null           | Provide location for booking (by default all locations will be available and location select page will be shown in case more than one location available in chain)                                                                                                                                                                       |
| enabled        |       boolean        |                                         `true \| false`                               | `true`           | Set to `false` to temporary disable online module.                                                                  |
| closeable |       boolean        |                                         `true \| false`                                    | `true`      | In case module is shown full screen (on mobile devices) - if module has separate "close" button to return to site.                    |
| designTheme    |        string        |                                    `soft \| normal \| strong`                         | `soft`           | Online module theme                                                                                                                                                                         |
| designColor    |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`      | `#6F3BF5`           | Online module color                                                                                                                                                                                        |
| modulePosition |        string        |                                    `left \| center \| right \| auto`                          | `auto`           | Online module position when module open (left side of screen, right side of screen, screen center). If `auto` is set: if "Book now!" button is shown, module will be shown left or right depending on `position` property; otherwise (`elementId` is set) module will be shown on the right side of screen. You can use this property if you want to place the module in a non-standard position. |
