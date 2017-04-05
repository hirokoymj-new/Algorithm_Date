# Algorithm - Date and String

## table of contents
1. [Show tomorrow date](#show-tomorrow-date)
2. [Define Date.nextDay using prototype](#)
3. [Get URL](#get-url)
4. [Get a domain name](#get-domain-name)
5. [Get a path name](#get-a-path-name)
6. [Get a file name](#get-a-file-name-from-url)
7. [Get domain and pathname using regular expression](#get-domain-name-and-path-name-using-regular-expression) 
8. [Differences between each() and forEach().](#differences-between-foreach-and-each)


## Show tomorrow date
```js
var today = new Date();
var tomorrow = new Date();
tomorrow.setDate(today.getDate()+1);

console.log(today);//Thu Dec 08 2016 08:35:42 GMT-0800 (PST)
console.log(tomorrow); //Fri Dec 09 2016 08:32:48 GMT-0800 (PST)
```

## Get URL.
```
window.location.href;
"https://github.com/hirokoymj/Algorithm_Date/edit/master/README.md"
```

## Get domain name.
```
window.location.host;
"github.com"
```

## Get a path name.
```
var pathname = window.location.pathname;
"/hirokoymj/Algorithm_Date/edit/master/README.md"
```

## Get a file name from url.
```
var url = "https://github.com/hirokoymj/Algorithm_Date/edit/master/README.md";
var filename = url.substring(url.lastIndexOf('/')+1);
console.log(filename); //README.md
```


## Get domain name, path name and file name using Regular expression
1. Creating a regular expression object. `var reg = new RegExp()`
2. ? means non-greedy.
3. () is grouping and can access using $n.
4. Use string.replace() method.
5. Use string.substring() method.
6. Use string.lastIndexOf() method.


```
var url = "https://github.com/hirokoymj/Algorithm_Date/edit/master/README.md";
var reg = new RegExp('https://(.+?)/(.+)');
var host = url.replace(reg, '$1');
var pathname = url.replace(reg, '$2');
var filename = url.substring(url.lastIndexOf('/')+1);

console.log(host); 		//github.com
console.log(pathname); //hirokoymj/Algorithm_Date/edit/master/README.md
console.log(filename); //README.md
```
## Differences between forEach() and each().
1. **forEach()** is JavaScript function to iterate for Array. - Array.forEach().
2. **each()** is jQuery function to iterate HTML.

forEach
```
var employees = [
  { skill: 'css', user: 'Bill' },
  { skill: 'javascript', user: 'Chad' },
  { skill: 'javascript', user: 'Bill' },
  { skill: 'css', user: 'Sue' },
  { skill: 'javascript', user: 'Sue' },
  { skill: 'html', user: 'Sue' }
];
employees.forEach(function(value, index){
	console.log(value + ',' + index);
})
```

$.each()
```
$('li a').each(function(value){
  console.log($(this).attr('href');
});
```



### References:
- [CSS-TRICKS GET URL and URL Parts in JavaScript](https://css-tricks.com/snippets/javascript/get-url-and-url-parts-in-javascript/)
- [MDN - Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
- [MDN - replace method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)
- [MDN - substring method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/substring)
- [MDN - lastIndexOf method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/lastIndexOf)
- [MDN - forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
- [jQuery - $.each](https://api.jquery.com/each/)
- [stackoverflow - regular expression](http://stackoverflow.com/questions/3809401/what-is-a-good-regular-expression-to-match-a-url)
