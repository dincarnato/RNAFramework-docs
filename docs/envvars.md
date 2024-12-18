Since v2.9.0, RNA Framework exports a number of environment variables, that will allow controlling specific behaviours of the framework.<br/><br/>


### RF_NOCHECKUPDATES

__Description:__ Disables check for updates from the RNA Framework's git repository at programs' startup.<br/>
__Accepted values:__<br/>
&nbsp;__0__ (update check enabled) [__default__]<br/>
&nbsp;__1__ (update check disabled)<br/>

__Example:__

```bash
$ export RF_NOCHECKUPDATES=1     # Disables check for updates
```
<br/>

### RF_VERBOSITY

__Description:__ Controls the level of verbosity of RNA Framework's warnings and exceptions.<br/>
__Accepted values:__<br/>
__-1__ (no warnings, without stack trace)<br/> 
&nbsp;__0__ (warnings and exceptions, without stack trace) [__default__]<br/>
&nbsp;__1__ (warnings and exceptions, with stack trace)<br/>

__Example:__

```bash
$ export RF_VERBOSITY=0     # Default, without stack trace

# [!] Exception [Core::Process::Queue->start()]:
#     Data looks binary, but binMode is not set to ":raw"
#     -> Caught at lib/Core/Base.pm line 118.



$ export RF_VERBOSITY=1     # With stack trace

# [!] Exception [Core::Process::Queue->start()]:
#     Data looks binary, but binMode is not set to ":raw"
#
#     Stack dump (descending):
#
#     [*] Package:    main
#         File:       rf-fold
#         Line:       502
#         Subroutine: Core::Process::Queue::start
# 
#     [*] Package:    Core::Process::Queue
#         File:       lib/Core/Process/Queue.pm
#         Line:       121
#         Subroutine: Core::Process::start
# 
#     [*] Package:    Core::Process
#         File:       lib/Core/Process.pm
#         Line:       108
#         Subroutine: main::fold
# 
#     [*] Package:    main
#         File:       rf-fold
#         Line:       981
#         Subroutine: (eval)
#         
#     -> Caught at lib/Core/Base.pm line 118.
```
<br/>

### RF_RPATH

__Description:__ Path to the R executable to be used by RNA Framework's modules for plotting.<br/>
__Accepted values:__ absolute path to `R`<br/>

__Example:__

```bash
$ export RF_RPATH=/usr/local/bin/R 
```
<br/>

