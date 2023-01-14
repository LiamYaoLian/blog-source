---
title: Debounce and Throttle
date: 2023-01-14 15:29:50
tags:
---
* debounce: no input after an interval, send a request
```JavaScript
function debounce(func, delay) {
  let timerId = null;
  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  }
}

const debounceFunc = debounce(func, 500);
const input = document.getElementById('some-id');
input.addEventListener('input', function(e) {
  debounceFunc(e.target.value);
})
```
* throttle: one request per interval
```JavaScript
function throttle(func, delay) {
  let timerId = null;
  let last = 0;
  return function(...args) {
    const now = Date.now();
    const remainingTime = last + delay - now;
    if (remainingTime <= 0) {
      func.apply(this, args);
      last = now;
    } else {
      clearTimeout(timerId);
      timerId = setTimeout(() => {
        func.apply(this, args);
        last = now;
      }, remainingTime);
    }
  }
}
```
