---
title: Updating KConfig files
draft: true
---

# Updating KConfig files

### Introduction

Sometimes you want to change the way how your application stores configuration information. This can have many reasons. Maybe you want to split your application into multiple components, or join multiple components to one component. Or you just think that the old configuration file format didn’t make much sense.

If you change the configuration file format, you need a way to migrate configuration information from the old format to the new format. You can add code to your application, which looks up both the old configuration and the new configuration, and then decides which one to use. However, if your intention is to simplify your application’s code, this will actually make your code more complicated.

If you use KConfig to store configuration information, there is a tool which can update your configuration files without adding complexity to your application code: kconf\_update. kconf\_update makes it easy to move configuration information from one place to another, but it is also possible to change the configuration data itself.

### How it works

Applications can install so called update files in `${KDE_INSTALL_KCONFUPDATEDIR}`. An update file has .upd as extension, and contains instructions for transferring and converting configuration information. Update files are separated into sections, and each section has an unique ID.

kconf\_update is started automatically when KDED detects a new update file in the above mentioned location. kconf\_update will first check in its own configuration file (kconf\_updaterc, usually located in \~/.config/) which sections of the update file haven’t been processed yet. When a section has been processed, its ID is stored in kconf\_updaterc and the affected configuration files, to make sure it will not be processed again.

If you overwrite an existing update file with a new version that contains a new section, only the update instructions from this new section will be processed.

If your application does not depend explicitly on KDED (e. g. if your target platform isn’t Linux or FreeBSD), kconf\_update may not be started automatically. You can start it manually using KConfig::checkUpdate(). Note that kconf\_update might process your update multiple times simultaneously in this case. This can cause problems if your update file uses the Script= command. See Development/Tools/Using\_kconf\_update#Note on side effects for more details. How to use\[edit]

To use kconf\_update in your application, you only need to write an update file and install it in the right place. An example update file is in the section #Moving configuration data around.

Add the following lines to your CMakeLists.txt:

```cmake
find_package(ECM ${KF5_REQUIRED_VERSION} CONFIG REQUIRED)
# Extra CMake Modules (ECM) component KDEInstallDirs
# defines variables like KDE_INSTALL_KCONFUPDATEDIR:
include(KDEInstallDirs)

# Makes your application compile with KConfig:
find_package(KF5 ${KF5_REQUIRED_VERSION} REQUIRED COMPONENTS
    Config
    [...]
)

# Installs the update file in the right place:
install(FILES okular.upd DESTINATION ${KDE_INSTALL_KCONFUPDATEDIR})
```

\{{< alert color="warning" title="Warning" >\}}

Before you deploy update files to your users, you should verify that they do what you want. Otherwise you will probably get bug reports, but you already missed your change to fix them. See the #Testing update files section.

\{{< /alert >\}}

### Moving configuration data around

Imagine your need to change the name of one entry in your configuration file. This happened in Okular, and this is the content of ${KDE\_INSTALL\_KCONFUPDATEDIR}/okular.upd:

```ini
#Configuration update for Okular
Version=5

#Convert user-defined annotation tools to quick annotation tools
Id=annotation-toolbar
File=okularpartrc
Group=Reviews
Key=AnnotationTools,QuickAnnotationTools
```

`#Configuration update for Okular`

Empty lines and lines that start with # are considered comments.

`Version=5`

Specifies that this update file is written for the KF5 version of kconf\_update. Every update file must start with this command.

`Id=annotation-toolbar`

Starts an update section called annotation-toolbar. You should give your section a name that describes why this update necessary. You can also include the year of your changes, like in Id=annotation-toolbar-2020.

`File=okularpartrc`

Tells kconf\_update which configuration file shall be affected.

`Group=Reviews`

Tells kconf\_update that it shall operate on the Reviews group of the configuration file.

`Key=AnnotationTools,QuickAnnotationTools`

Tells kconf\_update that it shall read from the entry AnnotationTools and write to the entry QuickAnnotationTools. In this case, this causes the entry to be renamed.

You can find the full command reference in [Update file command reference](../../../content/docs/features/configuration/Update\_file\_command\_reference/).

This update file will change the contents of okularpartrc from e. g.:

```init
[Reviews]
AnnotationTools=highlight,underline
```

to:

```init
[$Version]
update_info=okular.upd:annotation-toolbar

[Reviews]
QuickAnnotationTools=highlight,underline
```

Note that it changed the name of the AnnotationTools entry in the Reviews group, and added a new $Version group to mark the annotation-toolbar update as done.

To try this example, see the #Testing update files section. Testing update files\[edit]

You can run kconf\_update manually on a specific update file, to see how it will process it.

First, you need to locate your kconf\_update executable. (Try e. g. locate kconf\_update.) If you can’t locate it, you can also compile it from the KConfig repository. Let’s assume your kconf\_update executable is located at /usr/lib/kf5/kconf\_update.

kconf\_update offers the command line options --debug and --testmode. --debug enables helpful debugging output on the command line. --testmode activates QStandardPaths::setTestModeEnabled(), so kconf\_update will not update your actual configuration files, but files you have placed in a certain test directory. See the QStandardPaths::GenericConfigLocation documentation for your platform. Let’s assume this test directory is \~/.qttest/config/.

Now place your update file and a test configuration file in your working directory. Assuming you are testing the file from #Moving configuration data around, you run:

mkdir -p \~/.qttest/config/ cp okularpartrc \~/.qttest/config/ /usr/lib/kf5/kconf\_update --debug --testmode ./okular.upd

Then you should get an output like:

Automatically enabled the debug logging category kf.config.kconf\_update kf5.kconfig.update: Checking update-file "./okular.upd" for new updates kf5.kconfig.update: "okular.upd" : Found new update "annotation-toolbar" kf5.kconfig.update: "okular.upd" : Updating "okularpartrc" : "Reviews" : "QuickAnnotationTools" to "highlight,underline" kf5.kconfig.update: "okular.upd" : Removing "okularpartrc" : "Reviews" : "AnnotationTools" , moved.

Now you should open \~/.qttest/config/okularpartrc to verify that the update was performed correctly. If you see the new $Version group there, but AnnotationTools was not renamed, the update was performed, but something is wrong with your update instructions. If you don’t see the new $Version group, the update didn’t affect the correct configuration file. Tip For the first tests, it is convenient to write a configuration file that contains only the entries you are interested in (like in #Moving configuration data around). But you should also perform a test with a configuration file from an actual installation of your application.

Tip If you want to test the update file again, you need to restore an old version of the configuration file first.

Additionally, you need to open kconf\_updaterc, and remove the okular.upd group, otherwise kconf\_update will skip your update file. Since Frameworks version KF 5.76, kconf\_update will ignore kconf\_updaterc while in --testmode

Testing installed update files\[edit]

When you think your update file works fine, you may try to completely compile and install the new version of your application. After you have run e. g. make install, your update file should show up in the correct directory. KDED will detect this, and run kconf\_update. Your installed configuration file should now be updated, and you can open it to verify the update.

However, if you find problems now, you probably want to let the update run again. If you install a new update file, KDED will run kconf\_update again, but kconf\_update will not perform the update. What you need to do is:

```
Restore an old version of the configuration file, so there is something to update again. The old version shouldn’t contain the $Version group.
Locate kconf_updaterc (e. g. with the command locate kconf_updaterc), and remove the group named by your update file.
```

Now, kconf\_update will process the update again, as soon as you overwrite the update file. Since make install won’t overwrite the file with itself, you need to manually delete it first.

You can also try to run kconf\_update manually, e. g. with the command /usr/lib/kf5/kconf\_update --debug (without other arguments). Debug output while running in the background\[edit]

Because kconf\_update will now run in the background, you will not directly get debug output. After KDE Frameworks 5.57, debug output goes through the category kf.config.kconf\_update. If this category is enabled, debug output will be printed to stderr and consequently end up in xsession-errors or wayland-errors. You can use kdebugsettings to enable this logging category.

Alternatively, see the QLoggingCategory documentation for how to enable this logging category. Probably you can add the following group to a configuration file like /usr/share/qt5/qtlogging.ini:

\[Rules] kf.config.kconf\_update=true

If Qt was built with support for logging to system log facilities (such as systemd-journald), the output may be found there instead. Before 5.57, the output was always sent to stderr and may be found in the aforementioned errors files. Manipulating configuration data itself\[edit]

The above example update file okular.upd shows you how you can move configuration information around. But if you need to change the configuration data itself, you need to run configuration information through a script.

kconf\_update provides the Script= command, which will pipe configuration entries through a script before it is merged to the new configuration file.

You need to provide the script files in ${KDE\_INSTALL\_KCONFUPDATEDIR}. You should choose a script language whose interpreter will be available on your target platform. E. g. if your target platforms include Windows, you should not use shell scripts.

You can also use “compiled scripts”, these need to be placed in ${KDE\_INSTALL\_LIBDIR}/kconf\_update\_bin/. Compiled scripts allow you to use the C++ APIs of Qt, KConfig, your own application, and what you need else; and make you independent of interpreters.

The following sections give examples for both interpreted scripts and compiled scripts. Example interpreted update script\[edit]

Imagine you want to switch from a DoNotUseTabs configuration entry to a UseTabs configuration entry. This can be easily done using this small Python script:

import fileinput

for line in fileinput.input(): if line.startswith("DoNotUseTabs=false"): print("UseTabs=true") if line.startswith("DoNotUseTabs=true"): print("UseTabs=false")

print("# DELETE DoNotUseTabs")

print("UseTabs=true") When the script outputs the line UseTabs=true, kconf\_update will create a new configuration entry (unless it already exists).

print("# DELETE DoNotUseTabs") When the script outputs the line # DELETE DoNotUseTabs, kconf\_update will delete the old configuration entry.

See Development/Tools/Using\_kconf\_update#Script\_output\_format for detailed information on the script output format.

Your update file could look like this:

Version=5

Id=no-more-double-negation-2020 File=myApprc Group=UserInterface Script=myApp\_no\_more\_double\_negation\_2020.py,python3

Script=myApp\_no\_more\_double\_negation\_2020.py,python3 This instructs kconf\_update to look for the script file myApp\_no\_more\_double\_negation\_2020.py, and run it with python3. The script will get the contents of the group UserInterface as input, and the output will be merged back to the group UserInterface.

See Development/Tools/Using\_kconf\_update#Update\_script\_commands for a detailed reference on the Script= and ScriptArguments= commands.

Your CMakeLists.txt could look like this:

\[...] find\_package(ECM ${KF5\_REQUIRED\_VERSION} CONFIG REQUIRED) include(KDEInstallDirs) \[...] find\_package(KF5 ${KF5\_REQUIRED\_VERSION} REQUIRED COMPONENTS Config \[...] ) \[...] install(FILES conf/update/myApp\_configFiles.upd DESTINATION ${KDE\_INSTALL\_KCONFUPDATEDIR}) install(FILES conf/update/myApp\_no\_more\_double\_negation\_2020.py DESTINATION ${KDE\_INSTALL\_KCONFUPDATEDIR})

Don’t forget that your application now depends on python3. Example compiled update script\[edit]

Imagine your application made the state of a KActionMenu user configurable. Because you changed to the new QToolButton::ToolButtonPopupMode API, you need to migrate from the old delayed and stickyMenu entries to the new popupMode entry. You can do that with the following compiled update script, which uses the KConfig framework directly:

\#include #include

\#include #include

int main(int argc, char \*\*argv) { // Create a KConfig object from stdin. QFile inputFile; inputFile.open(stdin, QIODevice::ReadOnly); QTemporaryFile inputFileForKConfig("popupmode\_api\_tmp"); inputFileForKConfig.open(); inputFileForKConfig.write(inputFile.readAll()); inputFileForKConfig.close(); KConfig inputConfig(inputFileForKConfig.fileName(), KConfig::SimpleConfig);

```
// Create a text stream to stdout.
QTextStream outputStream(stdout, QIODevice::WriteOnly);

// Unify delayed and stickyMenu.
// delayed => 0 (DelayedPopup)
// !delayed && stickyMenu => 2 (InstantPopup)
// !delayed && !stickyMenu => 1 (MenuButtonPopup) (defined as default with KConfigXT)
KConfigGroup rootGroup = inputConfig.group("");
bool old_delayed = rootGroup.readEntry<bool>("delayed", true);
bool old_stickyMenu = rootGroup.readEntry<bool>("stickyMenu", true);
int new_popupMode = (old_delayed) ? 0 : (old_stickyMenu) ? 2 : 1;
if (new_popupMode != 1) {
    outputStream << "popupMode=" << new_popupMode << Qt::endl;
}
outputStream << "# DELETE delayed" << Qt::endl;
outputStream << "# DELETE stickyMenu" << Qt::endl;

return 0;
```

}

Your update file could look like this:

Version=5

Id=toolbuttonpopupmode-api-2020 File=myApprc Group=ActionMenuState Script=myApp-toolbuttonpopupmode-api-2020

Your CMakeLists.txt could look like this:

\[...] find\_package(ECM ${KF5\_REQUIRED\_VERSION} CONFIG REQUIRED) include(KDEInstallDirs) \[...] find\_package(KF5 ${KF5\_REQUIRED\_VERSION} REQUIRED COMPONENTS Config \[...] ) \[...] install(FILES myApp\_configFiles.upd DESTINATION ${KDE\_INSTALL\_KCONFUPDATEDIR})

add\_executable(myApp-toolbuttonpopupmode-api-2020 conf/update/scripts/popupmode.cpp) target\_link\_libraries(myApp-toolbuttonpopupmode-api-2020 KF5::ConfigCore ) install(TARGETS myApp-toolbuttonpopupmode-api-2020 DESTINATION ${KDE\_INSTALL\_LIBDIR}/kconf\_update\_bin/)

Because you don’t need an interpreter, you didn’t add any dependency to your application. Testing external update scripts\[edit]

The first step to test external update scripts is to call them directly, and feed them with your configuration files.

For example, you can test the script from #Example interpreted update script with this command:

cat myApprc\_UserInterface | python3 myApp\_no\_more\_double\_negation\_2020.py

If the script works correctly, you should get output like this:

UseTabs=true

## DELETE DoNotUseTabs

The second step is to test your update file together with the script. This means, you need to place the script file in a place where kconf\_update will find it while in --testmode. kconf\_update will look for the script in QStandardPaths::GenericDataLocation. Let’s assume this path is \~/.qttest/share/, then you need to place your script in \~/.qttest/share/kconf\_update/.

Place your update file, your update script, and your configuration file in your working directory, and run:

mkdir -p \~/.qttest/config/ cp myApprc \~/.qttest/config/ mkdir -p \~/.qttest/share/kconf\_update/ cp myApp\_no\_more\_double\_negation\_2020.py \~/.qttest/share/kconf\_update/ /usr/lib/kf5/kconf\_update --debug --testmode ./myApp\_configFiles.upd

You should get an output like:

Automatically enabled the debug logging category kf.config.kconf\_update kf.config.kconf\_update: Checking update-file "./myApp\_configFiles.upd" for new updates kf.config.kconf\_update: "demo9.upd" : Found new update "no-more-double-negation-2020" kf.config.kconf\_update: "demo9.upd" : Running script "myApp\_no\_more\_double\_negation\_2020.py" kf.config.kconf\_update: Script input stored in "/tmp/kconf\_update.lKlJpe" kf.config.kconf\_update: About to run "/usr/bin/python3" kf.config.kconf\_update: Script contents is: "\[...]" kf.config.kconf\_update: Successfully ran "/usr/bin/python3" kf.config.kconf\_update: Script output stored in "/tmp/kconf\_update.CCeaBP" kf.config.kconf\_update: Script output is: "UseTabs=true\n# DELETE DoNotUseTabs\n" kf.config.kconf\_update: "demo9.upd" : Script removes "myApprc" : ("UserInterface") : "DoNotUseTabs"

If you want to test compiled update scripts, the workflow is the same, except that you may place your compiled update script in one of your $PATH directories instead. Doing other stuff than updating configuration\[edit]

kconf\_update supports to run update scripts without input and discarding the output. You can use this e. g. to automatically rebuild the database of another service when the user updates your application. However, you should be careful about this. See Development/Tools/Using\_kconf\_update#Note\_on\_side\_effects for more information. Further example update files\[edit]

Another way to learn how to write scripts and update files is to look at existing ones.

With the following command, you can look which applications on your system have installed update files:

locate kconf\_update

You can also search through various KDE projects with this link:

https://lxr.kde.org/search?\_filestring=.%2B%5C.upd%5Cz&\_advanced=1
