# SonosLua
Lua library to control Sonos over local LAN using websockets, using the Fibaro HC3 web socket model

```lua
----- Sonos commands
sonos:play(playerName)                  -- Start playing group that player belong to
sonos:pause(playerName)                 -- Pause group that player belong to
sonos:volume(playerName,volume)         -- Set volume to group that player belong to
sonos:relativeVolume(playerName,delta)  -- Set relative volume to group that player belong to
sonos:mute(playerName,state)            -- Mute group that player belong to
sonos:togglePlayPause(playerName)       -- Toggle play/pause group that player belong to
sonos:skipToNextTrack(playerName)       -- Skip to next track in group that player belong to
sonos:skipToPreviousTrack(playerName)   -- Skip to previous track in group that player belong to
sonos:playFavorite(playerName,favorite,action,modes) -- Play favorite on group that player belong to
sonos:playPlaylist(playerName,playlist,action,modes) -- Play playlist on group that player belong to
sonos:playerVolume(playerName,volume)   -- Set volume of player
sonos:playerRelativeVolume(playerName,delta)         -- Set relative volume of player
sonos:playerMute(playerName,state)      -- Mute player
sonos:clip(playerName,url,volume)       -- Play audio clip on player
sonos:say(playerName,text,volume,lang)  -- Play TTS on player
sonos:playerGroup(playerName)           -- group that player belong to
sonos:playersInGroup(playerName)        -- players in group that player belong to
sonos:createGroup(playerNames,...)      -- group players.
sonos:removeGroup(groupName)            -- remove group. Ex. sonos:removeGroup(sonos:playerGroup(playerName))
sonos:getPlayer(playerName)             -- Get player object. Ex. p = sonos:getPlayer(playerName); p:pause()
sonos:cb(cb) 
```

Example of use:
```lua
local function delay(args)
  local t=0
  for i=1,#args,4 do
    local d,cond,f,doc=args[i],args[i+1],args[i+2],args[i+3]
    local function f0() print(">"..doc) f() end
    if cond then t=t+d setTimeout(f0,1000*t) end
  end
end

local clip = "https://github.com/joepv/fibaro/raw/refs/heads/master/sonos-tts-example-eng.mp3"
    function Sonos:eventHandler(event)
      print(event) -- Just print out events, could be used to ex. update UI
    end
    Sonos("192.168.1.6",function(sonos)
      self:debug("Sonos Ready")
      print("Players:",sonos.playerNames)
      print("Groups:",sonos.groupNames)
      local playerA = sonos.playerNames[1]
      local playerB = sonos.playerNames[2]
      local PA = sonos:getPlayerObject(playerA)
      print("PlayerA:",PA.name,PA.id)
      local PB = sonos:getPlayerObject(playerB)
      print("PlayerB:",PB.name,PB.id)
      print("PB:",PB)
      local favorite1 = (sonos.favorites[1] or {}).name
      local playlist1 = (sonos.playlists[1] or {}).name
      print(("PlayerA='%s', PlayerB='%s'"):format(playerA,playerB))
      print(("Favorite1='%s', Playlist1='%s'"):format(favorite1,playlist1))
      local function callback(headers,data) -- not used
        print("Callback",headers,data)
      end
      delay{
        -- 1,playerA,function() sonos:say(playerA,"Hello world",25) end, "TTS clip to player",
        -- 2,playerB,function() sonos:say(playerB,"Hello world again",25) end, "TTS clip to player",
        -- 2,playerA,function() sonos:clip(playerA,clip,25) end, "Audio clip to player with volume",
        -- 2,playerA,function() sonos:play(playerA) end, "Play group that player belongs to",
        -- 2,playerA,function() sonos:pause(playerA) end, "Pause group that player belongs to",
        -- 2,playerB,function() sonos:play(playerB) end, "Play group that player belongs to",
        --2,playerB,function() sonos:cb(callback):pause(playerB) end, "Pause group that player belongs to",
        -- 2,playerB and favorite1,function() sonos:playFavorite(playerB,favorite1) end, "Play favorite in group that player belongs to",
        -- 4,playerB,function() sonos:pause(playerB) end, "Pause group that player belongs to",
        -- 4,playerB and playlist1,function() sonos:playPlaylist(playerB,playlist1) end, "Play favorite in group that player belongs to",
        -- 4,playerB,function() sonos:pause(playerB) end, "Pause group that player belongs to",
        -- 5,playerA and playerB,function() sonos:createGroup(playerB,playerA) end, "Create group with players",
        -- 10,playerA,function() sonos:play(playerA) end, "Play group that playerA belongs to  (both players)",
        -- 4,playerA,function() sonos:removeGroup(sonos:playerGroup(playerA)) end, "Destroy group playerA belongs to",
        -- 4,playerA,function() sonos:play(playerA) end, "Play group that playerA belongs to (only playerA)",
      }
      -- sonos:volume(playerA,40) -- set volume to group that player belongs to
      -- sonos:playerVolume(playerA,30) -- set player volume
      -- local pl = sonos:getPlayer(playerA)
      -- pl:pause()
      -- local group = sonos:playerGroup(playerA) -- get group that player belongs to
      -- local players = sonos:playersInGroup(sonos:playerGroup(playerA)) -- get players in group
    end,{socket=true}) -- debugFlags
  end
  ```