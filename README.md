# laravel-html-purifier
Prečisovalnica HTML za Laravel

[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[![Software License][ico-license]](LICENSE)
[![Packagist](https://img.shields.io/packagist/v/sloyakuza/laravel-html-purifier)](https://packagist.org/packages/sloyakuza/laravel-html-purifier/)
[![Packagist Downloads](https://img.shields.io/packagist/dm/sloyakuza/laravel-html-purifier.svg?label=packagist%20downloads)](https://packagist.org/packages/sloyakuza/laravel-html-purifier)

Preprosta [Laravel](http://www.laravel.com/) ponudnik storitev za enostavno uporabo [HTMLPurifier](http://htmlpurifier.org/) v Laravelu. Z njihove spletne strani:

> HTML Purifier je standardno skladna HTML filtrska knjižnica, napisana v PHP. HTML Purifier ne bo odstranil le vse zlonamerne kode (bolj znane kot XSS) s temeljito revidiranim, varnim, a dovoljenim belim seznamom, temveč bo tudi zagotovil, da so vaši dokumenti skladni s standardi, nekaj le dosegljivega s celovitim poznavanjem W3C-jevih specifikacij. Naveličate se uporabe BBCode zaradi trenutne pokrajine pomanjkljivih ali negotovih HTML filtrov? Imate WYSIWYG urednik, vendar nikoli ni bil sposoben uporabljati? Iščete visokokakovostne, standardno skladne, odprtokodne komponente za to aplikacijo, ki jo gradite? HTML Prečisovalnica je za vas!

## Namestitev

### Za Laravel 5.5+

Zahtevaj ta paket s skladateljem:
```
composer require sloyakuza/laravel-html-purifier
```

Ponudnik storitev bo samodejno odkrit. Ponudnika ni treba dodajati nikjer.

### Za Laravel 5.0 do 5.4

Zahtevaj ta paket s skladateljem:
```
composer require sloyakuza/laravel-html-purifier
```

Poiščite `providers` ključ v `config/app.php` in registrirajte ponudnika storitev HTMLPurifier.

```php
    'providers' => [
        // ...
        SLOYakuza\Purifier\PurifierServiceProvider::class,
    ]
```

Poiščite `aliases` ključ v `config/app.php` in registrirati vzdevke.

```php
    'aliases' => [
        // ...
        'Purifier' => SLOYakuza\Purifier\Facades\Purifier::class,
    ]
```

## Navada


Uporabite te metode znotraj vaših zahtev ali srednje programske opreme, kjerkoli potrebujete HTML čiščenje:

```php
\clean(Input::get('inputname'));
```
ali

```php
Purifier::clean(Input::get('inputname'));
```

dinamična konfiguracija
```php
\clean('This is my H1 title', 'titles');
\clean('This is my H1 title', array('Attr.EnableID' => true));
```
ali

```php
Purifier::clean('This is my H1 title', 'titles');
Purifier::clean('This is my H1 title', array('Attr.EnableID' => true));
```

uporabiti [URI filter](http://htmlpurifier.org/docs/enduser-uri-filter.html)

```php
Purifier::clean('This is my H1 title', 'titles', function (HTMLPurifier_Config $config) {
    $uri = $config->getDefinition('URI');
    $uri->addFilter(new HTMLPurifier_URIFilter_NameOfFilter(), $config);
});
```

Druga možnost je, da v Laravel 7+, če iščete čiščenje HTML-ja v modelih Eloquent, lahko uporabite naše odlive po meri:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use SLOYakuza\Purifier\Casts\CleanHtml;
use SLOYakuza\Purifier\Casts\CleanHtmlInput;
use SLOYakuza\Purifier\Casts\CleanHtmlOutput;

class User extends Model
{
    protected $casts = [
        'bio'            => CleanHtml::class, // cleans both when getting and setting the value
        'description'    => CleanHtmlInput::class, // cleans when setting the value
        'history'        => CleanHtmlOutput::class, // cleans when getting the value
    ];
}
```

## Konfiguracijo

Če želite uporabiti lastne nastavitve, objavite konfiguracijo.

```
php artisan vendor:publish --provider="SLOYakuza\Purifier\PurifierServiceProvider"
```

Konfiguracija datoteke `config/purifier.php` to bi moralo biti všeč

```php

return [
    'encoding'           => 'UTF-8',
    'finalize'           => true,
    'ignoreNonStrings'   => false,
    'cachePath'          => storage_path('app/purifier'),
    'cacheFileMode'      => 0755,
    'settings'      => [
        'default' => [
            'HTML.Doctype'             => 'HTML 4.01 Transitional',
            'HTML.Allowed'             => 'div,b,strong,i,em,u,a[href|title],ul,ol,li,p[style],br,span[style],img[width|height|alt|src]',
            'CSS.AllowedProperties'    => 'font,font-size,font-weight,font-style,font-family,text-decoration,padding-left,color,background-color,text-align',
            'AutoFormat.AutoParagraph' => true,
            'AutoFormat.RemoveEmpty'   => true,
        ],
        'test'    => [
            'Attr.EnableID' => 'true',
        ],
        "youtube" => [
            "HTML.SafeIframe"      => 'true',
            "URI.SafeIframeRegexp" => "%^(http://|https://|//)(www.youtube.com/embed/|player.vimeo.com/video/)%",
        ],
        'custom_definition' => [
            'id'  => 'html5-definitions',
            'rev' => 1,
            'debug' => false,
            'elements' => [
                // http://developers.whatwg.org/sections.html
                ['section', 'Block', 'Flow', 'Common'],
                ['nav',     'Block', 'Flow', 'Common'],
                ['article', 'Block', 'Flow', 'Common'],
                ['aside',   'Block', 'Flow', 'Common'],
                ['header',  'Block', 'Flow', 'Common'],
                ['footer',  'Block', 'Flow', 'Common'],
				
				// Content model actually excludes several tags, not modelled here
                ['address', 'Block', 'Flow', 'Common'],
                ['hgroup', 'Block', 'Required: h1 | h2 | h3 | h4 | h5 | h6', 'Common'],
				
				// http://developers.whatwg.org/grouping-content.html
                ['figure', 'Block', 'Optional: (figcaption, Flow) | (Flow, figcaption) | Flow', 'Common'],
                ['figcaption', 'Inline', 'Flow', 'Common'],
				
				// http://developers.whatwg.org/the-video-element.html#the-video-element
                ['video', 'Block', 'Optional: (source, Flow) | (Flow, source) | Flow', 'Common', [
                    'src' => 'URI',
					'type' => 'Text',
					'width' => 'Length',
					'height' => 'Length',
					'poster' => 'URI',
					'preload' => 'Enum#auto,metadata,none',
					'controls' => 'Bool',
                ]],
                ['source', 'Block', 'Flow', 'Common', [
					'src' => 'URI',
					'type' => 'Text',
                ]],

				// http://developers.whatwg.org/text-level-semantics.html
                ['s',    'Inline', 'Inline', 'Common'],
                ['var',  'Inline', 'Inline', 'Common'],
                ['sub',  'Inline', 'Inline', 'Common'],
                ['sup',  'Inline', 'Inline', 'Common'],
                ['mark', 'Inline', 'Inline', 'Common'],
                ['wbr',  'Inline', 'Empty', 'Core'],
				
				// http://developers.whatwg.org/edits.html
                ['ins', 'Block', 'Flow', 'Common', ['cite' => 'URI', 'datetime' => 'CDATA']],
                ['del', 'Block', 'Flow', 'Common', ['cite' => 'URI', 'datetime' => 'CDATA']],
            ],
            'attributes' => [
                ['iframe', 'allowfullscreen', 'Bool'],
                ['table', 'height', 'Text'],
                ['td', 'border', 'Text'],
                ['th', 'border', 'Text'],
                ['tr', 'width', 'Text'],
                ['tr', 'height', 'Text'],
                ['tr', 'border', 'Text'],
            ],
        ],
        'custom_attributes' => [
            ['a', 'target', 'Enum#_blank,_self,_target,_top'],
        ],
        'custom_elements' => [
            ['u', 'Inline', 'Inline', 'Common'],
        ],
    ],

];
```
