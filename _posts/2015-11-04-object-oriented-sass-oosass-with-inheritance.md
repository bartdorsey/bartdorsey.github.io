---
layout: article
status: publish
published: true
title: Object Oriented SASS (OOSASS) with Inheritance
author:
  display_name: Bart Dorsey
  login: bart
  email: bart@bartdorsey.com
  url: http://bartdorsey.com
author_login: bart
author_email: bart@bartdorsey.com
author_url: http://bartdorsey.com
wordpress_id: 356
wordpress_url: https://bartdorsey.com/?p=356
date: '2015-11-04 17:06:54 -0500'
date_gmt: '2015-11-04 17:06:54 -0500'
categories:
- Programming
tags: []
comments: []
---
Mixins in SASS are great, however, there is a downside to using them for your OOSASS. Mixins bloat the output code. Every time you call a mixin it includes the output of the mixin in your final CSS. &nbsp;The obvious solution is placeholders, but placeholders don't work well with parameters or with BEM-style css class names.

I've come up with a solution that solves both problems. I lets you use a mixin for your OOSASS class, it lets you inherit from this OOSASS class and also produces efficient output using placeholders.

Say we are styling the following markup:

{% highlight HTML lineos %}
  <article class="my-article">
     <h1 class="my-article__title">Title</h1>
     <p class="my-article__body">Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est labour.</p>
  </article>
{% endhighlight %}

We might start by making a mixin for this component;

{% highlight sass linenos %}
@mixin article {
   &amp;__title {
      font-size: 18px;
   }
   &amp;__body {
      font-size: 12px;
   }
}
{% endhighlight %}

And then use it:
{% highlight sass lineos %}
.my-article {
  @include article;
}
{% endhighlight %}

The problem here comes when we try to apply this mixin to another component.

{% highlight sass lineos %}
.another-article {
    @incluce article
}
{% endhighlight %}

This is going to output this, which duplicates our CSS:
{% highlight sass lineos %}
.my-article__title {
   font-size: 18px;
}

.my-article__body {
   font-size: 12px;
}

.another-article__title {
   font-size: 18px;
}

.another-article__body {
   font-size: 12px;
}
{% endhighlight %}

What we want to do here is to create placeholders for every element in our mixin "class", and use them with extend inside our mixin.

{% highlight sass lineos %}
%article__title {
  font-size: 18px;
}

%article__body {
  font-size: 12px;
}

@mixin article {
  &amp;__title {
    @extend %article__title;
  }
  &amp;__body {
    @extend %article__body;
  }
  @content;
}

{% endhighlight %}

Now our previous code which did the @include article on the two components generates this:

{% highlight sass lineos %}
.my-article__title, .another-article__title {
  font-size: 18px;
}

.my-article__body, .another-article__body {
  font-size: 12px;
}
{% endhighlight %}

Now I have a reusable mixin that produces optimized SASS! &nbsp;But what if I want to override some style of an instance of a component? This is fine. Since we included @content in our mixin 'class' you can do this:

{% highlight sass lineos %}
.another-article {
  @include article {
    &amp;__title {
      color: black;
    }
  }
}
{% endhighlight %}
So what we have here is in essence OOSASS. .another-article is an instance of article. And it add styling to the title to color it black.
But what if we want to make &nbsp;a subclass of article? Say we want it to be a featured article and have larger bolder headlines?
We can do this:
{% highlight sass lineos %}
%featured-article__title {
  font-size: 24px;
  font-weight: bold;
}

%featured-article__body {
  font-size: 16px;
}

@mixin featured-article {
  @include article {
    &amp;__title {
      @extend %featured-article__title;
    }
    &amp;__body {
      @extend %featured-article__body;
    }
  }
  @content;
}
{% endhighlight %}

featured-article inherits from article. It overrides font-size and adds font-weight.
Here's an example of using two articles and two featured articles
{% highlight sass lineos %}
.an-article {
  @include article;
}

.another-article {
  @include article;
}

.a-featured-article {
  @include featured-article
}

.another-featured-article {
  @include featured-article
}
{% endhighlight %}
And the CSS output it produces:

{% highlight sass lineos %}
.an-article__title, .another-article__title, .a-featured-article__title, .another-featured-article__title {
  font-size: 18px;
}

.an-article__body, .another-article__body, .a-featured-article__body, .another-featured-article__body {
  font-size: 12px;
}

.a-featured-article__title, .another-featured-article__title {
  font-size: 24px;
  font-weight: bold;
}

.a-featured-article__body, .another-featured-article__body {
  font-size: 16px;
}
{% endhighlight %}

Because of the single level of specificity of BEM, the overrides work perfectly the and later rules for font-size win for featured articles. &nbsp;It would be nice to properly override the font-size and get it to exclude the first font-size declaration automatically, but sass doesn't really provide a mechanism to do this sort of thing.
Here's a <a href="http://sassmeister.com/gist/2350189b4130d7410fae">live example</a> of the above on sassmeister if you want to play with it.
