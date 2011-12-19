# Introduction

There are a lot of great javascripts modules around to bring translation to clientside web, but there 
where always some features i missed.

Mostly:

- Usage of namespaces: split needed translation in more than one file (to share them to other projects, eg. for buttons)
- load resourcesfiles from static source or from a dynamic generated file
- have a jquery function to translate all tags marked with a special attibute
- have an option to post missing translations to the server
- graceful fallback: if it doesn't find a translation in _en-US_ look first in _en_ befor taking value from fallback language


What else will you find:

- support for pluralized strings
- insertion of variables into translations
- translation nesting

What will you find in next release:

- support for localstorage

# Usage Sample

Add the i18next.js after the jquery JavaScript.

    <script type="text/javascript" src="jquery-1.6.4.min.js"></script>
    <script type="text/javascript" src="i18next.js"></script>

Add your resourcefile under /locales/en-US/translation.json

    {
        "app": {
          "name": "i18n"
        },
        "creator": {
          "firstname": "Jan",
          "lastname": "Mühlemann"
        }
    }

Init and use the module:

    $.i18n.init({}, function(t) { // will init i18n with default settings and set language from navigator
        var appName = t('app.name'); // -> i18n
        var creator = t('creator.firstname') + ' ' + t('creator.lastname'); // -> Jan Mühlemann
    });

After initialisation you can use __$.t()__ instead of setting translation in the init callback:

    var appName = $.t('app.name'); // -> i18n

If no resource string is found in the specific language (en-US) it will be looked up in _en_ befor taken from fallback language. 
If the key is not in the fallback language the key or a optional defaultValue will be displayed:

    var takeDefault = $.t('app.type', {defaultValue: 'OpenSource'}); // -> OpenSource

### or you can just use the jquery function provided

    // given resource
    "nav": {
      "home": "home",
      "1": "link1",
      "2": "link2"
    }

  // given html
  <ul class="nav">
    <li class="active"><a href="#" data-i18n="nav.home">home</a></li>
    <li><a href="#" data-i18n="nav.1">link1</a></li>
    <li><a href="#" data-i18n="nav.2">link2</a></li>
  </ul>

  // just do to translate all elements having attribute _data-i18n_
    $('.nav').i18n();

Every child element with an _data-i18n_ attribute will be translated with given key.

### Basic options for init

    $.i18n.init({
          lng: 'en-US'                               // defaults to get from navigator
        , fallbackLng: 'en'                          // defaults to 'dev'
        , ns: 'myNamespace'                          // defaults to 'tranalation'
        , resGetPath: 'myFolder/__lng__/__ns__.json' // defaults to 'locales/__lng__/__ns__.json' where ns = translation (default)
        , resStore: {...}                            // if you don't want your resources to be loaded you can provide the resources
    });


# Extended Samples

### loading multiple namespaces

Set the ns option to an array on i18n init:

    $.i18n.init({
        lng: 'en-US',
        ns: { namespaces: ['ns.common', 'ns.special'], defaultNs: 'ns.special'}
    });

This will load __en-US__, __en__ and __dev__ resources for two namespaces ('ns.special' and 'ns.common'):

    locales
       |
       +-- en-US
       |     |
       |     +-- ns.common
       |     +-- ns.special
       |      
       +-- en
       |    |
       |    +-- ns.common
       |    +-- ns.special
       | 
       +-- dev
            |
            +-- ns.common
            +-- ns.special

    // let's asume this will load following resource strings
    {
      "en_US": {
        "ns.special": {
          "app": {
            "name": "i18n"
          }
        },
        "ns.common": {}
      },
      "en": {
        "ns.special": {
          "app": {
            "area": "Area 51"
          }
        },
        "ns.common": {}
      },
      "dev": {
        "ns.common": {
          "app": {
            "company": {
              "name": "my company"
            }
          },
          "add": "add"
        }
      }
    }


you can translate using _$.t([namespace:]key, [options])_

    $.t('app.name');                   // -> i18n (from 'en-US' resourcefile)
    $.t('app.area');                   // -> Area 51 (from 'en' resourcefile)
    $.t('ns.common:app.company.name'); // -> my company (from 'dev' resourcefile)
    $.t('ns.common:add');              // -> add (from 'dev' resourcefile)

If no namespace is prepended to the resource key i18n will take the default namespace provided on initalisation.

### insert values into your translation

    // given resource
    "insert": "you are __myVar__",

	$.t('insert', {myVar: 'great'}) // -> you are great

### support for plurals

    // given resource
    "child": "__count__ child",
    "child_plural": "__count__ children"

	$.t('child', {count: 1}) // -> 1 child
    $.t('child', {count: 3}) // -> 3 children

You can set the _pluralSuffix_ as an option on initialisation.

### nesting

    // given resource
    "app": {
      "area": "Area 51",
      "district": "District 9 is more fun than $t(app.area)"
    }

    $.t('app.district') // -> District 9 is more fun than Area 51

### dynamic resouce route

Set the _dynamicLoad_ option to true on init

    $.i18n.init({
        lng: 'en-US'
        ns: { namespaces: ['ns.common', 'ns.special'], defaultNs: 'ns.special'},
        dynamicLoad: true,
        resGetPath: 'myPath/resources.json?ns=__ns__&lng__lng__' // set this according to your needs
    });

On init i18n will call the provided route with the querystring parameters language and namespaces (they will be 
seperated by a + which should be a space on serverside).

You will have to return something like:

    {
      "en_US": {
        "ns.special": {...},
        "ns.common": {...}
      },
      "en": {
        "ns.special": {..},
        "ns.common": {...}
      },
      "dev": {
        "ns.special": {..},
        "ns.common": {..}
        }
      }
    }

### post missing resources

Just init i18n with the according options (you shouldn't use this option in production):

    $.i18n.init({
        // ...
        sendMissing: true,
        resPostPath: 'myPath/add/__lng__/__ns__', // defaults to 'locales/add/__lng__/__ns__',
    });


## Inspiration

- [jsperanto](https://github.com/jpjoyal/jsperanto).