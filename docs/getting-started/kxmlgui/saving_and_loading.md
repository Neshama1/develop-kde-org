---
title: Saving and loading
weight: 4
aliases:
  - /docs/getting-started/saving_and_loading/
description: >-
  Introduces the KIO library while adding loading and saving support to our
  application.
---

# Saving and loading

### Introduction

Now that we have a basic text editor interface, it's time to make it do something useful. At the most basic, a text editor needs to be able to load files from data storage, save files that have been created/edited, and create new files.

KDE Frameworks provides a number of classes for working with files that make life a lot easier for developers. [KIO](docs:kio) allows you to easily access files through network-transparent protocols. Qt also provides standard file dialogs for opening and saving files.

![](../../../content/docs/getting-started/kxmlgui/saving_and_loading/saving\_and\_loading.webp)

### The Code

#### main.cpp

We don't need to change anything in here.

```c++
#include <QApplication>
#include <QCommandLineParser>
#include <KAboutData>
#include <KLocalizedString>
#include "mainwindow.h"

int main (int argc, char *argv[])
{
    using namespace Qt::Literals::StringLiterals;

    QApplication app(argc, argv);
    KLocalizedString::setApplicationDomain("texteditor");
    KAboutData aboutData(
        u"texteditor"_s,
        i18n("Text Editor"),
        u"1.0"_s,
        i18n("A simple editor capable of saving and loading"),
        KAboutLicense::GPL,
        i18n("(c) 2024"),
        i18n("Educational application..."),
        u"https://apps.kde.org/someappname/"_s,
        u"submit@bugs.kde.org"_s);

    aboutData.addAuthor(
        i18n("John Doe"),
        i18n("Tutorial learner"),
        u"john.doe@example.com"_s,
        u"https://john-doe.example.com"_s,
        u"johndoe"_s);

    KAboutData::setApplicationData(aboutData);

    QCommandLineParser parser;
    aboutData.setupCommandLine(&parser);
    parser.process(app);
    aboutData.processCommandLine(&parser);

    MainWindow *window = new MainWindow();
    window->show();

    return app.exec();
}
```

#### mainwindow.h

```c++
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <KXmlGuiWindow>

class KTextEdit;
class KJob;

class MainWindow : public KXmlGuiWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);

private:
    void setupActions();
    void saveFileToDisk(const QString &outputFileName);

private Q_SLOTS:
    void newFile();
    void openFile();
    void saveFile();
    void saveFileAs();

    void downloadFinished(KJob *job);

private:
    KTextEdit *textArea;
    QString fileName;
};

#endif // MAINWINDOW_H
```

To add the ability to load and save files, we must add the functions which will do the work. Since the functions will be called through [Qt's signal/slot](http://doc.qt.io/qt-6/signalsandslots.html) mechanism we must specify that these functions are slots using `Q_SLOTS`. Since we are using slots in this header file, we must also add the [Q\_OBJECT](docs:qtcore;QObject::Q\_OBJECT) macro, as only [Q\_OBJECTs](docs:qtcore;QObject::Q\_OBJECT) can have signals and slots.

We also want to keep track of the filename of the currently opened file, so we declare a [QString](docs:qtcore;QString) `fileName`.

#### mainwindow.cpp

```c++
#include <QApplication>
#include <QAction>
#include <QSaveFile>
#include <QFileDialog>
#include <QTextStream>
#include <QByteArray>
#include <KTextEdit>
#include <KLocalizedString>
#include <KActionCollection>
#include <KStandardAction>
#include <KMessageBox>
#include <KIO/StoredTransferJob>
#include "mainwindow.h"

MainWindow::MainWindow(QWidget *parent) : KXmlGuiWindow(parent), fileName(QString())
{
    textArea = new KTextEdit();
    setCentralWidget(textArea);
    setupActions();
}

void MainWindow::setupActions()
{
    using namespace Qt::Literals::StringLiterals;

    QAction *clearAction = new QAction(this);
    clearAction->setText(i18n("&Clear"));
    clearAction->setIcon(QIcon::fromTheme(u"document-new-symbolic"_s));
    actionCollection()->addAction(u"clear"_s, clearAction);
    actionCollection()->setDefaultShortcut(clearAction, Qt::CTRL | Qt::Key_L);
    connect(clearAction, &QAction::triggered, textArea, &KTextEdit::clear);

    KStandardAction::quit(qApp, &QCoreApplication::quit, actionCollection());
    KStandardAction::open(this, &MainWindow::openFile, actionCollection());
    KStandardAction::save(this, &MainWindow::saveFile, actionCollection());
    KStandardAction::saveAs(this, &MainWindow::saveFileAs, actionCollection());
    KStandardAction::openNew(this, &MainWindow::newFile, actionCollection());

    setupGUI(Default, u"texteditorui.rc"_s);
}

void MainWindow::newFile()
{
    fileName.clear();
    textArea->clear();
}

void MainWindow::saveFileToDisk(const QString &outputFileName)
{
    if (!outputFileName.isNull()) {
        QSaveFile file(outputFileName);
        file.open(QIODevice::WriteOnly);

        QByteArray outputByteArray;
        outputByteArray.append(textArea->toPlainText().toUtf8());

        file.write(outputByteArray);
        file.commit();

        fileName = outputFileName;
    }
}

void MainWindow::saveFileAs()
{
    saveFileToDisk(QFileDialog::getSaveFileName(this, i18n("Save File As")));
}

void MainWindow::saveFile()
{
    if (!fileName.isEmpty()) {
        saveFileToDisk(fileName);
    } else {
        saveFileAs();
    }
}

void MainWindow::openFile()
{
    const QUrl fileNameFromDialog = QFileDialog::getOpenFileUrl(this, i18n("Open File"));

    if (!fileNameFromDialog.isEmpty()) {
        KIO::Job *job = KIO::storedGet(fileNameFromDialog);
        fileName = fileNameFromDialog.toLocalFile();
        connect(job, &KJob::result, this, &MainWindow::downloadFinished);
        job->exec();
    }
}

void MainWindow::downloadFinished(KJob *job)
{
    if (job->error()) {
        KMessageBox::error(this, job->errorString());
        fileName.clear();
        return;
    }

    const KIO::StoredTransferJob *storedJob = qobject_cast<KIO::StoredTransferJob *>(job);

    if (storedJob) {
        textArea->setPlainText(QTextStream(storedJob->data(), QIODevice::ReadOnly).readAll());
    }
}
```

We'll get into the details of mainwindow.cpp in a while.

#### texteditorui.rc

This is identical to the `texteditorui.rc` from the \[previous tutorial]\(\{{< relref "using\_actions/#texteditoruirc" >\}}). We do not need to add any information about any of the [KStandardAction](docs:kconfigwidgets;KStandardAction) methods since the placement of those actions is handled automatically by the [KXmlGui](docs:kxmlgui) system.

### Explanation

Okay, now to implement the code that will do the loading and saving. This will all be happening in `mainwindow.cpp`.

```cpp
MainWindow::MainWindow(QWidget *parent) : KXmlGuiWindow(parent), fileName(QString())
```

The first thing we do is to initialize `fileName(QString())` in the MainWindow's [constructor initializer list](https://en.cppreference.com/w/cpp/language/constructor) to make sure that `fileName` is empty right from the beginning.

#### Adding the actions

We will then provide the outward interface for the user so they can tell the application to load and save. Like with the quit action in the \[previous tutorial]\(\{{< relref "using\_actions/#kstandardaction" >\}}), we will use [KStandardAction](docs:kconfigwidgets;KStandardAction). We add the actions in the same way we did for the quit action and, for each one, we connect it to the appropriate slot that we declared in the header file.

#### Creating a new document

The first function we create is the `newFile()` function.

```cpp
void MainWindow::newFile()
{
  fileName.clear();
  textArea->clear();
}
```

`fileName.clear()` sets the `fileName` string to be empty to reflect the fact that this document is not stored anywhere yet. `textArea->clear()` then clears the central text area using the same function that we connected the 'clear' action to in the \[previous tutorial]\(\{{< relref "using\_actions#creating-the-qaction-object" >\}}).

{% hint style="warning" %}Warning This example simply clears the text area without checking if the file has been saved first. It's only meant as a demonstration of file I/O and is _not_ an example of best programming practices. {% endhint %}

#### Saving a file

{% hint style="info" %}Note To make this tutorial simple, this example program can only save to local storage even though it can open any file from any location, even those from remote sources. {% endhint %}

#### saveFileToDisk(const QString &)

Now we get onto our first file handling code. We are going to implement a function which will save the contents of the text area to the file name given as a parameter. Qt provides a class for safely saving a file called [QSaveFile](docs:qtcore;QSaveFile).

The function's prototype is:

```cpp
void MainWindow::saveFileAs(const QString &outputFileName)
```

We then create our [QSaveFile](docs:qtcore;QSaveFile) object and open it with:

```cpp
QSaveFile file(outputFileName);
file.open(QIODevice::WriteOnly);
```

Now that we have our file to write to, we need to format the text in the text area to a format which can be written to file. For this, we create a [QByteArray](docs:qtcore;QByteArray) to serve as our temporary string buffer and fill it with the plain text version of whatever is in the text area:

```cpp
QByteArray outputByteArray;
outputByteArray.append(textArea->toPlainText().toUtf8());
```

Now that we have our [QByteArray](docs:qtcore;QByteArray), we use it to write to the file with [QSaveFile::write()](docs:qtcore;QSaveFile::write). If we were using a normal [QFile](docs:qtcore;QFile), this would make the changes immediately. However, if a problem occurred partway through writing, the file would become corrupted. For this reason, [QSaveFile](docs:qtcore;QSaveFile) works by first writing to a temporary file and then, when calling [QSaveFile::commit()](docs:qtcore;QSaveFile::commit), the changes are made to the actual file and the file is then closed.

```cpp
file.write(outputByteArray);
file.commit();
```

Finally, we set MainWindows's `fileName` member to point to the file name we just saved to.

```cpp
fileName = outputFileName;
```

#### saveFileAs()

This is the function that the `saveAs` slot is connected to. It simply calls the generic `saveFileToDisk(QString)` function and passes the file name returned by [QFileDialog::getSaveFileName()](docs:qtwidgets;QFileDialog::getSaveFileName).

```cpp
void MainWindow::saveFileAs()
{
    saveFileToDisk(QFileDialog::getSaveFileName(this, i18n("Save File As")));
}
```

[QFileDialog](docs:qtwidgets;QFileDialog) provides a number of static functions for displaying the common file dialog that is used by all KDE applications. Calling [QFileDialog::getSaveFileName()](docs:qtwidgets;QFileDialog::getSaveFileName) will display a dialog where the user can select the name of the file to save to or choose a new name. The function returns the full file name, which we then pass to `saveFileToDisk(QString)`.

#### saveFile()

```c++
void MainWindow::saveFile()
{
    if(!fileName.isEmpty()) {
        saveFileToDisk(fileName);
    } else {
        saveFileAs();
    }
}
```

There's nothing exciting or new about this function, just the logic to decide whether or not to show the save dialog. If `fileName` is not empty, then the file is saved to `fileName`. But if it is, then the dialog is shown to allow the user to select a file name.

#### Loading a file

Finally, we get around to being able to load a file, from local storage or from a remote location like an FTP server. The code for this is all contained in `MainWindow::openFile()`.

First we must ask the user for the name of the file they wish to open. We do this using [QFileDialog::getOpenFileUrl()](docs:qtwidgets;QFileDialog::getOpenFileUrl):

```c++
const QUrl fileNameFromDialog = QFileDialog::getOpenFileUrl(this, i18n("Open File"));
```

Here we use [QUrl](docs:qtcore;QUrl) to handle files from remote locations.

Then we use the [KIO](docs:kio) library to retrieve our file. This allows us to open the file even if it's stored in a remote location like an SFTP server. We make the following call to the [KIO::storedGet](docs:kio;KIO::storedGet) function with an argument for the file you wish to open or download:

```c++
KIO::Job *job = KIO::storedGet(fileNameFromDialog);
```

The function returns a handle to a [KIO::Job](docs:kio;KIO::Job), which we first connect to our `downloadFinished()` slot before "running" the job.

```c++
connect(job, &KJob::result, this, &MainWindow::downloadFinished);
job->exec();
```

The rest of the work happens in the `downloadFinished()` slot. First, the job is checked for errors. If it failed, we display a message box giving the error. We also make sure to clear the `fileName`, since the file wasn't opened successfully:

```c++
KMessageBox::error(this, job->errorString());
fileName.clear();
```

Otherwise, we continue with opening the file.

The data that `storedGet()` successfully downloaded, in this case the contents of our text file, is stored in the data member of a [KIO::StoredTransferJob](docs:kio;KIO::StoredTransferJob) class. But in order to display the contents of the file as text, we must use a [QTextStream](docs:qtcore;QTextStream). We create one by passing the data of the [KIO::StoredTransferJob](docs:kio;KIO::StoredTransferJob) to its constructor and then call its [QTextStream::readAll()](docs:qtcore;QTextStream::readAll) function to get the text from the file. This is then passed to the `setPlainText()` function of our text area.

```c++
const KIO::StoredTransferJob *storedJob = qobject_cast<KIO::StoredTransferJob *>(job);

if (storedJob) {
    textArea->setPlainText(QTextStream(storedJob->data(), QIODevice::ReadOnly).readAll());
}
```

{% hint style="info" %}Note Again, for simplicity's sake, this tutorial only saves text files to local disk. When you open a remote file for viewing and try to save it, the program will behave as if you were calling Save As on a completely new file. {% endhint %}

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)

project(texteditor)

set(QT_MIN_VERSION "6.6.0")
set(KF_MIN_VERSION "6.0.0")

find_package(ECM ${KF_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(FeatureSummary)

find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS
    Core    # QCommandLineParser, QStringLiteral
    Widgets # QApplication, QAction
)

find_package(KF6 ${KF_MIN_VERSION} REQUIRED COMPONENTS
    CoreAddons      # KAboutData
    I18n            # KLocalizedString
    XmlGui          # KXmlGuiWindow, KActionCollection
    TextWidgets     # KTextEdit
    ConfigWidgets   # KStandardActions

    WidgetsAddons   # KMessageBox

    KIO             # KIO

)

add_executable(texteditor)

target_sources(texteditor
    PRIVATE
    main.cpp
    mainwindow.cpp
)

target_link_libraries(texteditor
    Qt6::Widgets
    KF6::CoreAddons
    KF6::I18n
    KF6::XmlGui
    KF6::TextWidgets
    KF6::ConfigWidgets

    KF6::WidgetsAddons

    KF6::KIOCore

)

install(TARGETS texteditor ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})
install(FILES texteditorui.rc DESTINATION ${KDE_INSTALL_KXMLGUIDIR}/texteditor)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
```

Since we are now using the [KIO](docs:kio) library, we must tell CMake to link against it. We do this by passing `KIO` to the [`find_package()`](https://cmake.org/cmake/help/latest/command/find\_package.html) function and `KF6::KIOCore` to the [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target\_link\_libraries.html) function.

#### Running our application

Once again, you can repeat the same steps provided in \{{< ref "hello\_world#kxmlgui-running" >\}} to build and install the application. You can then run the project with:

```bash
kdesrc-build --run --exec texteditor kxmlgui-tutorial
```

or

```bash
texteditor
```
