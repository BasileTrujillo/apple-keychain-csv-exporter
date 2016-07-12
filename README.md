# Exporting Apple Keychains credentials to a CSV file

Here is an adaptation and safe backup of the folowing tutorial : https://gist.github.com/rmondello/b933231b1fcc83a7db0b
 
## Copy the iCloud Keychain into a local Keychain

One’s iCloud Keychain is stored on disk in a different format than a traditional keychain. To access the credentials, I first created a traditional keychain with the iCloud Keychain’s contents. To do this, I clicked `File > New Keychain (⌥⌘N)` in Keychain Access. In my case, I saved the new keychain to the desktop. I clicked on iCloud in the sidebar, selected all of the passwords, and copied them. I selected the new keychain I just created and pasted the passwords.

Keychain Access prompted me for the “Local Items” keychain password for every password I was pasting. In my case, this would have been over 200 times!

## Automating typing the keychain password and clicking “OK”

I ran the `Auto-Auth` script to take care of this:

```applescript
set keychainPassword to "YourPassword"

tell application "System Events"
	repeat while exists (processes where name is "SecurityAgent")
		try
			tell process "SecurityAgent"
				set value of text field 1 of window 1 to keychainPassword
				click button "OK" of window 1
			end tell
		on error errMsg
			try
				tell process "SecurityAgent"
					click button "Toujours autoriser" of window 1
				end tell
			on error errMsg
				try
					tell process "Keychain Access"
						click button "OK" of window 1
					end tell
				end try
			end try
		end try
		-- delay 0.2
	end repeat
end tell
```

Whatever process is running this script (Script Editor or a standalone bundle), it’ll need permission to “control your computer”.

![Screenshot Security & Privacy > Privacy > Accessibility](http://f.cl.ly/items/0d3V070O3g2i1K1S413a/control.png)

After that runs, the recently-created local keychain should contain all of the passwords stored in iCloud Keychain.

## Write all of the passwords from the keychain to a file

I grabbed a copy of Daniel Jalkut’s [“Usable Keychain Scripting”](http://www.red-sweater.com/blog/2035/usable-keychain-scripting-for-lion) utility to help with the next part, but someone more sane might turn to `security`.

I ran the following script to write the passwords out to disk:

```applescript
set the logFile to ((path to desktop) as string) & "Passwords"
set keychainPath to "/Users/Dad/Desktop/dad.keychain"

-- write_to_file taken from http://www.macosxautomation.com/applescript/sbrt/sbrt-09.html
on write_to_file(this_data, target_file, append_data)
    try
        set the target_file to the target_file as string
        set the open_target_file to open for access file target_file with write permission
        if append_data is false then set eof of the open_target_file to 0
        write this_data to the open_target_file starting at eof
        close access the open_target_file
        return true
    on error
        try
            close access file target_file
        end try
        return false
    end try
end write_to_file

tell application "Usable Keychain Scripting"
    set keychainItems to get every keychain item of keychain keychainPath
    repeat with keychainItem in keychainItems
        set aServer to server in keychainItem
        set anAccount to account in keychainItem
        set aPassword to password in keychainItem

        set csvEntry to aServer & "," & anAccount & "," & aPassword & "
"

        my write_to_file(csvEntry, logFile, true)
    end repeat
end tell
```

There’s a lot that can be improved with this code. For instance, I could have used a consistent naming style between copied and non-copied code. If I took the time to look up an array or list "join" routine, the intent of the could could have been better communicated.

Here again, OS X’s Keychain wanted to do its job, prompting me to allow access for each of the 200+ items.

```applescript
-- Taken from a comment by Mr. X on http://selfsuperinit.com/2014/01/20/exporting-icloud-keychain-passwords-as-a-plain-text-file/
tell application "System Events"
    repeat while exists (processes where name is "SecurityAgent")
        tell process "SecurityAgent"
            click button "Allow" of window 1
        end tell
        delay 0.2
    end repeat
end tell
```

After that, I had my file. Inelegant, but it got the job done, and I had fun.