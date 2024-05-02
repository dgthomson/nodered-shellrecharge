> __Warning__ 
>
> [21-09-2023] Shell Recharge and the Shell Go+ loyalty program now share the same login details. You may need to change your password inside the flow.
>
> [02-05-2024] If you get ``HTTP Error [403:x]``, try replacing the subflow with version 2.5.

## About this subflow

This subflow will control Shell Recharge or Newmotion car charging equipment.

The latest version is **sr-2-30.json** which needs **Node-RED version 3.02 or later**.

<a href="https://www.buymeacoffee.com/77gg4cxtp8r" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Getting started

Drag the Shell Recharge node into a flow.

Double-click on the new node.

Using the credentials used to login to the Shell Recharge website, enter the username and password into the respective fields of the `Edit subflow instance: <xxx>` box which appears.

To find suitable values for _Charger ID_ and _Card ID_, send a payload (using an inject node, for example) to **_Input_** containing:

> msg.payload = { "action": "setup" }

<img width="438" alt="Data" src="https://user-images.githubusercontent.com/2021563/221404599-3fc7cf22-d4d2-4f6d-a7b0-1c2da7774fa2.png">

Inspect the output (by connecting a debug node to the **_Debug_** output). Find your charger and RFID token card/keyfob details in this- these match the information shown on the Shell Recharge website. The array index for _msg.payload.chargePoints_ and _msg.payload.chargeTokens_ (highlighted by the red circles in the image above) should be entered in the configuration fields _Charger ID_ and _Card ID_ respectively. _These will be a number, usually `0`._

> __Warning__ 
>
> Card ID and Charger ID must only contain a number derived from the process above. Do not enter the charge card number as printed on a key fob/RFID token card or charge point serial number.
>
> <img width="493" alt="Screenshot 2023-02-26 at 11 15 27" src="https://user-images.githubusercontent.com/2021563/221416402-dcf6de7b-a260-4730-b201-865174e37598.png">

## Inputs

> msg.payload = { "action": "_ACTION_" }

...where _ACTION_ is:

**start** and **stop**

Start or stop a charging session using the configured charger and RFID card/token.

Override the node settings for _Charger ID ('cid')_ and _Card ID ('tid')_ with:

> msg.payload = { "action": "start", "cid": _\<number\>_, "tid": _\<number\>_ }

Add a random delay (between 0 and 5 minutes) to the start of a charge with:

> msg.payload = { "action": "start", "delay": true }

**reset**

Reset the charging equipment.

**poll**

Poll the charging equipment once for status information: this gives details directly from Shell Recharge like _'Charging'_ and _'Occupied'_. 

> msg.payload = { "action": "poll" }

**rpoll** and **nopoll**

Repeatedly poll the charging equipment for status information.

The polling interval (in seconds) can be set in (number) _msg.rpoll.interval_ and the duration (in seconds) can be set in (number) _msg.rpoll.duration_, for example:

> msg.payload = { "action": "rpoll", "poll": {"interval": 5, "duration": 20 } }

This would poll every 5 seconds for 20 seconds.

Stop polling with _nopoll_.

After _start_, _stop_ and _reset_, polling will automatically run every 20 seconds for 120 seconds.

**energy**

Get energy consumption information.

The start time is set in (number) _msg.range.start_ and the end time is set in (number) _msg.range.end_, both in Unix timestamp format, for example:

> msg.payload = { "action": "energy", "range": {"start": 1692054000000, "end": 1694645999999 } }

**expire**

Delete the current Shell Recharge API token. This will force a login attempt during the next interaction with Shell Recharge.

**flush**

Delete the current Shell Recharge API token, session and cache (which stores account details like chargers and token cards).

**dump**

Show the internal cache.

## Outputs

The **_Output_** payload is _(boolean) true_ if a charge is in progress, otherwise it is _(boolean) false_. This is also updated when polling is in progress. All other output is from **_Debug_** labelled with topics like _status_, _poll_, _response_, _energy_ and _error_.

## Problems

_HTTP Error [401:x]_

This means 'Unauthorised'. Check your username and password, or try _expire_.

_HTTP Error [409:x]_

``"The HTTP 409 Conflict response status code indicates a request conflict with the current state of the target resource."`` In the context of Shell Recharge, this probably means you're trying to start a charge while the system is in the process of stopping a session, or maybe resetting. Wait, then try again.

_"TypeError: Cannot read properties of undefined (reading '[evses|uuid]')"_

Do not enter the value of `uuid` or `rfid` given the `{"action":"setup"}` process into the `Edit subflow instance: <xxx>` section. `Charger ID` and `Card ID` should be a number, usually `0`.

Otherwise, get in touch.

## Contact

via [Github](https://github.com/dgthomson/nodered-shellrecharge)




