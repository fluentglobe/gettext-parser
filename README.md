gettext-parser
==============

[![Build Status](https://secure.travis-ci.org/andris9/gettext-parser.png)](http://travis-ci.org/andris9/gettext-parser)
[![NPM version](https://badge.fury.io/js/gettext-parser.png)](http://badge.fury.io/js/gettext-parser)

Parse and compile gettext *po* and *mo* files with node.js, nothing more, nothing less.

This module is slightly based on my other gettext related module [node-gettext](https://github.com/andris9/node-gettext). The plan is to move all parsing and compiling logic from node-gettext to here and leave only translation related functions (domains, plural handling, lookups etc.).

## ICONV NOTICE

By default *gettext-parser* uses pure JS [iconv-lite](https://github.com/ashtuchkin/iconv-lite) for encoding and decoding non UTF-8 charsets. If you need to support more complex encodings like EUC or Shift_JIS, you need to add [iconv](https://github.com/bnoordhuis/node-iconv) as a dependency for your project.

## Usage

Include the library:

    var gettextParser = require("gettext-parser");

Available methods:

  * `gettextParser.po.parse(buf[, defaultCharset])` where `buf` is a *po* file as a Buffer or an unicode string. `defaultCharset` is the charset to use if charset is not defined or is the default `"CHARSET"`. Returns gettext-parser specific translation object (see below)
  * `gettextParser.po.compile(obj)` where `obj` is a translation object, returns a *po* file as a Buffer
  * `gettextParser.mo.parse(buf[, defaultCharset])` where `buf` is a *mo* file as a Buffer (*mo* is binary format, so do not use strings). `defaultCharset` is the charset to use if charset is not defined or is the default `"CHARSET"`. Returns translation object
  * `gettextParser.mo.compile(obj)` where `obj` is a translation object, returns a *mo* file as a  Buffer

**NB** if you are compiling a previously parsed translation object, you can override the output charset with the `charset` property (applies both for compiling *mo* and *po* files).

    var obj = gettextParser.po.parse(inputBuf);
    obj.charset = "windows-1257";
    outputBuf = gettextParser.po.compile(obj);

Headers for the output are modified to match the updated charset.

## Data structure of parsed mo/po files

### Character set

The data is always in unicode but the original charset of the file can
be found from the `charset` property.

### Headers

Headers can be found from the `headers` object, all keys are lowercase and the value for a key is a string. This value will also be used when compiling.

### Translations

Translations can be found from the `translations` object which in turn holds context objects for `msgctx`. Default context can be found from `translations[""]`.

Context objects include all the translations, where `msgid` value is the key. The value is an object with the following possible properties:

  * **msgctx** context for this translation, if not present the default context applies
  * **msgid** string to be translated
  * **msgid_plural** the plural form of the original string (might not be present)
  * **msgstr** an array of translations
  * **comments** an object with the following properties: `translator`, `reference`, `extracted`, `flag`, `previous`.

Example

```json
{
	"charset": "iso-8859-1",

    "headers": {
        "content-type": "text/plain; charset=iso-8859-1",
        "plural-forms": "nplurals=2; plural=(n!=1);"
    },

    "translations":{
    	"": {
			"": {
                "msgid": "",
                "msgstr": ["Content-Type: text/plain; charset=iso-8859-1\n..."]
			}
		},

    	"another context":{
			"%s example":{
				"msgctx": "another context",
				"msgid": "%s example",
				"msgid_plural": "%s examples",
				"msgstr": ["% näide", "%s näidet"],
				"comments": {
					"translator": "This is regular comment",
					"reference": "/path/to/file:123"
				}
			}
		}
    }
}
```

Notice that the structure has both a `headers` object and a `""` translation with the header string. When compiling the structure to a *mo* or a *po* file, the `headers` object is used to define the header. Header string in the `""` translation is just for reference (includes the original unmodified data) but will not be used when compiling. So if you need to add or alter header values, use only the `headers` object.

If you need to convert *gettext-parser* formatted translation object to something else, eg. for *jed*, check out [po2json](https://github.com/mikeedwards/po2json).

## License

**MIT**
