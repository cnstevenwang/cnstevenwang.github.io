---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---
> A coder, build the world in code.
{: .prompt-tip }

> A loster, lost in the real world.
{: .prompt-tip }

```js
const coder = {
    name: "Steven",
    location: {
        country: "China",
        city: "Shanghai"
    },
    skills: {
        java: {
            since: 2011
        },
        springboot: {
            since: "2.0"
        },
        "psotgresql": "postgresql",
        "golang": {},
        "redis": {},
        "kafka": {},
        "python": {},
        "langchain": {}
    }
}

console.log(JSON.stringify(coder))
```