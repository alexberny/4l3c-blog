---
title: "Middleam"
date: 2014-04-21T15:31:02Z
draft: false
tags:
  - middleman
---

Voglio parlarvi di uno strumento che sto utilizzando ormai da un anno in quasi tutti i progetti di siti statici cui ho lavorato.
Si tratta di [Middleman](http://middlemanapp.com/), un generatore di siti statici, basato su Ruby e Sinitra.
L'utilità di Middleman non riguarda solo la generazione di siti statici, ma anche di blog (vedi questo), o la prototipizzazione di progetti che poi saranno implementati in altri modi.

### Cos'è un generatore di siti statici

Un generatore di siti statici è uno strumento che facilita la creazione di siti a partire da basi che sono proprie dei CMS o altri tool di sviluppo: separare i contenuti dal layout grafico, applicare vari temi a seconda del contenuto o incorporare risorse solo nelle pagine che ne hanno bisogno.
Il tutto senza scrivere centinaia di volte le stesse linee di codice ma partendo da risorse pre-compilate, o pre-processate, in linguaggi come Haml, Markdown, Sass, Less, Coffescreept.
Una volta compilato il tutto si otterranno le pagine html, i file css e javascript.

### Passo 1: l'installazione

Per installare Middleman basta digitare

``` bash
gem install middleman
```

da linea di comando.

Middleman mette a disposizione tre comandi:

``` bash
middleman init
middleman server
middleman build
```

Il primo inizializza un nuovo progetto, il secondo lancia il server locale per lo sviluppo del sito, il terzo genera le pagine che poi saranno caricate on-line.

### Passo 2: Creazione di un progetto

Creaiamo un nuovo progetto con il comando

``` bash
middleman init my_proj_name
```

Di default, Middleman mette a disposizione diversi template mentre altri sono liberamente scaricabili dal sito.
Per creare un pregetto utilizzando un template alternativo digitare il comando

``` bash
middleman init my_project --tempate=your_template
```

### Passo 3: La struttura del progetto

All'interno del progetto troverete una struttura simile a questa:

``` bash
proj
|- config.rb
|- Gemfile
|- source
   |- javascripts
      |- all.js
   |- stylesheets
      |- all.css
   |- layouts
      |- layout.erb
   |- images
   |- index.html.erb
```

Il file `config.rb` contiene i parametri di configurazione per lo sviluppo, il building e il deploy del progetto.
Il file `Gemfile` contiene le 'gemme' di Ruby necessarie al progetto.
Inutile dire che questi due file vanno modificati a secondo delle necessità.

Il resto sono i file javascript, css, il layout e un file index per l'inizio dello sviluppo del sito.

### Passo 4: Sviluppo del progetto

Scegliete il vostro editor preferito per lo sviluppo delle varie parti del progetto.
Lanciate il comando

``` bash
bundle exec middleman
```

per avviare Middleman e seguire la compilazione real-time del progetto all'url <http://localhost:4567> troverete il vostro progetto.
Volendo si può abilitare l'estensione `Livereload` in `config.rb` durante lo sviluppo in modo tale da ricaricare in automatico le pagine ad ogni modifica, liberandovi di questo 'ingrato' compito.

Vediamo in dettaglio alcuni aspetti dello sviluppo.

#### Scaricare i gem file necessari

Lanciare il comando

``` bash
bundle install
```

che provvederà ad installare le gemme necessarie al vostro progetto.

Ogni tanto è bene lanciare anche

``` bash
bundle update
```

per chiedere un aggiornamento di quelle già installate.

#### Aggiungere pagine al sito

Niente di più semplice: basta creare i file all'interno della directory `source` ed assegnargli un'estensione `.html.erb`.
Potete iniziare copiando il contenuto della pagina `index.html.erb` di esempio.
Aprendo questa pagina si nota che essa non è una normale pagina html.
Mancano infatti tutti i tag `html`, `head`, `body`, ... ma sono presenti una sezione racchiusa tra due linee di 3 trattini (- - -) e una parte di codice html.
Niente paura: i tag mancanti sono presenti all'interno del file di layout e Middleman si preoccuperà di integrarli all'interno della pagina al posto vostro.

Vediamo un attimo in dettaglio la pagina. La parte in alto, racchiusa tra i trattini, è formattata in stile *frontmatter* e contiene alcuni informazioni riguardo la pagina: il titolo, eventuali tag, data di creazione, autore, informazioni aggiuntive.
Queste informazioni sono accessibili a Middleman tramite il codice `current_page.data.dato`. Ogni volta che lo inserirete all'interno del layout sarà premura di Middleman sostituirlo con il contenuto del rispettivo `dato` del frontmatter nella pagina attuale.

La restante parte, invece, racchiude il codice html che compone la pagina in questione. Esso è associato al codice `yield` all'interno del layout.

In questo modo non dovrete ricordarvi di scrivere e riscrivere decine di tag, aprirli, chiuderli, indentarli, ecc in ogni pagina: lo fate una volta all'interno del layout e poi lasciate il lavoro sporco a Middleman.

Eccovi un esempio di pagina in *.erb*

``` bash

---
title: Welcome to Middleman
---

<div class="welcome">
  <h1>Middleman is Watching</h1>
  <p class="doc">
    <%= link_to "Read Online Documentation", "http://middlemanapp.com/" %>
  </p><!-- .doc -->
</div><!-- .welcome -->
```

#### Layout

Il file `layouts/layout.erb` contiene il layout di base per tutte le pagine html che andrete a creare all'interno della directory `source`.
Potete decidere di creare altri layout da applicare a sezioni\pagine del sito semplicemente creando altri file all'interno di questa directory e applicandoli poi alle singole pagine.
La potenza di Middleman permette anche di applicare ai layout il concetto di 'eredità' proprio dei linguaggi di programmazione: un layout di base che viene ereditato e migliorato da altri layout per le varie sezioni del sito.

Ecco un esempio di layout.erb

``` bash
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">

    <!-- Always force latest IE rendering engine or request Chrome Frame -->
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">

    <!-- Use title if it's in the page YAML frontmatter -->
    <title><%= current_page.data.title || "The Middleman" %></title>

    <%= stylesheet_link_tag "normalize", "all" %>
    <%= javascript_include_tag  "all" %>
  </head>

  <body class="<%= page_classes %>">
    <%= yield %>
  </body>
</html>
```

Notate le variabili `current_page.data.title` e `yield` citati in precedenza e alle quali saranno sostituiti rispettivamente i valori di *title* del frontmater e del contenuto html che compone la pagina.

All'occhio balzano anche le righe `<%= stylesheet_link_tag "normalize", "all" %>` e `<%= javascript_include_tag  "all" %>`; esse sono dei *template helpers* che approfondiremo tra poco.

In fase di building (sia durante lo sviluppo che in fase di deploy), Middleman crea una pagina `index.html` partendo dal `layout.html.erb` e sostituendovi i valori delle variabili e del contenuto del file `index.html.erb`.
Il file ottenuto è

``` html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">

    <!-- Always force latest IE rendering engine or request Chrome Frame -->
    <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">

    <!-- Use title if it's in the page YAML frontmatter -->
    <title>Welcome to Middleman</title>

    <link href="/stylesheets/normalize.css" rel="stylesheet" type="text/css" /><link href="/stylesheets/all.css" rel="stylesheet" type="text/css" />
    <script src="/javascripts/all.js" type="text/javascript"></script>
  </head>

  <body class="index">
    <div class="welcome">
  <h1>Middleman is Watching</h1>
  <p class="doc">
    <a href="http://middlemanapp.com/">Read Online Documentation</a>
  </p><!-- .doc -->
</div><!-- .welcome -->
  </body>
</html>
```

Per non associare nessun layout ad una pagina, per esempio alla sitemap, nel file `config.rb` inserite la riga

``` bash
page "/path/to/file.html", :layout => false
```

oppure la riga

``` bash
...
layout: false
...
```

all'interno del frontmatter della pagina in questione.

#### Partials

Sebbene l'utilizzo del file `layout` semplifichi molto la creazione del sito, molto spesso è preferibile dividere in più file  le varie sezioni della pagina.
Per fare ciò, Middleman mette a disposizione i cosiddetti *partial*: file parziali che vengono inclusi all'interno del layout.
La comodità dei partial sta nel semplificare il file di layout, e quindi la sua manutenibilità, e nel riutilizzo di parti di layout in caso di presenza di moltepici layout all'interno del progetto.

Supponendo di avere un file `_header.erb` nella cartella partials così definito:

``` bash
<head>
  <meta charset="utf-8">

  <!-- Always force latest IE rendering engine or request Chrome Frame -->
  <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible">

  <!-- Use title if it's in the page YAML frontmatter -->
  <title><%= current_page.data.title || "The Middleman" %></title>

  <%= stylesheet_link_tag "normalize", "all" %>
  <%= javascript_include_tag  "all" %>
</head>
```

il file di layout corrispondente si presenta in questo modo

``` bash
<!doctype html>
<html>
  <%= partial "partials/header" %>

  <body class="<%= page_classes %>">
    <%= yield %>
  </body>
</html>
```

Notare: il nome del partial inizia con underscore _ , che non viene inserito nel nome nell'inclusione, e non ha estensione html!

#### Layout multipli

Middleman offre la possibilità di definire molteplici layout all'interno di un'applicazione.
Questo risulta particolarmente comodo quando un sito presenta pagine con template differenti.
I layout multipli possono condividere un layout comune di base(vedi i 'layout annidati').

Per definire un nuovo layout basta creare il file corrispondente all'interno della cartella `layout`, ad esempio `altLayout.html.erb`.
Per associare un layout ad una pagina si può procedere in uno dei seguenti modi.

1. Richiamarlo nel frontmatter della pagina

    ``` bash
    ---
    title: pagina con altro layout
    layout: altLayout
    ---

    <div></div>
    ```

2. Nel file `config.rb`

    ``` bash
    ...
    page 'nome_pagina_con_altro_layout', :layout => 'altLayout'
    ...
    ```

#### Layout annidati - ovvero come ti eredito un layout

Middleman permette quella che potremmo definire 'eredità' dei layout. In alte parole si possono definire più layout impilati.
Un esempio di layout annidati lo avete con questo blog: il layout che si applica alle pagine statiche è stato implementato e ampliato per essere applicato ai post del blog.
L'associazione del layout alla pagina procede allo stesso modo di quanto visto nei layout multipli.

Prendendo come esempio il file `layout.html.erb` visto in precedenza come layout base, si è creato un file `article.html.erb` per applicarlo alle sole pagine dei post.

``` bash
<% wrap_layout :layout do %>
  <article>
    <%= yield %>
  </article>
<% end %>
```

In questo modo, Middleman sostituisce la vaiabile `yield` del layout `layout` con il contenuto del nuovo layout.

#### Template helpers

I template helpers non sono altro che degli *shortcut* che è possibile inserire all'interno dei layout per velocizzare la scrittura del progetto.
I più usati sono:

* stylesheet\_link\_tag 'layout', per l'inclusione di un file css
* javascript\_include\_tag 'application', per inclusione file javascript
* favicon\_tag 'images/favicon.png, per le favicon
* link\_to 'My Site', 'http://mysite.com', genera un link html
* content\_for :assets do, racchiude un blocco di codice che verrà sostituito all'interno del layout

Per un'approfondimento si rimanda allla sezione [helpers](http://middlemanapp.com/basics/helpers/) del sito di Middleman.

### Passo 5: La compilazione

La compilazione di un progetto si effettua con il comando:

``` bash
bundle exec middleman build
```

All'interno della directory `build` troverete le pagine html, i file css e javascript pronti per essere caricati sul server.
Nella sezione `configure :build do` del `config.rb` potete modificare alcune preferenze da applicare solo in fase di compilazione.
Risultano molto utili i *minify* e i *compress* per ottenere i file html, css e javascript già ridotti nelle dimensioni.

### Passo 6: Il deploy

L'ultimo passo è la pubblicazione del sito.
A seconda dell'hosting scelto, avete a disposizione varie strade:ftp, git, interfaccia web ...
Middleman offre alcune estensione atte a tale scopo.
Personalmente utilizzo `middleman-deploy` che risulta essere molto versatile in quanto offre la possibilita di effetuare il deploy in molteplici modi.

### Le estensioni

Guardando la pagina delle [estensioni](http://directory.middlemanapp.com/#/extensions/all) di Middleman si trovano molte estensioni che posso facilitare e velocizzare il proprio lavoro.
E' inoltre molto facile scriverne di nuove e in rete si trovano molte guide per farlo.

### Conclusioni

Questa breve introduzione a Middleman ha il solo scopo di aiutare a capire uno strumento leggero ma al tempo stesso potente per la creazione di siti.
