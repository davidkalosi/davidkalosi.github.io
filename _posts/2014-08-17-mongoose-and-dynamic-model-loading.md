---
layout: post
published: true
title: Mongoose and dynamic model loading
mathjax: false
featured: false
comments: true
modified: "2014-08-17"
categories: 
  - Node.JS
tags: "node,javascript,mongoose"
---

Here is another simple approach for maintaining a model per file structure with mongoose.

The basic idea is to have one model per file in a separate directory (called models for the purpose of this article) and then load them all automatically. This will simplify the process of defining new models - just create a new file and all done.

To achieve this first we need to establish a structure when defining a model:

models/User.js
{% highlight javascript %}
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var userSchema = new Schema({
    login: String,
    password: String,
    name: String,
});

var User = mongoose.model('User', userSchema);

exports.User = User;
{% endhighlight %}

And the models/index.js that takes care of the loading looks like this:
{% highlight javascript %}
var mongoose = require('mongoose');
var fs = require('fs');
var path = require('path');
var _ = require('lodash');

mongoose.connect('mongodb://localhost/app');

var model = {}

fs.readdirSync(__dirname)
    .filter(function(file) {
        return (file.indexOf('.') !== 0) && (file !== 'index.js')
    })
    .forEach(function(file) {
        model = _.extend(model, require(path.join(__dirname, file)));                      
    });

module.exports = model;
{% endhighlight %}

Then in the application you simply do 
{% highlight javascript %}
var model = require('./models');

model.User.findOne({login: 'davidkalosi'}, function (err, user) {
});
{% endhighlight %}

That's it ;)