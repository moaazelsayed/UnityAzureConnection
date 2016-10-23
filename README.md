## Connecting Unity3D to Cognitive Services (Using C#)
**This tutorial assumes that you have already created and setup your Cognitive Services as well as Easy Tables in your Azure Portal** If you have not, please start by doing so.

To be able to make REST requests to Azure/Cognitive Services, we will be using a part of Unity's built in **WWW** library.
More info can be found at https://docs.unity3d.com/ScriptReference/WWW.html.

To start off, we need to recognize the request format that cognitive services accepts. In the case for the Computer Vision API, we need to include `Ocp-Apim-Subscription-Key`, `Content-Type` as well as the correct url that we're hitting. In this case we use `https://api.projectoxford.ai/vision/v1.0/analyze?visualFeatures=Categories` with parameters already added to the url. 
To get your `Ocp-Apim-Subscription-Key`, go to your Azure Portal and select your cognitive services Api that you have already created and navigate to `keys` under `Resource Management`. You will find two keys; you will be able to use either.

```
	//Make a new form
	WWWForm form = new WWWForm();

	//Create and attach headers
	var headers = form.headers;
	headers["Ocp-Apim-Subscription-Key"] = "999999999xxxxxxxxxyourkeygoeshere";
	headers ["Content-Type"] = "application/octet-stream";
```

Your `Content-Type` will be different depending on the service. In this case, we analyzed an image that was captured, so we used `application/octet-stream`. 

**Note:** Do NOT use `form.AddBinaryData()` and `form.AddField()` to add the body of the request, we have found them to overwrite the content type in some of the cases. To overcome this, we will be feeding binary data straight into the `www` request function.

```
	//Declare byte array of image to be sent
	byte[] imageData = null;

	//You can either add straight into function or pull from storage
	FileInfo fileInfo = new FileInfo("pic.jpg");
	long imageFileLength = fileInfo.Length;
	FileStream fs = new FileStream("pic.jpg", FileMode.Open, FileAccess.Read);
	BinaryReader br = new BinaryReader(fs);

	//Map image into byte array
	imageData = br.ReadBytes((int)imageFileLength);
```
You can now declare your url as a string and make the request.

```
	string url = "https://api.projectoxford.ai/vision/v1.0/analyze?visualFeatures=Categories";

	WWW www = new WWW(url, imageData, headers);
```
Use IEnum function to get response. `StartCoroutine(WaitForRequest(www));`.
```
IEnumerator WaitForRequest(WWW www)
	{
		yield return www;
		// check for errors
		if (www.error == null)
		{
			//Do something with response
			Debug.Log("WWW Ok!: " + www.text);
		} else {
			Debug.Log("WWW Error: "+ www.error);
		}    
	}  
```

More info can be found at https://dev.projectoxford.ai/docs/services/56f91f2d778daf23d8ec6739/operations/56f91f2e778daf14a499e1fa.

---

## Connecting Unity3D to Azure Storage (Easy Tables)
Headers
```
	headers["ZUMO-API-VERSION"] = "2.0.0" ;
	headers["Content-Type"]="application/json";
```
Body json format
`string newObj = "{\"newTitle\":\"My Title\"}";`

If title is not present in table, it will be added.

define url
`string url = "http://account_name.azurewebsites.net/tables/table_name";`

since `www` request only takes in byte array as a body, the json will have to be cast into a byte array through `Encoding.ASCII.GetBytes(newObj)` this uses `System.Text`.
looking at it in the request, `WWW www = new WWW(url, Encoding.ASCII.GetBytes(newObj), headers);`.
``
Use IEnum function to get response. `StartCoroutine(WaitForRequest(www));`.
```
IEnumerator WaitForRequest(WWW www)
	{
		yield return www;
		// check for errors
		if (www.error == null)
		{
			//Do something with response
			Debug.Log("WWW Ok!: " + www.text);
		} else {
			Debug.Log("WWW Error: "+ www.error);
		}    
	}  
```

To find out more about how to insert into a table, check out:
https://msdn.microsoft.com/en-us/library/azure/dd179433.aspx

To find out more about how to query, check out:
https://msdn.microsoft.com/en-us/library/azure/dd179421.aspx

---

##If you would like to add onto this tutorial, do a pull request!