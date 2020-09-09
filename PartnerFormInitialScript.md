# AIHelps parter form init script

## Prewords:

This documentation describes actions needed to install AIHelps parter form on your site. If you have
any questions, feel free to contact _[telegram group](https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg)_
for more info.

## Get Started:

1. Include script `<script type="module">...</script>` don't forget set `type="module"`. All scripts
   are inserted in HTML between tags `<head> </head>` or between `<body> </body>` tags
2. The first you need to import `init` method from ES module like this
   ```js
   import { init } from "https://beautyprosoftware.com/partner-from-init/index.js";
   ```
3. Next step is you need to call the init method with custom [props](#props) for example:
   ```js
   init({
     phoneFieldId: "phoneFieldId",
     submitButtonId: "submitButtonId",
     emailFieldId: "emailFieldId",
     partnerCode: 12345,
     programType: "Fitness Pro",
     onError: (args) => {
       console.error("onError: ", args);
     },
     onSuccess: (args) => {
       console.log("onSuccess: ", args);
     },
   });
   ```
4. Then you need to check the devtools console, and if you don't see any
   [errors](#initialization_props_errors) like this: </br>
   <img src="https://raw.githubusercontent.com/AIHelpsSoft/documentations/master/images/partner_form_initial_script_img2.png" />
   this means everything is good, and you have done a great job, thanks for using AIHelps solutions

## Methods:

#### Init

This method just initialization the partner form for the download program, receive as an argument
[init props](#props). To import this method please see the code below:

```js
import { init } from "https://beautyprosoftware.com/partner-from-init/index.js";
```

The props `partnerCode, emailFieldId, phoneFieldId, submitButtonId` **are required**. You can see
all the [props](#props) in the table below.

Example:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Parter Form Test Page</title>
    <!-- ... snip ...  -->
    <script type="module">
      import { init } from "https://beautyprosoftware.com/partner-from-init/index.js";

      init({
        partnerCode: 12345,
        phoneFieldId: "phone_id",
        submitButtonId: "submit_button_id",
        emailFieldId: "email_id",
      });
    </script>
    <!-- ... snip ...  -->
  </head>
  <body>
    <!-- ... snip ...  -->
    <input type="tel" id="phone_id" placeholder="Phone Number" />
    <input type="email" id="email_id" placeholder="Email" />

    <button type="submit" id="submit_button_id">I'm button for submit</button>
    <!-- ... snip ...  -->
  </body>
</html>
```

## All init method props: <a name="props"></a>

| Prop name      | Type                          | Description                                                                                                                                                               |
| -------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| partnerCode    | five-digit number             | Partner code for new client. **Property is required**.                                                                                                                    |
| emailFieldId   | string                        | The id of email HTML Input Element. **Property is required**.                                                                                                             |
| phoneFieldId   | string                        | The id of phone number HTML Input Element. **Property is required**.                                                                                                      |
| submitButtonId | string                        | The id of submitting HTML Button Element, clicking on which sends a request to the API, and start downloading the program. **Property is required.**                      |
| nameFieldId    | string                        | The id of client name HTML Input Element.                                                                                                                                 |
| commentFieldId | string                        | The id of client comment HTML Input Element.                                                                                                                              |
| companyFieldId | string                        | The id of client company HTML Input Element.                                                                                                                              |
| programType    | [PROGRAM_TYPE](#program_type) | Program type to set what program you need to download. **By default set "Beauty Pro"**.                                                                                   |
| onSuccess      | function                      | The callback function called when the client has successfully sent the form to API gets as argument the [OnSuccessType](#on_success_type) as an argument to the function. |
| onError        | function                      | The callback function called when the client has failed to send the form to API gets the [OnErrorType](#on_error_type) as an argument to the function.                    |

## Types:

#### PROGRAM_TYPE <a name="program_type"></a>

This is specific string line one of `"Beauty Pro"`, `"Fitness Pro"`, `"Denta Pro"`.

#### OnSuccessType <a name="on_success_type"></a>

| Prop name    | Type                          | Description                                                             |
| ------------ | ----------------------------- | ----------------------------------------------------------------------- |
| program      | [PROGRAM_TYPE](#program_type) | Program type.                                                           |
| email        | string                        | Client email.                                                           |
| phone        | string                        | Client phone number.                                                    |
| type         | string                        | This is specific string line one of `"mac"`, `"mobile"`, `"database"`.  |
| partner_code | string                        | Partner code for new client. Five-digit number represented as a string. |
| person       | string                        | Client name. **Can be undefined**.                                      |
| comment      | string                        | Client comment. **Can be undefined**.                                   |
| company      | string                        | Client company. **Can be undefined**.                                   |
| timezone     | string                        | Client timezone in format `"Europe/Kiev"`.                              |
| code         | number                        | Client data base code. Six-digit number.                                |
| filename     | string                        | Program filename. **Can be undefined**.                                 |

#### OnErrorType <a name="on_error_type"></a>

| Prop name | Type                                    | Description                                                                            |
| --------- | --------------------------------------- | -------------------------------------------------------------------------------------- |
| errorType | [SUBMIT_ERROR_TYPE](#submit_error_type) | Some errors which appeared during sending API requests.                                |
| field     | HTMLInputElement                        | HTMLInputElement is from the field where the error has happened. **Can be undefined**. |
| fieldId   | string                                  | HTMLInputElement id where the error has happened. **Can be undefined**.                |
| value     | string                                  | Value from HTMLInputElement where the error has happened. **Can be undefined**.        |

## Errors:

#### SUBMIT_ERROR_TYPE <a name="submit_error_type"></a>

This is specific string line one of `"Wrong email"`, `"Wrong phone number"`,
`"Internal server error"`.

- `Wrong email` — Input email is invalid
- `Wrong phone number` — Input phone number is invalid
- `Internal server error` — During the sending request happened unexpected error

#### Initialization props errors <a name="initialization_props_errors"></a>

| Type of Error                 | Description |
| ----------------------------- | ----------- |
| PropsIsUndefined              |             |
| PropsIsNotObject              |             |
| PropPartnerCodeIsInvalid      |             |
| PropPhoneFieldIdIsUndefined   |             |
| PropSubmitButtonIdIsUndefined |             |
| PropEmailFieldIdIsUndefined   |             |
| CannotFindHTMLElement         |             |
| WrongPropType                 |             |
