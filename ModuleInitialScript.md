# AIHelps init script

## Preword

This this a documentation for a new version of the module which is not released yet!
Contact: **@kolomiieys (_telegram_)** for more info.

## to Get Started

1. Include the script in the HTML between tags `<head> </head>` or between `<body> </body>` tags
2. Then call init method `window.AIHelpsModule.init` and send in is this method settings, see [Property Settings](#setting).
3. Then call open method `window.AIHelpsModule.open` you can see module on own site. If you want close module call `window.AIHelpsModule.close`, More informations about method, see [AIHelpsModule methods](#AIHelpsModule).

## AIHelpsModule methods <a name="AIHelpsModule"></a>

### AIHelpsModule.init

This method needed to initialization [settings](#setting) to the module. You can send [settings](#setting) as the argument in this method. The other methods will not be work if you first do not call this method.

Example use:

```js
AIHelpsModule.init({
  codes: "503953",
  buttonPosition: "bottom_right",
  buttonMargin: [50, 50, 50, 50],
  buttonColor: "pink",
  buttonText: "Online Booking",
```

### AIHelpsModule.open

**TODO: not yet**

### AIHelpsModule.close

**TODO: not yet**

## Properties Settings <a name="setting"></a>

The setting is an object this object send in init method, see table below properties settings.

| Property                    |   Type   |     Default      |                        Values                         | Description                                                                                                    |
| --------------------------- | :------: | :--------------: | :---------------------------------------------------: | -------------------------------------------------------------------------------------------------------------- |
| code                        |  string  |    `"503953"`    |                           -                           | This own code your base **_is Required_**.                                                                     |
| elementId                   |  string  |   `undefined`    |                      any string                       | Element id on which click then script does display module if **elementId not set**, the script add own button. |
| buttonPosition              |  string  |  `bottom_right`  | `"bottom_right"` or `"bottom_left"` or `"top_right"`  | Position button on the site                                                                                    |
| buttonText                  |  string  | `Online Booking` |                      any string                       | Text on button                                                                                                 |
| buttonColor                 |   HEX    |    `#6F3BF5`     | "#0052F1", "#F55C3B", "#F53BEE", "#4DC602", "#CEA206" | Color button                                                                                                   |
| overrideCRMSettings         |  object  |   `undefined`    |                          see                          | Overwrite the settings of the online module that we get from CRM.                                              |
| professionalsDefault        |  string  |   `undefined`    |                    id professional                    | Set a selected professional                                                                                    |
| servicesDefault             |  string  |   `undefined`    |                      id services                      | Set a selected services                                                                                        |
| appointmentsDefaultLocation |  string  |   `undefined`    |                      id location                      | Set a selected location                                                                                        |
| moduleOpenedCallback        | function |   `undefined`    |                     any function                      | Callback function when the module will be open.                                                                |
| moduleClosedCallback        | function |   `undefined`    |                     any function                      | Callback function when the module will be close.                                                               |
