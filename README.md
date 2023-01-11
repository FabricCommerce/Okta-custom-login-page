*   [Login Page Customizations](#login-page-customizations)
*   [Registration Page Customization](#registration-page-customization)
*   [Email Customizations](#email-customizations)
*   [Adding custom fields](#adding-custom-fields)
    *   [Phone number Field (field supported by parseSchema event)](#phone-number-field-field-supported-by-parseschema-event)
    *   [Reconfirm password support](#reconfirm-password-support)
    *   [Date field (field NOT supported by parseSchema event)](#date-field-field-not-supported-by-parseschema-event)
*   [Supported Fields by parseSchema](#supported-fields-by-parseschema)
*   [Custom Validations](#custom-validations)
    *   [Validation on Registration / Signup Page:](#validation-on-registration--signup-page)
        *   [1\. Using preSubmit callback](#1-using-presubmit-callback-httpsgithubcomoktaokta-signin-widgetpresubmit)
        *   [2\. Using custom validation on onchange input field event](#2-using-custom-validation-on-onchange-input-field-event)
*   [Unsupported Feature by Okta](#unsupported-feature-by-okta)
*   [Sample working code for the customization:](#sample-working-code-for-the-customization)

Any cutomizations done in Sandbox, please document and create a ticket with Identity team. Identity team will take care of applying these changes to production. Only Identity team will have production access.

Login Page Customizations
-------------------------

[https://github.com/okta/okta-signin-widget#localization](https://github.com/okta/okta-signin-widget#localization)

[https://developer.okta.com/docs/guides/custom-widget/main/#modify-strings](https://developer.okta.com/docs/guides/custom-widget/main/#modify-strings)

[https://github.com/okta/okta-signin-widget/blob/master/docs/classic.md#okta-sign-in-widget-on-the-classic-engine](https://github.com/okta/okta-signin-widget/blob/master/docs/classic.md#okta-sign-in-widget-on-the-classic-engine)

[https://developer.okta.com/docs/guides/social-login/facebook/main/#add-the-identity-provider-to-the-embedded-okta-sign-in-widget](https://developer.okta.com/docs/guides/social-login/facebook/main/#add-the-identity-provider-to-the-embedded-okta-sign-in-widget)

Sample login screen after customization: [https://storefront.example.com/](https://storefront.example.com/)

![](attachments/2526281758/2525823295.png)

Registration Page Customization
-------------------------------

[https://developer.okta.com/docs/guides/set-up-self-service-registration/main/](https://developer.okta.com/docs/guides/set-up-self-service-registration/main/)

Note: if is there any requirement to support additional fields as part of Registration, please follow confluence page [here](https://yottadv.atlassian.net/wiki/spaces/all/pages/2459566326/Support+for+Custom+fields+User+Registration+for+CDP+Team).

Email Customizations
--------------------

Please check all email templates in Okta and customize them as required. Below are notifications have to be reviewed must.

1.  Registration - Activation
    
2.  Registration - Email Verification
    
3.  Password Changed
    
4.  Forgot password
    
5.  User activation
    
6.  Account Lockout
    

[https://help.okta.com/en-us/Content/Topics/Settings/Settings\_Email.htm](https://help.okta.com/en-us/Content/Topics/Settings/Settings_Email.htm)

[https://developer.okta.com/docs/guides/custom-email/main/](https://developer.okta.com/docs/guides/custom-email/main/)

[https://help.okta.com/en-us/Content/Topics/Settings/settings-customization-variables.htm](https://help.okta.com/en-us/Content/Topics/Settings/settings-customization-variables.htm)

Adding custom fields
--------------------

### Phone number Field (field supported by parseSchema event)

```js
config.registration = {
      parseSchema: function (schema, onSuccess, onFailure) {
        schema.profileSchema.properties.phoneNumber = {
          'type': 'string',
          'description': 'Phone number',
          'name': 'phoneNumber',
          'title': 'Enter Phone number',
          "pattern": phoneRegex
        };
        schema.profileSchema.fieldOrder.push('phoneNumber');
		onSuccess(schema);
    },
      preSubmit: (postData, onSuccess, onFailure) => {
          // validate here
          // eg. new RegExp(phoneRegex).test(postData.phoneNumber)) == false
      }
    }
    // Handle Error in afterError event
    signIn.on('afterError', function (context, error) {
    // handle preSubmit thrown error
    })
```

### Reconfirm password support

```html
config.registration = {
      parseSchema: function (schema, onSuccess, onFailure) {
      // This example will add an additional field to the registration form
        
        schema.profileSchema.properties.reenterPassword = {
          'type': 'password',
          'description': 'reenterPassword',
          'name': 'reenterPassword',
          'title': 'Reenter password',
          "allOf": schema.profileSchema.properties.password.allOf
        };
        // insert reenter password field after Password field
        schema.profileSchema.fieldOrder.splice(2, 0, 'reenterPassword');
        
		onSuccess(schema);
    },
    preSubmit: (postData, onSuccess, onFailure) => {
          /* handle preSubmit callback
        if (postData.password != postData.reenterPassword) {
        	const error = {
            "errorSummary": "Password_Mismatch",
            "errorCauses": [
                    {
                      "errorSummary": "Password and reentered passsword should be same",
                      "property": "userProfile.password",
                    }
                ]
            }
            onFailure(error);
          	return;
        }
    }
  }
```

Handle Error in `afterError` event

[https://github.com/okta/okta-signin-widget#aftererror](https://github.com/okta/okta-signin-widget#aftererror)

Ex:

```js
signIn.on('afterError', function (context, error) {
    console.log(context.controller);
    // registration

    console.log(error.name);
    // Password_Mismatch

    console.log(error.message);
    // The password does not meet the complexity requirements
    // of the current password policy.

    console.log(error.statusCode);
    // 403
});
```

### Date field (field NOT supported by parseSchema event)

Based on which context controller need the required input field, javascript snippet to add input element can be added. Ex: in login page

```html
oktaSignIn.on("ready", function (context) {
        console.log("onready:", context.controller);
        switch (context.controller) {
          case "registration":
            // ### insert date element after lastName 
            const lastNamefieldSet = document.querySelector("[name='lastName']")
                                      .closest('.o-form-fieldset');
                                      
            const dateFieldSet = lastNamefieldSet.cloneNode(true)
            dateFieldSet.querySelector('input').id = '';
            dateFieldSet.querySelector('input').type = 'date';
            
            lastNamefieldSet.after(dateFieldSet);
            
            break;
          default:
            console.log("controller not found");
        }
      });
```

Supported Fields by parseSchema
-------------------------------

Refer switch cases in okta formfactory file [https://github.com/okta/okta-signin-widget/blob/49971026a44db4a858e0d5c9ddcca08139c4f78a/packages/%40okta/courage-dist/esm/src/courage/views/forms/helpers/SchemaFormFactory.js#L73](https://github.com/okta/okta-signin-widget/blob/49971026a44db4a858e0d5c9ddcca08139c4f78a/packages/%40okta/courage-dist/esm/src/courage/views/forms/helpers/SchemaFormFactory.js#L73)

Custom Validations
------------------

### Validation on Registration / Signup Page:

#### 1\. Using preSubmit callback [https://github.com/okta/okta-signin-widget#presubmit](https://github.com/okta/okta-signin-widget#presubmit)  
Ex:

```java
config.registration = {
      parseSchema: function (schema, onSuccess, onFailure) {
      },
    preSubmit: (postData, onSuccess, onFailure) => {
          /* handle preSubmit callback
        if (postData.password != postData.reenterPassword) {
        	const error = {
            "errorSummary": "Password_Mismatch",
            "errorCauses": [
                    {
                      "errorSummary": "Password and reentered passsword should be same",
                      "property": "userProfile.password",
                    }
                ]
            }
            onFailure(error);
          	return;
        }
    }
  }
signIn.on('afterError', function (context, error) {
    console.log(context.controller);
    // registration

    console.log(error.name);
    // Password_Mismatch

    console.log(error.message);
    // The password does not meet the complexity requirements
    // of the current password policy.

    console.log(error.statusCode);
    // 403
});
```

#### 2\. Using custom validation on `onchange` input field event

Ex:

```java
// show custom error message on certain condition
$("#okta-signin-username").onchange = function(event, tar) {
    if (event.target.value.indexOf("@example.com") == -1) {
        var errorMessage = document.createElement('div');
        errorMessage.innerText = "Error: Email should have example.com";
        document.querySelector(somePath).append(errorMessage);
        return;
    }
}
```

Unsupported Feature by Okta
---------------------------

Customization can be done to support custom validation but **NOT** **recommended** by Fabric

1.  Validation on sigin page  
    You **cannot** do validation like preSubmit callback on signin page. You can add errormessage validation using **onchange** event but cannot change the behaviour of submit button.
    
2.  Validation on forgot password page  
    You **cannot** do validation like preSubmit callback on forgot password page. You can add errormessage validation using **onchange** event but cannot change the behaviour of submit button.
    

Sample working code for the customization:
------------------------------------------

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="robots" content="noindex,nofollow" />
    <!-- Styles generated from theme -->
    <link href="{{themedStylesUrl}}" rel="stylesheet" type="text/css">
    <!-- Favicon from theme -->
    <link rel="shortcut icon" href="{{faviconUrl}}" type="image/x-icon"/>
<script src="https://global.oktacdn.com/okta-auth-js/5.2.2/okta-auth-js.min.js" type="text/javascript"></script>
    <title>{{pageTitle}}</title>
  <style> .tab-view.active {border-bottom: solid 2px #000;
    padding-bottom: 3px;} main#okta-sign-in .okta-sign-in-header{ display: none; } #okta-login-container main#okta-sign-in { width: 100%; border-top: none; } </style>
    {{{SignInWidgetResources}}}
</head>
<body>
  <div class="login-bg-image tb--background" style="background:linear-gradient(135deg,#1c082c 25%,#ea47cc 85%,#cd3ace)"></div>
	<div class="top-header" style="height:75px;background:#000;margin-bottom:10px">
		<div style="float:left"><a style="color:#fff;font-family:montserrat;float:left;margin:33px 10px;transform:translate(0,-50%)" href="https://storefront.example.com/">Back to home</a></div>
		<div style="height:100%;overflow:hidden;text-align:center"><img src="https://fabric.inc/wp-content/uploads/2022/02/170220221645089413.png" height="170%" style="position:relative;top:-25px;left:-63px"></div>
	</div>
	<div class="custom-panel">
		<div class="inner-panel" style="width:90%;background:#fff;position:relative;left:95%;transform:translate(-100%,0)">
			<div class="tabs" style="padding:20px">
				<div class="all-tabs" style="border-bottom:solid 2px #c8c8c8;padding-bottom:3px">
					<div class="tab-signin tab-view active" style="float:left;margin-right:40px">
                      <a
                  class="Sign_Ip"
                  href="https://storefront.example.com/signin"
                         >Sign In</a></div>
					<div class="tab-signup tab-view">
                      <a
                  class="Sign_Up"
                  href="https://storefront.example.com/signin/register"
                         >Sign Up</a>
                  	</div>
				</div>
			</div>
			<div id="okta-login-container"></div>
		</div>
	</div>
	<div class="bottom-panel" style="height:75px;background:#292626">
		<div style="text-align:center;padding:10px">
			<select id="select-lang" style="width:324px;padding:10px">
				<option value="en">English</option>
				<option value="ja">Japanese</option>
			</select>
		</div>
	</div>  
  <!--
        "OktaUtil" defines a global OktaUtil object
        that contains methods used to complete the Okta login flow.
     -->
    {{{OktaUtil}}}

    <script type="text/javascript">
        // "config" object contains default widget configuration
        // with any custom overrides defined in your admin settings.
       var urlSearchParams = new URLSearchParams(window.location.search);
      var selectedLanguage = urlSearchParams.get('lang') || 'en';
      // "config" object contains default widget configuration
      // with any custom overrides defined in your admin settings.
      var config = OktaUtil.getSignInWidgetConfig();
	  config.features.showIdentifier = true;
		config.features.hideSignOutLinkInMFA = false;
		config.features.rememberMe = true
		config.features.autoFocus = true;
      	config.features.showPasswordToggleOnSignInPage = true;
      	config.features.showPasswordRequirementsAsHtmlList = true;
      //config.clientId = OktaUtil.getRequestContext();
      config.clientId = '0oatrevf6jatsMrZQ696';
      config.idps = [{
          type: 'Google',
          id: '0oa1epnapywOxn2pS697'
      }]
      console.log("getRequestContext", OktaUtil.getRequestContext());
      console.log("show config", config);
      //  config.redirectUri = //dev-lazurde.vercel.app;
      config.i18n = {
          // Overriding English properties
          'en': {
              'primaryauth.title': 'Welcome to fabric!',
              'primaryauth.username.placeholder': 'Fabric Email'
          },
          // Overriding Japanese properties
          'ja': {
              'primaryauth.title': 'のユーザー名 fabric!',
              'primaryauth.username.placeholder': 'Fabricのユーザー名'
          }
      };
      config.language = (supportedLanguages, userLanguages) => {
          var matchingLang = supportedLanguages.find(lang => lang == selectedLanguage);
          console.log(matchingLang);
          // supportedLanguages is an array of languageCodes, i.e.:
          // ['cs', 'da', ...]
          //
          // userLanguages is an array of languageCodes that come from the user's
          // browser preferences
          return matchingLang || 'en';
      }
      config.colors = {
          brand: '#ffffff'
      }
      config.logo = ''
      config.helpLinks = {
          help: 'https://storefront.example.com/help',
          forgotPassword: [location.protocol + '/',
                           location.hostname,
                           'signin',
                           'forgot-password'].join('/'),
          unlock: [location.protocol + '/',
                           location.hostname,
                           'signin',
                           'unlock-account'].join('/'),
          custom: [{
                  text: 'What is Okta?',
                  href: 'https://storefront.example.com/what-is-okta'
              },
              {
                  text: 'Storefront Website',
                  href: 'https://storefront.example.com',
                  target: '_blank'
              }
          ]
      }
    const phoneRegex = "^[0-9]{10}$";
	config.registration = {
      parseSchema: function (schema, onSuccess, onFailure) {
      // This example will add an additional field to the registration form
        console.log("SCHEMA BEFORE", schema);
        schema.profileSchema.properties.phoneNumber = {
          'type': 'string',
          'description': 'Phone number',
          'name': 'phoneNumber',
          'title': 'Enter Phone number',
          "pattern": phoneRegex
        };
        schema.profileSchema.properties.reenterPassword = {
          'type': 'password',
          'description': 'reenterPassword',
          'name': 'reenterPassword',
          'title': 'Reenter password',
          "allOf": schema.profileSchema.properties.password.allOf
        };
        schema.profileSchema.properties.dob = {
          'type': 'string',
          'description': 'Date of Birth',
          'name': 'dob',
          'title': 'Enter your date of birth'
        };
        schema.profileSchema.fieldOrder.splice(2, 0, 'reenterPassword');
        schema.profileSchema.fieldOrder.push('phoneNumber');
		schema.profileSchema.fieldOrder.push('dob');
		console.log("SCHEMA AFTER", schema);
        onSuccess(schema);
    },
      preSubmit: (postData, onSuccess, onFailure) => {
          /* handle preSubmit callback
        var dateofbirth = document.querySelectorAll("input");
        postData.fabricBirthDate = dateofbirth[5].value;
        postData.fabricAnniversaryDate = dateofbirth[6].value;   
        postData.fabricPrimaryContact = dateofbirth[7].value;
        postData.firstName = dateofbirth[3].value;*/
         console.log("postData",postData);
        
        if (postData.password != postData.reenterPassword) {
        	const error = {
            "errorSummary": "Password_Mismatch",
            "errorCauses": [
                    {
                      "errorSummary": "Password and reentered passsword should be same",
                      "property": "userProfile.password",
                    }
                ]
            }
            onFailure(error);
          	return;
        }
        
        if (!new RegExp(phoneRegex).test(postData.phoneNumber)) {
          const error = {
          "errorSummary": "Invalid_Phone_Number",
          "errorCauses": [
                  {
                    "errorSummary": "Phone number format is incorrect",
                    "property": "userProfile.phoneNumber",
                  }
              ]
          };
          onFailure(error);
          return;
        }
        
        onSuccess(postData);
      }
	}
      
      // Render the Okta Sign-In Widget
      var oktaSignIn = new OktaSignIn(config);
      config.issuer = "https://storefront.example.com/oauth2/default";
      const oktaAuth = new OktaAuth(config);
      function backButton() {
        if (window.location.href === "https://storefront.example.com/") {
          history.go(-3);
        } else if (
          window.location.href ===
          "https://storefront.example.com/signin/register-complete"
        ) {
          window.location.assign("https://storefront.example.com");
        } else {
          history.back();
        }
      }

      //Error Handling using afterError function

      oktaSignIn.on("afterError", function (context, error) {
        console.log("error", context.controller);
        // reset-password
        console.log("error name", error);
        // AuthApiError
        console.log("error msg", error.message);
        // The password does not meet the complexity requirements
        // of the current password policy.
        console.log(error.statusCode);

        if (context.controller === "primary-auth") {
          if (error.statusCode == 401) {
            SignInUnAuthanticatedError();
          }
        } else if (context.controller === "password-reset") {
          if (error.statusCode == 403) {
            resetUnAuthanticatedError();
          }
        } else if (context.controller === "registration") {
          if (error.message === "Password requirements were not met. ") {
            registrationUnAuthanticatedError();
          } else if (
            error.message === "A user with this Email already exists"
          ) {
            resetEmailUnAuthanticatedError();
          }
        }
      });
      // Identity the page using Context.controller function

      oktaSignIn.on("ready", function (context) {
        console.log("onready:", context.controller);

        /* 

        switch (context.controller) {
          case "primary-auth":
            signIn();
            break;
          case "registration":
            signUp();
            break;
          case "forgot-password":
            resetPassword();
            break;
          case "idp-discovery":
            discovery();
            break;
         case "recovery-loading":
            updatePassword();
            break;
          case "refresh-auth-state":
             updatePassword();
            break;
          default:
            console.log("controller not found");
        }*/
      });

      //AfterRender function for identity the page using Context.controller function

      oktaSignIn.on("afterRender", function (context) {
        console.log("AfterRender:", context.controller);
        /*if (context.controller === "password-reset") {
          updatePassword();
        } else if (context.controller === "primary-auth") {
          signIn();
        } else if (context.controller === "registration") {
          signUp();
        } else if (context.controller === "idp-discovery") {
          discovery();
        } else */
        if (context.controller === "forgot-password") {
          
       	var btn = document.querySelectorAll('a[data-se="email-button"]')[0];
          btn.addEventListener('click', function (e) {
            oktaAuth.forgotPassword({
            username: 'test@example.com',
            factorType: 'EMAIL',
          })
          .then(function(transaction) {
            console.log("RESP", transaction);
          })
          .catch(function(err) {
            console.error("ERR",err);
          });
            e.preventDefault();
            e.stopPropagation();
            // Custom link behavior
            console.log("STOPPING");
          });
          // resetPassword();
        }
        /* else if (context.controller === "password-reset-email-sent") {
          passwordResetEmailSent();
        } else if (context.controller === "registration-complete") {
          registrationComplete();
        }*/
      });
      // Render the Okta Sign-In Widget
     // var oktaSignIn = new OktaSignIn(config);
      oktaSignIn.renderEl({ el: '#okta-login-container' },
           function(res)   {
        		console.log("inside Success", res);
        		OktaUtil.completeLogin(res);
      	   },
           function(error) {
               // Logs errors that occur when configuring the widget.
               // Remove or replace this with your own custom error handler.
               console.log(error.message, error);
           }
       );
       
      var selectLang = document.getElementById("select-lang");
      selectLang.value = selectedLanguage;
      if (selectLang) {
          selectLang.onchange = function(event) {
              window.location.search = "?lang=" + event.srcElement.value;
          }
      }
    </script>
</body>
</html>
```