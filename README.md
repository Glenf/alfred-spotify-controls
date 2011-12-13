### Controls ####
# play, pause, stop, p, NULL
# mute, unmute, m
# next, n, >
# prev, pr, previous, <, <<
# << (takes you to the actual previous track, think double click)
# quit, end, kill, exit, e, q
# start, init, s
# now, i, current
# artist
# album
# disc
# duration, time, d
# count, plays
# track, t, name, song
# starred, star, fav
# popularity, rank, pop
# id
# url 
# search
################################
# Pause Example: spot pause
# Change Volume: spot 75
# Mute: spot mute
# Unmute: spot mute || spot unmute
# Start App: spot start
# Kill App: spot kill
# Current Track: spot current || spot now
# Track Info: spot duration || spot id || spot OPT
# Search: spot search de la soul
# Artist Only Search: spot search artist:de la soul
# Album Only Search: spot search album:stakes is high
# Track Only Search: spot search track:de la soul
### END ####

to calcTime(t)
	set m to (t div 60 as string)
	set s to t mod 60
	
	if s is less than 10 then
		set s to "0" & (s as string)
	else
		set s to (s as string)
	end if
	
	return m & ":" & s
end calcTime

on filterData(s, prepend, empty)
	if s is missing value or s is equal to "" or s is equal to "0" then
		if empty is true then
			return ""
		else
			return "Not Found"
		end if
	else
		return prepend & s
	end if
end filterData

on replaceText(find, replace, someText)
	set prevTIDs to text item delimiters of AppleScript
	set text item delimiters of AppleScript to find
	set someText to text items of someText
	set text item delimiters of AppleScript to replace
	set someText to "" & someText
	set text item delimiters of AppleScript to prevTIDs
	return someText
end replaceText

on sendMsg(nm, t, d, art)
	
	tell application "System Events"
		set isRunning to (count of (every process whose bundle identifier is "com.Growl.GrowlHelperApp")) > 0
	end tell
	
	if isRunning then
		tell application id "com.Growl.GrowlHelperApp"
			set the allNotificationsList to {nm}
			set the enabledNotificationsList to {nm}
			
			register as application "Spotify" all notifications allNotificationsList default notifications enabledNotificationsList icon of application "Spotify"
			
			if art is missing value or art is equal to "" then
				notify with name nm title t description d application name "Spotify" icon of application "Spotify"
			else
				notify with name nm title t description d application name "Spotify" image art
			end if
			
		end tell
	end if
	
end sendMsg

to splitString(aString, delimiter)
	set retVal to {}
	set prevDelimiter to AppleScript's text item delimiters
	log delimiter
	set AppleScript's text item delimiters to {delimiter}
	set retVal to every text item of aString
	set AppleScript's text item delimiters to prevDelimiter
	return retVal
end splitString

on alfred_script(q)
	
	set notify_name to "Track Information"
	set notify_title to ""
	set notify_desc to ""
	set notify_art to ""
	
	#Get i OPT
	if " i " is in q then
		try
			set tmp to my splitString(q, " ")
			set opt to item 2 of tmp
		on error
			set opt to "blank"
		end try
		
		#backwards compatibility
		if opt is not "blank" then
			set q to opt
		end if
	end if
	
	#Command Hashes
	set n to {"n", "next", ">"}
	set p to {"p", "play", "pause", "stop", ""}
	set pr to {"pr", "prev", "previous", "<", "<<"}
	set s to {"s", "start", "init"}
	set e to {"e", "q", "quit", "kill", "end", "exit"}
	set m to {"m", "mute", "unmute"}
	set i to {"i", "now", "current"}
	set t to {"t", "track", "name", "song"}
	set d to {"d", "duration", "time"}
	set c to {"count", "plays"}
	set f to {"star", "starred", "fav"}
	set po to {"pop", "popularity", "rank"}
	set ar to {"artist", "album_artist"}
	set h to {"help", "?"}
	
	
	tell application "Spotify"
		
		set rpeating to true
		
		try
			set notify_art to artwork of current track
		on error
			set notify_art to ""
		end try
		
		
		if q is in p then
			playpause
		else if q is in n then
			next track
			my alfred_script("i")
		else if q is in pr then
			previous track
			if q is equal to "<<" then
				previous track
			end if
			my alfred_script("i")
		else if q is in m then
			if sound volume is less than or equal to 0 then
				set sound volume to 100
			else
				set sound volume to 0
			end if
		else if q is in e then
			quit
		else if q is in s then
			activate
		else if q is in i then
			set c_album to my filterData(album of current track, " on ", true)
			set notify_title to name of current track & " (" & my calcTime(duration of current track) & ")"
			set notify_desc to "By " & artist of current track & c_album
			
		else if q is in ar then
			set arr to my filterData(artist of current track, "", false)
			set album_arr to my filterData(album artist of current track, "", true)
			if arr is equal to album_arr or album_arr is equal to "" then
				set notify_title to "Artist"
				set notify_desc to arr
			else
				set notify_title to "Artist / Album Artist"
				set notify_desc to "Artist: " & arr & "
Album Artist: " & album_arr
			end if
			
		else if q is equal to "album" then
			set notify_title to "Album Name"
			set notify_desc to my filterData(album of current track, "", false)
			
		else if q is equal to "disc" then
			set notify_title to "Disc Number"
			set notify_desc to my filterData((disc number of current track as string), "", false)
			
		else if q is in d then
			set notify_title to "Duration"
			set notify_desc to my calcTime(duration of current track)
			
		else if q is in c then
			set notify_title to "Play Count"
			set notify_desc to (played count of current track as string)
			
		else if q is in f then
			set notify_title to "Starred"
			if starred of current track is equal to true then
				set notify_desc to "Yes"
			else
				set notify_desc to "No"
			end if
			
		else if q is in po then
			set notify_title to "Popularity"
			set notify_desc to (popularity of current track as string) & " out of 100"
			
		else if q is equal to "id" then
			set notify_title to "ID"
			set notify_desc to id of current track
			
		else if q is in t then
			set num to my filterData((track number of current track as string), "", true)
			if num is not equal to "" then
				set num to " (#" & num & ")"
			end if
			
			set notify_title to "Current Track" & num
			set notify_desc to name of current track
			
		else if q is equal to "url" then
			set notify_title to "Spotify URL"
			set notify_desc to spotify url of current track
			
		else if "search" is in q then
			activate
			open location "spotify:search:" & my replaceText("search ", "", q)
			
		else if "app" is in q then
			activate
			open location "spotify:app:" & my replaceText("app ", "", q)
			
		else if q is equal to "shuffle" then
			if shuffling is enabled then
				set shuffling to false
			else
				set shuffling to true
			end if
			
		else if q is equal to "repeat" then
			if repeating is enabled then
				set repeating to false
			else
				set repeating to true
			end if
			
		else if q is equal to "dev" then
			set notify_title to "Developer Information"
			set notify_desc to "Jeff Johns | http://phpfunk.me | @phpfunk"
			set notify_art to ""
			
		else if q is in h then
			open location "https://github.com/phpfunk/alfred-spotify-controls/blob/spotify-0.8.0.873/README.md"
			
		else
			try
				(q as number) div 1
				set sound volume to q
			on error
				set notify_title to "Invalid Argument"
				set notify_desc to "The option '" & q & "' is invalid. Please try again."
				set notify_art to ""
			end try
		end if
	end tell
	
	if notify_desc is not equal to "" then
		set the clipboard to notify_desc as text
		sendMsg(notify_name, notify_title, notify_desc, notify_art)
	end if
end alfred_script