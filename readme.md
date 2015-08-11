Chef Real World Patterns
========================

##Introduction
Ok, you have all digested the basic foundations of Chef. You get the concept of using the basic building blocks in the form of **_resources_** which stack together and describe more complex things within **_recipes_**. Finally, one or more of these recipes and other supporting files go into a **_cookbook_**, which becomes the artifact of the desired state that you have now defined.

In the last meet-up I demonstrated local testing through **_Test Kitchen_**, where you could see your desired state carried out and verified on your local workstation, or even a test instance within your cloud provider. 

With these new skills, you should be able to show off to your friends by spinning up an instance of anything you could possibly **_desire_** (that was actually a pun).

##A Quick Review of Chef Basics

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/resources.png">
###Resources
Basic building block of Chef. Piece of the system and its desired state.

<br>

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/recipes.png">
###Recipes
Recipes are a collection of resources that together create higher level of functionality. 

<br>

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/cookbooks.png">
###Cookbooks
Cookbooks are a collection of recipes along with other items that allow it to address different outcomes. Cookbooks are the artifacts of Chef code.

####Tenants of Good Cookbook Design
There are some tenants of good cookbook design, that become especially important when you consider real world application. A lot of the following patterns assume these tenants are being applied.

#####The Application Cookbook
An application cookbook is where configuration specific to an application is written and maintained. For example, a cookbook developed to install and configure Apache Tomcat contains only the recipes necessary for installing tomcat binaries and configuring the service to listen on a desired port.

#####Abstracting Policy from Data
Cookbooks should be created in a way that makes them reusable, allowing them to serve different people in different situations.

This is accomplished by using cookbook attributes over hard-coded values whenever possible. This includes using templates over file resources, to take advantage of variable content. The result is a flexible cookbook where the end user can make it work for their own unique situations.

<br>

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/roles.png">
###Roles
a way to define certain patterns that exist across nodes within an organization

<br>

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/environments.png">

###Environments
a way to map an organizations real-life workflow to a chef server. For example, it allows you to lock specific versions of cookbooks to development, test and production environments.

####Environment Pinnings
Chef environments are used to group servers, generally mapped to your company’s software release cycle. Common Chef environment designations are development, stage, and production. Environments are also used to provide attributes which would be global to all nodes in that environment, allowing for a single point of controlling service configuration to a wide range of servers. The types of attributes typically reserved for the configuration of environments are NTP servers, DNS servers, subdomain naming, package repositories, etc.

Environments are also where cookbook pinning takes place. Pinning a cookbook version in an environment prevents newer versions of that cookbook from being applied to nodes in that environment. This allows for development to safely take place in a pre-production environment while protecting a staging or production environment from being impacted by changes to a cookbook.

It is recommended that all environments have specific cookbook versions defined for use. This will make it explicitly clear to any Chef developer working with the environment which cookbooks are expected to be used. It will also make promoting the expected versions into the next environment for testing.

The following is an example of an environment, including cookbook pins and an example attribute for that environment:

<img style="float: right" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/environments.png">

<br>

<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/organizations.png">
###Organizations
A way to provide multi-tenancy within a chef server. Policy cannot be shared between organizations


##Chef in the Real World
This is all well and good, but where do we go from here? How does this all work in the real world?

_A typical cookbook deployment life cycle_
<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/Chef-SLC.png">


###No Simple Answer
Because no one organization is the same as another, there is no generic answer to this question. 

###Chef Patterns
What I can do is provide a set of proven patterns that have been battle tested over time, along with some commonly accepted anti-patterns to avoid.

By selectively applying these patterns you can address your organization's unique requirements.

**_Note: These patterns are based on tribal knowledge, but not all tribes get along. You should look at these patterns objectively based on how they may (or may not) fit for you and / or the organization you work for._**

#On to Chef Patterns!

<br>

##Wrapper Cookbook Pattern
Speaking of end users, it is often the case that the author of the cookbook and the ultimate user of it are different people, perhaps in different organizations all together.

A common technique in this situation is to have the end user create a new cookbook, called a wrapper cookbook.

As its name implies, a wrapper cookbook is one which wraps itself around an existing cookbook, typically an application cookbook. A wrapper cookbook may extend functionality not found in an existing cookbook. In most use cases however it is generally used as a means of changing attributes found in an application cookbook. As an example, a wrapper cookbook which reconfigures Apache Tomcat beyond its community cookbook defaults would be called my-tomcat.

It would included the  **_application cookbook_** recipe through an _include_recipe_ statement: 
```ruby
incude_recipe "tomcat::server"
```

The attributes can be set through a _set_ method such as:
```ruby
default.set['tomcat']['base_version'] = 6
```
A simple entry into the *_metadata.rb_* is required to make the **_application cookbook_** available.
```ruby
depends "tomcat", ">1.2.1"
```
<img style="float: left" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/wrapper.png">

By inheriting the application cookbook, the end-user is left only to decide on the attributes needed for their specific desired state. They can even add recipes for additional company specific policy if required.

This pattern is especially common for using community cookbooks, where the community cookbook is the **_application cookbook_**.

###Anti-Pattern
Copying or forking the **_application cookbook_**. This often leads to drift as each cookbook evolves separately, at some point you just own a redundant cookbook, with all the original value of reuse lost.

<br>

##Base Cookbook Pattern
Each organization should have its own base cookbook. A base cookbook should provide only the minimal operating system components required to make a server network-ready, accessible to administrators, and suitable for application deployments. Examples of functionality in base cookbooks include configuration of network routes, DNS, LDAP/ActiveDirectory, and sshd.

*Base cookbook design pattern*
<img style="float: right" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/base-cookbook.png">

###Anti-Pattern
Adding this common base policy ala cart into many different roles and / or run-lists.

This can lead to an untenable management situation as this base policy evolves over time.

<br>

##Role Cookbooks
Within the Chef community, there are differing opinions on the use of roles. One opinion is that roles should not be used at all as they are not versioned in the Chef Server. As a result, roles can not be pinned the way cookbooks can. Therefore, a change in a role affects all nodes in all environments simultaneously. While this is a downside to roles there is an upside to roles being able to concatenate a potentially lengthy run list in a more easily describable manner. 

In the spirit of having one's cake and eating it to the role cookbook pattern emerged. This pattern consists of having a role with a single cookbook of the same name in its run list. This single cookbook (the role cookbook) maintains the intended run list through a series of include_recipe statements within the default recipe.
The result is a role you can assign to node's run lists that can be pinned and versioned through the single role cookbook associated with it.

<img style="float: right" src="https://s3-us-west-1.amazonaws.com/nedworks-web-images-01/Roles1.png">



