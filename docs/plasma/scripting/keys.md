---
title: Configuration keys
weight: 3
aliases:
  - /docs/plasma/scripting/keys/
description: List of all the configurations keys available for the scripting API
---

# Configuration keys

Here you find a list of commonly used configuration keys to use with the `writeConfig()` command. Where the documentation notes that a key is in a subgroup other than `General`, remember to first use `currentConfigGroup`.

### org.kde.plasma.keyboardlayout

#### General

* displayStyle: (`Enum`, default: `0`) Whether to show text, a flag, or both on the compact representation.

### org.kde.plasma.kicker

#### General

* icon: (`String`, default: `start-here-kde-symbolic`) The name of the icon to use for the compact representation (e.g. on a small panel).
* useCustomButtonImage: (`Bool`, default: `false`) Whether to use a custom image instead of an icon in the compact representation (e.g. on a small panel).
* customButtonImage: (`Url`, default: `null`) The URL of the custom image to use instead of an icon in the compact representation (e.g. on a small panel).
* appNameFormat: (`Int`, default: `0`) The format used in the display of application names: 0 = NameOnly, 1 = GenericNameOnly, 2 = NameAndGenericName, 3 = GenericNameAndName
* limitDepth: (`Bool`, default: `false`) Whether to flatten top-level menu categories to a single level instead of displaying sub-categories.
* alphaSort: (`Bool`, default: `false`) Whether to sort menu contents alphabetically or use manual/system sort order.
* recentOrdering: (`Int`, default: `0`) How should the previously used apps/docs be ordered: 0 = RecentFirst, 1 = PopularFirst
* favoriteApps: (`StringList`, default: `preferred://browser,org.kde.kontact.desktop,systemsettings.desktop,org.kde.dolphin.desktop,org.kde.discover`) List of general favorites. Supported values are menu id's (usually .desktop file names), special URLs that expand into default applications (e.g. preferred://browser), document URLs and KPeople contact URIs.
* favoriteSystemActions: (`StringList`, default: `logout,reboot,shutdown`) List of system action favorites.
* favoritesPortedToKAstats: (`Bool`, default: `false`) Are the favorites ported to use KActivitiesStats to allow per-activity favorites
* hiddenApplications: (`StringList`, default: `empty list`) List of menu id's (usually .desktop file names) of apps that should not be shown in the menu.
* showRecentApps: (`Bool`, default: `true`) Whether to show the "Recent Applications" category.
* showRecentDocs: (`Bool`, default: `true`) Whether to show the "Recent Files" category.
* useExtraRunners: (`Bool`, default: `true`) Whether to use additional KRunner plugins to produce results in the search.
* extraRunners: (`StringList`, default: `krunner_shell,krunner_bookmarksrunner,baloosearch,locations`) The plugin id's of additional KRunner plugins to use. Only used if useExtraRunners is true.
* alignResultsToBottom: (`Bool`, default: `true`) Whether to align search results to the bottom of the menu representation (e.g. panel popup) instead of the top.
* showIconsRootLevel: (`Bool`, default: `false`) Whether to show icons on the root level of the menu.

### org.kde.plasma.kickoff

#### General

* icon: (`String`, default: `start-here-kde-symbolic`) The name of the icon used in the compact representation (e.g. on a small panel).
* menuLabel: (`string`, default: `null`) Text label for the Panel button
* favorites: (`StringList`, default: `preferred://browser,org.kde.kontact.desktop,systemsettings.desktop,org.kde.dolphin.desktop,org.kde.discover.desktop`) List of general favorites. Supported values are menu id's (usually .desktop file names), special URLs that expand into default applications (e.g. preferred://browser), document URLs and KPeople contact URIs.
* systemFavorites: (`StringList`, default: `suspend,hibernate,reboot,shutdown`) List of system action favorites.
* primaryActions: (`Int`, default: `0`) Which actions should be displayed in the footer: 0 = Power, 1 = Session, 2 = Custom, 3 = Power and session
* favoritesPortedToKAstats: (`Bool`, default: `false`) Are the favorites ported to use KActivitiesStats to allow per-activity favorites
* systemApplications: (`StringList`, default: `systemsettings.desktop,org.kde.kinfocenter.desktop,org.kde.discover.desktop`) List of applications at the top of the "Computer" tab.
* paneSwap: (`Bool`, default: `false`) Whether to swap the sidebar and content panes
* favoritesDisplay: (`Int`, default: `0`) How to display favorites: 0 = Grid, 1 = List
* applicationsDisplay: (`Int`, default: `1`) How to display applications: 0 = Grid, 1 = List
* alphaSort: (`Bool`, default: `false`) Whether to sort menu contents alphabetically or use manual/system sort order.
* pin: (`Bool`, default: `false`) Whether the popup should remain open when another window is activated
* showActionButtonCaptions: (`Bool`, default: `true`) Whether to display captions ("shut down", "log out", etc.) for the footer action buttons
* compactMode: (`Bool`, default: `false`) Whether to use a compact display style for list items

### org.kde.plasma.kimpanel

#### Appearance

* vertical\_lookup\_table: (`Bool`, default: `false`)
* use\_default\_font: (`Bool`, default: `true`)
* font: (`Font`, default: `null`)
* hiddenList: (`StringList`, default: `empty list`)
* scaleIconsToFit: (`bool`, default: `false`) Whether to automatically scale System Tray icons to fix the available thickness of the panel. If false, tray icons will be capped at the smallMedium size (22px) and become a two-row/column layout when the panel is thick.

### org.kde.plasma.marginsseparator

#### General

* icon: (`String`, default: `user-desktop`)

### org.kde.plasma.pager

#### General

* displayedText: (`Enum`, default: `2`) The text to show inside the desktop rectangles.
* showWindowOutlines: (`Bool`, default: `true`) Whether to show window outlines.
* showWindowIcons: (`Bool`, default: `false`) Whether to show window icons inside the window rectangles.
* showOnlyCurrentScreen: (`Bool`, default: `false`) Whether to limit the Pager to the set of windows and the geometry of the screen the widget resides on.
* wrapPage: (`Bool`, default: `false`) Whether to wrap page when navigating with pager
* currentDesktopSelected: (`Enum`, default: `0`) What to do on left-mouse click on a desktop rectangle.
* pagerLayout: (`Enum`, default: `0`) The layout style used for the presentation.

### org.kde.plasma.showActivityManager

#### General

* showActivityName: (`Bool`, default: `true`) Show the current activity name
* showActivityIcon: (`Bool`, default: `true`) Show the current activity icon instead of the applet icon

### org.kde.plasma.showdesktop

#### General

* icon: (`String`, default: `user-desktop-symbolic`)

### org.kde.plasma.taskmanager

#### General

* showOnlyCurrentScreen: (`Bool`, default: `false`) Whether to show only window tasks that are on the same screen as the widget.
* showOnlyCurrentDesktop: (`Bool`, default: `true`) Whether to only show tasks that are on the current virtual desktop.
* showOnlyCurrentActivity: (`Bool`, default: `true`) Whether to show only tasks that are on the current activity.
* showOnlyMinimized: (`Bool`, default: `false`) Whether to show only window tasks that are minmized.
* unhideOnAttention: (`Bool`, default: `true`) Whether to unhide if a window wants attention.
* groupingStrategy: (`Enum`, default: `1`) How tasks are grouped: 0 = Do Not Group, 1 = By Program Name
* groupedTaskVisualization: (`Enum`, default: `0`) What happens when clicking on a grouped task: 0 = cycle through grouped tasks, 1 = try to show tooltips, 2 = try to show present Windows effect, 3 = show textual list (AKA group dialog)
* groupPopups: (`Bool`, default: `true`) Whether groups are to be reduced to a single task button and expand into a popup or task buttons are grouped on the widget itself.
* onlyGroupWhenFull: (`Bool`, default: `true`) Whether to group always or only when the widget runs out of space to show additional task buttons comfortably.
* groupingAppIdBlacklist: (`StringList`, default: `empty list`) The id's (usually .desktop file names) of applications that should not have their tasks grouped.
* groupingLauncherUrlBlacklist: (`StringList`, default: `empty list`) The launcher URLs (usually .desktop file or executable URLs) of applications that should not have their tasks grouped.
* sortingStrategy: (`Int`, default: `1`) How to sort tasks: 0 = Do Not Sort, 1 = Manually, 2 = Alphabetically, 3 = By Desktop, 4 = By Activity
* separateLaunchers: (`Bool`, default: `true`) Whether launcher tasks are sorted separately at the left side of the widget or can be mixed with other tasks.
* hideLauncherOnStart: (`Bool`, default: `true`) Whether launcher tasks should be hidden when their application is launched.
* maxStripes: (`Int`, default: `1`) The maximum number of rows (in a horizontal-orientation containment, i.e. panel) or columns (in a vertical-orientation containment) to layout task buttons in.
* forceStripes: (`Bool`, default: `false`) Whether to try and always layout task buttons in as many rows/columns as set via maxStripes.
* showToolTips: (`Bool`, default: `true`) Whether to show tooltips when hovering task buttons.
* taskMaxWidth: (`Enum`, default: `1`) Tune the max width of task items.
* wheelEnabled: (`Bool`, default: `true`) Whether using the mouse wheel with the mouse pointer above the widget should switch between tasks.
* wheelSkipMinimized: (`Bool`, default: `true`) Whether to skip minimized tasks when switching between them using the mouse wheel.
* highlightWindows: (`Bool`, default: `true`) Whether to request the window manager highlight windows when hovering corresponding task tooltips.
* launchers: (`StringList`, default: `applications:systemsettings.desktop,applications:org.kde.discover.desktop,preferred://filemanager,preferred://browser`) The list of launcher tasks on the widget. Usually .desktop file or executable URLs. Special URLs such as preferred://browser that expand to default applications are supported.
* middleClickAction: (`Enum`, default: `2`) What to do on middle-mouse click on a task button.
* indicateAudioStreams: (`Bool`, default: `true`) Whether to indicate applications that are playing audio including an option to mute them.
* fill: (`Bool`, default: `true`) Whether task manager should occupy all available space.
* taskHoverEffect: (`Bool`, default: `true`) Whether task buttons should change in appearance when the mouse pointer is above them.
* maxTextLines: (`Int`, default: `0`) The maximum number of text lines to show in a task button. 0 means no limit.
* minimizeActiveTaskOnClick: (`Bool`, default: `true`) Whether to minimize the currently-active task when clicked. If false, clicking on the currently-active task will do nothing.
* reverseMode: (`Bool`, default: `false`) Whether to grow the tasks in according to system configuration or opposite to system configuration.
* iconSpacing: (`Int`, default: `1`) Spacing between icons in task manager. Margin is multiplied by this value.

### org.kde.plasma.windowlist

#### General

* showText: (`Bool`, default: `true`) Whether to show the window title when the applet is used on a horizontal panel

### org.kde.plasma.binaryclock

#### General

* showOffLeds: (`Bool`, default: `true`)
* showSeconds: (`Bool`, default: `true`)
* useCustomColorForActive: (`Bool`, default: `false`)
* customColorForActive: (`Color`, default: `green`)
* useCustomColorForInactive: (`Bool`, default: `false`)
* customColorForInactive: (`Color`, default: `red`)
* useCustomColorForGrid: (`Bool`, default: `false`)
* customColorForGrid: (`Color`, default: `blue`)

### org.kde.plasma.colorpicker

#### General

* history: (`StringList`, default: `empty list`)
* autoClipboard: (`Bool`, default: `true`)
* defaultFormat: (`String`, default: `#RRGGBB`)
* pickOnActivate: (`Bool`, default: `true`)
* compactPreviewCount: (`Int`, default: `1`)

### org.kde.plasma\_applet\_dict

#### General

* dictionary: (`String`, default: `all`)

### org.kde.plasma.fifteenpuzzle

#### Appearance

* imagePath: (`String`, default: `empty string`)
* useImage: (`Bool`, default: `false`)
* showNumerals: (`Bool`, default: `true`)
* boardColor: (`Color`, default: `#333333`)
* numberColor: (`Color`, default: `#ffffff`)
* boardSize: (`Int`, default: `4`)

### org.kde.plasma.fuzzyclock

#### Appearance

* fuzzyness: (`Int`, default: `1`)
* boldText: (`Bool`, default: `false`)
* italicText: (`Bool`, default: `false`)

### org.kde.plasma.keyboardindicator

#### General

* key: (`StringList`, default: `Caps Lock`)

### org.kde.plasma.mediaframe

#### General

* interval: (`Double`, default: `10.0`)
* randomize: (`Bool`, default: `true`)
* pauseOnMouseOver: (`Bool`, default: `true`)
* leftClickOpenImage: (`Bool`, default: `true`)
* showCountdown: (`Bool`, default: `true`)
* fillMode: (`int`, default: `1`)

#### Paths

* pathList: (`StringList`, default: `empty list`)

### org.kde.plasma.notes

#### General

* color: (`String`, default: `yellow`)
* fontSize: (`int`, default: `null`)
* noteId: (`String`, default: `empty string`)
* scrollX: (`Double`, default: `0`)
* scrollY: (`Double`, default: `0`)
* cursorPosition: (`Int`, default: `0`)

### org.kde.plasma.quicklaunch

#### General

* maxSectionCount: (`Int`, default: `1`)
* showLauncherNames: (`Bool`, default: `false`)
* enablePopup: (`Bool`, default: `false`)
* title: (`String`, default: `empty string`)
* launcherUrls: (`StringList`, default: `empty list`)
* popupUrls: (`StringList`, default: `empty list`)

### org.kde.plasma.timer

#### General

* predefinedTimers: (`StringList`, default: `30,60,120,300,450,600,900,1200,1500,1800,2700,3600`)
* running: (`int`, default: `0`)
* seconds: (`int`, default: `0`)
* savedAt: (`DateTime`, default: `null`)
* showTitle: (`Bool`, default: `false`)
* title: (`String`, default: `Timer`)
* showRemainingTime: (`Bool`, default: `true`)
* showSeconds: (`Bool`, default: `true`)
* showTimerToggle: (`Bool`, default: `true`)
* showProgressBar: (`Bool`, default: `false`)
* showNotification: (`Bool`, default: `true`)
* notificationText: (`String`, default: `Timer finished`)
* runCommand: (`Bool`, default: `false`)
* command: (`String`, default: `empty string`)

### org.kde.plasma.userswitcher

#### General

* showFace: (`Bool`, default: `false`)
* showName: (`Bool`, default: `true`)
* showFullName: (`Bool`, default: `true`)
* showTechnicalInfo: (`Bool`, default: `false`)

### org.kde.plasma.weather

#### WeatherStation

* updateInterval: (`int`, default: `30`)
* source: (`string`, default: `null`)

#### Appearance

* showTemperatureInCompactMode: (`bool`, default: `false`)
* showTemperatureInBadge: (`bool`, default: `false`)
* showTemperatureInTooltip: (`bool`, default: `true`)
* showWindInTooltip: (`bool`, default: `true`)
* showPressureInTooltip: (`bool`, default: `false`)
* showHumidityInTooltip: (`bool`, default: `true`)

#### Units

* temperatureUnit: (`int`, default: `null`)
* pressureUnit: (`int`, default: `null`)
* speedUnit: (`int`, default: `null`)
* visibilityUnit: (`int`, default: `null`)

### org.kde.plasma.webbrowser

#### General

* useDefaultUrl: (`Bool`, default: `false`)
* defaultUrl: (`String`, default: `https://www.kde.org/`)
* url: (`String`, default: `https://www.kde.org/`)
* useMinViewWidth: (`Bool`, default: `true`)
* minViewWidth: (`Int`, default: `600`)
* constantZoomFactor: (`Int`, default: `100`)
* icon: (`String`, default: `empty string`) The name of the icon used in the compact representation (e.g. on a small panel).
* favIcon: (`String`, default: `empty string`) The url of the website's favicon that is saved in the last session.
* useFavIcon: (`Bool`, default: `true`) Use the website's favicon instead of a system icon
* enableNavigationBar: (`Bool`, default: `true`)

### org.kde.plasma.analogclock

#### General

* showSecondHand: (`Bool`, default: `false`)
* showTimezoneString: (`Bool`, default: `false`)

### org.kde.plasma.appmenu

#### Appearance

* compactView: (`Bool`, default: `false`) If true it only shows a button for the application menu.

### org.kde.plasma.calendar

#### Agenda

* startOfWorkingDay: (`int`, default: `9`)
* endOfWorkingDay: (`int`, default: `17`)
* showWeekNumbers: (`Bool`, default: `false`)
* compactDisplay: (`String`, default: `d`)

### org.kde.plasma.clipboard

#### General

* barcodeType: (`String`, default: `QRCode`)

### org.kde.plasma.devicenotifier

#### General

* removableDevices: (`Bool`, default: `true`) If true it lists removable devices, such as USB thumbdrives. Only one between removableDevices, nonRemovableDevices and allDevices should be set.
* nonRemovableDevices: (`Bool`, default: `false`) If true it lists non removable devices, such as internal harddrives. Only one between removableDevices, nonRemovableDevices and allDevices should be set.
* allDevices: (`Bool`, default: `false`) If true it lists all kind of devices. Only one between removableDevices, nonRemovableDevices and allDevices should be set.
* popupOnNewDevice: (`Bool`, default: `true`) If true it tries to open the plasmoid when a new device is inserted, as a kind of notification.

### org.kde.plasma.digitalclock

#### Appearance

* showLocalTimezone: (`Bool`, default: `false`) Whether the time zone should be displayed when the clock is showing the local time zone.
* showSeconds: (`Enum`, default: `1`) How seconds should be shown in the clock.
* showDate: (`Bool`, default: `true`) Whether the date should be shown next to the clock.
* dateFormat: (`string`, default: `shortDate`) The date format to display. Options are: shortDate, longDate, isoDate or custom.
* customDateFormat: (`string`, default: `ddd d`) Custom date format string.
* autoFontAndSize: (`Bool`, default: `true`) Use Plasma default font and automatically determine font size.
* fontFamily: (`string`, default: `null`) Font family. e.g "arial". The system font is used if this is not set.
* boldText: (`Bool`, default: `false`) Sets the font to bold.
* italicText: (`Bool`, default: `false`) Sets the font to italic.
* fontWeight: (`Int`, default: `50`) Sets font weight.
* fontStyleName: (`string`, default: `null`) Sets font style
* fontSize: (`Int`, default: `10`) Sets font size.
* timeFormat: (`string`, default: `default`)
* selectedTimeZones: (`StringList`, default: `Local`) A list of the configured time zones. Format is "Europe/London". Special entry "Local" indicates system time zone.
* lastSelectedTimezone: (`String`, default: `Local`) When multiple time zones are configured, this is the one shown on widget restore. Typically the system's current time zone.
* wheelChangesTimezone: (`Bool`, default: `false`) Whether the mouse wheel switches between the time zones configured in selectedTimeZones.
* displayTimezoneFormat: (`Enum`, default: `0`) Whether the time zone is displayed as a code i.e. "GMT", full text i.e. "London" or UTC offset "+1".
* showWeekNumbers: (`Bool`, default: `false`) Whether the calendar should show week numbers.
* use24hFormat: (`UInt`, default: `1`) Force the clock to use 12/24 hour time, instead of following the user locale.
* firstDayOfWeek: (`Int`, default: `-1`) Force the calendar to use a specific week day as first day of a week. -1 means follow user locale, 0 is Sunday, 1 is Monday, etc.
* enabledCalendarPlugins: (`StringList`, default: `empty list`) A list of plugins where additional calendar event data can be sourced.
* pin: (`Bool`, default: `false`) Whether the popup should remain open when another window is activated
* dateDisplayFormat: (`Enum`, default: `0`) Whether the date should be shown below or beside the time

### org.kde.plasma.icon

#### General

* url: (`String`, default: `empty string`)
* localPath: (`String`, default: `empty string`)

### org.kde.plasma.lock\_logout

#### General

* show\_requestShutDown: (`Bool`, default: `false`) Show an option to shut down the system.
* show\_requestReboot: (`Bool`, default: `false`) Show an option to reboot the system.
* show\_requestLogout: (`Bool`, default: `false`) Show an option to logout.
* show\_requestLogoutScreen: (`Bool`, default: `true`) Show an option to display the logout screen.
* show\_lockScreen: (`Bool`, default: `true`) Show an option to lock the system.
* show\_switchUser: (`Bool`, default: `false`) Show an option to switch user.
* show\_suspendToDisk: (`Bool`, default: `false`) Show an option to suspend the system to disk (hibernate).
* show\_suspendToRam: (`Bool`, default: `false`) Show an option to suspend the system suspend.

### org.kde.plasma.panelspacer

#### General

* expanding: (`bool`, default: `true`) If true, the spacer tries to take all the available space in the panel.
* length: (`Int`, default: `-1`) length in pixels of the spacer. Configuration effective only if expanding is set to false. A negative value means an invalid value that should be completely ignored.
