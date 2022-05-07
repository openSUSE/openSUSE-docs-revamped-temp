## KDE
KDE is an international technology team that creates free and open source software for desktop and portable computing.
Plasma, made by KDE is the default graphical desktop environment of openSUSE.
The latest version, Plasma 5, is a fresh, elegant and powerful desktop for both beginner and advanced users.

KDE software is not limited to a desktop environment. The software made by the community includes:

* Plasma, the graphical desktop.
* KDE Applications, a collection of a wide variety of applications for communication, work, education and entertainment, including famous programs like Kate.
* Additional high-quality applications, such as DigiKam and Krita.
* KDE Frameworks, a series of modules to easily build new applications upon.

What makes KDE software on openSUSE special:
* It has a team of dedicated engineers who spend much of their time fixing bugs so you can rely on KDE.
* It has a large, active and highly experienced community team around it who bring multiple points of view and different interests to the project so that KDE software
 on openSUSE meets many users' needs.
* Through its innovation it stimulates the openSUSE distribution to develop, evolve and progress.

### X11, XWayland & Wayland sessions

### Settings & Keybindings

KDE settings can be customized through the __System Settings__ application found in
_Application Launcher --> Settings --> System Settings_. A convenient shortcut for System Settings can be found on the system panel as well.
KDE is extremely customizable,to find more specific help visit the [Plasma Wiki](https://userbase.kde.org/Plasma)

Keybindings or keyboard shortcuts are an efficient way to interact with KDE without having to use the mouse.

In __System Settings__ click __Shortcuts__ under the "Workspace" category.
You will be able to find any pre-existing shortcuts for many KDE applications as well as create your own shortcuts for any applications you install yourself.
The __Custom Shortcuts__ menu allows you to create shortcuts for almost any system task.
<br><br>
The Shortcuts window has three main areas.

* The shortcut categories, __Shortcuts__ and __Custom Shortcuts__.
* List of applications or custom shortcuts.
* Shortcut editing area.

![Shortcut Settings](image/Shortcuts.png)

Many KDE applications are already in the list. If you want to add additional applications simply
click __Add Application__ and make a selection from the list. Click __OK__ to confirm the selection.

![Choose Application](image/Shortcuts_ChooseApplication.png)

Once this step is done, the new application will be displayed in the Applications column.

#### Changing Keyboard Shortcuts

Select the application you want to change the shortcut for from the Applications column. The list of shortcut actions will be displayed in the editing area on the right. Click on any shortcut action to expand its settings (if there is only one shortcut available it will be automatically expanded).
<br>
Shortcut actions can have default and custom keyboard shortcuts. You can disable the default by un-marking the box next to it or leave it active alongside any custom shortcuts you choose.

Click __Add custom shortcut__, the button will change to indicate it is waiting for the next key combination, whatever keys are pressed next will be recorded as the custom shortcut. Click the "cancel" button, which has now appeared, if you change your mind. When a custom shortcut is recorded, you have the option to delete it by clicking the "trash" icon or additional custom shortcuts can be added.
<br>

![Modify Shortcuts](image/Shortcuts_Spectacle.png)

Once you are satisfied with the changes, click __Apply__ in the bottom right corner of the shortcuts window.

#### Adding Custom Shortcuts

Select the __Custom Shortcuts__ category from the left hand column of the __Shortcuts__ settings window. Custom shortcuts allow you to trigger your own events from simple to complex that would not normally be available. For demonstration purposes we are going to setup a keyboard shortcut for a program called _Rofi_. You may not want to use this program yourself but the process is the same for other programs.
<br><br>
First we will add a new __Custom Shortcut__ by clicking _Edit --> New --> Global Shortcut --> Command/URL_. A new entry will be added to the list in the middle column.

![New Custom Shortcut](image/CustomShortcuts_AddNew.png)

The entry will be highlighted and its name selected so you can rename it immediately just by typing. If you want to change the name later on, simply double-click the name in the list and start typing.

![Rename Custom Shortcut](image/CustomShortcuts_NameNew.png)

Using a name that is descriptive will help us remember what this custom shortcut does. In this case, the program _Rofi_ will display a list of open applications and switch between their windows with the keyboard.
<br><br>
Now add the command that will be run when the keyboard shortcut is pressed. When an action is selected from the middle column, the right hand area will change to the editing view. You will see tabs near the top for __Comment__, __Trigger__, and __Action__. Click the __Action__ tab and add the command to the text box labeled "Command/URL".

![Add Custom Command](image/CustomShortcuts_Command.png)

Next click the __Trigger__ tab. This is where the keyboard shortcut that triggers the command will be set. Click the button labeled "Shortcut" which will display the current keyboard shortcut if there is one. The button changes color and waits for you to type the desired shortcut.

![Add Trigger](image/CustomShortcuts_Trigger.png)

If this keyboard shortcut is already used, a warning will be displayed. Click "Cancel" to try a different shortcut or click "Reassign" if you wish to use it anyway. For this demonstration, "Reassign" is selected. Click __Apply__ at the bottom right of the window.
<br><br>
Now the custom keyboard shortcut runs the command system wide, no matter what applications are open.

![Demonstrate Custom Shortcut](image/CustomShortcuts_Demonstration.png)

#### Using panes, bars, widgets & docks

#### Learning more


## References & credits
- [KDE official documentation & authors](https://docs.kde.org/)