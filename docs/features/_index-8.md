---
title: Sonnet
group: features
authors:
  - SPDX-FileCopyrightText: 2006 Jacob R Rideout <kde@jacobrideout.net>
SPDX-License-Identifier: LGPL-2.0-or-later
aliases:
  - /docs/features/spellchecking/
description: Spellchecking made easy
---

# Sonnet

Sonnet is a useful framework provided by KDE for software developers who want to solve the problem of spellchecking in text editors. It has a plugin based architecture with support for HSpell, Enchant, ASpell and HUNSPELL plugins. It even supports automated language detection, based on a combination of different algorithms.

### Spellchecking in your QTextEdit

Sonnet can be easily integrated into your QTextEdit as follows:

```cpp
    QTextEdit *textEdit = new QTextEdit;
    textEdit->setText(
        QString::fromLatin1("This is a sample buffer. Whih this thingg will "
                            "be checkin for misstakes. Whih, Enviroment, govermant. Whih."));

    Sonnet::SpellCheckDecorator *installer = new Sonnet::SpellCheckDecorator(textEdit);
    installer->highlighter()->setCurrentLanguage(QStringLiteral("en_US"));
```

Sonnet::SpellCheckDecorator can also be extended in various ways to spell check text that is formatted differently, for example in emails.

```cpp
class MailSpellCheckDecorator : public Sonnet::SpellCheckDecorator
{
public:
    explicit MailSpellCheckDecorator(QTextEdit *edit)
        : Sonnet::SpellCheckDecorator(edit)
    {
    }

protected:
    bool isSpellCheckingEnabledForBlock(const QString &blockText) const override
    {
        qDebug() << blockText;
        return !blockText.startsWith(QLatin1Char('>'));
    }
};
```

So, you can use MailSpellCheckDecorator in exactly the same way as you would use SpellCheckDecorator, but with the added functionality that `MailSpellCheckDecorator` will ignore quoted parts of a email.

### Language Detection in Sonnet

Sonnet can determine the difference between \~75 languages for a given string. It is based off a perl script originally written by Maciej Ceglowski called Languid. His script used a two-part heuristic to determine language. First the text is checked for the scripts it contains, next for each set of languages using those scripts a n-gram frequency model of a given language is compared to a model of the text. The most similar language model is assumed to be the language. If no language is found an empty string is returned.

Here you see a simple example of language detection using the GuessLanguage class from Sonnet:

```cpp
GuessLanguage languageGuesser;
QString lang = languageGuesser.identify("Mon super texte");
```

### GUI Widgets provided by Sonnet

Sonnet also provides some GUI widgets that can be used by Qt applications to configure settings in Sonnet; for example Qt applications can use the `DictionaryComboBox` class from Sonnet to get a `QComboBox` that can configure the dictionary used by Sonnet.

```cpp
void TestDialog::check(const QString &buffer)
{
    Sonnet::Dialog *dlg = new Sonnet::Dialog(new BackgroundChecker(this), nullptr);
    connect(dlg, &Dialog::spellCheckDone, this, &TestDialog::doneChecking);
    dlg->setBuffer(buffer);
    dlg->show();
}
```

The ConfigDialog class from Sonnet provides a more advanced configuration dialog to configure settings such as whitelisting words, skipping run-together words as well as enabling or disabling auto detection of the language.
