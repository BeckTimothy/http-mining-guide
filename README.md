If you open `https://bio2024.mapyourshow.com/8_0/explore/exhibitor-gallery.cfm?featured=false` and open the network tab then refresh. You'll see a request hit the `remote-proxy.cfm` endpoint.

This request hits url "https://bio2024.mapyourshow.com/8_0/ajax/remote-proxy.cfm"
with payload:
- action: search
- searchtype: exhibitorgallery
- searchsize: 50
  or url encoded: `action=search&searchtype=exhibitorgallery&searchsize=50`

![Pasted image 20240606231317.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606231317.png)


We can see in the response section that the first response returns '1nHealth' which also happens to be the first result populated in page.

![Pasted image 20240606222856.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606222856.png)


by right clicking the request in network -> selecting copy -> then selecting copy as curl (cmd)

![Pasted image 20240606231401.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606231401.png)


we can then import the curl into insomnia (from curl)

![Pasted image 20240606223013.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606223013.png)


This will let us mess around with the  request, params, and headers necessary to get the desired result.

In insomnia, or postman or whatever your favorite request-emulator/api-debugger is, you'll see that if we 'mute' all of the Headers and resend the curl request we'll get this forbidden access notice:

![Pasted image 20240606224115.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606224115.png)


The goal here is to determine which headers are necessary for this api to return our desired response before we create an automation that abuses the api. If we re-enable all of the Headers and then resend a request disabling one header per request, we'll eventually find out that the api requires us to send a single header. `'X-Requested-With':'XMLHttpRequest'`

from here we can write a small python script to execute this curl:
```python
import requests  
headers = {'x-requested-with':'XMLHttpRequest'}  
r = requests.get('https://bio2024.mapyourshow.com/8_0/ajax/remote-proxy.cfm?action=search&searchtype=exhibitorgallery&searchsize=50', headers=headers)  
print(r.json())
```

When ran we will notice that we receive data. But it's not all the data from the api and this api isn't paginated. (Edit: it actually is but navigating pagination isn't the easiest solution)

By printing `print(len(bytes(json.dumps(r.json()), "utf-8")))` to get the length of the return we can see if we modify the search size query param to 99999, the length of the return is far less; likely due to an error message recieved. but if we change the searchsize query param to 9999 we get a far longer response size.

![Pasted image 20240606225531.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606225531.png) 

We also know from looking at the web requests in the browser after hitting the load more button and the subsequent load all button that there are only 1557 data points, so we know by using search size 9999 we are receiving all the data from the api.

![Pasted image 20240606225644.png](https://raw.githubusercontent.com/BeckTimothy/http-mining-guide/img/20240606225644.png)

Leaving us with the final python code of:
```python
import requests  
headers = {'x-requested-with':'XMLHttpRequest'}  
r = requests.get('https://bio2024.mapyourshow.com/8_0/ajax/remote-proxy.cfm?action=search&searchtype=exhibitorgallery&searchsize=9999', headers=headers)  
print(r.json())
```
and no need to utilize a web scraping library.