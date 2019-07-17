# AIHelps init script

## Prewords

This this a documentation for a new version of the module which is **not released
yet!** Contact: **@kolomiieys (_telegram_)** for more info.

## to Get Started

1. All scripts are inserted in HTML between tags `<head> </head>` or between
   `<body> </body>` tags
2. The first you need to import the init method from ES module like this
   ```js
   import { init } from "https://beautyprosoftware.com/b";
   ```
3. Then call init method `init()` and send in is this method settings, see
   [Property Settings](#setting).
4. Then call open method `openModule()` you can see module on own site. If you
   want close module call `closeModule()`, More informations about method, see
   [AIHelpsModule methods](#AIHelpsModule).

## AIHelpsModule methods <a name="AIHelpsModule"></a>

### Init

This method needed to initialization [settings](#setting) to the module. You can
send [settings](#setting) as the argument in this method. The other methods will
not be work if you first don't call this method.

Example use:

```js
import { init } from "https://beautyprosoftware.com/b";

init({
  database: 503953 /** your database code. this value is required */,
  color: "#a7a7d7",
  position: "bottom_right",
  text: "Click me!",
  designTheme: "normal",
  onOpen: () => console.log("module open"),
  onClose: () => console.log("module close"),
});
```

### openModule

**TODO: No documentation for this yet**

Example use:

```js
import { init, openModule } from "https://beautyprosoftware.com/b";

init({
  /** ... */
});

/** You can call this method before the init method */
openModule();
```

### closeModule

**TODO: No documentation for this yet**

Example use:

```js
import { init, closeModule } from "https://beautyprosoftware.com/b";

init({
  /** ... */
});

/** You can call this method before the init method */
closeModule();
```

## Properties Settings <a name="setting"></a>

The setting is an object this object send in init method, see table below
properties settings.

| Property     |         Type         |                                              Values                                              | Description                                                                                                    |
| ------------ | :------------------: | :----------------------------------------------------------------------------------------------: | -------------------------------------------------------------------------------------------------------------- |
| database     |        string        |                                        code your database                                        | This own code your base **_is Required_**.                                                                     |
| elementId    |        string        |                                         your element id                                          | Element id on which click then script does display module if **elementId not set**, the script add own button. |
| position     |        string        | `${top \| bottom \| right \| left} ${top \| bottom \| right \| left}` for example: `"top right"` | Position button on the site                                                                                    |
| text         |        string        |                                            any string                                            | Text on button                                                                                                 |
| color        |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`                 | Color button                                                                                                   |
| onOpen       |       function       |                                           any function                                           | Callback function when the module will be open.                                                                |
| onClose      |       function       |                                           any function                                           | Callback function when the module will be close.                                                               |
| professional |        string        |                                         id professional                                          | Set a selected professional                                                                                    |
| services     | `string \| [string]` |                                           id services                                            | Set a selected services                                                                                        |
| location     |        string        |                                           id location                                            | Set a selected location                                                                                        |
| enabled      |       boolean        |                                         `true \| false`                                          | enabled                                                                                                        |
| designTheme  |        string        |                                    `soft \| normal \| strong`                                    | designTheme                                                                                                    |
| designColor  |         HEX          |                 `#6F3BF5 \| #0052F1 \| #F55C3B \| #F53BEE \| #4DC602 \| #CEA206`                 | designColor                                                                                                    |
