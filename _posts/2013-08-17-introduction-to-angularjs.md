---
layout: post
title: Introduction to AngularJS
category: howto,angularjs
tags: angularjs, speackerdeck, presentation, introduction, howto
---

![AngularJS logo](introduction-to-angularjs/AngularJS.jpg)

I discovered recently this exciting framework, and even if I'm still a beginner in it, I presented it to my colleagues to share my enthusiasm.
Here are the slides i used for the presentation.

<script async class="speakerdeck-embed" data-id="7a7a33c0bb5301309b3a023921bdbc68" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

I'll try to sum up here some the first slides and present an example of application using AngularJS.

Why another framework?
----------------------

AngularJS is a **client side** (like for instance jquery) open-source javascript framework that help creating **web application** structured around the MVC pattern. It was historically developed by [Miško Hevery][1] and Adam Abrons, until Google promoted it (after Adam left).

The main difference between AngularJS and the other client side web application framework (jquery, ember, backbone) is how it embraces the HTML DOM: it extends it to keep the capability to describe web interfaces in a descriptive way, and it bind it with Angular javascript controllers, to update dynamically views according to model changes.

Writing apps with AngularJS is extremely elegant.

How difficult is it to deploy?
------------------------------

Creating an AngularJS app is very concise, you just have to source the javascript library:

    <script src="lib/angular/angular.js" />

And define the app in your html page

    <body ng-app>

That's all!

Identi.ca client application example
------------------------------------

[Identi.ca][2] is a social micro-blogging platform, like Twitter, which relies on open-source softwares. Identi.ca also supports the Twitter API 1.0, which doesn't exist anymore on Twitter since June 11th 2013. It allows us to use the old search API, which in our case is great because it doesn't require authentication and we could as a consequence easily interact with it.

###The API

The API we will use is very simple: it's a a RESTFUL API, and you can find its definition [here][3].

We will only use these elements:

 * url: `http://identi.ca/api`
 * action: ` search.json`
 * parameter: `q`

###The JS Controller

AngularJS provide a very cool service to manage REST API: **$resource**. To be able to use this service, we have to inject the *ngResource* module.

    angular.module('identicaSearch', ['ngResource']);

We then create the Controller, and inject him the $scope and the $resource services: you can switch the two arguments, the matching with the rights object are based on the name (And you have to take that into account if you want to do minification).

    function IdenticaCtrl($scope,$resource){
    }

In it we are going to affect a variable with the resource, set with defaults values.

    $scope.identicaSearch = $resource(
                                'http://identi.ca/api/:action',
                                {action:'search.json', q: 'angularjs', callback:'JSON_CALLBACK'},
                                {get:{method:'JSONP'}}
                            );



