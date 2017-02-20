# errisy-server
an out-of-box nodejs server

## why not express?

```javascript
express.get('/', function(req, res){
  res.send('hello world!');
})
```
How does front end consume it? Typically, xhr (XMLHttpRequest).

Both server and client sides are not well structure and managable.

## My solution is remote procedure call.
See [errisy-tsc](https://github.com/errisy/errisy-tsc) for remote procedure call details.

### What's in the box?
1. http and https server that support a standard production setup of http/https servers.
2. RPCMiddleware for handling remote procedure calls with the *.rpc.ts files. They can be loaded dynamically with vm in dev mode, but can be loaded with require as module in production mode.
3. FrontEndRouterMiddleware to direct links that should be handled by front end router to the specified html file.
4. FileMiddleware for handling file request.
5. SessionMiddleware for setup sessions. (This will be updated with my own code as the module node-session can crash during dev mode hot reload. But it does not affect production.).


## How does remote procedure call work?
** this following can be found at [errisy-tsc](https://github.com/errisy/errisy-tsc) **

Simple write your service as classes in *.rpc.ts files and extends from rpc.RPCService.

use **@rpc.service** to decorate the service class that you want to expose to the client.
use **@rpc.member** to decorate the member that you want to expose to the client.

**app.rpc.ts:**
```typescript
/// <transpile path="C:\HTTP\npm\errisy-tsc\csclient.cs"/>
import * as rpc from 'errisy-rpc';
import { ImageItem } from './imageitem';
/**
 * the RPC service example.
 */
@rpc.service
export class AppService extends rpc.RPCService {
    /**
     * this is genarate a list of image slides for the front end to display
     */
    @rpc.member
    public async ImageList(): Promise<ImageItem[]> {
        return [
            new ImageItem('img/1.jpg', 300, 1000),
            new ImageItem('img/2.jpg', 500, 2000),
            new ImageItem('img/3.jpg', 200, 400),
            new ImageItem('img/4.jpg', 100, 800),
            new ImageItem('img/5.jpg', 300, 1400),
        ]
    }
}
```
So here is the transpiled client for Angular 2.
You can simply inject it into any Angular 2 component and use it to call it by await (because all rpc calls are wrapped in the async functions)
You can also invoke it by calling the [ImageList url] field, which is the link for it: "/app/app.rpc.js?AppService-ImageList&"

**Check out the magic of comments** The comments are also copied from the service to the client. That means the front end developers will see the output.

**app.rpc.ts:**
```typescript
//Client file generated by RPC Compiler.
import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import { Observer } from 'rxjs/Observer';
import 'rxjs/add/operator/toPromise';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';
import * as rpc from 'errisy-rpc';

import { ImageItem } from './imageitem';
/**
 * the RPC service example.
 */
@Injectable()
export class AppService {
	constructor(private $_Angular2HttpClient: Http){
	}
	/**Please set Base URL if this Remote Procedure Call is not sent to the default domain address.*/
	public $baseURL: string = "";
		/**
		 * this is genarate a list of image slides for the front end to display
		 */
		public ImageList(): Promise<ImageItem[]>{
			return this.$_Angular2HttpClient.post(this.$baseURL + '/app/app.rpc.js?AppService-ImageList', rpc.buildClientData()).map(rpc.Converter.convertJsonResponse).toPromise();
		}
		public get "ImageList url"():string{
			return this.$baseURL + "/app/app.rpc.js?AppService-ImageList&";
		}
}

```

## Transpiled C#:

C# can call the service as well with automatically generated front end client file! (For example, Xamarin project, Windows Desktop/WPF/Winform projects).
You must include the C# PolyFill for your C# front end.
Make sure you set the transpile path properly: 
**/// <transpile path="path in your computer"/>**
```CSharp
//Data Type Definition Generated by RPC compiler. Please do not modify this file.
namespace app
{
	/// <summary>
	/// the RPC service example.
	/// </summary>
	public class AppService
	{
		public AppService(string baseUrl){
			BaseUrl = baseUrl;
		}
		public string BaseUrl { get; set; }
		/// <summary>
		/// this is genarate a list of image slides for the front end to display
		/// </summary>
		public async System.Threading.Tasks.Task<ImageItem[]> ImageList()
		{
			return NodeJSRPC.Converter.convertJsonResponse<ImageItem[]>(await NodeJSRPC.HttpClient.Post(this.BaseUrl + "/app/app.rpc.js?AppService-ImageList", ));
		}
	}
}
```

#Can this handle file upload?
Yes! 
Use rpc.Polyfill.File to handle client side JavaScript File objects and rpc.Polyfill.Blob to handle client side JavaScript Blob objects.
This works for C# as well. Check it out yourself in the client files.

```typescript
/// <transpile path="C:\HTTP\npm\errisy-tsc\csclient.cs"/>

import * as rpc from 'errisy-rpc';
import { ImageItem } from './imageitem';

/**
 * the RPC service example.
 */
@rpc.service
export class AppService extends rpc.RPCService {

    /**
     * this is genarate a list of image slides for the front end to display
     */
    @rpc.member
    public async ImageList(): Promise<ImageItem[]> {
        return [
            new ImageItem('img/1.jpg', 300, 1000),
            new ImageItem('img/2.jpg', 500, 2000),
            new ImageItem('img/3.jpg', 200, 400),
            new ImageItem('img/4.jpg', 100, 800),
            new ImageItem('img/5.jpg', 300, 1400),
        ]
    }
    /**
     * File upload
     * @param file
     */
    @rpc.member
    public async upload(file: rpc.Polyfill.File): Promise<string> {

    }
    /**
     * transfer of bytes
     * @param file
     */
    @rpc.member
    public async transfer(file: rpc.Polyfill.Blob): Promise<boolean> {

    }
}
```