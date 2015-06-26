---
layout: post
title:  "Remote pagination with kaminari"
date:   2015-06-26 16:10:00
categories: rails pagination
---

[Kaminari](https://github.com/amatsuda/kaminari) is an awesome gem to paginate your list view

According to the document to make ajax links you just add "remote: true" like this:

{% highlight ruby %}
<%= paginate @users, remote: true %>
{% endhighlight %}

Crazy simple but it doesn't work in my case! Maybe the document has not been updated yet.

I have to add this file to make it works:

{% highlight ruby %}
# index.js.erb
$("#users").html("<%= escape_javascript render(@users) %>");
$("#paginator").html("<%= escape_javascript(paginate(@users, remote: true)) %>")
{% endhighlight %}

You can see the view details in this example:

[Kaminari example](https://github.com/amatsuda/kaminari_example/tree/ajax/app/views/users)


Hope this can help you!
