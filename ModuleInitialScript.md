# AIHelps init script

## Prewords

This this a documentation for a new version of the module which is **not released
yet!** Contact: **@kolomiieys (_telegram_)** for more info.

## to Get Started

1. All scripts are inserted in HTML between tags `<head> </head>` or between
   `<body> </body>` tags
2. The first you need to import the init method from ES module like this
   ```js
   import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";
   ```
3. Then call init method `init()` and send in is this method settings, see
   [Property Settings](#setting).
4. Then call open method `openModule()` you can see module on own site. If you
   want close module call `closeModule()`, More informations about method, see
   [AIHelpsModule methods](#AIHelpsModule).

for example:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Online Booking</title>
  </head>
  <body>
    <script type="module">
      import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";

      init({
        database: 503953,
      });
    </script>
  </body>
</html>
```

## AIHelpsModule methods <a name="AIHelpsModule"></a>

### Init

This method needed to initialization [settings](#setting) to the module. You can
send [settings](#setting) as the argument in this method. The other methods will
not be work if you first don't call this method.

Example use:

```js
import { init } from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  database: 503953 /** your database code. this value is required */,
  color: "#a7a7d7",
  position: "bottom right",
  text: "Click me!",
  designTheme: "normal",
  onOpen: () => console.log("module open"),
  onClose: () => console.log("module close"),
});
```

### openModule

This function can be called after the init method and use to show the module.

Example use:

```js
import {
  init,
  openModule,
} from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  /** ... */
});

/** You can call this method before the init method */
openModule();
```

### closeModule

This function can be called after the init method and use to hide the module.

Example use:

```js
import {
  init,
  closeModule,
} from "https://beautyprosoftware.com/online-booking-init/index.js";

init({
  /** ... */
});

/** You can call this method before the init method */
closeModule();
```

## Properties Settings <a name="setting"></a>

The setting is an object this send in init method, see table below
properties settings.

| Property       |         Type         |                                              Values                                              | Description                                                                                                          |
| -------------- | :------------------: | :----------------------------------------------------------------------------------------------: | -------------------------------------------------------------------------------------------------------------------- |
| database       |        string        |                                        code your database                                        | This own code your base **_is Required_**.                                                                           |
| elementId      |        string        |                                         your element id                                          | Element id on which click then the script does display module if **elementId not set**, the script added own button. |
| position       |        string        | `${top \| bottom \| right \| left} ${top \| bottom \| right \| left}` for example: `"top right"` | Position own button on the site. Doesn't work if you set elementId                                                   |
| text           |        string        |                                            any string                                            | Text on own button. Doesn't work if you set elementId                                                                |
| color          |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`                 | Color button. Doesn't work if you set elementId                                                                      |
| onOpen         |       function       |                                           any function                                           | Callback function when the module will be open.                                                                      |
| onClose        |       function       |                                           any function                                           | Callback function when the module will be close.                                                                     |
| professional   |        string        |                                         id professional                                          | Set a selected professional                                                                                          |
| services       | `string \| [string]` |                                           id services                                            | Set a selected services                                                                                              |
| location       |        string        |                                           id location                                            | Set a selected location                                                                                              |
| enabled        |       boolean        |                                         `true \| false`                                          | This property use when you need to enabled or disabled Online Module                                                 |
| isNotCloseable |       boolean        |                                         `true \| false`                                          | If this property has true then module show on start and cannot be closed. Default false                              |
| designTheme    |        string        |                                    `soft \| normal \| strong`                                    | DesignTheme                                                                                                          |
| designColor    |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`                 | DesignColor                                                                                                          |
| positionModule |        string        |                                    `center \| right \| left`                                     | **Not recommended to use**. Module appearance position                                                               |
