# AIHelps parther form init script

## Prewords:

This documentation describes actions needed to install AIHelps parther form on your site. If you
have any questions, feel free to contact **https://t.me/joinchat/EVNM_kgTp_iDmHv0Z-1npg
(_telegram_)** for more info.

## Get Started:

1. Include script `<script type="module">...</script>` don't forget set `type="module"`. All scripts
   are inserted in HTML between tags `<head> </head>` or between `<body> </body>` tags
2. The first you need to import `init` method from ES module like this
   ```js
   import { init } from "Here is link to script";
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

## All init method props: <a name="props"></a>

| Prop name      | Type                          | Description                                                                                                                                                  |
| -------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| partnerCode    | five-digit number             | Partner code for new client. Property is **required**                                                                                                        |
| emailFieldId   | string                        | The id of email HTML Input Element. Property is **required**                                                                                                 |
| phoneFieldId   | string                        | The id of phone number HTML Input Element. Property is **required**                                                                                          |
| submitButtonId | string                        | The id of submit HTML Button Element. Property is **required**                                                                                               |
| nameFieldId    | string                        | The id of client name HTML Input Element.                                                                                                                    |
| commentFieldId | string                        | The id of client comment HTML Input Element.                                                                                                                 |
| companyFieldId | string                        | The id of client company HTML Input Element.                                                                                                                 |
| programType    | [PROGRAM_TYPE](#program_type) | Program type to set what program you need to download. **By default set "Beauty Pro"**                                                                       |
| onSuccess      | function                      | The callback function called when the client successfully sended the form to API, gets the [OnSuccessType](#on_success_type) as an argument to the function. |
| onError        | function                      | The callback function called when the client successfully sended the form to API, gets the [OnErrorType](#on_error_type) as an argument to the function.     |

## Types:

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
| timezone     | string                        | Client timezone.                                                        |
| code         | string                        | Client data-base code. Six-digit number represented as a string.        |
| filename     | string                        | Program filename. **Can be undefined**.                                 |

#### OnErrorType <a name="on_error_type"></a>

| Prop name | Type             | Description                                                                                             |
| --------- | ---------------- | ------------------------------------------------------------------------------------------------------- |
| errorType | string           | This is specific string line one of `"Wrong email"`, `"Wrong phone number"`, `"Internal server error"`. |
| field     | HTMLInputElement | HTMLInputElement is from the field where happened error. **Can be undefined**.                          |
| fieldId   | string           | Id HTMLInputElement where happened error. **Can be undefined**.                                         |
| value     | string           | Value from HTMLInputElement where happened error. **Can be undefined**.                                 |

#### PROGRAM_TYPE <a name="program_type"></a>

This is specific string line one of `"Beauty Pro"`, `"Fitness Pro"`, `"Denta Pro"`.
