# Algorithm - Date and String

## table of contents
1. [Show tomorrow date](#show-tomorrow-date)
2. [Define Date.nextDay using prototype](#)
3. [Get URL](#get-url)
4. [Get a domain name](#get-domain-name)
5. [Get a path name](#get-a-path-name)
6. [Get a file name](#get-a-file-name-from-url)
7. [Get domain and pathname using regular expression](#get-domain-name-and-path-name-using-regular-expression) 


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


## Get domain name and path name using Regular expression
1. Creating a regular expression using enclosed between slashes or RegExp object.
2. ? means non-greedy.
3. () is grouping and can access using $n.

```
var url = 'https://github.com/hirokoymj/Algorithm_Date/edit/master/README.md';
var reg = new RegExp('(http|https)://(.+?)\/(.+)');
var protocol = url.replace(reg, '$1');
var domain = url.replace(reg, '$2');
var pageName = url.replace(reg, '$3');
console.log(protocol); //https
console.log(domain);	//github.com
console.log(pageName);	//hirokoymj/Algorithm_Date/edit/master/README.md
```

### References:
- [CSS-TRICKS GET URL and URL Parts in JavaScript](https://css-tricks.com/snippets/javascript/get-url-and-url-parts-in-javascript/)
- [MDN - Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
- [stackoverflow - regular expression](http://stackoverflow.com/questions/3809401/what-is-a-good-regular-expression-to-match-a-url)
