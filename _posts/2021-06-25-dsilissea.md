---
layout: page
categories:
   - programming
tags : 
   - dislessia
title: Dsilissea
teaser: ""
meta_description: ""
image:
  title: "/2021-06-25-dsilissea/header.jpg"
  thumb: "/2021-06-25-dsilissea/header-thumb.jpg"
  caption: ""
  caption_url: ""
---

<p>La dislessia fa parte dei disturbi specifici dell’apprendimento o DSA (manuale DSM-5) ed è una condizione caratterizzata da problemi con la lettura e la diagnosi che si formula è indipendente dall’intelligenza della persona. Diverse persone ne sono colpite in misura diversa; i problemi possono includere difficoltà nella pronuncia delle parole, nella lettura veloce, nella scrittura a mano, nella pronuncia delle parole durante la lettura ad alta voce e nella comprensione di ciò che si legge. Spesso queste difficoltà vengono notate inizialmente a scuola. In caso di compromissione totale delle capacità di lettura si parla di alessia. Le difficoltà sono involontarie e le persone con questo disturbo hanno un normale desiderio di apprendimento.</p>*Fonte: [Wikipedia](https://it.wikipedia.org/wiki/Dislessia)*
<p>Nel brano sottostante si è voluto creare un esempio dell'esperienza di lettura di una persona affetta da questo disturbo</p>

<div id="dislessiatext">
<blockquote>
<p>“Signor curato,” disse un di que’ due, piantandogli gli occhi in faccia.</p>

<p>“Cosa comanda?” rispose subito don Abbondio, alzando i suoi dal libro, che gli restò spalancato nelle mani, come sur un leggìo.</p>

<p>“Lei ha intenzione,” proseguì l’altro, con l’atto minaccioso e iracondo di chi coglie un suo inferiore sull’intraprendere una ribalderia, “lei ha intenzione di maritar domani Renzo Tramaglino e Lucia Mondella!”</p>

<p>“Cioè...” rispose, con voce tremolante, don Abbondio: “cioè. Lor signori son uomini di mondo, e sanno benissimo come vanno queste faccende. Il povero curato non c’entra: fanno i loro pasticci tra loro, e poi... e poi, vengon da noi, come s’anderebbe a un banco a riscotere; e noi... noi siamo i servitori del comune.”</p>

<p>“Or bene,” gli disse il bravo, all’orecchio, ma in tono solenne di comando, “questo matrimonio non s’ha da fare, né domani, né mai.”</p>
</blockquote>
</div>



*Fork di [https://geon.github.io/programming/2016/03/03/dsxyliea](https://geon.github.io/programming/2016/03/03/dsxyliea)*




<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
<script type="text/javascript">

"use strict";

$(function(){

	var getTextNodesIn = function(el) {
	    return $(el).find(":not(iframe,script)").addBack().contents().filter(function() {
	        return this.nodeType == 3;
	    });
	};

	// var textNodes = getTextNodesIn($("p, h1, h2, h3"));
	var textNodes = getTextNodesIn($('div[id="dislessiatext"]'));
	
	for (var i = 0; i < textNodes.length; i++) {
			var node = textNodes[i];
			node.oldValue=node.nodeValue;
	}


	function isLetter(char) {
		return /^[\d]$/.test(char);
	}


	var wordsInTextNodes = [];
	for (var i = 0; i < textNodes.length; i++) {
		var node = textNodes[i];

		var words = []

		var re = /\w+/g;
		var match;
		while ((match = re.exec(node.oldValue)) != null) {

			var word = match[0];
			var position = match.index;

			words.push({
				length: word.length,
				position: position
			});
		}

		wordsInTextNodes[i] = words;
	};


	function messUpWords () {
		for (var i = 0; i < textNodes.length; i++) {

			var node = textNodes[i];

			for (var j = 0; j < wordsInTextNodes[i].length; j++) {

				// Only change a tenth of the words each round.
				/*if (Math.random() > 3/10) {

					continue;
				}*/

				var wordMeta = wordsInTextNodes[i][j];

				var word = node.oldValue.slice(wordMeta.position, wordMeta.position + wordMeta.length);
				var before = node.nodeValue.slice(0, wordMeta.position);
				var after  = node.nodeValue.slice(wordMeta.position + wordMeta.length);

				node.nodeValue = before + messUpWord(word) + after;
			};
		};
	}

	function messUpWord (word) {

		if (word.length < 3) {

			return word;
		}

		return word[0] + messUpMessyPart(word.slice(1, -1)) + word[word.length - 1];
	}

	function messUpMessyPart (messyPart) {

		if (messyPart.length < 2) {

			return messyPart;
		}

		var a, b;
		while (!(a < b)) {

			a = getRandomInt(0, messyPart.length - 1);
			b = getRandomInt(0, messyPart.length - 1);
		}

		return messyPart.slice(0, a) + messyPart[b] + messyPart.slice(a+1, b) + messyPart[a] + messyPart.slice(b+1);
	}

	// From https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
	function getRandomInt(min, max) {
		
		return Math.floor(Math.random() * (max - min + 1) + min);
	}


	setInterval(messUpWords, 1000);
});


</script>
