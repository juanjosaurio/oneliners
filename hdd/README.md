# HDD

Getting HDDs price list sorted by $/GB

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
	<div class="top-caluga">
	        <div class="id-caluga"><span class="txt-id  force-select">24883</span></div>
		<div class="status-caluga">
			<span>+100 Unid.</span>
		</div>
	</div>
	<div class="center-caluga">
	    	<a data-tag="ir_producto" class="noselect" href="/producto/24883-wd-disco-externo-750gb-2-5-usb-3-0-elements-negro">
			        <div class="center-titulo">
				       	<span itemprop="brand" itemscope="" itemtype="http://schema.org/Brand">
				       		<span itemprop="name" class="marca">WD  &reg; </span>
				       	</span>
			             <span class="nombre" itemprop="name">Disco Externo 750GB 2.5" USB 3.0 Elements Negro</span>			           
			        </div>
			        <div class="caluga-img">
			        	
			                <img data-id="caluga_foto" alt="Foto No Disponible" itemprop="image" src="/public/foto/24883/1_100.jpg?t=1549467628"></div>
			        <div class="caluga-txt" itemprop="offers" itemscope="" itemtype="http://schema.org/Offer">			        	
			                <span class="txt">Precio Efectivo</span>
			                <span class="txt-precio"> $       36.990</span>
			                
			                	<meta itemprop="price" content="36990.0000"><meta itemprop="priceCurrency" content="CLP"></div>
		    </a>
	    </div>
	    <div class="bot-caluga">
		    			
		    			<div class="comparador">
		    				<input type="checkbox" id="comparador_24883" value="24883"><a style="cursor:default;">Comparar</a>
		    			</div>
	    </div>
```

* In the following steps some data will be simulated, redacted or modified to suit the explanation. At then end there will be a faithful and honest execution of the command and its full output.

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

Given what Inspect Element showed, the first attribute to extract was the name of the product, located in a *span* element with tags ```class="nombre" itemprop="name"```. So the first attempt was something like this:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/'

			             <span class="nombre" itemprop="name">Disco Externo 750GB 2.5" USB 3.0 Elements Negro</span>			           
			             <span class="nombre" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Elements Negro</span>			           
			             <span class="nombre" itemprop="name">Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3</span>			           
			             <span class="nombre" itemprop="name">Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro</span>
			             <span class="nombre" itemprop="name">Disco Externo 3TB 2.5" USB 3.0 Elements  </span>			           
			             <span class="nombre" itemprop="name">Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul</span>			           
			             <span class="nombre" itemprop="name">Disco Externo 4TB 2.5" USB 3.0 Portable Expansi&oacute;n </span>			           
[...]
```
We have the product name and the HDD capacity. We could have used *sed* to extract the content of the *span* element but there's a fun way to do it in *awk*, the *match* function. Acording to the [GNU Manual Page](https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html) *match* will search a string for a regular expression and return the position of the occurence. If a third parameter is provided, it will also fill that array with the matched portion of the string or the parenthesized matching subexpressions.

So we should use *match* with a regular expression to extract what's inside the *span* element. And what regular expression could extract that? Start with the end of the opening tag ```>``` followed by whatever is not the beggining of the closing tag ```[^>]*``` and finish with the beggining of the closing tag ```>```, with parenthesis for the sake of extraction.

The next attempt was something like this:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; print product;}'

Disco Externo 750GB 2.5" USB 3.0 Elements Negro			           
Disco Externo 1TB 2.5" USB 3.0 Elements Negro			           
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3			           
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro
Disco Externo 3TB 2.5" USB 3.0 Elements  			           
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul			           
Disco Externo 4TB 2.5" USB 3.0 Portable Expansi&oacute;n 			           
[...]
```

#### Capacity

Now, to extract the capacity, let's use *match* again, but this time with a regular expression representing a number followed by a unit (GB or TB).

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; print tmpcap}'

750GB			           
1TB		           
1TB			           
1TB
3TB  			           
2TB			           
4TB 			           
[...]
```

If we want to compare, and avoid decimals for now, we should convert capacity to GB. So we need to check for the unit to set a factor (to later multiply number times factor):


```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} print factor;}'

1			           
1024		           
1024			           
1024
1024  			           
1024			           
1024 			           
[...]
```

Now, to extract the number, we use *match* again, but this time the parenthesis in the regular expression will only capture the number and not the unit.


```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); print f[1];}'

750			           
1		           
1			           
1
3  			           
2			           
4 			           
[...]
```

Now we calculate the capacity:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor; print capacity;}'

750			           
1024		           
1024			           
1024
3072  			           
2048			           
4096 			           
[...]
```

#### Price

Now we must extract the price, which is located in a *meta* element inside a *span* element, with tags *itemprop* always equals to *price* (this will be the pattern to look for). For the extraction, *match* to the rescue (again). This time with a regular expression to match with the tags but to only extract the integer part of the price.

```bash
cat page.html | awk '/meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print price}'

89990
93990
94990
97990
110990
113990
119990 			           
[...]
```

### Getting what we want

Now we integrate both *awk* pattern+action into a single script:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}'

Disco Externo 750GB 2.5" USB 3.0 Elements Negro;750;89990;119.9866
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1024;93990;91.7871
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3;1024;94990;92.7636
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro;1024;97990;95.6933
Disco Externo 3TB 2.5" USB 3.0 Elements;3072;110990;36.1295
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2048;113990;55.6591
Disco Externo 4TB 2.5" USB 3.0 Portable Expansi&oacute;n;4096;119990;29.2944
[...]
```

And here we find a problem. The arbitrary field separator used here was ```;``` and according to *awk* not all the lines have the same number of fields:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}' | awk -F';' '{print NF}'

4
4
4
4
4
4
5
[...]
```

The product description contains HTML encoded text (the last line shown), so we must find some way to decode it. I had *perl* at hand so I installed *perl-HTML-Parser* and this is how it went:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);'

Disco Externo 750GB 2.5" USB 3.0 Elements Negro;750;89990;119.9866
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1024;93990;91.7871
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3;1024;94990;92.7636
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro;1024;97990;95.6933
Disco Externo 3TB 2.5" USB 3.0 Elements;3072;110990;36.1295
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2048;113990;55.6591
Disco Externo 4TB 2.5" USB 3.0 Portable Expansi?n;4096;119990;29.2944
[...]
```

It still wasn't showing the right character, so we must use *iconv*:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8

Disco Externo 750GB 2.5" USB 3.0 Elements Negro;750;89990;119.9866
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1024;93990;91.7871
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3;1024;94990;92.7636
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro;1024;97990;95.6933
Disco Externo 3TB 2.5" USB 3.0 Elements;3072;110990;36.1295
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2048;113990;55.6591
Disco Externo 4TB 2.5" USB 3.0 Portable Expansión;4096;119990;29.2944
[...]
```

Finally it's showing the information the way I want. Now we must sort it to find the cheapest GB with *sort*:

```bash
cat page.html | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8 | sort -r -t ';' -k 4n

Disco Externo 4TB 2.5" USB 3.0 Portable Expansión;4096;119990;29.2944
Disco Externo 3TB 2.5" USB 3.0 Elements;3072;110990;36.1295
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2048;113990;55.6591
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1024;93990;91.7871
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3;1024;94990;92.7636
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro;1024;97990;95.6933
Disco Externo 750GB 2.5" USB 3.0 Elements Negro;750;89990;119.9866
[...]
```

The first line shows the cheapest GB (not necesarily the cheapest buy).

### The Promise

And now, the live command with *curl* and its real output:

```bash
curl -s "https://www.pcfactory.cl/discos-externos?categoria=422&papa=706" | awk '/class="nombre" itemprop="name"/{match($0, />([^<]*)</, f); product=f[1]; match(product, /([0-9]+[GT]B)/, f);tmpcap=f[1]; if(tmpcap ~ /TB/){factor=1024;}else{factor = 1;} match(tmpcap, /([0-9]+)[GT]B/, f); capacity=f[1]*factor;} /meta itemprop="price"/{match($0, /price" content="([0-9]+)\./, f);price=f[1]; print product";"capacity";"price";"price/capacity;}' | perl -MHTML::Entities -pe 'decode_entities($_);' | iconv -f iso-8859-1 -t utf-8 | sort -r -t ';' -k 4n

Disco Externo 4TB 2.5" USB 3.0 Portable Expansión ;4096;79990;19.5288
Disco Externo 4TB 2.5" USB3.0 Elements Negro;4096;94990;23.1909
Disco Externo 4TB Canvio Basics Black;4096;97990;23.9233
Disco Externo 3TB 2.5" USB 3.0 Elements  ;3072;79990;26.0384
Disco Externo 2TB 2,5" USB 3.0 Portable Expansion Drive Negro;2048;54990;26.8506
Disco Externo 2TB 2.5" USB 3.0 Elements Negro ;2048;54990;26.8506
Disco Externo 4TB Canvio Advance Black;4096;110990;27.0972
Disco Externo 8TB 3.5" USB 3.0 Back Up Plus Hub Desktop Software de Respaldo;8192;237490;28.9905
Disco Externo 3TB 2.5" USB 3.0 My Passport Respaldo Automático Negro ;3072;89990;29.2936
Disco Externo 4TB para Mac 2.5" USB 3.0 My Passport Respaldo Automático;4096;119990;29.2944
Disco Externo 4TB 2.5" USB 3.0 My Passport Respaldo Automático;4096;123490;30.1489
Disco Externo 3TB 3,5" USB 3.0 Expansion Desktop Negro;3072;93990;30.5957
Disco Externo 6TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;6144;189990;30.9229
Disco Externo 8TB 3.5" USB 3.0 My Book Desktop Respaldo Automático;8192;256490;31.3098
Disco Externo 2TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;2048;64990;31.7334
Disco Externo 2TB 2.5" USB 3.0 My Passport Respaldo Automático Azul ;2048;64990;31.7334
Disco Externo 2TB 2.5" USB 3.0 Canvio Basics Black A3;2048;66490;32.4658
My Passport 4TB para Mac USB 3.0;4096;142490;34.7876
Disco Externo 4TB My Cloud Home Almacenamiento Online;4096;149990;36.6187
Disco Externo 3TB 2.5" USB 3.0 (c/ adaptador USB C) Canvio Premium Plateado P2 - Respalda con Seguridad;3072;113990;37.1061
Disco Externo 1TB 2.5" USB 3.0 Elements Negro;1024;39990;39.0527
Disco Externo 1TB 2.5" USB 3.0 Canvio Basics Black A3;1024;39990;39.0527
Disco Externo Antigolpes 2TB 2,5" USB 3.0 Azul;2048;79990;39.0576
Disco Externo 6TB My Cloud Home Almacenamiento Online;6144;239990;39.0609
Disco Externo 2TB 2.5" USB 3.0 (c/ adaptador USB C) Canvio Premium Plateado P2 - Respalda con Seguridad;2048;85490;41.7432
Disco Externo 2TB 2.5" USB 3.0 My Passport Ultra Metal Negro-Gris;2048;89990;43.9404
Disco Externo 2TB 2.5" USB 3.0 My Passport Ultra Metal Blanco-Gold;2048;89990;43.9404
Disco Externo 1TB 2,5" USB 3.0 Portable Expansion Drive Negro;1024;45990;44.9121
Disco Externo 1TB 2.5" USB 3.0 Advance V9 Rojo - Respalda, SIncroniza y Comparte;1024;45990;44.9121
Disco Externo 1TB 2.5" USB 3.0 Advance V9 Negro - Respalda, SIncroniza y Comparte;1024;45990;44.9121
Disco Externo 1TB 2.5" USB 3.0 Advance V9 Azul - Respalda, SIncroniza y Comparte;1024;45990;44.9121
Disco Externo Antigolpes 1TB 2,5" USB 3.0 Azul;1024;49990;48.8184
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Automático Negro;1024;49990;48.8184
Disco Externo 1TB 2.5" USB 3.0 My Passport Respaldo Automático Blanco;1024;49990;48.8184
Disco Externo 750GB 2.5" USB 3.0 Elements Negro;750;36990;49.32
Disco Externo 1TB 2.5" Para Mac USB 3.0 My Passport Respaldo Automático Negro;1024;56990;55.6543
Disco Externo 1TB para Mac 2.5" USB 3.0 My Passport Respaldo Automático;1024;59990;58.584
Disco Externo 1TB 2.5" USB 3.0 My Passport Ultra Metal Negro-Gris;1024;69990;68.3496
Disco Externo 1TB 2.5" USB 3.0 My Passport Ultra Metal Blanco-Gold;1024;69990;68.3496
Disco Externo 2TB My Cloud Home Almacenamiento Online;2048;139990;68.3545
Disco Externo 1TB 2.5" USB 3.0 Rugged Triple Interface Antigolpes;1024;89990;87.8809
Disco Externo 500GB Wireless ;500;64990;129.98
Disco Externo Cloud 2TB Time Capsule WiFi para Red Inalambrica;2048;279990;136.714
Disco Externo 1TB 2.5"  My Passport Wireless;1024;153190;149.6
```

### Limitations

This is a one-liner, an optimistic, quick and dirty way to do something. A lot of thing can break it:

* Any change in the web page structure, elements and/or tags used
* The way the capacity is displayed (without a space between the number and the unit)
* A new unit (MB? PB?)

### Update

As of today, March 17th 2021, the one-liner still works :-)

