---
layout: post
title: "Uploading Files With ASP.NET Core and Angular"
subtitle: "This article discuss about uploading files from an Angular application to ASP.NET Core backend."
date: 2021-03-04 00:00:00
categories: [Angular,AspNetCore]
tags: [Angular,AspNetCore]
author: "Anuraj"
image: /assets/images/2021/03/angular_file_upload.png
---
This article discuss about uploading files from an Angular application to ASP.NET Core backend. First you need to create an ASP.NET Core Angular project. Once it is done, you need to create an angular component. And then you can modify the backend implementation.

### Creating an Angular component

To create an angular component, you need to run the following command - `ng generate component fileupload --skip-import --skip-tests -s`. Next you need to modify the typescript file - `fileupload.component.ts` like this.

{% highlight TypeScript %}
upload(files) {
  if (files.length === 0)
    return;

  const formData = new FormData();

  for (const file of files) {
    formData.append(file.name, file);
  }

  const uploadReq = new HttpRequest('POST', this.baseUrl + 'FileManagement/upload', formData, {
    reportProgress: true,
  });

  this.http.request(uploadReq).subscribe(event => {
    if (event.type === HttpEventType.UploadProgress) {
      this.progress = Math.round(100 * event.loaded / event.total);
    };
  });
}
{% endhighlight %}

In the above code, when a user browse files, the on change event will be fired. And then upload the file with `httpclient` using FormData object.
And in the `fileupload.component.html` you need to modify and include HTML like this.

{% highlight HTML %}
<div class="form-group">
  <label for="picture">Picture</label>
  <div class="custom-file">
    <input #file type="file" id="customFile" accept=".jpg,.png,.gif" multiple (change)="upload(file.files)" />
    <label class="custom-file-label" for="customFile">Choose file</label>
  </div>
</div>
<div class="progress">
  <div class="progress-bar" role="progressbar" [style.width.%]="progress"></div>
</div>
{% endhighlight %}

In the next section you will learn how to read and save in ASP.NET Core.

### Implementing the backend

In ASP.NET Core you can create a controller, and create an action method. In the action method, you can use `Request.Form.Files` to read the uploaded files with Form Data from Angular client. 

{% highlight CSharp %}
[HttpPost]
[Route("upload")]
public async Task<IActionResult> Upload()
{
    var files = Request.Form.Files;
    foreach (var file in files)
    {
        var blobContainerClient = new BlobContainerClient("UseDevelopmentStorage=true","images");
        blobContainerClient.CreateIfNotExists();
        var containerClient = blobContainerClient.GetBlobClient(file.FileName);
        var blobHttpHeader = new BlobHttpHeaders
        {
            ContentType = file.ContentType
        };
        await containerClient.UploadAsync(file.OpenReadStream(), blobHttpHeader);
    }

    return Ok();
}
{% endhighlight %}

The above code is using the `Request.Form.Files` options. You can also use the `Request.ReadFormAsync()` method which helps you to read the files asynchronously, here is the code for reading the file asynchronously with the help of `Request.ReadFormAsync()` method.

{% highlight CSharp %}
[HttpPost]
[Route("upload")]
public async Task<IActionResult> Upload()
{
    var formCollection = await Request.ReadFormAsync();
    var files = formCollection.Files;
    foreach (var file in files)
    {
        var blobContainerClient = new BlobContainerClient("UseDevelopmentStorage=true", "images");
        blobContainerClient.CreateIfNotExists();
        var containerClient = blobContainerClient.GetBlobClient(file.FileName);
        var blobHttpHeader = new BlobHttpHeaders
        {
            ContentType = file.ContentType
        };
        await containerClient.UploadAsync(file.OpenReadStream(), blobHttpHeader);
    }

    return Ok();
}
{% endhighlight %}

ASP.NET Core also supports File Upload with the help of form binding. Here is the implementation in the ASP.NET Core controller.

{% highlight CSharp %}
[HttpPost]
[Route("createprofile")]
public async Task<IActionResult> CreateProfile([FromForm] Profile profile)
{
    if (ModelState.IsValid)
    {
        var tempProfile = profile;
        return Ok();
    }

    return BadRequest();
}
{% endhighlight %}

And here is the `Profile` class.

{% highlight CSharp %}
public class Profile
{
    [Required]
    public string Name { get; set; }
    [Required]
    public string Email { get; set; }
    [Required]
    public IFormFile Picture { get; set; }
}
{% endhighlight %}

And from Angular client you can use the Angular forms. Here is the Angular code which upload and submit the data to server.

{% highlight Typescript %}
const formData = new FormData();
for (const key of Object.keys(this.profileForm.value)) {
  const value = this.profileForm.value[key];
  formData.append(key, value);
}
this.http.post(this.baseUrl + 'FileManagement/createprofile', formData, {
  reportProgress: true,
  observe: 'events'
}).subscribe(event => {
  if (event.type === HttpEventType.UploadProgress) {
    this.progress = Math.round((100 * event.loaded) / event.total);
  }
  if (event.type === HttpEventType.Response) {
    console.log(event.body);
    this.profileForm.reset();
  }
});
{% endhighlight %}

And there is one specific code block which helps to populate the File value to Angular form control. Here is the code, which triggered on the change event of File Upload control.

{% highlight Typescript %}
onFileChanged(event) {
  if (event.target.files.length > 0) {
    const file = event.target.files[0];
    this.labelImport.nativeElement.innerText = file.name;
    this.profileForm.patchValue({
      picture: file,
    });
  }
}
{% endhighlight %}

Here is the screenshot of the application running.

![Angular File Upload]({{ site.url }}/assets/images/2021/03/angular_file_upload.png)

This way you can implement File Upload from Angular to ASP.NET Core and how to store them in Azure Blob storage. You can use the model binding implementation if you are planning to use upload a file along with other form fields like name or email.

Happy Programming :)