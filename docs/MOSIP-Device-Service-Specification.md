# MOSIP Device Specification

## Introduction & Background

### Objective
The objective of this specification document is to establish the technical and compliance standards/ protocols that are necessary for a biometric device to be used in MOSIP solutions.

### Target Audience
This is a biometric device specification document and aims to help the biometric device manufactures, their developers, and their designers in building MOSIP compliant devices.  It is assumed that the readers are familiar with MOSIP registration and authentication services.

### MOSIP Devices
All devices that collect biometric data for MOSIP should operate within the specification of this document.

## Revision History
|Version|State|Date|Changes|
---|---|---|---|
|0.9.2|Frozen|Aug-2019||
|0.9.3|Frozen|Feb-2020||
|0.9.5|Draft|13-Jun-2020||
|0.9.5|Draft|10-Aug-2020|Signature for API to retrieve encryption certificate has been changed from GET to POST and Device Stream now supports an optional parameter - timeout
|0.9.5|Draft|04-Dec-2020|In the header of JWT Signature, the key to store the type has been changed to "typ" from "type" as per JWT standards. Kindly view the digital id specification for the change.
|0.9.5|Draft|26-Feb-2021|Updated the [FTM criteria](#certification) to include PCI PED 2.0 and CC.
|0.9.5|Draft|24-Mar-2021|The reference to L2 devices has been removed from this document.<br>The biometric specification listed here has been moved to a new section [Biometric Specification](Biometric-Specification.md) and the old specification is now in [0.9.5 Biometric Specifications](0.9.5-Biometric-Specification.md) for reference.
|0.9.5|Draft|07-Apr-2021|Device De registration API spec was updated.
|0.9.5|Draft|08-Apr-2021|We will be following datetime values in ISO 8601 with format yyyy-mm-ddTHH:MM:ssZ. The same has been updated throughout the document.

## Glossary of Terms
* Device Provider - An entity that manufactures or imports the devices in their name. This entity should have legal rights to obtain an organization level digital certificate from the respective authority in the country.
* Foundational Trust Provider - An entity that manufactures the foundational trust module.
* Device - A hardware capable of capturing biometric information.
* L1 Certified Device / L1 Device - A device certified as capable of performing encryption in line with this spec in its trusted zone.
* L0 Certified Device / L0 Device - A device certified as one where the encryption is done on the host machine device driver or the MOSIP device service.
* Foundational Trust Provider Certificate - A digital certificate issued to the "Foundational Trust Provider". This certificate proves that the provider has successfully gone through the required Foundational Trust Provider evaluation. The entity is expected to keep this certificate in secure possession in an HSM. All the individual trust certificates are issued using this certificate as the root. This certificate would be issued  by the countries in conjunction with MOSIP.
* Device Provider Certificate - A digital certificate issued to the "Device Provider". This certificate proves that the provider has been certified for L0/L1 respective compliance. The entity is expected to keep this certificate in secure possession in an HSM. All the individual trust certificates are issued using this certificate as the root. This certificate is issued by the countries in conjunction with MOSIP.
* Registration - The process of applying for a Foundational Id.
* KYC - Know Your Customer. The process of providing consent to perform profile verification and update.
* Auth - The process of verifying one’s identity.
* FPS - Frames Per Second
* Management Server - A server run by the device provider to manage the life cycle of the biometric devices.
* Device Registration - The process of registering the device with MOSIP servers.
* Signature - All signature should be as per RFC 7515.
* header in signature - Header in signature means the attribute with "alg" set to RS256 and x5c set to base64encoded certificate.
* payload is the byte array of the actual data, always represented as base64urlencoded.
* signature - base64urlencoded signature bytes
* ISO format timestamp - ISO 8601 with format yyyy-mm-ddTHH:MM:ssZ (Example: 2020-12-08T09:39:37Z)

---

## Device Specification
The MOSIP device specification provides compliance guidelines to devices for them to work with MOSIP. The compliance is based on device capability, trust and communication protocols. A MOSIP compliant device would follow the standards established in this document. It is expected that the devices are compliant to this specification and tested and validated. The details of each of these are outlined in the subsequent sections.

### Device Capability
The MOSIP compliant device is expected to perform the following,
* Should have the ability to collect one or more biometric
* Should have the ability to sign the captured biometric image or template.
* Should have the ability to protect secret keys
* Should have no mechanism to inject the biometric

### Base Specifications for Devices
For details about biometric data specifications please view the page [MOSIP Biometric Specification](Biometric-Specification.md).

We recommend that countries look at ergonomics, accessibility, ease of usage, and common availability of devices while choosing devices for use in registration and authentication scenarios.

---

## Device Trust
MOSIP compliant devices provide a trust environment for the devices to be used in registration, KYC and AUTH scenarios. The trust level is established based on the device support for trusted execution.

* L1 - The trust is provided by a secure chip with secure execution environment.
* L0 - The trust is provided at the software level. No hardware related trust exist. This type of compliance is used in controlled environments.

### Foundational Trust Module (FTM)
The foundational trust module would be created using a secure microprocessor capable of performing all required biometric processing and secure storage of keys. The foundational device trust would satisfy the below requirements.

* The module has the ability to securely generate, store and process cryptographic keys.
* Generation of asymmetric keys and symmetric keys with TRNG.
* The module has the ability to protect keys from extraction.
* The module has to protect the keys from physical tampering, temperature, frequency and voltage related attacks.
* The module could withstand against Hardware cloning.
* The module could withstand probing attacks
* The module provides memory segregation for cryptographic operations and protection against buffer over flow attacks
* The module provides ability to withstand cryptographic side channel attacks like Differential Power analysis attacks, Timing attacks.
* CAVP validated implementation of the cryptographic algorithm.
* The module has the ability to perform a cryptographically validatable secure boot.
* The module has the ability to run trusted applications.

The foundational device trust derived from this module is used to enable trust-based computing for biometric capture. The foundational device trust module provides for a trusted execution environment based on the following:

* Secure Boot
    * Ability to cryptographically verify code before execution.
    * Ability to check for integrity violation of the module/device.
    * Halt upon failure.
    * Ability to securely upgrade and perform forward only upgrades, to thwart downgrade attacks.
    * SHA256 hash equivalent or above should be used for all hashing requirements
    * All root of trust is provisioned upon first boot or before.
    * All upgrades would be considered success only after the successful boot with proper hash and signature verification.
    * The boot should fail upon hash/signature failures and would never operate in a intermediary state.
    * Maximum of 10 failed attempts should lock the upgrade process and brick the device. However chip manufactures can decide to be less than 10.
* Secure application
    * Ability to run applications that are trusted.
    * Protect against downgrading of applications.
    * Isolated memory to support cryptographic operations.
    * All trust are anchored during the first boot and not modifiable.

#### Certification
The FTM should have a at least one of the following certifications in each category to meet the given requirement.

##### Category: Cryptographic Algorithm Implementation
* CAVP (RSA, AES, SHA256, TRNG (DRBGVS), ECC)

{% hint style="info" %}
The supported algorithm and curves are listed [here](#cryptography)
{% endhint %}

##### Category: FTM Chip
(ONE of the following certifications)
* FIPS 140-2 L3 or above
* PCI PTS 5 or above (Pre-certified)
* PCI - PED 2.0 or above (Pre-Certified)
* One of following Common Criteria (CC) certification
	* https://www.commoncriteriaportal.org/files/ppfiles/pp0035a.pdf
	* https://www.commoncriteriaportal.org/files/ppfiles/pp0084a_pdf.pdf

##### Category: Tamper
* For L1 level compliance the FTM should support tamper evidence. Tamper responsiveness is recommended for the whole device/system.

#### Threats to Protect
The FTM should protect against the following threats.

* Hardware cloning attacks - Ability to protect against attacks that could result in a duplicate with keys.
* Hardware Tamper attacks
    * Physical tamper - No way to physically tamper and obtain it secrets. 
    * Voltage & frequency related attacks - Should shield against voltage leaks and should prevent against low voltage. The FTM should always be in either of the state operational normally or inoperable. The FTM should never be operable when its input voltages are not met.
    * Temperature attacks on crypto block - Low or High the FTM is expected to operate or reach inoperable state. No state in between.
* Differential Power Analysis attack.
* Probing attacks - FTM should protect its surface area against any probe related attacks.
* Segregation of memory for execution of cryptographic operation (crypto block should be protected from buffer overflow type attacks).
* Vulnerability of the cryptographic algorithm implementation.
* Attacks against secure boot & secure upgrade.
* TEE/Secure processor OS attack.

#### Foundational Trust Module Identity
Upon an FTM provider approved by the MOSIP adopters, the FTM provider would submit a self signed public certificate to the adopter. Let us call this as the FTM root. The adopter would use this certificate to seed their device trust database. The FTM root and their key pairs should be generated and stored in FIPS 140-2 Level 3 or more compliant devices with no possible mechanism to extract the keys.  The foundational module upon its first boot is expected to generate a random asymmetric key pair and provide the public part of the key to obtain a valid certificate.  The FTM provider would validate to ensure that the chip is unique and would issue a certificate with the issuer set to FTM root.  The entire certificate issuance would be in a secured provisioning facility. Auditable upon notice by the adopters or its approved auditors.  The certificate issued to the module will have a defined validity period as per the MOSIP certificate policy document defined by the MOSIP adopters. This certificate and private key within the FTM chip is expected to be in it permanent memory.

{% hint style="info" %}
The validity for the chip certificate can not exceed 20 years from the date of manufacturing.
{% endhint %}

### Device
Mosip devices are most often used to collect biometrics. The devices are expected to follow the specification for all level of compliance and its usage. The mosip devices fall under the category of Trust Level 3 (TL3) as defined in MOSIP architecture. At TL3 device is expected to be whitelisted with a fully capable PKI and secure storage of keys at the hardware.

* L0 - A device can obtain L0 certification when it uses software level cryptographic library with no secure boot or FTM.  These devices will follow different device identity and the same would be mentioned as part of exception flows.
* L1 - A device can obtain L1 certification when its built in secure facility with one of the certified FTM.

#### Device Identity
It is imperative that all devices that connect to MOSIP are identifiable. MOSIP believes in cryptographic Identity as its basis for trust.

##### Physical ID
An identification mark that shows MOSIP compliance and a readable unique device serial number (minimum of 12 digits), make and model. The same information has to be available over a 2D QR Code or Barcode. This is to help field support and validation.

##### Digital ID
A digital device ID in MOSIP would be a signed JSON (RFC 7515) as follows:
```JSON
{
  "serialNo": "Serial number",
  "make": "Make of the device",
  "model": "Model of the device",
  "type": "Type of the biometric device",
  "deviceSubType": "Subtypes of the biometric device",
  "deviceProvider": "Device provider name",
  "deviceProviderId": "Device provider id",
  "dateTime": "Current datetime in ISO format"
}
```

Signed with the JSON Web Signature (RFC 7515) using the "Foundational Trust Module" Identity key, this data is the fundamental identity of the device. Every MOSIP compliant device will need the foundational trust module.

The only exception to this rule is for the L0 compliant devices that have the purpose (explained below during device registration) as "Registration". L0 devices would sign the Digital Id with the device key.

Signed Digital ID would look as follows:
```
"digitalId": "base64urlencoded(header).base64urlencoded(payload).base64urlencoded(signature)"
```

The header in the digital id would have:
```
"alg": "RS256",
"typ": "JWT",
"x5c": "<Certificate of the FTM chip, If in case the chain of certificates are sent then the same will be ignored">
```

MOSIP assumes that the first certificate in the x5c is the FTM's chip public certificate issued by the FTM root certificate.

Unsigned digital ID would look as follows:
```
"digitalId": "base64urlencoded(payload)"
```
Payload is the Digital ID JSON object.

{% hint style="info" %}
For a L0 unregistered device the digital id will be unsigned. In all other scenarios, except for a discovery call, the digital ID will be signed either by the chip key (L1) or the device key (L0).
{% endhint %}

##### Accepted Values for Digital ID
Parameters | Description
-----------|-------------
serialNo | <ul><li>Serial number of the device.</li><li>This value should be same as printed on the device (Refer [Physical ID](#physical-id)).</li></ul>
make | <ul><li>Brand name.</li><li>This value should be same as printed on the device (Refer [Physical ID](#physical-id)).</li><ul>
model | <ul><li>Model of the device.</li><li>This value should be same as printed on the device (Refer [Physical ID](#physical-id)).</li></ul>
type | <ul><li>Currently allowed values for device type are "Finger", "Iris" or "Face".</li><li>More types can be added based on Adopter's implementation.</li></ul>
deviceSubType | <ul><li>Device Sub type is based on the device type.</li><li>For Finger - "Slap", "Single" or "Touchless"</li><li>For Iris - "Single" or "Double"</li><li>For Face - "Full face"</li></ul>
deviceProvider | <ul><li>Name of the device provider.</li><li>Device provider should be a legal entity in the country.</li></ul>
dateTime | <ul><li>Current time during the issuance of the request.</li><li>This is in ISO format.</li></ul>

### Keys
List of keys used in the device and their explanation.

* **Device Key**

Each biometric device would contain an authorized private key after the device registration. This key is rotated frequently based on the requirement from the respective adopter of MOSIP. By default MOSIP recommends 30 days key rotation policy for the device key. The device keys are created by the device providers inside the FTM during a successful registration.  The device keys are used for signing the biometric. More details of the signing and its usage will be [here](#device-service-communication-interfaces).  This key is issued by the device provider and the certificate of the device key is issued by the device provider key which in turn is issued by the MOSIP adopter after approval of the device providers specific model.

* **FTM Key**

The FTM key is the root of the identity. This key is created by the FTM provider during the manufacturing/provisioning stage. This is a permanent key and would never be rotated. This key is used to sign the Digital ID.

* **MOSIP Key**

The MOSIP key is the public key provided by the MOSIP adopter. This key is used to encrypt the biometric biometric. Details of the encryption is listed below. We recommend to rotate this key every 1 year.

## Device Service - Communication Interfaces
The section explains the necessary details of the biometric device connectivity, accessibility, discover-ability and protocols used to build and communicate with the device.

The device should implement only the following set of APIs.  All the API’s are independent of the physical layer and the operating system, with the invocation being different across operating systems. While the operating system names are defined in this spec a similar technology can be used for unspecified operating systems.  It is expected that the device service ensures that the device is connected  locally to the host.

### Device Discovery
Device discovery would be used to identify MOSIP compliant devices in a system by the applications. The protocol is designed as simple plug and play with all the necessary abstraction to the specifics.

#### Device Discovery Request
```JSON
{
  "type": "type of the device"
}
```

#### Accepted Values for Device Discovery Request
* type - "Biometric Device", "Finger", "Face", "Iris"

{% hint style="info" %}
"Biometric Device" - is a special type and used in case if you are looking for any biometric device.
{% endhint %}

#### Device Discovery Response
```JSON
[
  {
    "deviceId": "Internal ID",
    "deviceStatus": "Device status",
    "certification": "Certification level",
    "serviceVersion": "Device service version",
    "deviceSubId": ["Array of supported device sub Ids"],
    "callbackId": "Base URL to reach to the device",
    "digitalId": "Unsigned Digital ID of the device",
    "deviceCode": "A unique code given by MOSIP after successful registration",
    "specVersion": ["Array of supported MDS specification version"],
    "purpose": "Auth  or Registration or empty if not registered",
    "error": {
      "errorCode": "101",
      "errorInfo": "Invalid JSON Value Type For Discovery.."
    }
  },
  ...
]
```

#### Accepted Values for Device Discovery Response
Parameters | Description
-----------|-------------
deviceStatus | Allowed values are "Ready", "Busy", "Not Ready" or "Not Registered".
certification | Allowed values are "L0" or "L1" based on level of certification.
serviceVersion | Version of the MDS specification that is supported.
deviceId | Internal ID to identify the actual biometric device within the device service.
deviceSubId | <ul><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
callbackId | <ul><li>This differs as per the OS.</li><li>In case of Linux and windows operating systems it is a HTTP URL.</li><li>In the case of android, it is the intent name.</li><li>In IOS, it is the URL scheme.</li><li>The call back URL takes precedence over future request as a base URL.</li></ul>
digitalId | Digital ID as per the Digital ID definition but it will not be signed.
deviceCode | A unique code given by MOSIP after successful registration.
specVersion | Array of supported MDS specification version.
purpose | Purpose of the device in the MOSIP ecosystem. Allowed values are "Auth" or "Registration".
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code defined in the [error code section](#error-codes).
error.errorInfo | Description of the error that can be displayed to end user. Multi lingual support.

{% hint style="info" %}
* The response is an array that we could have a single device enumerating with multiple biometric options.
* The service should ensure to respond only if the type parameter matches the type of device or the type parameter is a "Biometric Device".
* This response is a direct JSON as show in the response.
{% endhint %}

#### Windows/Linux
All the device API will be based on the HTTP specification. The device always binds to with any of the available ports ranging from 4501 - 4600. The IP address used for binding has to be 127.0.0.1 and not localhost.

The applications that require access to MOSIP devices could discover them by sending the HTTP request to the supported port range. We will call this port as the device_service_port in the rest of the document.

**_HTTP Request:_**
```
MOSIPDISC http://127.0.0.1:<device_service_port>/device
HOST: 127.0.0.1: <device_service_port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:http://127.0.0.1:<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
* The pay loads are JSON in both the cases and are part of the body.
* CallbackId would be set to the `http://127.0.0.1:<device_service_port>`. So, the caller will use the respective HTTP verb / method and the URL to call the service.
{% endhint %}

#### Android
All devices on an android device should listen to the following intent "io.mosip.device".

Upon invocation of this intent the devices are expected to respond back with the json response filtered by the respective type.

{% hint style="info" %}
In Android, the CallbackId would be set to the appId. So, the caller will create the intent "appId.Info" or "appId.Capture".
{% endhint %}

#### IOS
All device on an IOS device would respond to the URL schema as follows:
```
MOSIPDISC://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the caller using the call-back-app-url with the base64 encoded json as the URL parameter for the key data.

{% hint style="info" %}
* In IOS there are restrictions to have multiple apps registering to the same URL schema.
* CallbackId would be set to the device service appname. So, the caller has to call appnameInfo or appnameCapture as the URL scheme.
{% endhint %}

### Device Info
The device information API would be used to identify the MOSIP compliant devices and their status by the applications.

#### Device Info Request
NA

#### Accepted Values for Device Info Request
NA

#### Device Info Response
```
[
  {
    "deviceInfo": {
      "deviceStatus": "Current status",
      "deviceId": "Internal ID",
      "firmware": "Firmware version",
      "certification": "Certification level",
      "serviceVersion": "Device service version",
      "deviceSubId": ["Array of supported device sub Ids"],
      "callbackId": "Baseurl to reach to the device",
      "digitalId": "Signed digital id as described in the digital id section of this document.",
      "deviceCode": "A unique code given by MOSIP after successful registration",
      "env": "Target environment",
      "purpose": "Auth  or Registration",
      "specVersion": ["Array of supported MDS specification version"],
    },
    "error": {
      "errorCode": "101",
      "errorInfo": "Invalid JSON Value "
    }
  }
  ...
]
```

The final JSON is signed with the JSON web signature using the "Foundational Trust Module" Identity key, this data is the fundamental identity of the device. Every MOSIP compliant device will need the foundational trust module.

So the API would respond in the following format.
```
[
  {
    "deviceInfo": "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
    "error": {
      "errorCode": "100",
      "errorInfo": "Device not registered. In this case the device info will be only base64urlencode(payload)"
    }
  }
]
```

#### Allowed values for Device Info Response
Parameters | Description
-----------|-------------
deviceInfo | <ul><li>The deviceInfo object is sent as JSON Web Token (JWT).</li><li>For device which is not registered, the deviceInfo will be unsigned.</li><li>For device which is registered, the deviceInfo will be signed using the device key.</li></ul>
deviceInfo.deviceStatus | <ul><li>This is the status of the device.</li><li>Allowed values are "Ready", "Busy", "Not Ready" or "Not Registered".</li></ul>
deviceInfo.deviceId | Internal Id to identify the actual biometric device within the device service.
deviceInfo.firmware | <ul><li>Exact version of the firmware.</li><li>In case of L0 this is same as serviceVersion.</li></ul>
deviceInfo.certification | <ul><li>Allowed values are "L0" or "L1" based on the level of certification.</li></ul>
deviceInfo.serviceVersion | Version of the MDS specification that is supported.
deviceInfo.deviceId | Internal ID to identify the actual biometric device within the device service.
deviceSubId | <ul><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
deviceInfo.callbackId | <ul><li>This differs as per the OS.</li><li>In case of Linux and windows operating systems it is a HTTP URL.</li><li>In the case of android, it is the intent name.</li><li>In IOS, it is the URL scheme.</li><li>The call back URL takes precedence over future request as a base URL.</li></ul>
deviceInfo.digitalId | <ul><li>The digital id as per the digital id definition.</li><li>For L0 devices which is not registered, the digital id will be unsigned.</li><li>For L0 devices which is registered, the digital id will be signed using the device key.</li><li>For L1 devices, the digital id will be signed using the FTM key.</li></ul>
deviceInfo.env | <ul><li>The target enviornment.</li><li>For devices that are not registered the enviornment is "None".</li><li>For device that is registered, then send the enviornment in which it is registered.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>
deviceInfo.purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>For devices that are not registered the purpose is empty.</li><li>Allowed values are "Auth" or "Registration".</li></ul>
deviceInfo.specVersion | Array of supported MDS specification version.
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code defined in the [error code section](#error-codes).
error.errorInfo | Description of the error that can be displayed to end user. Multi lingual support.

{% hint style="info" %}
* The response is an array that we could have a single device enumerating with multiple biometric options.
* The service should ensure to respond only if the type parameter matches the type of device or the type parameter is a "Biometric Device".
{% endhint %}

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range. The device always binds to with any of the available ports ranging from 4501 - 4600. The IP address used for binding has to be 127.0.0.1 and not localhost.

**_HTTP Request:_**
```
MOSIPDINFO http://127.0.0.1:<device_service_port>/info
HOST: 127.0.0.1:<device_service_port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:http://127.0.0.1:<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
The pay loads are JSON in both the cases and are part of the body.
{% endhint %}

#### Android
On an android device should listen to the following intent "appId.Info".

Upon invocation of this intent the devices are expected to respond back with the JSON response filtered by the respective type.

#### IOS
On an IOS device would respond to the URL schema as follows:
```
APPIDINFO://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the called using the call-back-app-url with the base64 encoded JSON as the URL parameter for the key data.

{% hint style="info" %}
In IOS there are restrictions to have multiple app registering to the same URL schema.
{% endhint %}

### Capture
The capture request would be used to capture a biometric from MOSIP compliant devices by the applications. The capture call will respond with success to only one call at a time. So, in case of a parallel call the device info details are sent with status as "Busy".

#### Capture Request
```
{
  "env": "Target environment",
  "purpose": "Auth  or Registration",
  "specVersion": "Expected version of the MDS spec",
  "timeout" : "Timeout for capture",
  "captureTime": "Capture request time in ISO format",
  "domainUri": "URI of the auth server",
  "transactionId": "Transaction Id for the current capture",
  "bio": [
    {
      "type": "Type of the biometric data",
      "count":  "Finger/Iris count, in case of face max is set to 1",
      "bioSubType": ["Array of subtypes"],
      "requestedScore": "Expected quality score that should match to complete a successful capture",
      "deviceId": "Internal Id",
      "deviceSubId": "Specific Device Sub Id",
      "previousHash": "Hash of the previous block"
    }
  ],
  customOpts: {
    //Max of 50 key value pair. This is so that vendor specific parameters can be sent if necessary. The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications. Vendors are free to include additional parameters and fine-tuning parameters. None of these values should go undocumented by the vendor. No sensitive data should be available in the customOpts.
  }
}
```

{% hint style="info" %}
Count value should be driven by the count of the bioSubType for Iris and Finger. For Face, there will be no bioSubType but the count should be "1".
{% endhint %}

#### Allowed Values for Capture Request
Parameters | Description
-----------|-------------
env | <ul><li>The target environment.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>
purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>For devices that are not registered the purpose is empty.</li><li>Allowed values are "Auth" or "Registration".</li></ul>
specVersion | Expected version of MDS specification.
timeout | <ul><li>Max time the app will wait for the capture.</li><li>Its expected that the API will respond back before timeout with the best frame.</li><li>All timeouts are in milliseconds.</li></ul>
captureTime | <ul><li>Time of capture in ISO format.</li><li>The time is as per the requesting application.</li></ul>
domainUri | <ul><li>URI of the authentication server.</li><li>This can be used to federate across multiple providers or countries or unions.</li></ul>
transactionId | <ul><li>Unique Id of the transaction.</li><li>This is an internal Id to the application thats providing the service.</li><li>Different id should be used for every successful auth.</li><li>So even if the transaction fails after auth we expect this number to be unique.</li></ul>
bio.type | Allowed values are "Finger", "Iris" or "Face".
bio.count | <ul><li>Number of biometric data that is collected for a given type.</li><li>The device should validate and ensure that this number is in line with the type of biometric that's captured.</li></ul>
bio.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
bio.requestedScore | Upon reaching the quality score the biometric device is expected to auto capture the image.
bio.deviceId | Internal Id to identify the actual biometric device within the device service.
bio.deviceSubId | <ul><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
bio.previousHash | <ul><li>For the first capture the previousHash is hash of empty UTF-8 string.</li><li>From the second capture the previous captures hash (as hex encoded) is used as input.</li><li>This is used to chain all the captures across modalities so all captures have happened for the same transaction and during the same time period.</li></ul>
customOpts | <ul><li>In case, the device vendor wants to send additional parameters they can use this to send key value pair if necessary.</li><li>The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications.</li><li>Vendors are free to include additional parameters and fine-tuning the process.</li><li>None of these values should go undocumented by the vendor.</li><li>No sensitive data should be available in the customOpts.</li></ul>

#### Capture Response
```
{
  "biometrics": [
    {
      "specVersion": "MDS spec version",
      "data": {
        "digitalId": "digital Id as described in this document",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "MDS version",
        "bioType": "Finger",
        "bioSubType": "UNKNOWN",
        "purpose": "Auth  or Registration",
        "env": "Target environment",
        "domainUri": "URI of the auth server",
        "bioValue": "Encrypted with session key and base64urlencoded biometric data",
        "transactionId": "Unique transaction id",
        "timestamp": "Capture datetime in ISO format",
        "requestedScore": "Floating point number to represent the minimum required score for the capture",
        "qualityScore": "Floating point number representing the score for the current capture"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "sessionKey": "encrypted with MOSIP public key (dynamically selected based on the uri) and encoded session key biometric",
      "thumbprint": "SHA256 representation of thumbprint of the certificate that was used for encryption of session key. All texts to be treated as uppercase without any spaces or hyphens",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value"
      }
    },
    {
      "specVersion" : "MDS spec version",
      "data": {
        "digitalId": "Digital Id as described in this document",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "MDS version",
        "bioType": "Finger",
        "bioSubType": "Left IndexFinger",
        "purpose": "Auth  or Registration",
        "env": "target environment",
        "domainUri": "uri of the auth server",
        "bioValue": "encrypted with session key and base64urlencoded biometric data",
        "transactionId": "unique transaction id",
        "timestamp": "Capture datetime in ISO format",
        "requestedScore": "Floating point number to represent the minimum required score for the capture",
        "qualityScore": "Floating point number representing the score for the current capture"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "sessionKey": "encrypted with MOSIP public key and encoded session key biometric",
      "thumbprint": "SHA256 representation of thumbprint of the certificate that was used for encryption of session key. All texts to be treated as uppercase without any spaces or hyphens",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value"
      }
    }
  ]
}
```

#### Accepted Values for Capture Response
Parameters | Description
-----------|------------- 
specVersion | Version of the MDS specification using which the response was generated.
data | <ul><li>The data object is sent as JSON Web Token (JWT).</li><li>The data block will be signed using the device key.</li></ul>
data.digitalId | <ul><li>The digital id as per the digital id definition in JWT format.</li><li>For L0 devices, the digital id will be signed using the device key.</li><li>For L1 devices, the digital id will be signed using the FTM key.</li></ul>
data.deviceCode | A unique code given by MOSIP after successful registration
data.deviceServiceVersion | MDS version
data.bioType | Allowed values are "Finger", "Iris" or "Face".
data.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
data.purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>Allowed values is "Auth".</li></ul>
data.env | <ul><li>The target environment.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>
data.domainUri | <ul><li>URI of the authentication server.</li><li>This can be used to federate across multiple providers or countries or unions.</li></ul>
data.bioValue | <ul><li>Biometric data is encrypted with random symmetric (AES GCM) key and base-64-URL encoded.</li><li>For symmetric key encryption of bioValue, (biometrics.data.timestamp XOR transactoinId) is computed and the last 16 bytes and the last 12  bytes of the results are set as the aad and the IV(salt) respectively.</li><li>Look at the Authentication document to understand more about the encryption.</li></ul>
data.transactionId | Unique transaction id sent in request
data.timestamp | <ul><li>Time as per the biometric device.</li><li>Note: The biometric device is expected to sync its time from the management server at regular intervals so accurate time could be maintained on the device.</li></ul>
data.requestedScore | Floating point number to represent the minimum required score for the capture.
data.qualityScore | Floating point number representing the score for the current capture.
hash | The value of the previousHash attribute in the request object or the value of hash attribute of the previous data block (used to chain every single data block) concatenated with the hex encode sha256 hash of the current data block before encryption.
sessionKey | The session key (used for the encrypting of the bioValue) is encrypted using the MOSIP public certificate with RSA/ECB/OAEPWITHSHA-256ANDMGF1PADDING algorithm and then base64-URL-encoded.
thumbprint | <ul><li>SHA256 representation of thumbprint of the certificate that was used for encryption of session key.</li><li>All texts to be treated as uppercase without any spaces or hyphens.</li></ul>
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code defined in the [error code section](#error-codes).
error.errorInfo | Description of the error that can be displayed to end user. Multi lingual support.

The entire data object is sent in JWT format. So, the data object will look like:
```
"data" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
payload - is defined as the entire byte array of data block.
```

#### Windows/Linux
The applications that requires to capture biometric data from a MOSIP devices could do so by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
CAPTURE [http://127.0.0.1:<device_service_port>/capture](http://127.0.0.1/capture)
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:[http://127.0.0.1](http://127.0.0.1):<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
The pay loads are JSON in both the cases and are part of the body.
{% endhint %}

#### Android
All device on an android device should listen to the following intent appid.capture.  Upon this intend the devices are expected to respond back with the JSON response filtered by the respective type.

#### IOS
All device on an IOS device would respond to the URL schema as follows.
```
APPIDCAPTURE://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the called using the call-back-app-url with the base64 encoded json as the URL parameter for the key data.

### Device Stream
The device would open a stream channel to send the live video streams. This would help when there is an assisted operation to collect biometric. Please note the stream API’s are available only for registration environment.

Used only for the registration module compatible devices. This API is visible only for the devices that are registered for the purpose as "Registration".

#### Device Stream Request
```
{
  "deviceId": "Internal Id",
  "deviceSubId": "Specific device sub Id",
  "timeout": "Timeout for stream"
}
```

#### Allowed Values for device Stream Request
Parameters | Description
-----------|--------------
deviceId | Internal Id to identify the actual biometric device within the device service.
deviceSubId | <ul><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
timeout | <ul><li>Max time after which the stream should close.</li><li>This is an optional paramter and by default the value will be 5 minutes.</li><li>All timeouts are in milliseconds.</li></ul>

#### Device Stream Response
Live Video stream with quality of 3 frames per second or more using [M-JPEG](https://en.wikipedia.org/wiki/Motion_JPEG).

{% hint style="info" %}
Preview should have the quality markings and segment marking. The preview would also be used to display any error message to the user screen. All error messages should be localized.
{% endhint %}

#### Error Response for Device Stream
```
{
  "error": {
    "errorCode": "202",
    "errorInfo": "No Device Connected."
  }
}
```

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
STREAM   http://127.0.0.1:<device_service_port>/stream
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
HTTP Chunk of frames to be displayed. Minimum frames 3 per second.

#### Android
No support for streaming

#### IOS
No support for streaming

### Registration Capture
The registration client application will discover the device. Once the device is discovered the status of the device is obtained with the device info API. During the registration the registration client sends the RCAPTURE API and the response will provide the actual biometric data in a digitally signed non encrypted form. When the Device Registration Capture API is called the frames should not be added to the stream. The device is expected to send the images in ISO format.

The requestedScore is in the scale of 1-100. So, in cases where you have four fingers the average of all will be considered for capture threshold. The device would always send the best frame during the capture time even if the requested score is not met.

The API is used by the devices that are compatible for the registration module. This API should not be supported by the devices that are compatible for authentication.

#### Registration Capture Request
```
{
  "env":  "Target environment",
  "purpose": "Auth  or Registration",
  "specVersion": "Expected MDS spec version",
  "timeout": "Timeout for registration capture",
  "captureTime": "Time of capture request in ISO format",
  "transactionId": "Transaction Id for the current capture",
  "bio": [
    {
      "type": "Type of the biometric data",
      "count":  "Finger/Iris count, in case of face max is set to 1",
      "bioSubType": ["Array of subtypes"], //Optional
      "exception": ["Finger or Iris to be excluded"],
      "requestedScore": "Expected quality score that should match to complete a successful capture.",
      "deviceId": "Internal Id",
      "deviceSubId": "Specific device Id",
      "previousHash": "Hash of the previous block"
    }
  ],
  customOpts: {
    //max of 50 key value pair. This is so that vendor specific parameters can be sent if necessary. The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications. Vendors are free to include additional parameters and fine-tuning parameters. None of these values should go undocumented by the vendor. No sensitive data should be available in the customOpts.
  }
}
```

#### Accepted Values for Registration Capture Request
Parameters | Description
-----------|-------------
env | <ul><li>The target environment.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>
purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>For devices that are not registered the purpose is empty.</li><li>Allowed values are "Auth" or "Registration".</li></ul>
specVersion | Expected version of MDS specification.
timeout | <ul><li>Max time the app will wait for the capture.</li><li>Its expected that the API will respond back before timeout with the best frame.</li><li>All timeouts are in milliseconds.</li></ul>
captureTime | <ul><li>Time of capture in ISO format.</li><li>The time is as per the requesting application.</li></ul>
transactionId | <ul><li>Unique Id of the transaction.</li><li>This is an internal Id to the application thats providing the service.</li><li>Different id should be used for every successful auth.</li><li>So even if the transaction fails after auth we expect this number to be unique.</li></ul>
bio.type | Allowed values are "Finger", "Iris" or "Face".
bio.count | <ul><li>Number of biometric data that is collected for a given type.</li><li>The device should validate and ensure that this number is in line with the type of biometric that's captured.</li></ul>
bio.bioSubType | <ul><li>Array of bioSubType for respective biometric type.</li><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li><li>This is an optional parameter.</li></ul>
bio.exception | <ul><li>This is an array and all the exceptions are marked.</li><li>In case exceptions are sent for face then follow the exception photo specification above.</li><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb"]</li><li>For Iris: ["Left", "Right"]</li></ul>
bio.requestedScore | Upon reaching the quality score the biometric device is expected to auto capture the image.
bio.deviceId | Internal Id to identify the actual biometric device within the device service.
bio.deviceSubId | <ul><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
bio.previousHash | <ul><li>For the first capture the previousHash is hash of empty UTF-8 string.</li><li>From the second capture the previous captures hash (as hex encoded) is used as input.</li><li>This is used to chain all the captures across modalities so all captures have happened for the same transaction and during the same time period.</li></ul>
customOpts | <ul><li>In case, the device vendor wants to send additional parameters they can use this to send key value pair if necessary.</li><li>The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications.</li><li>Vendors are free to include additional parameters and fine-tuning the process.</li><li>None of these values should go undocumented by the vendor.</li><li>No sensitive data should be available in the customOpts.</li></ul>

#### Registration Capture Response
```
{
  "biometrics": [
    {
      "specVersion": "MDS Spec version",
      "data": {
        "digitalId": "Digital id of the device as per the Digital Id definition..",
        "bioType": "Biometric type",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "MDS version",
        "bioSubType": "Left IndexFinger",
        "purpose": "Auth  or Registration",
        "env": "Target environment",
        "bioValue": "base64urlencoded biometrics (ISO format)",
        "transactionId": "Unique transaction id sent in request",
        "timestamp": "2019-02-15T10:01:57.086Z",
        "requestedScore": "Floating point number to represent the minimum required score for the capture. This ranges from 0-100.",
        "qualityScore": "Floating point number representing the score for the current capture. This ranges from 0-100."
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block)",    
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value Type For Discovery.. ex: {type: 'Biometric Device' or 'Finger' or 'Face' or 'Iris' } "
      }
    },
    {
      "specVersion" : "MDS Spec version",
      "data": {
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "bioType" : "Finger",
        "digitalId": "Digital id of the device as per the Digital Id definition.",
        "deviceServiceVersion": "MDS version",
        "bioSubType": "Left MiddleFinger",
        "purpose": "Auth  or Registration",
        "env":  "Target environment",
        "bioValue": "base64urlencoded extracted biometric (ISO format)",
        "transactionId": "Unique transaction id sent in request",
        "timestamp": "2019-02-15T10:01:57.086Z",
        "requestedScore": "Floating point number to represent the minimum required score for the capture. This ranges from 0-100",
        "qualityScore": "Floating point number representing the score for the current capture. This ranges from 0-100"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value Type For Discovery.. ex: {type: 'Biometric Device' or 'Finger' or 'Face' or 'Iris' }"
      }
    }
  ]
}
```

#### Allowed Values for Registration Capture Response
Parameters | Description
-----------|-------------
specVersion | Version of the MDS specification using which the response was generated.
data | <ul><li>The data object is sent as JSON Web Token (JWT).</li><li>The data block will be signed using the device key.</li></ul>
data.bioType | Allowed values are "Finger", "Iris" or "Face".
data.digitalId | <ul><li>The digital id as per the digital id definition in JWT format.</li><li>For L0 devices, the digital id will be signed using the device key.</li><li>For L1 devices, the digital id will be signed using the FTM key.</li></ul>
data.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
data.deviceServiceVersion | MDS Version
data.env | <ul><li>The target environment.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>
data.purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>Allowed values are "Auth" or "Registration".</li></ul>
data.bioValue | Base64-URL-encoded biometrics (in ISO format)
data.transactionId | Unique transaction id sent in request
data.timestamp | <ul><li>Time as per the biometric device.</li><li>Note: The biometric device is expected to sync its time from the management server at regular intervals so accurate time could be maintained on the device.</li></ul>
data.requestedScore | Floating point number to represent the minimum required score for the capture.
data.qualityScore | Floating point number representing the score for the current capture.
hash | The value of the previousHash attribute in the request object or the value of hash attribute of the previous data block (used to chain every single data block) concatenated with the hex encode sha256 hash of the current data block before encryption.
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code defined in the [error code section](#error-codes).
error.errorInfo | Description of the error that can be displayed to end user. Multi lingual support.

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
RCAPTURE  http://127.0.0.1:<device_service_port>/capture
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
HTTP response.

#### Android
No support for Registration Capture

#### IOS
No support for Registration Capture

---

## Device Server
The device server exposes two external device APIs to manage devices. These will be consumed from Management Server created by the device provider. Refer to the subsequent section in this document.

### Registration
The MOSIP server would provide the following device registration API which is white-listed to the management servers of the device provider or their partners.

{% hint style="info" %}
This API is exposed by the MOSIP server to the device providers.
{% endhint %}

**Version:** v1

#### Device Registration Request URL
`POST https://{base_url}/v1/masterdata/registereddevices`

#### Device Registration Request
```
{
  "id": "io.mosip.deviceregister",
  "request": {
    "deviceData": {
      "deviceId": "Unique Id to identify a biometric capture device",
      "purpose": "Auth or Registration. Can not be empty.",
      "deviceInfo": {
        "deviceSubId": "An array of sub Ids that are available",
        "certification":  "certification level",
        "digitalId": "Signed digital id of the device",
        "firmware": "Firmware version",
        "deviceExpiry": "Device expiry date",
        "timestamp":  "Current time in ISO format"
      },
      "foundationalTrustProviderId" : "Foundation trust provider Id, in case of L0 this is empty"
    }
  },
  "requesttime": "Current timestamp in ISO format from management server",
  "version": "Registration server api version as defined above"
}
```

#### Accepted Values for Registration Request
Parameters | Description
-----------|-------------
deviceData | <ul><li>The device data object is sent as JSON Web Token (JWT).</li><li>The device data block will be signed using the device provider certificate.</li></ul>
deviceData.deviceId | <ul><li>Unique device id that the device provider uses to identify the device.</li><li>This can also be serial no if the device provider is sure of maintaining the uniqueness across all their devices.</li></ul>
purpose | <ul><li>The purpose of the device in the MOSIP ecosystem.</li><li>For devices that are not registered the purpose is empty.</li><li>Allowed values are "Auth" or "Registration".</li></ul>
deviceData.deviceInfo | <ul><li>The device info object is sent as JSON Web Token (JWT).</li><li>The device info block will be signed using the device key.</li></ul>
deviceInfo.deviceSubId | <ul><li>An array of sub Ids that are supported for the device.</li><li>Allowed values are 1, 2 or 3.</li><li>The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement.</li><li>Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present.</li><li>In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises.</li><li>The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).</li><ul>
deviceInfo.certification | <ul><li>The certificate level of the device.</li><li>Allowed values are L0 or L1</li></ul>
deviceInfo.digitalId | <ul><li>The digital id as per the digital id definition.</li><li>For L0 devices, the digital id will be signed using the device key.</li><li>For L1 devices, the digital id will be signed using the FTM key.</li></ul>
deviceInfo.firmware | Version of the firmware of the device.
deviceInfo.deviceExpiry | <ul><li>Expiry date of the device.</li><li>Device will not work post that expiry date and it cannot be registered again.</li></ul>
deviceInfo.timestamp | <ul><li>The timestamp when the request was created.</li><li>For device registration the request should reach within 5 mins of this timestamp to MOSIP or the request will be rejected.</li></ul>
foundationalTrustProviderId | <ul><li>Foundation trust provider id whoes chip is in the device.</li><li>For L0 device, this is empty.</li></ul>

{% hint style="info" %}
During the registration of L0 devices please sign using the key thats generated inside the device and send the public key in x509 encoded spec form. <br>
After successful registration the management server should issue a certificate against the same public key as a response to the registration call.
{% endhint %}

* The entire device data is sent as a JWT format. So the it will look like:
```
"deviceData" : base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
```
* Payload is the object in deviceData.
* The request is signed with the device provider key at the management server.

#### Device Registration Response
```
{
  "id": "io.mosip.deviceregister",
  "version": "Registration server API version as defined above",
  "responsetime": "ISO time format",
  "response": {
    "status":  "Registration status",
    "digitalId": "Digital id of the device a sent by the request",
    "deviceCode": "UUID RFC4122 Version 4 for the device issued by the mosip server",
    "timestamp": "Timestamp in ISO format",
    "env": "prod/development/stage"
  },
  "error": [
    {
      "errorCode": "Error code if registration fails. Remaining keys above are dropped in case of errors.",
      "message": "Description of the error code"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:
```
"response" : base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
```

### Accepted Values in Device Registration Response
Parameters | Description
-----------|-------------
response | <ul><li>The entire response block will be sent in JWT format.</li><li>This will be signed by MOSIP using their public signature certificate.</li></ul>
response.status | <ul><li>This is the status of the device.</li><li>After successful registration the status will sent as "Registered".</li></ul>
response.digitalId | This is the same digital ID that was sent in the request.
response.deviceCode | <ul><li>This is the device code issued by MOSIP server post registration.</li><li>This will be in UUID RFC4122 Version 4 format</li><li>Once device is registered the device code needs to be set in the device.</li></ul>
response.timestamp | <ul><li>This is the timestamp when the device was registered.</li><li>This will be in ISO format.</li></ul>
response.env | <ul><li>The target environment where the device is registered.</li><li>Allowed values are "Staging", "Developer", "Pre-Production" or "Production".</li></ul>

The response should be sent to the device. The device is expected to store the deviceCode within its storage in a safe manner. This device code is used during the capture stage.

{% hint style="info" %}
The device once registered for a specific purpose can not be changed after successful registration. The device can only be used for that specific mosip process.
{% endhint %}

### De-Register
The MOSIP server would provide the following device de-registration API which is whitelisted to the management servers of the device provider or their partners.

**Version:** v1

#### Device De-Registration Request URL
`POST https://{base_url}/v1/masterdata/device/deregister`

#### Device De-Registration Request
```
{
  "id": "io.mosip.devicederegister",
  "version": "de-registration server api version as defined above",
  "request": {
    "device": {
      "deviceCode": "<device code>",
      "env": "<environment>"
    },
    "isItForRegistrationDevice": true
  }
  "requesttime": "current timestamp in ISO format"
}
```

The device data in request is sent as a JWT format. So the final request will look like:
```
"request": {
  "device" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
}
```

#### Accepted Values for Device De-Registration Request
```
env - Allowed values are Staging| Developer| Pre-Production | Production
deviceCode - This is the device code issued by MOSIP server post registration. This will be in UUID RFC4122 Version 4 format. Once device is registered the device code needs to be set in the device.
isItForRegistrationDevice - It is set as true when it is a registration device and false when it is an authentication device.
```

#### Device De-Registration Response
```
{
  "id": "io.mosip.devicederegister",
  "version": "de-registration server api version as defined above",
  "responsetime": "Current time in ISO format",
  "response": {
    "status": "Success",
    "deviceCode": "<device code>",
    "env": "<environment>",
    "timestamp": "timestamp in ISO format"
  },
  "error": [
        {
      "errorCode" : "<error code if de-registration fails>",
      "message" : "<human readable description of the error code>"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:

```
"response" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
```

---

### Certificates
The MOSIP server would provide the following retrieve encryption certificate API which is white-listed to the management servers of the device provider or their partners.

#### Retrieve Encryption Certificate Request URL
`POST https://{base_url}/v1/masterdata/device/encryptioncertficates`

**Version:** v1

#### Retrieve Encryption Certificate Request
```
{
  "id": "io.mosip.auth.country.certificate",
  "version": "certificate server api version as defined above",
  "request": {
    "data": {
      "env":  "target environment",
      "domainUri": "uri of the auth server"
    }
  },
  "requesttime": "current timestamp in ISO format"
}
```

The request is sent as a JWT format. So the final request will look like:
```
"request": {
  "data": "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
}
```

#### Accepted Values for Retrieve Certificate Request
```
env - Allowed values are Staging| Developer| Pre-Production | Production
domainUri - unique uri per auth providers. This can be used to federate across multiple providers or countries or unions.
```

#### Encryption Certificate Response
```
{
  "id": "io.mosip.auth.country.certificate",
  "version": "certificate server api version as defined above",
  "responsetime": "Current time in ISO format",
  "response": [
    {
      "certificate": "base64encoded certificate as x509 V3 format"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:
```
"response" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
```

---

## Management Server and Management Client

### Management Server Functionalities and Interactions
The management server has the following objectives.

1. Validate the devices to ensure its a genuine device from the respective device provider. This can be achieved using the device info and the certificates for the Foundational Trust Module.
1. Register the genuine device with the MOSIP device server.
1. Manage/Sync time between the end device the server. The time to be synced should be the only trusted time accepted by the device.
1. Ability to issue commands to the end device for
    1. De-registration of the device (Device Keys)
    1. Collect device information to maintain, manage, support and upgrade a device remotely.
1. A central repository of all the approved devices from the device provider.
1. Safe storage of keys using HSM FIPS 140-2 Level 3. These keys are used to issue the device certificate upon registration.
The Management Server is created and hosted by the device provider outside of MOSIP software. The communication protocols between the MDS and the Management Server can be decided by the respective device provider. Such communication should be restricted to the above specified interactions only. No transactional information should be sent to this server.
1. Should have the ability to push updates from the server to the client devices.

{% hint style="info" %}
*As there are no adopter specific information being exchanged at the management server or at the FTM provisioning server, there are no mandates from MOSIP where these are located globally. However the adopter is recommended to have audit and contractual mechanisms to validate the compliance of these components at any point in time.*
{% endhint %}

### Management Client
Management client is the interface that connects the device with the respective management server. Its important that the communication between the management server and its clients are designed with scalability, robustness, performance and security. The management server may add many more capabilities than what is described here, But the basic security objectives should be met at all times irrespective of the offerings.

1. For better and efficient handling of device at large volume, we expect the devices to auto register to the Management server.
1. All communication to the server and from the server should follow that below properties.
    1. All communication are digitally signed with the approved algorithms
    1. All communication to the server are encrypted using one of the approved public key cryptography (HTTPS – TLS1.2/1.3 is required with one of the [approved algorithms](#cryptography).
    1. All request has timestamps attached in ISO format to the milliseconds inside the signature.
    1. All communication back and fourth should have the signed digital id as one of the attribute.
1. Its expected that the auto registration has an absolute way to identify and validate the devices.
1. The management client should be able to detect the devices in a plug and play model.
1. All key rotation should be triggered from the server.
1. Should have the ability to detect if its speaking to the right management server.
1. All upgrades should be verifiable and the client should have ability to verify software upgrades.
1. Should not allow any downgrade of software.
1. Should not expose any API to capture biometric. The management server should have no ability to trigger a capture request.
1. No logging of biometric data is allowed. (Both in the encrypted and unencrypted format)

---

## Compliance
**L1 Certified Device / L1 Device** - A device certified as capable of performing encryption on the device inside its trusted zone. <br>
**L0 Certified Device / L0 Device** - A device certified as one where the encryption is done on the host inside its device driver or the MOSIP device service.

### Secure Provisioning
Secure provisioning is applicable to both the FTM and the Device providers.

1. The devices and FTM should have a mechanism to protect against fraudulent attempts to create or replicate.
1. The device and FTM trust should be programmed in a secure facility which is certified by the respective MOSIP adopters.
1. Organization should have mechanism to segregate the FTM's and Devices built for MOSIP using cryptographically valid and repeatable process.
1. All debug options within the FTM or device should be disabled permanently
1. All key creations need for provisioning should happen automatically using FIPS 140-2 Level 3 or higher devices. No individual or a group or organization should have mechanism to influence this behavior.
1. Before the devices/FTM leaving the secure provisioning facility all the necessary trust should be established and should not be re-programmable.

{% hint style="info" %}
*As there are no adopter specific information being exchanged at the management server or at the FTM provisioning server, there are no mandates from MOSIP where these are located globally. However the adopter is recommended to have audit and contractual mechanisms to validate the compliance of these components at any point in time.*
{% endhint %}

### Compliance Level
API     | Compatible
----|-----------
Device Discovery | L0/L1
Device Info | L0/L1
Capture | L1
Registration Capture | L0/L1

---

## Cryptography
Supported algorithms:

Usage | Algorithm | Key Size | Storage
------|-----------|----------|---------
Encryption of biometrics - Session Key | AES | >=256 | No storage, Key is created with TRNG/DRBG inside FTM
Encryption session key data outside of FTM | RSA OAEP | >=2048 | FTM trusted memory
Encryption session key data outside of FTM | ECC curve 25519 | >=256 | FTM trusted memory
Biometric Signature | RSA | >=2048 | Key never leaves FTM created and destroyed
Biometric Signature | ECC curve 25519 | >=256 | Key never leaves FTM created and destroyed
Secure Boot | RSA | >=256 | FTM trusted memory
Secure Boot | ECC curve 25519 | >=256 | FTM trusted memory

{% hint style="info" %}
No other ECC curves supported.
{% endhint %}

## Signature
In all the above APIs, some of the requests and responses are signed with various keys to verify the requester's authenticity. Here we have detailed the key used for signing a particular block in a request or response body of various APIs.

Request/Response | Block | Signature Key
-----------------|-------|---------------
Device Discovery Response | Device Info | NA as it will not be signed
Device Discovery Response | Digital ID | NA as it will not be signed
Device Info Response | Device Info | <ul><li>NA in case of unregistered device</li><li>Device Key in case of registered device</li></ul>
Device Info Response | Digital ID | <ul><li>For L0 device using device key</li><li>For L1 device using FTM chip key</li></ul>
Capture Response | Data | Device key is used
Capture Response | Digital ID | FTM chip key is used
Registration Capture Response | Data | Device key is used
Registration Capture Response | Digital ID | <ul><li>For L0 device using device key</li><li>For L1 device using FTM chip key</li></ul>
Device Registration Request | Device Data | Device Provider certificate is used
Device Registration Request | Device Info | Device key is used
Device Registration Request | Digital ID | <ul><li>For L0 device using device key</li><li>For L1 device using FTM chip key</li></ul>
Device De-registration Request | Device | Device Provider certificate is used
Device Registration Response | Response | MOSIP Signature certificate is used
Device Registration Response | Digital ID | Should be same as request
Device De-registration Response | Device | MOSIP Signature certificate is used

## Error Codes
Code | Message
-----|--------
0 | Success
100     | Device not registered
101     | Unable to detect a biometric object
102     | Technical error during extraction.
103     | Device tamper detected
104     | Unable to connect to management server
105     | Image orientation error
106     | Device not found
107     | Device public key expired
108     | Domain public key missing
109     | Requested number of biometric (Finger/IRIS) not supported
5xx     | Custom errors. The device provider is free to choose his error code and error messages.
