---
layout: post
title:  "Asynchronous AngularJS directives & services"
date:   2015-09-10 09:00:00
categories: javascript angularjs directives memoization
---

The purpose of this post is to show an example how to create AngularJS services utilizing memoization and asynchronicity.

With [Pouta Blueprints][blueprints] I had to get some configurable variables from its REST API and built the views based on these.
As this is needed in multiple places in the application, creating AngularJS service for this was a no-brainer.
As variables are fetched from the API, it would be wasteful not to cache the API response in the service.
Because the service utilizes REST API, it would also be wasteful to wait until the request made by the service returns.

The service described below achieves this by returning a promise from the REST call using the awesome [Restangular][restangular] library.

{% highlight javascript %}
app.service('ConfigurationService', ['Restangular', function(Restangular) {
    var configEndpoint = Restangular.all('config');
    this._config = null;

    this.getValue = function(key) {
        if (this._config == null) {
            this._config = configEndpoint.getList().then(function(response) {
                var config = {};
                angular.forEach(response, function(v, k) {
                    config[v.key] = v.value;
                });
                return config;
            });
        }
        return this._config;
    };
}]);
{% endhighlight %}

The service above can then be used in directives by injecting the service into the directive as dependency. 
Below the directive updates the text of an element only after the promise is resolved.

{% highlight javascript %}
app.directive('configurableValue', ['ConfigurationService', function(ConfigurationService) {
    return {
        restrict: 'E',
        link: function(scope, element, attrs) {
            ConfigurationService.getValue().then(function (data) {
                element.text(data[attrs.key]);
            }); 
        }
    };
}]);
{% endhighlight %}

Am I doing it wrong? Comments? I would be grateful to hear your insights, so
[tweet] me if you care.

[blueprints]:   https://github.com/CSC-IT-Center-for-Science/pouta-blueprints
[restangular]:  https://github.com/mgonto/restangular
[tweet]: https://twitter.com/harrmi
