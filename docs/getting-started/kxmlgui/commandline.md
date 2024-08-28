---
title: Command line interface
weight: 6
aliases:
  - /docs/getting-started/commandline/
description: >-
  Adds the ability to specify which file to open from the command line to our
  text editor.
---

# Command line interface

### Introduction

We now have a working text editor that can open and save files. We might, however, want to extend its utility by enabling users to more quickly and efficiently use it to edit files. In this tutorial we will make the editor act more like a desktop application by enabling it to open files from command line arguments or even using Open with from within Dolphin.

![](../../../content/docs/getting-started/kxmlgui/commandline/commandline.webp)

### Code and Explanation

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

    void openFileFromUrl(const QUrl &inputFileName);


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

Here we have done nothing but add a new `openFileFromUrl()` function which takes a [QUrl](docs:qtcore;QUrl). Again, we use URLs instead of strings so that we can also work with remote files as if they were local.

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

    openFileFromUrl(QFileDialog::getOpenFileUrl(this, i18n("Open File")));

}


void MainWindow::openFileFromUrl(const QUrl &inputFileName)

{

    if (!inputFileName.isEmpty()) {

        KIO::Job *job = KIO::storedGet(inputFileName);

        fileName = inputFileName.toLocalFile();

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

There's no new code here, only rearranging. Everything from `void openFile()` has been moved into `void openFileFromUrl(const QUrl &inputFileName)` except the call to [QFileDialog::getOpenFileUrl()](docs:qtwidgets;QFileDialog::getOpenFileUrl).

This way, we can call `openFile()` if we want to display a dialog, or we can call `openFileFromUrl(const QUrl &)` if we know the name of the file already. Which will be the case when we feed the file name through the command line.

#### main.cpp

```c++
#include <QApplication>
#include <QCommandLineParser>

#include <QDir>

#include <QUrl>

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
        i18n("A simple text area using QAction etc."),
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

    parser.addPositionalArgument(u"file"_s, i18n("Document to open"));


    parser.process(app);
    aboutData.processCommandLine(&parser);

    MainWindow *window = new MainWindow();
    window->show();


    if (parser.positionalArguments().count() > 0) {

        window->openFileFromUrl(QUrl::fromUserInput(

            parser.positionalArguments().at(0),

            QDir::currentPath()));

    }


    return app.exec();
}
```

This is where all the [QCommandLineParser ](docs:qtcore;QCommandLineParser)magic happens. In previous examples, we only used the class to feed [QApplication](docs:qtwidgets;QApplication) the necessary data for using flags like `--version` or `--author`. Now we actually get to use it to process command line arguments.

First, we tell [QCommandLineParser ](docs:qtcore;QCommandLineParser)that we want to add a new positional argument. In a nutshell, these are arguments that are not options. `-h` or `--version` are options, `file` is an argument.

```c++
parser.addPositionalArgument(u"file"_s, i18n("Document to open"));
```

Later on, we start processing positional arguments, but only if there is one. Otherwise, we proceed as usual. In our case we can only open one file at a time, so only the first file is of interest to us. We call the `openFileFromUrl()` function and feed it the URL of the file we want to open, whether it is a local file like "$HOME/foo" or a remote one like "ftp.mydomain.com/bar". We use the overloaded form of [QUrl::fromUserInput()](docs:qtcore;QUrl::fromUserInput) in order to set the current path. This is needed in order to work with relative paths like "../baz".

```c++
if (parser.positionalArguments().count() > 0) {
    window->openFileFromUrl(QUrl::fromUserInput(parser.positionalArguments().at(0), QDir::currentPath()));
}
```

#### CMakeLists.txt

We don't need to change anything in here.

```c++
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

### Running our application

Again, you can repeat the same steps provided in \{{< ref "hello\_world#kxmlgui-running" >\}} to build and install the application.

However, we will test if our application handles files from the command line correctly. Create a simple file:

```bash
echo "It works!" > testfile.txt
```

Now you may pass it as argument with:

```bash
kdesrc-build --run --exec texteditor kxmlgui-tutorial testfile.txt
```

or

```bash
texteditor testfile.txt
```

You should then see your application run and load `testfile.txt` directly from its UI, showing "It works!" in your `textArea`.
