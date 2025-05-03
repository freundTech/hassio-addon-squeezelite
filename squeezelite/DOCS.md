# Home Assistant Add-on: Squeezelite

## How to use

Simply install and start this add-on.

## Configuration

**Note**: _Remember to restart the add-on when the configuration is changed._

### Option: `player_name`

The name your Squeezelite player advertises as. Defaults to `Home Assistant`

### Option: `mac_address`

The MAC address your player advertises as. This can be any hexadecimal value in the form AB:CD:EF:12:34:56.
The MAC address needs to be different from all other Squeezelite players on your network.
If your change the address your media server might see your player as a new player.

When no address is given the add-on generates a random address and persists it across restarts.
