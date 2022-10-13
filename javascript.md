```javascript
function var_dump(obj)
{
    let keys = Object.keys(obj);
    for (let i = 0; i < keys.length; i++) {
        let val = obj[keys[i]];
        console.log(val);
    }
}

function var_dump2(obj)
{
    for (const [key, val] of Object.entries(obj)) {
        console.log([key, val]);
    }
}
```
[1]: https://www.w3docs.com/snippets/javascript/how-to-get-all-property-values-of-a-javascript-object.html
