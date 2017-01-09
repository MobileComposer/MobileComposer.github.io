---
published: true
layout: post
title: Overview of Angular JS and Datacontext w/ Breezejs
date: 2016-12-23 05:23
author: kam
comments: true
tags: [JavaScript, Angular JS, Web Development, Breeze.js, Mobile Composer]
---

Before I started at [Mobile Composer](https://www.mcomposer.com/) my coding experience came from backend coding languages. I used Java mostly, and previously used JavaScript for a summer in other internships. I never had experience building a front end system that would then connect to the logic we had on the
backend. In this post I will go over some of the logic we use to connect our web application to the server that holds our data. 
This will be more of a summary and overview to show the logic flow and processes that the team of interns learned when creating content for the web application.
As interns, we used Angular JS which is an MV* framework. We use it as an MVC framework, the MVC stands for Model, View, Controller.  

# ViewModel and Controller
For this example I will use one of the Admin Screens I created that allows Administrators to manage content through our web application. 

<img src="{{AppBrandList}}/MobileComposer.github.io/images/2017-1-2/AppBrandList.png" style="width: 500px;"/>






We use [AngularJS 1.5](https://angularjs.org/) to build out our admin screens, this is an example of a view which is a AppBrandList page that we connect to our controller. 

```javascript
    <section class="mainbar" data-ng-controller="appBrandController as vm">
        <section class="matter">
            <div class="container">
                <div class='row'>
                    <div class='col-sm-12'>
                        <div class='box bordered-box blue-border' style='margin-bottom:0;'>
                            <div class='box-header blue-background'>
                                <div class='title'>App Brands</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
     </section>
    
```
A ViewModel is a pattern that encapsulates the presentation state and logic. Our view interacts with this ViewModel, as the ViewModel acts as a intermediary between the view and model.
This is an example of how we use our view and the ng-controller directive to link this page to our appBrandController file. This is not the full AppBrandList view but only a portion. The first "section class" line is the one that connects to our controller.
In our controller we declare what modules the controller is dependent on, the one we will look at is the 'datacontext' module. This module allows our viewmodel to access data. The controller controls the application flow of our logic.


```javascript
var controllerId = 'appBrandController';

    angular.module('app')
         .controller(controllerId, ['$http', '$rootScope', '$scope', 'common', 'datacontext', '$location', '$route', '$routeParams', 'propsBrand', appBrandController]);


```



# Datacontext and Breeze.js
In our controller we explictly list datacontext as a module. The datacontext allows the ViewModel to access data easily. For example, if we wanted to access a list of AppBrands that we have in a organization we would just call datacontext.getAppBrands. 
Our ViewModel does not know how this call is being made it just knows that if it calls this function a list of AppBrands will be returned. 
In our datacontext we use [Breeze.js](http://www.getbreezenow.com/breezejs) which is a client side JavaScript library that manages data. Breeze.JS allows us to easily query data within our datacontext. An example of a query that we use is below: 


```javascript
 function _getAppBrands() {
            var orderBy = 'name';
            var appData = [];

            return EntityQuery.from('AppBrand')
                    .orderBy(orderBy)
                    .using(manager).execute()
                    .then(querySucceded, _queryFailed);

           
            function querySucceded(data) {
                appData = data.results;
                return appData;
            }
        }
```

This query allows us to get the AppBrands and order them by name. This type of set up allows for easier access and caching of data. When we wrap Breeze inside of our datacontext it allows us to only have one place to go for the data that we need.  
Although there are different ways of connecting Angular to your database this method has worked very well for us. Breeze provides a helpful SPA template and tutorial for setting it up with your datacontext [here](http://www.getbreezenow.com/ng-spa-template). I highly recommend it! 
