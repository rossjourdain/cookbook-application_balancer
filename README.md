Description
===========

This cookbook provides convenience wrappers, and LWRP's around the haproxy2 cookbook.
It enables configuration of proxy config sections within haproxy via recipes without having to manage solely in the attributes.
Each config via an LWRP sets a named attribute section meaning multiple recipes dependent on the same proxy config can both declare it
(both on the same server) and that section won't be defined twice.

None of the configuration options are linted or validated, simply passed straight through and written to the haproxy config file, so be warned

Requirements
============

Cookbook haproxy2 (https://github.com/jeremyolliver/cookbook-haproxy2) - fork of (https://github.com/demonccc/chef-repo/blob/master/cookbooks/haproxy2)

Attributes
==========

The attributes for this cookbook are only a placeholder keyed by the name passed to the LWRP to prevent doubling up if included twice.
All attributes are translated into another set to be interpreted by the haproxy2 cookbook

Usage
=====

Important!

to use the LWRP in one of your application cookbooks, then include the recipe "application_balancer::default".
Then use the LWRP's provided to configure haproxy as desired. To write the config, the recipe "application\_balancer::server" must be included. This needs to be included only once, otherwise you'll get duplicates. If you have only one application to configure for per node, you can include this in your wrapper cookbook, otherwise you'll need to only include "application\_balancer::server" at the end of your run\_list

Examples of some various options. `role_app`, `extra` and `member_options` are special terms that haproxy2 uses to search for nodes with the given chef role and declares those as the servers
with the given `member_options` and `extra` attributes. Alternately any param may be declared, for valid options see section 4.1 of http://haproxy.1wt.eu/download/1.3/doc/configuration.txt
for which keywords are valid for which contexts (`balanced_app` provides a `listen` block.) For keywords that can be declared multiple times (such as use_backend and server) pass in the value
as an array.

    balanced_app "myappname" do
      role_app "mychefrole"
      member_options({"port" => "80"})
      bind "0.0.0.0:4444"
    end


    application_frontend "myfrontendsite" do
      bind "IPADDRESS:8080"
      default_backend "mainserver"
      acl "apiservice    path_beg /v1/api"
      use_backend "api if apiservice"
    end

    application_backend "api" do
      no_option "redispatch"
      server "#{node.fqdn} 127.0.0.1:7000"
    end


TODO
====

Convert the "LWRP"'s to real resource definitions not just wrapper methods storing config in attributes, then we should be able to avoid the need to manually include the ::server recipe at the end of the run\_list. Doing so should allow more consolidation of this cookbook and the haproxy2 cookbook together, and better extensibility
