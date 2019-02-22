# AIHelps init script

## Preword

####**Warning!**
This this a documentation for a new version of the module which is not released yet!
Contact: **@kolomiieys (_telegram_)** for more info.

## to Get Started

1. Include the script in the HTML between tags `<head> </head>`
2. Then call init method `window.AIHelpsModule.init` and send in is this method settings, see [Property Settings](#setting).
3. Then call open method `window.AIHelpsModule.open` you can see module on own site. If you want close module call `window.AIHelpsModule.close`, More informations about method, see [AIHelpsModule methods](#AIHelpsModule).

## AIHelpsModule methods <a name="AIHelpsModule"></a>

### AIHelpsModule.init

This method needed to initialization [settings](#setting) to the module. You can send [settings](#setting) as the argument in this method. The other methods will not be work if you first do not call this method.

Example use:

```js
AIHelpsModule.init({
  codes: "503953",
  positionModule: "center",
  buttonProperties: {
    position: ["bottom", "left"],
    margin: [50, 50, 50, 50],
    template: "pink",
    text: "Online Booking",
  });
```

### AIHelpsModule.open

### AIHelpsModule.close

## Properties Settings <a name="setting"></a>

The setting is an object this object send in init method, see table below properties settings.

| Property                    |   Type   |              Default               |           Values            | Description                                                                                                          |
| --------------------------- | :------: | :--------------------------------: | :-------------------------: | -------------------------------------------------------------------------------------------------------------------- |
| codes                       |  string  |               503953               |              -              | This own code your base **_is Required_**.                                                                           |
| positionModule              |  string  |              `center`              |     `left|center|right`     | Positions module on site.                                                                                            |
| buttonId                    |  string  |             undefined              |         any string          | Button id on which click then script does display module if buttonId not set, the script add own button.             |
| buttonProperties            |  object  | [Object](<(#customizationButton)>) | [see](#customizationButton) | Customization button [Properties object](#customizationButton) this property need if you **_did not set buttonId_**. |
| overrideCRMSettings         |  object  |             undefined              |             see             | Overwrite the settings of the online module that we get from CRM.                                                    |
| professionalsDefault        |  string  |             undefined              |       id professional       | Set a selected professional                                                                                          |
| servicesDefault             |  string  |             undefined              |         id services         | Set a selected services                                                                                              |
| appointmentsDefaultLocation |  string  |             undefined              |         id location         | Set a selected location                                                                                              |
| moduleOpenedCallback        | function |             undefined              |        any function         | Callback function when the module will be open.                                                                      |
| moduleClosedCallback        | function |             undefined              |        any function         | Callback function when the module will be close.                                                                     |

#### Own button properties customization <a name="customizationButton"></a>

| Property |       Type       |       Default        |                                               Values                                               | Description                                                    |
| -------- | :--------------: | :------------------: | :------------------------------------------------------------------------------------------------: | -------------------------------------------------------------- |
| position | array of strings | `["bottom", "left"]` | First element in array `bottom|top` second element in array `left|right`. `[vertical, horizontal]` | This property sets display position for the default button.    |
| margin   | array of numbers |  `[50, 50, 50, 50]`  |                                    `[top, left, bottom, right]`                                    | Set an indent value (in pixels) for the default 'open' button. |
| template |      string      |        `pink`        |                                     Choose one theme of `pink`                                     | Set the theme default 'open' button.                           |
| text     |      string      |   `Online Booking`   |                                             any string                                             | Text on button.                                                |
