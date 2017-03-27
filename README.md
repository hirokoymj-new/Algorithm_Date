# Algorithm - Date

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


### Get url string.
```
var url = window.location.href;
console.log(url); //http://www.cherokeenursingscholarship.com/content/about.php"
```

### Get a page name.
```
var url = window.location.href; //http://www.cherokeenursingscholarship.com/content/about.php
var pageName = url.substring(url.lastIndexOf('/')+1);
console.log(pageName); //about.php

```
### Get a page name without domain.
```
var url = "http://www.cherokeenursingscholarship.com/content/about.php";

var tmp = url.substring(7);
console.log(tmp);

var tmp1 = tmp.substring(tmp.indexOf('/')+1);
console.log(tmp1);
```
https://css-tricks.com/snippets/javascript/get-url-and-url-parts-in-javascript/
