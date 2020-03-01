---
title: "Javascript WeakSet Use Case"
date: 2020-03-27T23:32:26+08:00
draft: true
---

WeakSet is 

# Detecting circular references

The first thing WeakSet can help you is checking whether the object has circular references, which may cause bugs.
For example, every time when you try to serialize a JSON object with circular references using `JSON.stringify`, the browser would throw an error, like `TypeError: Converting circular structure to JSON` in Chrome.
With WeakSet you can handle this issue easily. When iterating an JSON object, you simply add every object to a WeakSet, and if the object already exists in the WeakSet, th. The following is the example from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cyclic_object_value):

```javascript
const getCircularReplacer = () => {
  const seen = new WeakSet();
  return (key, value) => {
    if (typeof value === "object" && value !== null) {
      if (seen.has(value)) {
        return;
      }
      seen.add(value);
    }
    return value;
  };
};

JSON.stringify(circularReference, getCircularReplacer());
```

1. 