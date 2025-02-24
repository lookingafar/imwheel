## This is a fork of imwheel utility for linux, originally made by Jonathan Atkins <jcatki@jonatkins.org>
> http://imwheel.sourceforge.net/

imwheel allows to grab input from the mouse, modify it and send it to the system.
For example, it's one of the easiest way to change the scrolling wheel speed on linux.
You can also modify mouse behaviour for selected programs.

Detailed instructions can be found here:
https://wiki.archlinux.org/index.php/IMWheel
					 

# Overview

This is a not so short little ditty that does the simple conversion of mouse
button presses into key presses.  By grabbing only the 4th and 5th mouse buttons
the program is able to receive input from the Intellimouse series mice.  There
is also a modified version of gpm included that has, already, support for the
Intellimouse, and I added a wheel fifo.  The user library may also be used,
although this may need to be debugged.  See the gpm section below for more.
The wheel button can always be used as middle button, this program does not
affect how the XServer, nor GPM, react towards it.  A mouse with a wheel button
is a 3 button mouse, and should be configured as such!

# Required


XFree86 >= 3.3.2 (or other XServer with wheel to mouse button support)
    not version 4.0.0 for gpm or jam use, FIFO support is broken there.
	it is fixed in 4.0.1 so upgrade if you are on 4.0.0
Intellimouse or Logitech MouseMan+ (other wheel mice work too...)

to compile from source:
    
The X11 include files and libraries. (Not the X server development packages)

# Installation

```
configure --help
configure (and any options you want to set)
make
make install
```
***

**READ THE MANPAGE FOR UP TO DATE DOCS!**                        
**everything after this point may help, but is not up to date.**

***

The mouse must be setup in XF86Config to send the mouse buttons 4 and 5 for 
wheel actions.  To do this:

## METHOD #1 : XGrabButton

1. edit /etc/XF86Config with your favorite editor (I suggest vim!  It's crunchy)
    
    * a. add the following line to the "Pointer" section.

        for 1 wheel:
            ZAxisMapping    4 5
        for two wheels or a stick:
            ZAxisMapping    4 5 6 7

    * b. Make sure your Protocol is set to either "IMPS/2" for a PS/2 mouse
       or for serial mice set it to "IntelliMouse" or "Auto".
       It should work for the following mice, which have wheels, knobs, or
       buttons that are mappable with ZAxisMapping (according to XFree86
       documentation):
       
* ASCII  (serial, PS/2)

* Genius NetScroll (PS/2)

* Genius NetMouse and NetMouse Pro (serial, PS/2)

* Logitech MouseMan+ and FirstMouse+ (serial, PS/2)


Half functionality may be obtained from the following mice that have
at least a fourth button capability:

* ALPS Glide-Point (serial, PS/2) ('Tapping' is button 4)**

The half functionality will only go up and left, unless you include the
-4 option on the command-line.


   * c. Here's my current setup for an example of a PS/2 Intellimouse:

```
        Section "Pointer"
            Protocol        "IMPS/2"        #change for your mouse type.
            Device          "/dev/psaux"    #change for your device.
            BaudRate        1200            #This is probably not needed!
            Resolution      100             #And neither is this!
            ZAxisMapping    4 5             #This is necessary!
            Buttons         3               #Use this instead of Emulate3 stuff!
        EndSection
```
* d. Restart XWindows if you need to!

2. make

3. make install
   This will currently install to /usr/local/bin, so edit the Makefile to
   change this behavior.
   Don't forget to choose whether you want imwheel to be setuid root.  This is
   required for any users who need to use imwheel and the pidfile stuff to track
   running imwheels.

4. After XWindows is started run imwheel like this:

`imwheel -k`

   imwheel will automatically background itself this way.
   I added the -k to force any old imwheel processes to stop.  This will not
   cause imwheel to die if there are no previous imwheels.

**XFree86 Settings:**
    Make sure Emulate3 phrases are either deleted or commented out to use the
wheel button as the middle button(2), and then the right button is button 3.

## STUPID FACT:
Obviously this method can be used for a real 5(+) button mouse as 
well...but why?!  The 4th and 5th buttons may be used as if they were the wheel 
rolling up and down, but this will remove the regular functionality of the 4th 
and 5th buttons, unless you "exclude" the windows in which you need the 4th and 
5th buttons.

## METHOD #2 : wheel fifo

This method is REQUIRED for Accelerated-X or any other server without wheel
mouse support built in.  It is also the recommended method by me! This method
will currently support ONLY the ps/2 Intellipoint Mice, not the serial version,
nor any other mice for that fact...unless you want to add support in gpm
yourself or send me a mouse so that I can implement it in gpm.  To add a new
mouse to this method no change to imwheel is required.

1. Edit /etc/XF86Config with your favorite editor (I suggest vim!  It's crunchy)
OR use XF86Setup.  That's easier!

    * b. Make sure your Protocol is set to "MouseSystems"
       Mouse support is dependent on gpm.  Right now I KNOW imps2 works!
       others that may work for you include mm+ps2, marblefx, pcnps2, and ms3.
       try others if these don't work or another matches your mouse better.

    * c. Here's my current setup for an example of a PS/2 Intellimouse:
```
        Section "Pointer"
            Protocol        "MouseSystems"  #gpm -R is MouseSystems!
            Device          "/dev/gpmdata"  #this is the gpm -R output
            BaudRate        1200            #works for me...
            Resolution      100             #i dunno...works again!
            Buttons         3               #use this,it's better than Emulate3!
                                            #if you set this higher...more
                                            #configuration is REQUIRED
        EndSection
```
   * d. Restart XWindows if you need to!

2. make

3. make install
   This will currently install to /usr/local/bin, so edit the Makefile to
   change this behavior.
   if you want ALL the gpm stuff enter that directory and "make install" there.
   The usual install only installs the gpm executable.
   Don't forget to choose whether you want imwheel to be setuid root.  This is
   required for any users who need to use imwheel and the pidfile stuff to track
   running imwheels.

4. After XWindows is started run :

`imwheel -k --wheel-fifo`

imwheel will automatically background itself this way.
I added the -k to force any old imwheel processes to stop.  This will not
cause imwheel to die if there are no previous imwheels.


**NOTE:** the @Exclude command has no bearing in this method, and is ignored.
mainly because by excluding a window in this method there is nothing
sent to the window for the wheel, whereas in the other mode 
exclusion turns off the button grabbing and allows the usual button 4 and
5 events again for those clients that either need to grab all the buttons
or want to receive the buttons 4 and/or 5 for their own purposes.

# gpm

The included gpm is stable, a patch is included as well, and I will try to keep 
up with the next stable versions.  Using the included gpm is only necessary if 
you are going to use method #2 from above.  To use gpm and imwheel together you 
must run gpm and imwheel as follows:
```
gpm -W -R -t mousetype(see above for known working types)
```
**NOTE:** gpm may be killed and restarted with imwheel running, imwheel and gpm
can both handle this interruption just fine.  When gpm is dead, both
the XServer and imwheel will not respond to the mouse.  You will need
to run gpm again!

Other options may be added of course!! use -w to force wheel. use gpm -h to see
them.

```
imwheel -k -W /dev/gpmwheel
```
or
```
imwheel -k --wheel-fifo
```
or
```
imwheel -k --wheel-fifo /dev/gpmwheel
```
or
```
imwheel -k --wheel-fifo /dev/jam_imwheel:0      # number zero may be higher
```

if you want ALL the gpm stuff enter that directory and "make install" there.
The usual install only installs the gpm executable.

**NOTE:** gpm will create the fifo if it needs to, imwheel will not.
/dev/gpmwheel is simple it output a 4 or 5 (byte not char!) for up and down.
You may use it in other programs if imwheel is not used...but why?  od the fifo
to watch your wheel at work (od is an octal dumper!), it requires a line for
output so wheel a lot to see your mouse actions!  The gpm currently included is
a hacked beta, which may sound bad, but it's been very stable for my imps2.

# JAM

    For jam to work with imwheel you will have to place the imwheel output 
module in the mice/file that you are using for your mouse.  see the mice/imps2 
file for an example.  See also the JAM documentation.  JAM will then output 
imwheel compatible streams on the /dev/jam_imwheel:* set of FIFOs, see JAM docs 
on multiple FIFO outputs.  JAM is the only way to use multiple running 
imwheels, one for each DISPLAY you may have, for multiple displays.  If you 
only have one screen then this doesn't really matter to you.
    XWheel is the full client to be paired with JAM and intended to replace 
imwheel.  IMWheel is now an aging product...only maintanance work is being done 
on imwheel and it's version/hack of GPM.

# .imwheelrc

## OVERVIEW

The configuration of specific clients is taken care of in the file called 
"$HOME/.imwheelrc" or "/etc/X11/imwheelrc".

All arguments in the file are separated by commas.

The pound(#) symbol may be used anywhere for comments. 

Case is VERY important in the configuration file!

## WINDOWS

In this file windows are configured by title name, resource name, or class name.

Each window is configured on multiple lines.  The first line in a window 
section is the window pattern string enclosed in double-quotes("), then one 
line each for each action in that window.

Double-quotes must surround window identifier strings, and only these strings 
are allowed to be quoted.  Without the double-quotes that window's section will 
be parsed as if it is in the previous window's section!

The windows are matched at run-time using regular expression syntax, so to 
match a group of windows make up the appropriate regex pattern and use that as 
the identifier.

## REGULAR EXPRESSIONS 

Regular expressions are used, for one reason, because they are built into C, and
the other, they are powerful!

One thing to note is that if you don't put a ^ (caret) at the beginning of each 
string then you may end up with a string matching a substring that doesn't 
begin with the expression.  Which is why in the distributed file most entries 
begin with a caret now.

O'Reilly & Associates has a great Regular expressions book that you can get, 
and it covers the type of expression syntax used here.

## COMMANDS

Commands are began with an at('@') character.  They apply to the window whose
section they are in.  Mixing commands and window wheel actions may achieve
undesired results.  For now mixing is moot, being that the only command is to
turn off wheel actions!

Available commands:
`@Exclude`
    
This will un-grab the mouse buttons for the current window, it will attempt
to re-grab the mouse buttons when the keyboard focus is changed.  Once the
focus window has changed from the excluded window, every motion of the mouse
will cause an attempt to re-grab the buttons.  The program will catch all
errors during this period.  Once the buttons are grabbed successfully the
program resumes normal wheel operation.
**NOTE:** This command is required for the "xv grab" window!
This command is unused with the --wheel-fifo option, see gpm.

## KEYSYMS 

The program expects combinations of keysyms to be used by using pipe(|) 
characters to combine them together.
Example: 
```
Alt_R|Shift_R
```
Means Right Alt AND Right shift together, not just either-or!

Modifier Keysym names used in X:
```
    Shift_L     Shift_R
    Control_L   Control_R
    Alt_L       Alt_R
```
  These are not currently assigned any keys, unless you xmodmap them in:
  ```
    Super_L     Super_R
    Hyper_L     Hyper_R
```
  And here's some that you may use, and they are somewhere on your keyboard!
  Here's where they were on my keyboard, again, this is not universal.
  Use the "xev" program to test your own keys on your keyboard!
```
Caps_Lock   =The Caps Lock key! (This still turns on and off caps lock!)
Num_Lock    =The Num Lock key!  (This is not good to use...)
Multi_key   =The scroll lock key! (Go figure!)
Mode_switch =Right Alt...for me anyways. (this mean I cannot use Alt_R)
```
**NOTE:**
    You can add support for the windows keys.  They may be set up for you as
Super_L and Super_R.  If not use xmodmap or better, Xkeycaps the graphical
frontend, to set them up as these Super keys.

To find keysym names for any keys available see the 
/usr/X11R6/include/X11/keysymdef.h file, and for any define in that file remove 
the "XK_" for the usable keysym name in the configuration file.

If no input keysyms are named, any input modifier keys will be ignored, and 
that action will match if the wheel action is matched!  Order matters here 
a lot!  Put a no-input-modifier keysym at the end of it's section for what you 
most likely want to happen!  The "None" keysym is preferred in this situation, 
however.

**"None"** is another special keysym used to indicate no input modifier is allowed. 
This is used to make the action not match when a modifier is pressed.

**"Pass"** is a special keysym used to tell imwheel to pass the button press 
through to the client window as if imwheel was not present. (Not implemented 
yet!)

The Modifier keysym(s) are the expected keys to be pressed along with the wheel 
actions.  If the modifier is not a match, that action is not done.

The second field is the mouse wheel action, "Up" or "Down".

The Output Keysyms (the second set of keysyms) are the keys that are pressed
when the following conditions are met:

1. Current Focused window (not a mouse over thing!) has a match as a title,
        resource, or class name to the window regex pattern specified as the
        window identifier.
2. Current pressed keys on keyboard match the Modifier Keysym(s),
        or there are no modifier keysyms to match.
3. The wheel is moving up or down and that matches the second argument.

## REPS 

Reps (Repetitions) lets you say a number for how many times you want the
output keysyms to be pressed.  See the chart on the default bindings for the
default number of reps for each modifier-combo (The chart is near the end of
this document).

## DELAYS 

Two delays are configurable in the "/etc/X11/imwheelrc" and/or the
"$HOME/.imwheelrc" files.  The first delay is the delay between repetitions.
The second delay is the delay between key down and key up events.  If you make
the delay too long expect the likelyness of the sticky keys bug to occur (see
BUGS file).  The delays are specified in microseconds.  see the included
imwheelrc file (which you should copy/merge) for the netscape configuration
that works best for me!

## FILES 

See the included imwheelrc file for a few examples of configured windows.  This 
file is a good start and may be copied into the users home directory as 
".imwheelrc" and/or into /etc/X11/imwheelrc for a default setup for all users.  
Then modify your individual copy or the /etc copy to add more windows.

## CONFIGURATION HELPER 

There is a "hidden" configuration helper that can be called up while running 
XWindows and imwheel.  The helper is called by wiggling the wheel up and down
in X's root window until it appears.
The helper is able to do the following:
1. Grab windows and display relevant configuration information, that is:
   Window title, Resource name, and Class name.  The class name is usually
   the best thing to use, but this is up to you to decide!
2. Grab wheel actions and display the modifiers to use in the imwheelrc file.
3. Reload the imwheelrc file, this actually restarts imwheel for you!

The Cancel button merely exits the helper, however the imwheelrc file is not
rescanned, so no changes will be reflected in continued wheel usage.

To grab a window, click on the image on the left, and then click on a window.
If the window can be used with imwheel it will appear in the big button.  This
is determined soley by window focus.  I use focus on mouse over so this works
great for me, however those who use click-focus may not see the expected
results.  It's just trying to help...

To grab a wheel action, press the button label as such and then press and hold
you desired modifier keys then roll the wheel in the direction to configure.
The wheel direction to use in the file will be shown with a list of the
modifiers below it.  Output keys can be grabbed somewhat similarly in this
manner however the shifted keys may need to be translated to the actual 
characters desired, this was not the case for XTerm though.

## PLEAD 

Please send your configuration additions to me at one of the addresses stated 
below for possible inclusion in further releases.  If deemed useful and 
sensible it will be added to the distribution.  Please don't write asking for 
help with the rc file!  Ask for features and bug fixes, sure, but not any more 
of a tutorial than this!

## SYNTAX 

The following syntax is available ('<' and '>' characters signify variables 
required to be filled in, '[' and ']' are optional variables):

blank lines are ignored, and not needed, as are comments ignored.
whitespace is ignored unless contained in quotes.
```
"<window identifier>"
[Modifier keysym(s)/None],<Up/Down>,[Keysym(s)/Pass],[Reps],[RepDelay],[DelayUp]
[Modifier keysym(s)/None],<Up/Down>,[Keysym(s)/Pass],[Reps],[RepDelay],[DelayUp]
[Modifier keysym(s)/None],<Up/Down>,[Keysym(s)/Pass],[Reps],[RepDelay],[DelayUp]
...more modifiers, etc...
"<next window identifier>"
@<Command>
"<next window identifier>"
[Modifier keysym(s)/None],<Up/Down>,[Keysym(s)/Pass],[Reps],[RepDelay],[DelayUp]

...and so forth...
```
The best example is availble in the supplied imwheelrc file, in the netscape 
section, there is a commented-out version of Up and Down that do smooth 
repeating up or down keys respectively and it is tuned to have a working delay 
for netscape to catchup.

# Options

Use the -h or --help option for all available options, and documentation.
--long-args can also be written as -long-args, but this sometimes causes other
combinations of short-args to fail due to ambiguity, just state them separately
to avoid the ambiguity.  (e.g.: -pdD => -p -d -D)
**Notation:** "<>" surrounds required args
"[]" surrounds optional args.
```
-4
--flip-buttons 
                Flips the mouse buttons so that 4 is 5 and 5 is 4, reversing
                the Up and Down actions.  This would make 4 buttons more useful!
                See also the manpage for xmodmap.
-f
--force
                Forces the X event subwindow to be used instead of the original
                hack that would replace the subwindow in the X event with a
                probed focus query (XGetInputFocus).  This should fix some
                compatability problems with some window managers, such as
                window maker, and perhaps enlightenment.  If nothing seems to be
                working right, try toggling this on or off...
-W <fifo-path>
--wheel-fifo [fifo-path]
                Use the gpm wheel fifo instead of XGrabMouse.  See gpm section.
                This method allows only one X display to be used.
                This is required for method #2 to work.
                @Exclude commands in the rc file are unused in this mode.
                fifo   names the named pipe created by gpm.
                       It defaults to "/dev/gpmwheel". (--wheel-fifo only)
                       Must exist before running imwheel in this mode.
-k
--kill          Attempts to kill old imwheel (written in --wheel-fifo method
                only.) Pidfile must be created for this to work.  Process is
                tested using /proc/${pid}/status Name: field ?= imwheel. 
                If /proc is not mounted then this fails everytime!
-p
--pid           Don't write a pidfile for --wheel-fifo method.  This is the only
                method that uses the pidfile.  XGrab doesn't need it, so it just
                issues a warning about starting multiple imwheels on the same
                display.

-X <display>
--display <display>
                use XServer at a specified display in standard X form.
                using this mode allows for multiple displays.
                this is useful in method #1.
-D
--debug         Show all possible debug info while running.  This spits out alot
                and I also suggest using the -d option to prevent imwheel from
                detaching from the controlling terminal.
-d
--detach        Actually this does the opposite, it prevents detachment from the
                controlling terminal.  (no daemon...)  Control-C stops, etc...
-h
--help          short help on options plus version/author info.
-q
--quit          Quit imwheel before entering event loop.  Usful in killing an
                imwheel running in --wheel-fifo mode after exiting XWindows.
                (e.g.: imwheel -k -q = kill and quit)
-s <sum-minimum>
--sensitivity <sum-minimum>
                Used with gpm only and then only with recognized stick mice.
                like -t only this sets a minimum total amount of movment of the
                stick or marble, before any action is taken.  This works good
                with the Marble type devices.  See also --threshhold.
-t <minimum-pressure>
--threshhold <minimum-pressure>
                Used with gpm only and then only with recognized stick mice.
                stick mice send a pressure value ranging from 0(no pressure) to
                7(hard push).  This sets the minimum required pressure for
                movement. setting it to zero will cause realtime sticking, which
                is usually too much action for X to keep up. (max rate i saw was
                100 events a second!)  The default is 2, to avoid slight presses
                on the 90degree direction of the intended while still getting to
                the intended direction.  Setting this to 7 is insane, because it
                requires the user to press the hardest everytime they want
                something to happen!  See also --sensitivity.

```
# Usage

The default modifier/wheel actions that are hard-coded will always be present
when an imwheelrc file doesn't provide a match or doesn't exist.
Default modifier effects chart:
```

    Modifier                Effect (Up, Down)           Repetitions
    --------                -----------------           -----------
    None                    Page_Up, Page_Down          1
    Shift                   Up, Down                    1
    Control                 Page_Up, Page_Down          2
    Shift+Control           Page_Up, Page_Down          5
    Meta(that's Alt!)       Left, Right                 10
    Shift+Meta              Left, Right                 1
    Control+Meta            Left, Right                 20
    Shift+Control+Meta      Left, Right                 50
```

For left and Right on wheels and sticks the repetitions are all that matter,
because they only go left and right by default.  This can be changed in the
imwheelrc.  Us the Left or Right keyword for the input action keyword to alter
left and right motions of a wheel or stick.

# Lefty X Usage

For left handed users this command may help at the start of X:
xmodmap -e "pointer = 3 2 1 4 5"

# Contact Info
>Jonathan Atkins
>2012 River Run Trail
>Fort Wayne, IN 46825-6041

jcatki@jonatkins.org
