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

### Extracting the data

So far I could user *curl* to get the web page in HTML format:

```
curl -s https://www.pcfactory.cl/discos-externos?categoria=422&papa=706
```

For the sake of not getting my IP address blacklisted I'll store the contents of the web page in *page.html* and will replace *curl*ing the URL with *cat*ing the file.

```
cat page.html
```

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

So we should use *match* with a regular expression to extract what's inside the *span* element. And what regular expression could extract that? Start with the end of the opening tag ```>``` followed by whatever is not the beggining of the closing tag ```[^>]*``` and finish with the beggining of the closing tag ```>```.

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


