# Home Assistant App: Sendspin

## How to use

Simply install and start this app.

## Configuration

**Note**: _Remember to restart the app when the configuration is changed._

### Option: `player_name`

The name your Sendspin player advertises as. Defaults to `Home Assistant`

### Option: `player_id`

The ID your player advertises as. The ID needs to be different from all other Sendspin 
players on your network. If your change the ID your media server might see your player
as a new player.

When no ID is given the app generates an ID from your hostname.

### Option: `static_delay_ms`

This is an extra time delay in milliseconds added to sound played by your player 
after the sendspin clock synchronization. You can use this to adjust for slight
timing differences between different speakers.
