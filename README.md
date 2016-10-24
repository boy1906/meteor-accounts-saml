Meteor-accounts-saml
==================

SAML v2 login support. This plugin can be used for login with existing password based accounts and for creating new ones. For security reasons you can sign the requests between your app and the SSO-Service.



-----


## Important Notes

* **This package is working but may have issues with various saml providers** - it has only been tested and verified with [OpenIDP](https://openidp.feide.no/) and [OpenAM](https://www.forgerock.org/openam).
* Most SAML IDPs don't allow SPs with a _localhost (127.0.0.1)_  address. Unless you run your own IDP (eg via your own OpenAM instance) you might exprience issues.
* The accounts-ui loggin buttons will not include saml providers, this may be implemented as a future enhancement, see below for how to build a custom login button.

## Usage

Put SAML settings in eg `server/lib/settings.js` like so:

```
let fs = Npm.require('fs');

settings = {"saml":[{
    "provider":"openam",
    "entryPoint":"https://openam.idp.io/openam/SSORedirect/metaAlias/zimt/idp",
    "issuer": "https://sp.zimt.io/", //replace with url of your app
    "cert":"MIICizCCAfQCCQCY8tKaMc0 LOTS OF FUNNY CHARS ==",
    "idpSLORedirectURL": "http://openam.idp.io/openam/IDPSloRedirect/metaAlias/zimt/idp",
     "privateKeyFile": "certs/mykey.pem",  // path is relative to $METEOR-PROJECT/private
     "publicCertFile": "certs/mycert.pem",  // eg $METEOR-PROJECT/private/certs/mycert.pem
     "samlFields": ["Mail","Vorname"], // which fields has to be served by the SSO provider
     "keyMeteorUser": "username", // identifier of user object, e.g. username - will be used for check with nameID
     "createNewUser": true, // it is allowed to create new users
     "identifierFormat":"urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified", // which format of nameID should we get from SSO
     "privateCert": fs.readFileSync('./assets/app/certs/server.crt', 'utf-8'),
     "privateKey": fs.readFileSync('./assets/app/certs/key.pem', 'utf-8'),
  }]}
  
Meteor.settings = settings;
```

in some template

```
<a href="#" class="saml-login" data-provider="openam">OpenAM</a>
```

in helper function

```
'click .saml-login': function(event, template){
    event.preventDefault();
    var provider = $(event.target).data('provider');
    Meteor.loginWithSaml({
	    provider:provider
	}, function(error, result){
		//handle errors and result
    });
  }
```

and if SingleLogout is needed

```
'click .saml-login': function(event, template){
    event.preventDefault();
    var provider = $(event.target).data('provider');
    Meteor.logoutWithSaml({
	    provider:provider
	}, function(error, result){
		//handle errors and result
    });
  }
```

## Setup SAML SP (Consumer)

1. Create a Meteor project by `meteor create sp` and cd into it.
2. Add `boy1906:meteor-accounts-saml`
3. Create `server/lib/settings.js` as described above. Since Meteor loads things in `server/lib` first, this ensures that your settings are respected even on Galaxy where you cannot use `meteor --settings`. 
4. Put your private key and your cert (not the IDP's one) into the "private" directory. Eg if your meteor project is at `/Users/steffo/sp` then place them in `/Users/steffo/sp/private`
5. Check if you can receive SP metadata eg via `curl http://localhost:3000/_saml/metadata/openam`. Output should look like:

```
<?xml version="1.0"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" entityID="http://localhost:3000/">
  <SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <KeyDescriptor>
      <ds:KeyInfo>
        <ds:X509Data>
          <ds:X509Certificate>MKA.... lot of funny chars ... gkqhkiG9w0BAQUFADATMREwDwYDVQQDEwgxMC4w
		  </ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
      <EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes256-cbc"/>
      <EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes128-cbc"/>
      <EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#tripledes-cbc"/>
    </KeyDescriptor>
    <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
    <AssertionConsumerService index="1" isDefault="true" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://localhost:3000/_saml/validate/openam"/>
  </SPSSODescriptor>
</EntityDescriptor>
  ```
## Generate Custom Certificate
```
openssl req -newkey rsa:2048 -new -nodes -keyout key.pem -out csr.pem
openssl x509 -req -in csr.pem -signkey key.pem -out server.crt
```

## Credits
Based Steffo's Meteor/SAML package steffo:meteor-accounts-saml which is
heavily derived from https://github.com/bergie/passport-saml.
