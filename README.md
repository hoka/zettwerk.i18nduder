Introduction
============

The tasks to add translations to a plone product are more or less always the same:

* Creating and registering the locales directory
* Adding languages
* Call i18ndude with rebuild-pot command
* Call i18ndude with sync command
* Run the last two commands in combination with find-untranslated a couple of times
* And maybe directly compile the mo files

zettwerk.i18nduder tries to make this a little bit easer, by wrapping the i18ndude commands and avoid the folder and path handling by calling it directly from your buildout home folder. It makes nothing, that you can't do with i18ndude directly, but in our internal usage, it made working with translations a little bit easier and faster.

To make this work there is only one assumption made, which should be common in a paster created plone product: your translation domain has the same name as your python package.


Installation
============

Put a i18nduder part to your buildout.cfg and define it like this::

  [i18nduder]
  recipe = zc.recipe.egg
  eggs = ${instance:eggs}
         zettwerk.i18nduder


It is important to include your instance eggs, cause duder gets the path of the package by importing it. Also note, that zettwerk.i18nduder includes a dependency to i18ndude.


Available commands and usage
============================

The generated script is called duder. To see a list of commands use the --help argument::

  > ./bin/duder --help
  > Usage: duder command -p package.name or --help for details
  >
  > Available commands:
  >   create		Create the locales folder and/or add given languages
  >   update		Call rebuild-pot and sync
  >   mo		Call msgfmt to build the mo files
  >   find		Find untranslated strings (domain independent)
  >
  > Options:
  >   -h, --help            show this help message and exit
  >   -p PACKAGE, --package=PACKAGE
  >                         Name of the package
  >   -d DOMAIN, --domain=DOMAIN
  >                         Name of the i18n domain. Only needed if it is different
  >                         from the package name.
  >   -l LANGUAGES, --languages=LANGUAGES
  >                         List of comma-separated names of language strings.
  >                         Only used for create command. For example: -l en,de

Examples
========

The following commands are used against a newly created, and registered in your buildout archetype product with the package name dummy.example

First, we create the locales folder and add some languages::

  > ./bin/duder create -p dummy.example -l en,de
  > Created locales folder at: /home/joerg/Instances/Plone4/src/dummy.example/dummy/example/locales
  > Don't forget to add this to your configure.zcml:
  > <i18n:registerTranslations directory="locales" /  >
  > - en language added
  > - de language added

To add another language just call it again::

  > ./bin/duder create -p dummy.example -l nl
  > Locales folder already exists at: /home/joerg/Instances/Plone4/src/dummy.example/dummy/example/locales
  > - nl language added

After working with dummy.exmaple some translations must be done::

  > ./bin/duder update -p dummy.example
  > nl/LC_MESSAGES/dummy.example.po: 1 added, 0 removed
  > en/LC_MESSAGES/dummy.example.po: 1 added, 0 removed
  > de/LC_MESSAGES/dummy.example.po: 1 added, 0 removed

Note, that the pot/po headers are not manipulated with this. This must be done by hand.

Find-untranslated is also available::

  > ./bin/duder find -p dummy.example
  > wrote output to: /home/joerg/Instances/Plone4/dummy.example-untranslated

And the mo files can be compiled like this::

  > ./bin/duder mo -p dummy.example

This calls "msgfmt", so it must be available on your system.

Notes about domains
===================

It is maybe good to now, that i18ndude can't differentiate domains by the MessageFactory. So if you run duder with the domain option will result with the msgids of the given domain (when used explicitly in your template files with i18n:domain) *and* the translations, which are created via the MessageFactory of you default domain.

Feel free to contact us for suggestions and improvments.
