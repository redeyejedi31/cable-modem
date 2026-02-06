# Gaining API access to Cable Modem Hub 5 

## Introduction
I have written a script (work in progress) that retrieves the Cable Modems (CM) Sagemcom F3896LG-VMB stats by querying the hub5's Api, the data is stored in mysql and stats displayed in Grafana. I will will be uploaded the script to github soon.

### How to access APi
A bearer token is generated each time you login.
#### Get Token
```console
curl --location --request POST 'http://192.168.0.1/rest/v1/user/login' \
--header 'Content-Type: application/json' \
--data-raw '{"password":"ABC123"}'
```
Use the password printed on your Cable Modem (CM) to access the configuration page. Replace ABC123 with your password, ensuring it remains enclosed in quotes.

The query will return:
```JSON
{
    "created": {
        "token": "e7a3c625616b8ad58493e1a9c91181c3",
        "userLevel": "regular",
        "userId": 3
    }
}
```
Generating a token will disable web browser login.  Tokens should be deleted after use or they will automatically expire after 10 minutes of inactivity.
#### Delete Token
```console
curl --location --request DELETE 'http://192.168.0.1/rest/v1/user/3/token/e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
A successful Delete Token request will not return a response.  An invalid or incorrect key will result in the following:
```json
{"status":401,"message":"Unauthorized","errorCode":7}
```

## Example Queries

#### Ping Servers 
Ping (ICMP) and Traceroute operations create jobs requiring a POST, GET, and DELETE sequence.  A new job cannot be initiated until the previous one is completed and deleted.
```console 
curl --location --request POST 'http://192.168.0.1/rest/v1/system/diagnostics/ping/jobs' \
--header 'Token: Bearer e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Content-Type: application/json' \
--data-raw '{"pingJob":{"parameters":{"host":"1.1.1.1","numberOfPings":3,"dataBlockSize":64}}}'
```
#### Retrieve Ping
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/system/diagnostics/ping/job/1'
```
This will return:
```JSON
{
    "pingJob": {
        "state": "complete",
        "parameters": {
            "host": "1.1.1.1",
            "interface": "erouter0",
            "intervalMs": null,
            "numberOfPings": 3,
            "dataBlockSize": 64,
            "timeoutMs": 10
        },
        "results": {
            "successCount": 3,
            "failureCount": 0,
            "averageResponseTimeMs": "16.268",
            "minimumResponseTimeMs": "16.169",
            "maximumResponseTimeMs": "16.396"
        }
    }
}
```
#### Delete Ping Job
```console
curl --location --request DELETE 'http://192.168.0.1/rest/v1/system/diagnostics/ping/job/1' \
--header 'Token: Bearer e7a3c625616b8ad58493e1a9c91181c3' \
--header 'Content-Type: application/json' \
--data-raw '{"pingJob":{"parameters":{"host":"1.1.1.1","numberOfPings":3,"dataBlockSize":64}}}'
```
#### GET Information from DHCP Server
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/network/ipv4/dhcp' \
--header 'Accept-Language: en-GB,en;q=0.9,en-US;q=0.8' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
```JSON
{
    "dhcp": {
        "enable": true,
        "minAddress": "192.168.0.10",
        "maxAddress": "192.168.0.254",
        "subnetMask": "255.255.255.0",
        "leaseTime": 86400,
        "maxIps": 254,
        "allowedIps": [
            {
                "lanAllowedSubnetIp": "192.168.0.1",
                "lanAllowedSubnetMask": "255.255.255.0"
            }
        ],
        "blockedIps": [
            {
                "lanBlockedSubnetIp": "x.x.x.x",
                "lanBlockedSubnetMask": "0.0.0.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.252.0"
            },
            {
                "lanBlockedSubnetIp": "xx.xx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.252.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.252"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xx.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.248"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.248"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.xxx.xxx",
                "lanBlockedSubnetMask": "255.255.255.252"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            },
            {
                "lanBlockedSubnetIp": "xxx.xxx.x.x",
                "lanBlockedSubnetMask": "255.255.255.0"
            }
        ]
    }
}
```
### List Connected Devices
```console
curl --location --request GET 'http://192.168.0.1/rest/v1/network/hosts?connectedOnly=true' \
--header 'Accept-Language: en-GB,en;q=0.9,en-US;q=0.8' \
--header 'Authorization: Bearer e7a3c625616b8ad58493e1a9c91181c3'
```
```JSON
{
    "hosts": {
        "hosts": [
            {
                "macAddress": "B9:1E:BC:42:90:15",
                "config": {
                    "connected": true,
                    "deviceName": "",
                    "deviceType": "unknown",
                    "hostname": "unknown",
                    "interface": "ethernet",
                    "speed": 1000,
                    "ethernet": {
                        "port": 4
                    },
                    "ipv4": {
                        "address": "192.168.0.111",
                        "leaseTimeRemaining": 80492
                    }
                }
            },
            {
                "macAddress": "DE:3D:04:5C:7E:F3",
                "config": {
                    "connected": true,
                    "deviceName": "",
                    "deviceType": "unknown",
                    "hostname": "unknown",
                    "interface": "wifi",
                    "speed": 390,
                    "wifi": {
                        "band": "band5g",
                        "ssid": "VM1234567890_5G",
                        "rssi": -75
                    },
                    "ipv4": {
                        "address": "192.168.0.144",
                        "leaseTimeRemaining": 70714
                    }
                }
            }
        ]
    }
}
```
Here are the API-accessible pages, without detailing the specific commands or their functionality: 
### Exposed api endpoints

| Endpoint | Function |
|---|---|
| **Static Resources** |  |
| `/resources/languages*` | Base path for language JSON resources (used to load `/resources/languages/{lang}.json`) |
| `/resources/languages/uk.json*` | UK translation strings (language pack JSON) |
| **Cable Modem Endpoints** |  |
| `/rest/v1/cablemodem/` | Cable modem overview/summary (DOCSIS status bundle) |
| `/rest/v1/cablemodem/downstream` | DOCSIS downstream channel table |
| `/rest/v1/cablemodem/downstream/primary_` | Primary downstream channel summary |
| `/rest/v1/cablemodem/eventlog` | Cable modem event log (DOCSIS log entries) |
| `/rest/v1/cablemodem/registration*` | Cable modem registration/provisioning state |
| `/rest/v1/cablemodem/serviceflows` | DOCSIS service flow information |
| `/rest/v1/cablemodem/state` | Cable modem state/link status (DOCSIS state object) |
| `/rest/v1/cablemodem/state_` | Cable modem state (alternate/legacy variant used by UI) |
| `/rest/v1/cablemodem/upstream` | DOCSIS upstream channel table |
| `/rest/v1/cablemodem/upstream/primary_` | Primary upstream channel summary |
| `/rest/v1/cablemodem/{direction}` | DOCSIS channel table by direction (`downstream` \| `upstream`) |
| `/rest/v1/cablemodem/{direction}/primary_` | Primary channel summary by direction (`downstream` \| `upstream`) |
| **Echo Endpoint** |  |
| `/rest/v1/echo` | Service availability/connectivity probe endpoint (used by UI during reboot/polling) |
| **MTA Endpoints** |  |
| `/rest/v1/mta/lines` | Telephony (MTA) line status |
| **Network Endpoints** |  |
| `/rest/v1/network/hosts` | LAN host/device list (supports filters such as `connectedOnly` / `interface` depending on variant) |
| `/rest/v1/network/hosts?connectedOnly=true&interface=wifi` | Host list filtered to currently-connected WiFi devices |
| `/rest/v1/network/hosts_?connectedOnly=true&interface=ethernet` | Host list (alternate endpoint) filtered to currently-connected Ethernet devices |
| `/rest/v1/network/hosts_?connectedOnly=true&interface=wifi` | Host list (alternate endpoint) filtered to currently-connected WiFi devices |
| `/rest/v1/network/ipportfilters` | IP/port filter rules (access control/filtering configuration) |
| `/rest/v1/network/ipv4/dhcp` | DHCP server configuration (IPv4) |
| `/rest/v1/network/ipv4/dmz` | DMZ configuration (IPv4) |
| `/rest/v1/network/ipv4/firewall` | Firewall configuration (IPv4) |
| `/rest/v1/network/ipv4/info` | IPv4 interface/addressing information |
| `/rest/v1/network/ledlight` | Front-panel LED/light settings |
| `/rest/v1/network/macfilters` | MAC filtering rules |
| `/rest/v1/network/mtu` | WAN/LAN MTU configuration |
| `/rest/v1/network/portforwarding` | Port forwarding rules (NAT forwards) |
| `/rest/v1/network/porttriggers` | Port triggering rules |
| `/rest/v1/network/reservedipaddresses` | DHCP reservation list (static leases) |
| `/rest/v1/network/upnp` | UPnP configuration and/or status |
| `/rest/v1/network/mbu` | MBU status/info (keep if verified on-device; name suggests a status/info object) |
| **PON Endpoints** |  |
| `/rest/v1/pon/state*` | PON/optical link state (only relevant if device exposes a PON module) |
| **System Endpoints** |  |
| `/rest/v1/system/diagnostics/ping/job/` | Ping job instance endpoint (job lifecycle/result; ID required in practice) |
| `/rest/v1/system/diagnostics/ping/job/1` | Ping job instance endpoint for job ID 1 (get state/result; delete job) |
| `/rest/v1/system/diagnostics/ping/job/1?stateOnly=true` | Ping job state-only view for job ID 1 |
| `/rest/v1/system/diagnostics/ping/job/{id}` | Ping job instance endpoint (get state/result; delete job) |
| `/rest/v1/system/diagnostics/ping/job/{id}?stateOnly=true` | Ping job instance state-only view |
| `/rest/v1/system/diagnostics/ping/job/{pingJobs}` | Bulk ping job lookup endpoint (multiple IDs encoded in path) |
| `/rest/v1/system/diagnostics/ping/jobs` | Ping job collection (create new job; list jobs) |
| `/rest/v1/system/diagnostics/ping/jobs?stateOnly=true` | Ping job collection state-only view |
| `/rest/v1/system/diagnostics/traceroute/job/` | Traceroute job instance endpoint (job lifecycle/result; ID required in practice) |
| `/rest/v1/system/diagnostics/traceroute/job/1` | Traceroute job instance endpoint for job ID 1 (get state/result; delete job) |
| `/rest/v1/system/diagnostics/traceroute/job/1?stateOnly=true` | Traceroute job state-only view for job ID 1 |
| `/rest/v1/system/diagnostics/traceroute/job/{id}` | Traceroute job instance endpoint (get state/result; delete job) |
| `/rest/v1/system/diagnostics/traceroute/job/{id}?stateOnly=true` | Traceroute job instance state-only view |
| `/rest/v1/system/diagnostics/traceroute/job/{traceRouteJobs}` | Bulk traceroute job lookup endpoint (multiple IDs encoded in path) |
| `/rest/v1/system/diagnostics/traceroute/jobs` | Traceroute job collection (create new job; list jobs) |
| `/rest/v1/system/diagnostics/traceroute/jobs?stateOnly=true` | Traceroute job collection state-only view |
| `/rest/v1/system/firstinstall*` | Initial setup/first-run state |
| `/rest/v1/system/gateway/provisioning*` | Gateway provisioning/activation status |
| `/rest/v1/system/info` | System/device information (model, firmware, uptime, etc) |
| `/rest/v1/system/languages` | Supported languages list (system/UI locales) |
| `/rest/v1/system/localization` | Current localisation settings (locale/timezone/language selection) |
| `/rest/v1/system/modemmode` | Modem mode/router mode configuration and state |
| `/rest/v1/system/reboot` | Trigger reboot |
| `/rest/v1/system/restore` | Factory reset/restore defaults |
| `/rest/v1/system/softwareupdate*` | Firmware/software update status (and possibly trigger/check) |
| `/rest/v1/system/ui/defaults` | UI defaults/initial UI configuration values |
| `/rest/v1/system/ui/screens*` | UI screen/page registry (what screens are available/enabled) |
| **User Endpoints** |  |
| `/rest/v1/user/` | User resource collection/current user info (may support CRUD depending on implementation) |
| `/rest/v1/user/3/language` | Get/set language preference for user ID 3 |
| `/rest/v1/user/3/password` | Change password for user ID 3 |
| `/rest/v1/user/login` | Login/authentication endpoint (session/token workflow) |
| `/rest/v1/user/{user}/language` | Get/set language preference for specified user ID |
| `/rest/v1/user/{user}/token/{authRetrieveToken}` | Token/session management endpoint (retrieve/revoke token depending on verb) |
| **WiFi Endpoints** |  |
| `/rest/v1/wifi/` | WiFi overview/summary |
| `/rest/v1/wifi/band2g/config` | 2.4 GHz WiFi configuration |
| `/rest/v1/wifi/band2g/guest/config` | 2.4 GHz guest WiFi configuration |
| `/rest/v1/wifi/band2g/macfilters` | 2.4 GHz MAC filtering configuration |
| `/rest/v1/wifi/band2g/state` | 2.4 GHz WiFi status/state |
| `/rest/v1/wifi/band2g/state_` | 2.4 GHz WiFi status/state (alternate/legacy variant) |
| `/rest/v1/wifi/band2g/wps/config` | 2.4 GHz WPS configuration |
| `/rest/v1/wifi/band2g/wps/pairing/jobs` | 2.4 GHz WPS pairing job collection/status |
| `/rest/v1/wifi/band5g/config` | 5 GHz WiFi configuration |
| `/rest/v1/wifi/band5g/guest/config` | 5 GHz guest WiFi configuration |
| `/rest/v1/wifi/band5g/macfilters` | 5 GHz MAC filtering configuration |
| `/rest/v1/wifi/band5g/state` | 5 GHz WiFi status/state |
| `/rest/v1/wifi/band5g/state_` | 5 GHz WiFi status/state (alternate/legacy variant) |
| `/rest/v1/wifi/band5g/wps/config` | 5 GHz WPS configuration |
| `/rest/v1/wifi/band5g/wps/pairing/jobs` | 5 GHz WPS pairing job collection/status |
| `/rest/v1/wifi/capabilities` | WiFi capability flags/features supported |
| `/rest/v1/wifi/smartmode` | Smart WiFi/band steering mode configuration |
| `/rest/v1/wifi/{band}/config` | WiFi configuration by band (`band2g` \| `band5g`) |
| `/rest/v1/wifi/{band}/guest/config` | Guest WiFi configuration by band (`band2g` \| `band5g`) |
| `/rest/v1/wifi/{band}/state` | WiFi status/state by band (`band2g` \| `band5g`) |


