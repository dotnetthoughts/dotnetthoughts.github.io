---
layout: post
title: "Load Data while Scrolling Page Down with jQuery and ASP.Net MVC"
subtitle: "Load Data while Scrolling Page Down with jQuery and ASP.Net MVC"
date: 2014-01-08 09:16
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, Javascript]
tags: [.Net, ASP.Net, ASP.Net MVC, C#.Net, Javascript, JQuery]
header-img: "img/post-bg-01.jpg"
---
This post is about Facebook style data loading while scrolling down the page. To identify the page scroll down, you can use the following snippet.

{% highlight Javascript %}
$(window).scroll(function () {
    if ($(window).scrollTop() == $(document).height() - $(window).height()) {
        //This is an Ajax method which will fetch the data from server
        FetchDataFromServer();
    }
});
{% endhighlight %}

This condition will satisfy when user scroll down and reached the bottom of the page. This will invoke the Ajax get method, which will return JSON data from server. For rendering the JSON, I am using jsrender template-ing engine. Here is the complete implementation. I am not using any database for this sample.

Here is the server side (MVC controller)

{% highlight CSharp %}
public ActionResult FetchData(int pageIndex = 0)
{
    var posts = new List<Post>();
    int first = pageIndex + 1;
    int last = pageIndex + 10;
    for (int i = first; i <= last; i++)
    {
        posts.Add(new Post()
        {
            Id = i,
            Date = DateTime.UtcNow,
            Content = "Content from Database with Id" + i,
            Title = "Title " + i
        });
    }

    return Json(posts, JsonRequestBehavior.AllowGet);
}
{% endhighlight %}

Client side (JQuery and JSRender)

{% highlight Javascript %}
$(function () {
    FetchDataFromServer();
    $(window).scroll(function () {
        if ($(window).scrollTop() ==
            $(document).height() - $(window).height()) {
            if ($('#loadMessage').css('display') == 'none') {
                FetchDataFromServer();
            }
        }
    });
});

function FetchDataFromServer() {
    $("#loadMessage").toggle();
    var id = $(".post:last").attr("id");
    $.ajax("/Home/FetchData", {
        type: "GET",
        contentType: "application/json",
        data: { pageIndex: id },
        dataType: "json",
        success: function (data) {
            var postsTemplate = $.templates("#posts");
            var html = postsTemplate.render(data);
            $(".post:last").after(html);
            $("#loadMessage").toggle();
        },
        error: function () {
            $("#loadMessage").toggle();
        }
    });
}
{% endhighlight %}

And here is the HTML part.

{% highlight HTML %}
<div id="result">
    <div id="0" class="post"></div>
    <div id="loadMessage" style="display:none;">
        ![](~/Content/loader.gif)
    </div>
</div>
{% endhighlight %}

And here is the JSRender template

{% highlight HTML %}
{% raw %}
<script type="text/x-jsrender" id="posts">
    <div id="{{:Id}}" class="post">
        ## {{:Title}}
        <hr />
        <div>
            {{:Content}}
        </div>
    </div>
</script>
{% endraw %}
{% endhighlight %}

For identifying the Page, I am using the Div Id. If you don't want that approach you can create a hidden variable and can do the same.

Happy Coding :)
