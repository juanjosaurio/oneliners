# HDD

Getting HDDs price list sorted by $/TB

## The Motivation

In the past, when considering the idea of buying a HDD, I used to open a popular webpage selling HDD and copy-pasting (or typing) information into an Excel spreadsheet in order to calculate the cost-per-gigabyte and sorting the list by that column.

Last time I thought that I could try to make it a one-liner just for fun, so here's the story.

## Beginning

### Fair Warning

There's absolutely no endorsement of this particular commerce, I'm not getting paid for using their URL, so I'll avoid naming them. I'll leave the URL as is so the steps shown here can be reproduced.

### The URL

The main site URL is ```https://www.pcfactory.cl/```

The specific URL for the External HDDs category is ```https://www.pcfactory.cl/discos-externos?categoria=422&papa=706``` which will get you a ~470Kb HTML file.

### Finding the data

One of the more simple ways is to load the page in a modern browser, find the data visually, right-clik, Inspect Element and *voilà*. You get the general idea of the structure around the data.

```HTML
<div class="product " data-class="Caluga" id="caluga_42040">
    <input type="hidden" id="data_mentalidad_web_42040"
        value='[{"name":"Disco Externo 14TB 3.5\" USB 3.0 Elements Desktop","id":"42040","price":"329990","brand":"WD  \u00ae ","category":"Discos Externos","variant":"","quantity":1,"position":42}]'>

    <div class="product__heading" itemprop="brand" itemscope="" itemtype="http://schema.org/Brand">
        <a class="product-ab-link" data-tag="ir_producto"
            href="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop" id="42040"></a>

        <div class="card-title color-dark" itemprop="name">WD &reg; </div>
        <p class="link color-primary-1 product__category-title">Discos Externos</p>
    </div>

    <div class="p-relative">
        <a class="product-ab-link" data-tag="ir_producto"
            href="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop" id="42040"></a>
        <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 14TB 3.5" USB 3.0 Elements
            Desktop</div>
    </div>

    <div class="product__units">
        <a class="product-ab-link" data-tag="ir_producto"
            href="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop" id="42040"></a>

        <div class="p-relative">
            <p class="link color-dark"><strong>ID</strong> <span>42040</span></p>
            <button class="product-ab-link copy-id copy-id-42040" data-clipboard-text="42040"
                data-link="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop"></button>
        </div>

        <p class="link--sm color-gray-1">8 Unid.</p>
    </div>

    <div class="product__image">
        <a class="product-ab-link" data-tag="ir_producto"
            href="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop" id="42040"></a>
        <img alt="Foto No Disponible" itemprop="image"
            src="https://www.pcfactory.cl/public/foto/42040/1_200.jpg?t=1712343871825">
    </div>

    <div class="product__price">

        <div class="product__price-texts" itemprop="offers" itemscope="" itemtype="http://schema.org/Offer">
            <a class="product-ab-link" data-tag="ir_producto"
                href="/producto/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop" id="42040"></a>
            <div class="title-md color-primary-1 alineado-porcentaje-precio"><span
                    class="porcentaje-card-precio">-29%</span> $ 329.990</div>
            <div class="title-sm color-gray-2 texto-tachado">$ 464.090</div>
            <div class="price color-dark">Precio Oferta Efectivo</div>
            <meta itemprop="price" content="$ 329.990">
            <meta itemprop="priceCurrency" content="CLP">
        </div>

        <div>
            <button class="button-no-decoration button-pointer-e-none"
                onmousedown="try { rrApi.addToBasket(42040) } catch(e) {}" id="addtocart_42040_1">
                <img src="/public/dist/images/design/add_cart.svg">
            </button>
        </div>
    </div>
    <div class="product__price " style="gap:0 !important; justify-content:initial !important;">
        <div>
            <img src="/public/dist/images/design/icono_tarjeta_credito.svg" style="width:30px;">
        </div>
        <div>
            <p class="price">Hasta 24 cuotas<strong> sin inter&eacute;s*</strong></p>
        </div>
    </div>
    <div class="product__footer">
        <input type="hidden" id="json_producto_compara_42040" data-type="json"
            value='{"marca":"WD  \u00ae ","categoria":"Discos Externos","nombre":"Disco Externo 14TB 3.5\" USB 3.0 Elements Desktop","precio_titulo":"Precio Oferta Efectivo","precio":"$ 329.990","foto":"https:\/\/www.pcfactory.cl\/public\/foto\/42040\/1_200.jpg?t=1712343871825","url":"\/producto\/42040-wd-disco-externo-14tb-3-5-usb-3-0-elements-desktop"}'>
        <button class="button-icon button-icon--small" id="comparador_42040" value="42040">
            <img src="/public/dist/images/design/compare.svg">
            <span class="color-gray-1">Comparar</span>
        </button>
    </div>

    <div class="product__add-to-cart">
        <div class="button-bg-icon button-bg-icon--bordered">
            <button id="addtocart">A&ntilde;adir al carro</button>
            <img src="/public/dist/images/design/add_cart.svg">
        </div>
    </div>

    <input data-id="producto_familia" type="hidden">
</div>
```

* In the following steps some data will be simulated, redacted or modified to suit the explanation. At the end there will be a faithful and honest execution of the command and its full output.

### Extracting the data

So far I could use *curl* to get the web page in HTML format:

```
curl -s "https://www.pcfactory.cl/discos-externos?categoria=422&papa=706"
```

For the sake of not getting my IP address blacklisted I'll store the contents of the web page in *page.html* and will replace *curl*ing the URL with *cat*ing the file.

```
cat page.html
```

#### Product Name

Given what Inspect Element showed, the first attribute to extract was the name of the product, located in a *div* element with tags ```class="price color-dark-2  product__card-title" itemprop="name"```. So the first attempt was something like this:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/'

          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Elements Negro</div>
          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Basic</div>
          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5</div>
          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green</div>
          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Autom&aacute;tico Negro</div>
          <div class="price color-dark-2  product__card-title" itemprop="name">Disco Externo 2TB 2.5" USB 3.0 Elements Negro </div>
			           
[...]
```
We have the product name and the HDD capacity. We could have used *sed* to extract the content of the *div* element but there's a fun way to do it in *awk*, the *match* function. Acording to the [GNU Manual Page](https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html) *match* will search a string for a regular expression and return the position of the occurence. If a third parameter is provided, it will also fill that array with the matched portion of the string or the parenthesized matching subexpressions.

So we should use *match* with a regular expression to extract what's inside the *div* element. And what regular expression could extract that? Start with the end of the opening tag ```>``` followed by whatever is not the beggining of the closing tag ```[^>]*``` and finish with the beggining of the closing tag ```>```, with parenthesis for the sake of extraction.

The next attempt was something like this:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; print product;}'

Disco Externo 1TB 2.5" USB 3.0 Elements Negro
Disco Externo 1TB 2.5" USB 3.0 Basic
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5
Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Autom&aacute;tico Negro
Disco Externo 2TB 2.5" USB 3.0 Elements Negro
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul		           
[...]
```

#### Capacity

Now, to extract the capacity, let's use *match* again, but this time with a regular expression representing a number followed by a unit (GB or TB).

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1]; print capacity}'

1
1
1
1
1
2
1
2
2
2
1 			           
[...]
```

In the previous version of this page and script, there was a section devoted to normalize the capacity in GB, but now there are no products being offered with its capacity expressed in GB.

#### Price

Now we must extract the price, which is located in a *meta* element inside a *div* element, with tags *itemprop* always equals to *price* (this will be the pattern to look for) and the actual price is the value for tag *content*. For the extraction, *match* to the rescue (again). This time with a regular expression to match with the tags but to only extract the integer part of the price.

```bash
cat page.html | awk '/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1]; print price}'

49.990
56.690
58.190
59.390
59.990
66.990
73.290
75.990		           
[...]
```

To remove the dot, we must add a simple substitution using the *sub* function.

```bash
cat page.html | awk '/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print price}'

49990
56690
58190
59390
59990
66990
73290
75990
[...]
```

### Getting what we want

Now we integrate both *awk* pattern+action into a single script:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}'

Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1;49990;49990
Disco Externo 1TB 2.5" USB 3.0 Basic;1;56690;56690
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5;1;58190;58190
Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green;1;59390;59390
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Autom&aacute;tico Negro;1;59990;59990
Disco Externo 2TB 2.5" USB 3.0 Elements Negro ;2;66990;33495
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul;1;73290;73290
Disco Externo 2TB 2.5" USB 3.0 Basic;2;75990;37995
[...]
```

And here we find a problem. The arbitrary field separator used here was ```;``` and according to *awk* not all the lines have the same number of fields:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}' | awk -F';' '{print NF}'

4
4
4
4
5
4
4
4
4
[...]
```

The product description contains HTML encoded text (the last line shown), so we must find some way to decode it. I had *perl* at hand so I installed *perl-HTML-Parser* and this is how it went:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);'

Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1;49990;49990
Disco Externo 1TB 2.5" USB 3.0 Basic;1;56690;56690
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5;1;58190;58190
Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green;1;59390;59390
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Autom▒tico Negro;1;59990;59990
Disco Externo 2TB 2.5" USB 3.0 Elements Negro ;2;66990;33495
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul;1;73290;73290
Disco Externo 2TB 2.5" USB 3.0 Basic;2;75990;37995
[...]
```

It still wasn't showing the right character, so we must use *iconv*:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}'' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8

Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1;49990;49990
Disco Externo 1TB 2.5" USB 3.0 Basic;1;56690;56690
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5;1;58190;58190
Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green;1;59390;59390
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;1;59990;59990
Disco Externo 2TB 2.5" USB 3.0 Elements Negro ;2;66990;33495
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul;1;73290;73290
Disco Externo 2TB 2.5" USB 3.0 Basic;2;75990;37995
[...]
```

Finally it's showing the information the way I want. Now we must sort it to find the cheapest GB with *sort*:

```bash
cat page.html | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8 | sort -r -t ';' -k 4n

Disco Externo 14TB 3.5" USB 3.0 Elements Desktop;14;329990;23570.7
Disco Externo 12TB 3.5" USB 3.0 Desktop Expansion.;12;305990;25499.2
Disco Externo 8TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;8;209990;26248.8
Disco Externo 6TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;6;164990;27498.3
Disco Externo 5TB WD_Black P10 Game Drive;5;144990;28998
Disco Externo 4TB 2.5" USB 3.0 Canvio Advance v10 Green;4;118290;29572.5
Disco Externo 4TB Canvio Basics Black A5;4;118690;29672.5
Disco Externo 4TB 2.5" USB3.0 Elements Negro;4;119990;29997.5
Disco Externo 5TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;5;149990;29998
Disco Externo 5TB 2.5" USB 3.0 Elements Negro;5;151990;30398
[...]
```

The first line shows the cheapest TB (not necesarily the cheapest buy).

### The Promise

And now, the live command with *curl* and its real output:

```bash
curl -s "https://www.pcfactory.cl/discos-externos?categoria=422&papa=706" | awk '/class="price color-dark-2  product__card-title" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+)TB/, f);capacity=f[1];}/meta itemprop="price"/{match($0, /price" content="\$ ([0-9\.]+)"/, f);price=f[1];sub(/\./, "", price); print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8 | sort -r -t ';' -k 4n

Disco Externo 14TB 3.5" USB 3.0 Elements Desktop;14;329990;23570.7
Disco Externo 12TB 3.5" USB 3.0 Desktop Expansion.;12;305990;25499.2
Disco Externo 8TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;8;209990;26248.8
Disco Externo 6TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;6;164990;27498.3
Disco Externo 5TB WD_Black P10 Game Drive;5;144990;28998
Disco Externo 4TB 2.5" USB 3.0 Canvio Advance v10 Green;4;118290;29572.5
Disco Externo 4TB Canvio Basics Black A5;4;118690;29672.5
Disco Externo 4TB 2.5" USB3.0 Elements Negro;4;119990;29997.5
Disco Externo 5TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;5;149990;29998
Disco Externo 5TB 2.5" USB 3.0 Elements Negro;5;151990;30398
Disco Externo 4TB 2.5" USB 3.0 My Passport Respaldo Automático;4;124990;31247.5
Disco Externo 4TB 2.5" USB 3.0 Basic;4;128190;32047.5
Disco Externo 4TB 2.5" Canvio Flex;4;131490;32872.5
Disco Externo 2TB 2.5" USB 3.0 Elements Negro ;2;66990;33495
Disco Externo 5TB 2.5" USB 3.0 Firecuda Gaming;5;169990;33998
Disco Externo 4TB WD_Black P10 Game Drive;4;139990;34997.5
Disco Externo 3TB 2.5" USB 3.0 Elements  ;3;106090;35363.3
Disco Externo 2TB 2.5" USB 3.0 Basic;2;75990;37995
Disco Externo Antigolpes 4TB 2.5" USB 3.0;4;153090;38272.5
Disco Externo 2TB 2.5" USB 3.0 Canvio Basics Black A5;2;76990;38495
Disco Externo 2TB 2.5" USB 3.0 Canvio Advance v10 Green;2;77590;38795
Disco Externo 2TB WD_Black P10 Game Drive;2;84990;42495
Disco Externo 2TB 2.5" para PS4;2;89990;44995
Disco Externo 2TB 2.5" Canvio Gaming;2;89990;44995
Disco Externo 2TB 2.5" Canvio Flex ;2;92990;46495
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2;94890;47445
Disco Externo 4TB G-DRIVE ArmorATD USB 3.2;4;198790;49697.5
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1;49990;49990
Disco Externo 2TB 2.5" USB 3.0 Firecuda Gaming;2;104990;52495
Disco Externo 6TB G-DRIVE Enterprise Class USB 3.2;6;319790;53298.3
Disco Externo 2TB 2.5" WD_Black P10 Game Drive Edición Call of Duty;2;112090;56045
Disco Externo 1TB 2.5" USB 3.0 Basic;1;56690;56690
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A5;1;58190;58190
Disco Externo 1TB 2.5" USB 3.0 Canvio Advance v10 Green;1;59390;59390
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;1;59990;59990
Disco Externo 2TB 2.5" USB 3.0 Rugged Mini;2;122390;61195
Disco Externo 2TB 2.5" Starwars Edition Baby Yoda;2;127490;63745
Disco Externo 2TB G-DRIVE ArmorATD USB 3.2;2;145290;72645
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul;1;73290;73290
Disco Externo Portable 1TB 2.5" USB-C;1;82990;82990
Disco Externo 1TB 2.5" USB 3.0 Rugged Mini;1;95890;95890
Tarjeta de Expansión Xbox Series X | S 1TB;1;249690;249690
```

### Limitations

This is a one-liner, an optimistic, quick and dirty way to do something. A lot of things can break it:

* Any change in the web page structure, elements and/or tags used
* The way the capacity is displayed (without a space between the number and the unit)
* A new unit (MB? PB?)

### Update

~~As of today, March 17th 2021, the one-liner still works :-)~~
Updated on April 27th 2024 to reflect changes on the web page and capacities only expressed on TB.

