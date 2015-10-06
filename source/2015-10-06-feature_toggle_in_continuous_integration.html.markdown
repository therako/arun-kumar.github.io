---
title: "Feature Toggles in Continuous Integration"
date: 2015-10-06 00:13 UTC
tags: [Devops, FeatureToggle, ContinuousIntegration]
comments: true
---

Hi all, this is a journey through how we did Continuous Integration in our project with Feature toggles.

### Why Feature Toggle?
A few days before the First ever release (when the feature to be released is dev done), there was a discussion on whether to do Feature branches or Feature Toggles. After that discussion we choose to do Feature Toggles as our team was growing and splitting into multiple small feature teams. I am not gonna bore everyone with why and what's of Feature toggles, here a link to Martin Fowler's blog on Feature Toggles.

### Some example of Feature toggles
A small example of how feature toggles looks like in a Rails app,
{% highlight ruby %}
 Application.routes.draw do
  root 'pages#welcome'
   get '/welcome' => 'pages#welcome'

   if FeatureFlag.allow_comments?
     get  '/comments' => 'comments#all'
     get  '/comment'  => 'comments#new'
     post '/comment'  => 'comments#create'
   end
 end
{% endhighlight %}

Here the entire routes are toggled. The FeatureFlag is a wrapper around Environment config variables.

{% highlight ruby %}
 class FeatureFlag
   def self.allow_comments?
     Config['COMMENTS_SECTION_AVAILABLE'] == 'true'
   end
 end
{% endhighlight %}

The routes to see, add new comments will be available only if the Feature toggle is turned ON, and user won't be able to access those pages/features when toggled OFF. When the comments feature is fully developed and ready to release, it can be turned ON by changing just a config file.

### Discussion and Decision

Back then Our CI server was just doing unit and automation test nothing more. Now there was a bigger issues of How are we going to release a new builds with some bug fixes for the old feature. Of course the answer is just to toggle off the new feature in the production environment config and deploy. But the problem here was, we were also doing a more changes on Navigation and other trivial flows which are common on to all the features.

The discussion started with the example of Navigation changes,

To accomplish the new feature the Navigations have to be changed, but it doesn't make sense to release just the new navigation with old features. So we had them under the feature toggles and now we have two Navigation flows. We have decided to test both the flows.

And then we came up this idea of Branching our CI pipelines into two layers (Development and Release).

{% img center /images/feature_toggle_in_continuous_integration_1.png  %}

**Note:** Packaging/ Versioning is a stage where we create a deployable rpm package and version it. Here we package the app once, then test by installing it in test environments and finally install it in Production.

This is behind the idea of achieving Feature Branching Advantages in Feature Toggles. How?

We split Feature Toggle configs into two environments

- Development config - where every feature is always toggled ON.

- Release Config - Only the currently deployed into production Feature are ON. (This is also used in times of release to turn on one particular Feature and run it through the pipelines to get it released)

Initially It was hard to split the Automation to run based on Feature Toggles, but it became easy and we went along. Just about adding new tests and also keeping the old tests. In this case we had two different Navigation tests.

### Conclusion
We run the set of commits picked by the CI tool through both these pipelines. so in order to call your story complete It has to pass through both versions of feature toggles. And thus we created a way to make these set of commits deployable, whenever the business wants it to be deployed.
  
**Note:** The set of commits differs from one tool to another and also based on how many commits is being made in a particular interval which the CI tool takes to poll for the new changes. In our case there is a default wait time of about 30 seconds, so there could be two commits by two different developers going in together in the same build.


This is how our Final setup looks like.

{% img center /images/feature_toggle_in_continuous_integration_2.png  %}