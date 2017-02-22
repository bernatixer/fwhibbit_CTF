# fwhibbit_CTF
Solución para el reto del QR, del CTF de Follow The White Rabbit

Buenas! soy bernatixer, en este repositorio podréis encontrar el write-up de uno de los retos del CTF de fwhibbit, concretamente el del QR.

La solución esta en **JavaScript**, si queréis ver la solución en **Python** podéis ir a los siguientes write-ups:
 * e0d1n: https://roothalt.github.io/dev/fwhibbit-ctf-dev/
 * Kalrong: https://blog.kalrong.net/es/2017/02/20/follow-the-white-rabbit-ctf-magic-qr/

Bueno, vamos a empezar!
Si nos dirigimos a la página del reto (http://dev1.ctf.followthewhiterabbit.es:8008/) podremos ver lo siguiente

![QR](http://image.prntscr.com/image/b147c9b8f1f64e37893ff485618a34e1.png)

Como podemos observar tenemos que sacar de alguna manera un hash en SHA1 del código QR antes de que el conejo se coma la zanahoria, es decir tenemos 4 segundos para hacerlo.

Yo empecé estudiando el código QR, a simple vista se ve un poco raro, si además lo intentamos escanear con algún teléfono móvil veremos que en la mayor parte de los casos no funciona. La solución a este problema es invertir los colores. Luego de invertir el blanco y negro ya pude escanear el código, este dio como resultado una serie de ‘-’ y ‘.’, podemos deducir que se trata de un código en morse. Solo falto pasar de morse a texto para poder obtener el hash en SHA1. Todo bien, tenemos el método para resolver este problema pero tarde bastante mas que 4 segundos, así que vamos a automatizarlo con JavaScript.

Resumiendo lo que debemos hacer en nuestro código:
 - Invertir los colores de la imagen
 - Escanear el código QR
 - Pasar de morse a texto

Para invertir los colores de la imagen usaremos Pixastic (http://dph.am/pixastic-docs/docs/actions/invert/)
Para escanear el código QR usaremos QRCodeDecoder (https://cirocosta.github.io/qcode-decoder/)
Para pasar de morse a texto usaremos esta pequeña libreria (https://neocotic.com/mor.js/)

Solo nos queda juntarlo todo en un archivo así que empezaremos creando un archivo html con una imagen, preferiblemente copiaremos el código de una de las que nos da la web del reto:

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAUAAAAFACAIA . . . . . . XfgVlYLLUAAAAAAElFTkSuQmCC">
```
... y importamos las librerías que hemos comentado anteriormente ...
```html
<script src="http://dev1.ctf.followthewhiterabbit.es:8008/js/jquery-3.1.1.min.js"></script>
<script src="https://cirocosta.github.io/qcode-decoder/build/qcode-decoder.min.js"></script>
<script src="mor.min.js"></script> <!-- Tengo el script localmente, link (https://raw.githubusercontent.com/neocotic/mor.js/master/mor.js) -->
<script src="http://dph.am/pixastic-docs/lib/download/pixastic.js"></script>
```
Como podéis ver estoy importando jQuery, ya que la página del reto también lo usa, así que nos aprovecharemos de eso

Bien, ya tenemos las librerías importadas, solo nos queda usarlas! Así que empezamos con Pixastic
```javascript
var img = document.getElementsByTagName("img")[0]; // Seleccionamos la imagen
Pixastic.process(img, "invert"); // Invertimos la imagen (fácil, no? :D)
var canvas = document.getElementsByTagName("canvas")[0]; // Al invertir la imagen nos devuelve un canvas, así que lo seleccionamos
var dataURL = canvas.toDataURL("image/png"); // Obtenemos el dataURL del canvas
```
Con estas cuatro líneas ya tenemos la dataURL de la imagen invertida, solo nos faltan dos pasos, escanear y traducir, vamos a por ello:
```javascript
QCodeDecoder().decodeFromImage(dataURL, function (err, result) { // result nos dara el codigo morse que queremos
	console.log(result);
	console.log(morjs.decode(result, {mode: 'simple'})); // Comprobamos que esto funciona
});
```
Veamos como queda el código completo:
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAUAAAAFACAIA . . . . . . XfgVlYLLUAAAAAAElFTkSuQmCC">
<script src="http://dev1.ctf.followthewhiterabbit.es:8008/js/jquery-3.1.1.min.js"></script>
<script src="https://cirocosta.github.io/qcode-decoder/build/qcode-decoder.min.js"></script>
<script src="mor.min.js"></script>
<script src="http://dph.am/pixastic-docs/lib/download/pixastic.js"></script>
<script>
var img = document.getElementsByTagName("img")[0];
Pixastic.process(img, "invert");
var canvas = document.getElementsByTagName("canvas")[0];
var dataURL = canvas.toDataURL("image/png");

QCodeDecoder().decodeFromImage(dataURL, function (err, result) { // result nos dara el codigo morse que queremos
	console.log(result);
	console.log(morjs.decode(result, {mode: 'simple'})); // Comprovamos que esto funciona
});
</script>
```
Solo nos queda probarlo, así que abrimos el archivo html y vamos a la consola
![Consola](http://image.prntscr.com/image/55dcda7a114a48938c892485352be9bf.png)
Funciona correctamente! El último paso es ponerlo en la página del reto, para eso tendremos que hacer algunas modificaciones, ya que lo que haremos sera pegar el código JavaScript en la consola local de la web del reto, para esto tendremos que importar los scripts manualmente. Una manera fácil es minimizar (https://javascript-minifier.com/) el código. nos quedara algo así:
```javascript
.
.
.
código de los tres archivos (Pixastic, QRCodeDecoder, mor.js)
.
.
.
var img = document.getElementsByTagName("img")[0];
Pixastic.process(img, "invert");
var canvas = document.getElementsByTagName("canvas")[0];
var dataURL = canvas.toDataURL("image/png");

QCodeDecoder().decodeFromImage(dataURL, function (err, result) {
	console.log(morjs.decode(result, {mode: 'simple'}));
	$('#solution').val(morjs.decode(result, {mode: 'simple'})); // Pondra automaticamente el hash SHA1 en el input
});
```
Finalmente cuando cargue la página del reto, pegamos rapidamente este código en la consola y se nos pondrá el SHA1, le damos a 'Send' y *voilà*
![Voila](http://image.prntscr.com/image/e160b491ed2342598a812a012ba5ad6c.png)
