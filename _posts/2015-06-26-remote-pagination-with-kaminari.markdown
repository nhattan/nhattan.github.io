---
layout: post
title:  "Remote pagination with kaminari"
date:   2015-06-26 16:10:00
categories: rails pagination
---

[Kaminari (雷)](https://github.com/amatsuda/kaminari) is a Japanese word meaning "thunder", is also a popular gem used to paginate in Rails.

According to the document to make ajax links you just have to add a parameter like this

{% highlight ruby %}
<%= paginate @users, remote: true %>
{% endhighlight %}

Yeah, crazy simple but it is not enough to make it works out of the box!

So after adding this file, it worked as expected

{% highlight ruby %}
# index.js.erb
$("#users").html("<%= escape_javascript render(@users) %>")
$("#paginator).html("<%= escape_javascript(paginate(@users, remote: true)) %>")
{% endhighlight %}

With ```users``` is the id of users list.

Remember to have ```jquery-rails``` gem in your application and your ```application.js``` having this like default settings:
{% highlight javascript %}
// application.js
//= require jquery
//= require jquery_ujs
{% endhighlight %}

You can see the view details in this example:
[Kaminari ajax example](https://github.com/amatsuda/kaminari_example/tree/ajax/app/views/users)


Hope this helps!
