# Algorithm - Date and String

## table of contents
1. [Show tomorrow date](#show-tomorrow-date)
2. [Define Date.nextDay using prototype](#)


## Show tomorrow date
```js
var today = new Date();
var tomorrow = new Date();
tomorrow.setDate(today.getDate()+1);

console.log(today);//Thu Dec 08 2016 08:35:42 GMT-0800 (PST)
console.log(tomorrow); //Fri Dec 09 2016 08:32:48 GMT-0800 (PST)
```

# Algorithm - String


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

## Get domain name and path name using Regular expression
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
[CSS-TRICKS GET URL and URL Parts in JavaScript](https://css-tricks.com/snippets/javascript/get-url-and-url-parts-in-javascript/)
[]()

