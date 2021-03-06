

# Useful programs
* `Inspect.exe` for looking at Window information (Windows 10 SDK)
* `ILSpy` is somewhat useful
* ProcessExplorer (Sysinternals)
* OLE Viewer


# Crunchyroll
Making some progress on this, definitely more than Funimation.
There are actual win32 UI objects (WinForms?) that are able to be emumerated
and interacted with. Using `Inspect.exe`, I was able to find the name of
currently playing episode, episodes in the list when in the menu, and player
elements. Might be able to get current time from the player elements,
and the show by constantly monitoring and saving some sort of state
when user starts watching. It'd be brittle, but better than nothing.


# Funimation
I suspect the episode "id" is incremented by two per episode since there may be two audo languages, English and Japanese.
The app doesn't make it easy to switch languages, however, so don't have a easy way to validate this hypothesis.
Either way, we can get the language from the filename.

### File Path example
```
\\AppData\\Local\\Packages\\FunimationProductionsLTD.FunimationNow_nat5s4eq2a0cr\\AC\\INetHistory
\\mms\\HUOPOA32\\1345116_English_431a16b9-c95a-e711-8175-020165574d09[1].dat
```

## Some examples
* Star Blazers 2202 Episode 6: 1757485 (English)
* Full Metal Panic Season 2 Episode 4: 1345116 (English)
* Full Metal Panic Season 2 Episode 7: 1345122 (English)
* Full Metal Panic Season 2 Episode 8: 1345124 (English)
* Full Metal Panic Season 2 Episode 9: 1345126 (English)
* Full Metal Panic Season 2 Episode 10: 1345128 (English)
* Full Metal Panic Season 3 Episode 1: 1755434 (English)

## Looking at the Funimation servers
Homepage IP (`nslookup funimation.com`)
* 107.154.106.169

No episode playing:
*  172.217.12.14 - This is a Google analytics server

### Playing episodes
*  107.154.108.80:443
*  143.204.31.65:443
*  172.217.12.14:80
*  172.217.12.4:443
*  38.122.56.118:443


# Format
Want to follow Discord's recommendations as much as possible here
https://discordapp.com/developers/docs/rich-presence/best-practices

### Ideal status information
*  Show name
*  Season number
*  Episode number
*  Episode name
*  Start time, current time, or end time (so we can show elapsed/remaining using startTimestamp/endTimestamp)

### Example
```
  Funimation
  Full Metal Panic!
  S2 E9 | Her Problem
  Elapsed: 6:03
```

### Format
* App name
* Details (this will wrap lines)
* Status (string)
* start (int)

# Windows service
So...wasted 45 minutes trying to get this darn thing to just start
Turns out you need to run post-install script for pywin32
Hopefully won't have to do this if I bundle as a exe...we'll see
```powershell
# Fix pywin32 service install
python 'C:\Program Files\Python36\Scripts\pywin32_postinstall.py' -install`

# To install as automatic service
python service.py --startup=auto install
```

## Pause/Continue/Shutdown
```python
# Uncomment these to implement "Pause" functionality
# def SvcPause(self):
#     pass
#
# def SvcContinue(self):
#     pass

# Uncomment this to implement logic that should occur at system shutdown
# def SvcShutdown(self):
#     pass
```

# Code snippets

## psutil
```python
from pprint import pprint
print(process)
pprint(process.open_files())
pprint(process.connections('all'))
print(process.exe())
print(process.cwd())
print(process.cmdline())
```

There seems to be some...interesting permissions on the executable,
making it only spawnable by Ye Olde svchost. Will likely need to
open it the "canonical" way, whatever that is for UWP apps.

```python
cmdline = process.cmdline()
logging.debug("Program cmdline: %s", ' '.join(cmdline))
```

## pyinstaller
```powershell
pyinstaller --clean -y --onefile --icon .\installer\astolfo.ico --specpath .\installer\ -n Astolfo --version-file .\installer\file_version_info.
txt .\astolfo.py

pyinstaller --clean -y .\installer\Astolfo.spec
```

# Errors
```
(presence) PS Astolfo>python .\astolfo.py --debug --verbose funimation
Exception ignored in: <object repr() failed>
Traceback (most recent call last):
  File "C:\Program Files\Python36\lib\asyncio\proactor_events.py", line 95, in __del__
    warnings.warn("unclosed transport %r" % self, ResourceWarning,
  File "C:\Program Files\Python36\lib\asyncio\proactor_events.py", line 54, in __repr__
    info.append('fd=%s' % self._sock.fileno())
  File "C:\Program Files\Python36\lib\asyncio\windows_utils.py", line 152, in fileno
    raise ValueError("I/O operatioon on closed pipe")
ValueError: I/O operatioon on closed pipe
```

# Credit

Credit to:
    Chris Umbel (chrisumbel.com/article/windows_services_in_python)
    Zen_Z (codeproject.com/Articles/1115336/Using-Python-to-Make-a-Windows-Service)
    pywin32 (github.com/mhammond/pywin32/win32/Demos/service/serviceEvents.py)
            (github.com/mhammond/pywin32/win32/Lib/win32serviceutil.py#L747)
    django-windows-tools (github.com/antoinemartin/django-windows-tools)
