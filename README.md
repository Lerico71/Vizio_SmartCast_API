# Vizio SmartCast API (2016+ Models)

## Overview
* API Server runs on port `9000` using https. Will not respond to http.

	*Don't port forward it. There are some commands that can be executed 
without authentication.*
* Certificate's CN is `BG2.prod.vizio.com` so will likely fail SSL validation.
* API includes a `Status` object, `URI` requested, and Execution `time` on every response. It's included below, but excluded in all of the examples for redundancy.

	```
	{
		...
		"STATUS": {
			"RESULT": String,
			"DETAIL": String
		},
		"URI": String,
		"TIME": String
	}
	```
	API does not seem to be fully restful, and doesn't apply proper status codes in responses. Verify request's result from the `status` object. Result codes included where applicable.
* All requests made should contain a JSON body with the header `Content-Type: application/json` set.
* When authentication is required, send `Auth: AUTH_TOKEN` header with request.
* This does not cover any MyVizio Account APIs.
* See an issue? Have a question? Open an issue, or find me on twitter [@exiva](https://twitter.com/exiva)

## Pairing
***required to control set.***
### Start Pairing
`PUT /pairing/start`

#### Body
```
{
	"DEVICE_NAME": String,
	"DEVICE_ID": String
}
```

#### Response
```
{
	"ITEM": {
		"PAIRING_REQ_TOKEN": Integer,
		"CHALLENGE_TYPE": Integer
	},
	...
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -X PUT -d '{"DEVICE_ID":"12345","DEVICE_NAME":"cURL Example"}' https://myVizioTV:9000/pairing/start`

#### Notes
Save `DEVICE_ID`, you'll need it for the challenge.

### Submit Challenge
`PUT /pairing/pair`

#### Body
```
{
  "DEVICE_ID": String,
  "CHALENGE_TYPE": Integer,
  "RESPONSE_VALUE": Integer,
  "PAIRING_REQ_TOKEN": Integer  
}
```

#### Response
```
{
	"ITEM": {
		"AUTH_TOKEN": String
	},
	...
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -X PUT -d '{"DEVICE_ID": "12345","CHALLENGE_TYPE": 1,"RESPONSE_VALUE": "1234","PAIRING_REQ_TOKEN": 0}' https://myVizioTV:9000/pairing/pair`

#### Notes
`RESPONSE_VALUE` key is the code displayed on the TV.

### Cancel Pairing
`PUT /pairing/cancel`

#### Body
```
{
  "DEVICE_NAME": String,
  "DEVICE_ID": String
}
```

#### Response
```
{
	"ITEM": {},
	...
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -X PUT -d '{"DEVICE_ID":"12345","DEVICE_NAME":"cURL"}' https://myVizioTV:9000/pairing/cancel`

### Appendix
####Status Results
| RESULT                   | Meaning                       |
| ------------------------ | ----------------------------- |
| INVALID_PARAMETER        | Malformed Request             |
| MAX\_CHALLENGES_EXCEEDED | Too many failed pair attempts |
| SUCCESS                  | Successfully Paired           |
| PAIRING_DENIED           | Incorrect Pin                 |
| VALUE\_OUT\_OF_RANGE     | Pin out of range              |
| CHALLENGE_INCORRECT      | Incorrect challenge           |
| BLOCKED                  | Pairing in progress already   |


## Remote Control Methods
### Turning Set on From Sleep
The HTTP API server turns off when the set is sleeping. Send a WoL magic packet to turn it on. *not supported when set runs in Eco mode.*

### Send remote control button press
*Authenticated*

`PUT /key_command`

####Body
```
{
	"KEYLIST": [{
		"CODESET": Integer,
		"CODE": Integer,
		"ACTION": String
	}]
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X PUT -d '{"KEYLIST": [{"CODESET": 5,"CODE": 0,"ACTION":"KEYPRESS"}]}' https://myVizioTV:9000/key_command`

#### Notes
You can string together long remote actions by adding to the `keylist` array.

### Appendix
| Action   |
| -------- |
| KEYDOWN  | 
| KEYUP    |
| KEYPRESS |

| Event Name   | Codeset | Code |
| ------------ | :-----: | :--: |
| Channel Down | 8       | 0    |
| Channel Up   | 8       | 1    |
| Previous Ch  | 8       | 2    |
| Volume Down  | 5       | 0    |
| Volume Up    | 5       | 1    |
| Mute Toggle  | 5       | 4    |
| Mute On      | 5       | 3    |
| Mute Off     | 5       | 2    |
| Cycle Input  | 7       | 1    |
| Power Off    | 11      | 0    |
| Power On     | 11      | 1    |
| Power Toggle | 11      | 2    |

## Video Input Methods
### Get currently selected input
*Authenticated*

`GET /menu_native/dynamic/tv_settings/devices/current_input`

#### Response
```
{
	"ITEMS": [{
		"NAME": String,
		"CNAME": String,
		"TYPE": String,
		"VALUE": String,
		"ENABLED": Boolean,
		"HASHVAL": Integer
	}],
	"NAME": String,
	"HASHLIST": Array,
	"GROUP": String,
	"PARAMETERS": {
		"HASHONLY": String,
		"FLAT": String,
		"HELPTEXT": String
	},
	...
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X GET https://myVizioTV:9000/menu_native/dynamic/tv_settings/devices/current_input`

### Get list of inputs
*Authenticated*

`GET /menu_native/dynamic/tv_settings/devices/name_input`

#### Response
```
{
	"ITEMS": [{
		"NAME": String,
		"CNAME": String,
		"TYPE": String,
		"VALUE": {
			"NAME": String,
			"METADATA": String
		},
		"ENABLED": Boolean,
		"HASHVAL": Integer
	},
	...
	],
	"NAME": String,
	"HASHLIST": Array,
	"GROUP": String,
	"PARAMETERS": {
		"HASHONLY": String,
		"FLAT": String,
		"HELPTEXT": String
	},
	...
}
```

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X GET https://myVizioTV:9000/menu_native/dynamic/tv_settings/devices/name_input`

### Change Input
*Authenticated*

`PUT /menu_native/dynamic/tv_settings/devices/current_input`

#### Body
```
{
	"REQUEST": "MODIFY",
	"VALUE": String,
	"HASHVAL": Integer
}
```

##### Expected Values
| PUT `current_input` | From `name_input`   |
| ------------------- | ------------------- |
| VALUE               | ITEMS[x].CNAME      |
| HASHVAL             | ITEMS[x].HASHVAL    |

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X PUT -d '{"REQUEST": "MODIFY","VALUE": "HDMI-1","HASHVAL": 1384176329}' https://myVizioTV:9000/menu_native/dynamic/tv_settings/devices/current_input`

## Display Settings
### Read Settings
*Authenticated*

`GET /menu_native/dynamic/tv_settings/SETTINGS_CNAME`
*(See [Settings CNAMEs](#settings-cnames) for `SETTINGS_CNAME` values)*

#### Response
```
{
	"ITEMS": [{
		"NAME": String,
		"CNAME": String,
		"TYPE": String,
		"VALUE": String,
		"ENABLED": Boolean,
		"HASHVAL": Int,
		"ELEMENTS": Array
	},
	...
	],
	"NAME": String,
	"HASHLIST": Array,
	"GROUP": String,
	"PARAMETERS": {
		"HASHONLY": String,
		"FLAT": String,
		"HELPTEXT": String
	},
	...
}
```

#### Notes
`ELEMENTS` array is conditional.

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X GET https://myVizioTV:9000/menu_native/dynamic/tv_settings/picture`

### Write Settings
*Authenticated*

**Warning** Writing an invalid value may have the potential to brick sets. Use the `TYPE` key from `SETTINGS_CNAME` `ITEMS` array to determine the data type to write.

`PUT /menu_native/dynamic/tv_settings/SETTINGS_CNAME/ITEMS_CNAME`

#### Body
```
{
	"REQUEST": "MODIFY",
	"HASHVAL": Integer,
	"VALUE": String/Integer/Boolean
}
```

#### Notes
Obtain `ITEMS_CNAME` and `HASHVAL` values from the `SETTINGS_CNAME` `ITEMS` array.

#### cURL Example
`curl -k -H "Content-Type: application/json" -H "AUTH: 123A456B" -X PUT -d '{"REQUEST": "MODIFY", "HASHVAL": 831638627, "VALUE": "On"}' https://myVizioTV:9000/menu_native/dynamic/tv_settings/picture/game_low_latency`

### Appendix
#### Settings CNAME's
| SETTINGS_CNAME                                 |
| ---------------------------------------------- |
| picture                                        |
| picture/picture_size                           |
| picture/picture_position                       |
| picture/picture_mode_edit                      |
| picture/color\_calibration                     |
| picture/color\_calibration/color_tuner         |
| picture/color\_calibration/calibration_tests   |
| audio                                          |
| timers                                         |
| network                                        |
| channels                                       |
| closed_captions                                |
| devices                                        |
| system                                         |
| system/system\_information                     |
| system/system\_information/tv_information      |
| system/system\_information/tuner_information   |
| system/system\_information/network_information |
| system/system\_information/uli_information     |
| mobile_devices                                 |
| cast                                           |


## Misc

#### Status Responses
| `RESULT`                      |
| ----------------------------- |
| success                       |
| failure                       |
| uri\_not_found                |
| aborted                       |
| busy                          |
| blocked                       |
| requires_pairing              |
| requires\_system_pin          |
| requires\_new\_system_pin     |
| net\_wifi\_needs\_valid_ssid  |
| net\_wifi\_already_connected  |
| net\_wifi\_missing_password   |
| net\_wifi\_not_existed        |
| net\_wifi\_missing_password   |
| net\_wifi\_needs\_valid_ssid  |
| net\_wifi\_auth_rejected      |
| net\_wifi\_connect_timeout    |
| net\_wifi\_connect_aborted    |
| net\_wifi\_connection_error   |
| net\_ip\_manual\_config_error |
| net\_ip\_dhcp_failed          |
| net\_unknown_error            |

#### Types
| `ITEM`                      |
| --------------------------- |
| T\_UNKNOWN_V1               |
| T\_NO\_TYPE_V1              |
| T\_HIDDEN\_NETWORK_V1       |
| T\_DST\_LIST_V1             |
| T\_MENU_V1                  |
| T\_MENU\_X_V1               |
| T\_LIST_V1                  |
| T\_LIST\_X_V1               |
| T\_VALUE\_ABS_V1            |
| T\_VALUE_V1                 |
| T\_STRING_V1                |
| T\_PIN_V1                   |
| T\_IP\_ADDRESS_V1           |
| T\_MAC\_ADDRESS_V1          |
| T\_MATRIX_V1                |
| T\_HEADER_V1                |
| T\_ROW_V1                   |
| T\_DEVICE_V1                |
| T\_ACTION_V1                |
| T\_APS_V1                   |
| T\_AP_V1                    |
| T\_PASSWORD_V1              |
| T\_SOFTAP\_CONFIG_V1        |
| T\_LIST\_PAIRED\_DEVICES_V1 |
| T\_TEST\_CONNECTION_V1      |
| T\_IETF\_2822\_STRING_V1    |
| T\_DATE_V1                  |
| T\_LIST\_CEC\_DEVICE_V1     |
| T\_MANUAL\_IP\_CONFIG_V1    |
| T\_VIZIO\_DEVICE\_INFO_V1   |
| T\_UPDATE\_INFO_V1          |
| T\_ARRAY_V1                 |
| T\_EMAIL_V1                 |
| T\_LIST\_VALUES_V1          |
| T\_CEC\_DEVICE_V1           |

