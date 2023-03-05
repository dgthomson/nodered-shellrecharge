## About the Node-RED-shellrecharge subflow

This subflow will control Shell Recharge or Newmotion car charging equipment.

The most recent version is nr-shellrecharge.json.

> __Note__
>
> 26-Feb-2023: changes at Shell Recharge have broken the retrieval of energy consumption data for 'info' and 'energy', however the information needed for _Charger ID_ and _Card ID_ is still provided.

## Getting started

In Node-RED, click the top-right MENU icon (hamburger) then click **Import**. The option **Paste flow json or select a file to import** can be found here. Choose **nr-shellrecharge.json** (downloaded from this webpage) and then drop the imported nodes on to your flow.

Double-click on the new node.

Using the credentials used to login to the Shell Recharge website, enter the username and password into the respective fields.

To find suitable values for _Charger ID_ and _Card ID_, send a payload (using an inject node, for example) to **_Input_** containing:

> msg.payload = {"action":"setup"}

Inspect the output (by connecting a debug node to the **_Debug_** output). Choose a suitable charger and RFID token card/keyfob- these match the details shown on the Shell Recharge website. The array index for _msg.payload.chargePoints_ and _msg.payload.chargeTokens_ should be entered in the configuration fields _Charger ID_ and _Card ID_ respectively.

<img width="438" alt="Data" src="https://user-images.githubusercontent.com/2021563/221404599-3fc7cf22-d4d2-4f6d-a7b0-1c2da7774fa2.png">

_Poll_ should be left unchecked initially (see below).

_Automatically refresh token_ will keep a session active by refreshing the Shell Recharge API token every 24 hours.

> __Warning__ 
>
> Card ID and Charger ID must only contain a number derived from the process above. Do not enter the charge card number as printed on a key fob/RFID token card or charge point serial number.
>
> <img width="493" alt="Screenshot 2023-02-26 at 11 15 27" src="https://user-images.githubusercontent.com/2021563/221416402-dcf6de7b-a260-4730-b201-865174e37598.png">

## Inputs

> msg.payload = {"action":"_ACTION_"}

...where _ACTION_ is:

**start**

Start a charging session using the configured charger and RFID card/token.

**stop**

Stop the current charging session.

**reset**

Reset the charging equipment.

**info**

This gives similar output to  **setup** above. The information may be returned from the cache; use **flush** first to update it if changes have been made on the Shell Recharge website.

~~Energy consumption data is also returned if there is an active Shell Recharge subscription _(beta)_. Additional payload parameters are _from_ and _to_ which should be set as a Unix Epoch time which will output for the period interval specified. Usually, 50 results are given per query; use _offset_ with _(number) n_ to select a batch of results from _n_ to _(n + 50)_.~~

**poll** and **nopoll**

Continually poll the charging equipment for status information: this gives details directly from Shell Recharge like _'Charging'_ and _'Occupied'_. 

Select _Poll_ on the setup page to automatically poll after **start**, **stop** and **reset** actions.

The polling interval is every 5 minutes but only for 4 hours in total, at which point polling will be disabled automatically (in order to play fair with Shell Recharge's servers).

**flush**

Delete the current Shell Recharge API token, session and cache (which stores account details like chargers and token cards/keyfobs). This will force a fresh login attempt during the next interaction with Shell Recharge and get up-to-date account information. The token is expired internally after 24 hours.

**token**

Manually request a new Shell Recharge API token. Expires internally after 24 hours. Set _msg.payload.quiet_ to _true_ to suppress the initial token flush debug message.

**ready**

Tests whether a live API token is available: outputs _(boolean) true_ or _(boolean) false_.

**refresh** and **norefresh**

Keep a session active by refreshing the Shell Recharge API token every 24 hours after **start**, **stop** and **reset** actions.

**vehicles**

List the vehicles set up on the Shell Recharge account.

~~**energy**~~

~~Similar to **info**, but uses another (less-reliable) Shell Recharge API to return energy consumption data.~~

## Outputs

The **_Output_** payload is _(boolean) true_ if a charge is in progress after a **start** command (see above) and _(boolean) false_ when a charge has been stopped by **stop** (see above). This is also updated when polling is in progress. All other output is from **_Debug_** labelled with topics like _status_, _poll_, _response_, _token_, _ready_ and _error_.

## Problems

Sometimes Shell Recharge just doesn't allow the system to log in... it's not clear why. The node will indicate if this is the case. The node will try 5 times before giving up.

## Contact

via [Github](https://github.com/dgthomson/nodered-shellrecharge)
