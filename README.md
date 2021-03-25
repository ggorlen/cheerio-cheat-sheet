# Cheerio Cheat Sheet
Cheerio Cheat Sheet with the most needed stuff..





# load html
```javascript
const $ = cheerio.load(html);
```


# get html after you change something
```javascript
const $ = cheerio.load('<h2 class="title">Hello world</h2>');

$('h2.title').text('Hello there!');
$('h2').addClass('welcome');

const newHTML = $.html();
```




$.html();


<br><br>
______________________________________
______________________________________
<br><br>
