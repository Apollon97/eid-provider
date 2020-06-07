[![stability-stable](https://img.shields.io/badge/stability-prerelease-lightgrey.svg)](#)
[![version](https://img.shields.io/badge/version-0.0.3-green.svg)](#)
[![maintained](https://img.shields.io/maintenance/yes/2020.svg)](#)
[![maintainer](https://img.shields.io/badge/maintainer-daniel%20sörlöv-blue.svg)](https://github.com/DSorlov)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://img.shields.io/github/license/DSorlov/eid-provider)

# eid-provider
This module is developed to enable rapid deployment of identity based authentication for Node.js by creating a common interface to most of the suppliers for official electronic identification and it allows you to mix and match your suppliers. This is a reusability code port of code that I have contributed to [teams-app-eid](https://github.com/DennizSvens/teams-app-eid) with some smart addons and international support.

**Supported modules:**

| Module | Vendor | Authentication | Signing | Geographies | Readiness |
| --- | --- | --- | --- | --- | --- |
| bankid | BankID | :heavy_check_mark: | :heavy_check_mark: | :sweden: | Production ready |
| frejaeid | Verisec (Freja eID) | :heavy_check_mark: | :heavy_check_mark: | :sweden: :denmark: :norway: :finland: | Production ready |
| frejaorgid | Verisec (Freja eID) | :heavy_check_mark: | :heavy_check_mark: | :sweden: :denmark: :norway: :finland: | Production ready |
| ftbankid | Funktionstjänster (CGI) | :heavy_check_mark: | :heavy_check_mark: | :sweden: | Production ready |
| ftfrejaeid | Funktionstjänster (CGI) | :heavy_check_mark: | :heavy_check_mark: | :sweden: :denmark: :norway: :finland: | Not recomended* |
| gbankid | Svensk e-Identitet | :heavy_check_mark: | :heavy_check_mark: | :sweden: | Not recomended** |
| gfrejaeid | Svensk e-Identitet | :x: | :x: | :sweden: | Not recomended*** |

<sup>* The API key we have been supplied with does not allow for freja authentication so largely untested but complies with api.<br/>
** The GrandID keys we have got for testing have stopped working so have not been tested.<br/>
*** GrandID do not officially support Freja eID for silent logins. Using some ugly workarounds tbh, so not for production I think.
</sup>

## Examples
### Simple example
This is a very simple example of calling authentication via frejaeid for the ssn 200101011212 and when final results are in dump them out on the console.
```javascript
const  eidprovider = require('./eid-provider.js')('frejaeid');  
const  config = eidprovider.settings.testing;
eidprovider.initialize(config);

eidprovider.authRequest('200101011212').then(function(result){
	console.log(result);
});
```
### Simple example with objects as input
Now we are authenticating a norweigan user instead, also it shows of how to send objects as arguments to the functions.
```javascript
const  eidprovider = require('./eid-provider.js')('frejaeid');  
const  config = eidprovider.settings.testing;
eidprovider.initialize(config);

var user = {ssn:'14023526620',country='NO'}

eidprovider.authRequest(user).then(function(result){
	console.log(result);
});
```
### Another simple example in async function
This is a very simple example of calling authentication via bankid for the ssn 200101011212 and when final results are in dump them out on the console, however this time since we are in a async function we could simply use await if needed be.
```javascript
const  eidprovider = require('./eid-provider.js')('bankid');  
const  config = eidprovider.settings.testing;
eidprovider.initialize(config);

const result = await eidprovider.authRequest('200101011212');
console.log(result);
```
### Little more detailed example
This is a build on the simple example where we instead use bankid for the same ssn, the final results are still dumped on the console but we also dump the init and status update messages to the console via callbacks. 
```javascript
function handleInit(status) {
	console.log(status);
}

function handleUpdate(status) {
	console.log(status);
}

const  eidprovider = require('./eid-provider.js')('bankid');  
const  config = eidprovider.settings.testing;
eidprovider.initialize(config);

eidprovider.authRequest('200101011212'}, handleInit, handleUpdate).then(function(result){
	console.log(result);
});
```
### Handling the magic yourself
You can even handle the checks and stuff yourself instead of relying on the internals of the library. Could be useful for some scenarios where you perhaps check for signing in another app etc.. you need to handle loading config and stuff as normal...
```javascript
// This initiates a auth request and returns a id, poll it later to see results.
eidprovider.initAuthRequest('200101011212').then(function(result) {
 doMagicStuff(result);
}

[[..some other code..]]
// This polls for results, use pollSignStatus for a signing status
eidprovider.pollAuthStatus(idfrominit).then(function(result) {
 doEvenMoreMagicStuff(result);
}

[[..some other code..]]
// This ends a session forcefully
eidprovider.cancelAuth(idfrominit);
```
### Supplying custom configuration
It's simple to configure the modules. Just get the default objects and override what you need. Different modules accept different configuration options. Check out the settings object for each module below.
```javascript
const  eidprovider = require('./eid-provider.js')('frejaeid');  
const  config = eidprovider.settings.production;
config.client_cert = fs.readFileSync('supersecret.pfx');
config.password = 'mysupersecretpassword';
eidprovider.initialize(config);
```
## Modules
There are basically right now two main types of integrations: one is working directly with the service apis and the other kind is working with a broker service. The broker services can be usefull if you have many integrations or other sources in your enterprise and you wish to use the same sources for these.

The configuration options below should be quite obvious as what they do. If you are unsure your supplier will most probably be able to determine what information you need. Most modules have sane values, certificates etc for most testing services and production services however there is no production credentials and you need to strike an agreement with the services yourself to obtain these.

### BankID 
This module works directly with the BankID api. It is supplied with working testing credentials and basic production details.
>**Default production configuration (settings.production)**
```
endpoint:  'https://appapi2.bankid.com/rp/v5',
client_cert:  '',
ca_cert:  fs.readFileSync(`./certs/bankid_prod.ca`),
allowFingerprint:  true,
password:  ''
```
>**Default testing configuration (settings.testing)**
```
endpoint:  'https://appapi2.test.bankid.com/rp/v5',
client_cert:  fs.readFileSync('./certs/bankid_test.pfx'),
ca_cert:  fs.readFileSync(`./certs/bankid_test.ca`),
allowFingerprint:  true,
password:  'qwerty123'
```
### Freja eID
This module works directly with the Freja eID REST API. It is supplied with working testing credentials and basic production details.
>**Default production configuration (settings.production)**
```
endpoint:  'https://services.prod.frejaeid.com',
client_cert:  '',
ca_cert:  fs.readFileSync(`./certs/bankid_prod.ca`),
jwt_cert:  fs.readFileSync(`./certs/frejaeid_prod.jwt`),
minimumLevel:  'EXTENDED',
password:  '',
default_country: 'SE'
```
>**Default testing configuration (settings.testing)**
```
endpoint:  'https://services.test.frejaeid.com',
client_cert:  fs.readFileSync('./certs/frejaeid_test.pfx'),
ca_cert:  fs.readFileSync(`./certs/frejaeid_test.ca`),
jwt_cert:  fs.readFileSync(`./certs/frejaeid_test.jwt`),
minimumLevel:  'EXTENDED',
password:  'test',
default_country: 'SE'
```
### Freja OrgID
This module works directly with the Freja eID REST API for Organizational IDs. It is not supplied with any testing credentials. Contact Verisec AB.
>**Default production configuration (settings.production)**
```
endpoint:  'https://services.prod.frejaeid.com',
client_cert:  '',
ca_cert:  fs.readFileSync(`./certs/bankid_prod.ca`),
jwt_cert:  fs.readFileSync(`./certs/frejaeid_prod.jwt`),
password:  '',
default_country: 'SE'
```
>**Default testing configuration (settings.testing)**
```
endpoint:  'https://services.test.frejaeid.com',
client_cert:  '',
ca_cert:  fs.readFileSync(`./certs/frejaeid_test.ca`),
jwt_cert:  fs.readFileSync(`./certs/frejaeid_test.jwt`),
password:  '',
default_country: 'SE'
```
### Funktionstjänster (same for both the ftbankid and ftfrejaeid modules)
This module works by interfacing the Funktionstjänster service. It is supplied working testing credentials and basic production details.
>**Default production configuration (settings.production)**
```
endpoint:  'https://grp.funktionstjanster.se:18898/grp/v2?wsdl',
ca_cert:  fs.readFileSync(`./certs/ftbankid_prod.ca`),
display_name:  '',
policy:  ''

```
>**Default testing configuration (settings.testing)**
```
endpoint:  'https://grpt.funktionstjanster.se:18898/grp/v2?wsdl',
ca_cert:  fs.readFileSync(`./certs/ftbankid_test.ca`),
display_name:  'test',
policy:  'logtest020'
```
### GrandID (same for both the gbankid and gfrejaeid modules)
This module works by interfacing the GrandID service. It is supplied only with basic information. You need to obtain your own credentials.
>**Default production configuration (settings.production)**
```
endpoint: 'https://client.grandid.com/',
servicekey: '',
apikey: ''
```
>**Default testing configuration (settings.testing)**
```
endpoint: 'https://client.grandid.com/',
servicekey: '',
apikey: ''
```