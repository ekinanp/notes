## Introducing Classes and Defined Types

`classes` and `defined types` (also refered to as `defines`) are Puppet's equivalents to methods or functions in other programming languages. They let you write re-usable Puppet code.

## Defining and declaring classes

A Puppet class is essentially a container of Puppet resource declarations. It's created once (class definition), and used by all nodes that need it. Classes represent a well-known subset of system config., like `ntp`, `nginx`, and `ssh`.

One classic use case is a class that installs the Apache web server and applies some basic settings, shown below:
```
class apache {
  package { 'apache2':
    ensure => present,
  }
  file { '/etc/apache2/apache2.conf':
    ensure => 'file',
    source => 'puppet:///modules/apache/etc/apache2/apache2.conf',
  },
  service { 'apache2':
    ensure => running,
    enable => true,
    subscribe => File['/etc/apache2/apache2.conf'],
  }
}
```

Any webserver node that wants to make use of this class needs the following line in their manifest:
```
include apache
```

^ That is called "including" a class, or declaring it. You can even have that be the only thing in a node's manifest:
```
node 'webserver01' {
  include apache
}
```

## Creating and using defined types

A defined type is a new resource type that makes use of existing resource types. It is useful when you have a repeating list of existing resource types, b/c you can wrap them up in a defined type.

As a class, it consists of a body containing the manifest code. But a defined type can also take arguments that make their values available in its body as local variables (exactly why they are like functions in other programming languages). For example, consider the Apache virtual host configuration:
```
define virtual_host(
  String $content,
  String[3,3] $priority = '050'
) {
  file { "/etc/apache2/sites-available/${name}":
    ensure => 'file',
    owner => 'root',
    group => 'root',
    mode => '0644',
    content => $content
  },
  file { "/etc/apache2/sites-enabled/${priority}-${name}":
    ensure => 'link',
    target => "../sites-available/${name}";
  }
}
```

So here, you can see that the parameters are `content` and `priority`, each with their own types (`String` for `content`, and `String[3,3]` for `priority`). The variable `name` is the title of the defined type that is automatically injected when it is declared (you can also reference it via. `title`). There are several different ways to declare a defined type, shown below:
```
virtual_host { 'example.net':
  content => file('apache/vhosts/example.net')
}
virtual_host{ 'fallback':
  priority => '999',
  content => file('apache/vhosts/fallback')
}
```
```
virtual_host {
  'example.net':
    content => 'foo';
  'fallback':
    priority => '999',
    content => ...,
}
```

where the second declaration encapsulates declaring multiple resources of the same type. Notice how declaring a defined type is identical to defining a resource -- you have your title, as well as some parameters/properties here and there.

Note some subtleties that are not outlined by the text:
1. Resources declared inside a defined type should still have unique titles, b/c you can still refer to those resources via. the standard `Type['title']` syntax. For example, I can reference `File['/etc/apache2/sites-available/example.net']` that is defined via. the 'example.net' `virtual_host`. This is b/c defined types are syntatic sugar to wrap up repeated resources under a single container.

Note there are other `magic` variables like `name/title` that you can access in a defined type's body, such as `require` when the defined type is defined with the `require` metaparameter. If the metaparameter is not used, the variable value is empty.

## Understanding and leveraging the differences

The purposes of Puppet's class and defined types are specific, and do not overlap.

A class declares resources and properties that are centric to the system. It is a finalized description of one, or sometimes more, aspect of your system as a whole. Whatever the class represents, it can only exist in one form. To Puppet, each class is a singleton, a fixed set of information that applies to your system (class is included), or it does not.

Typically, a class will encapsulate the following resources:
* Packages that should be installed (or removed)
* Some config. file in `/etc`
* A common directory, needed to store scripts or configs for many subsystems
* Cron jobs that should be mostly identical on all applicable systems

The define is used for all things that exist in multiple instances, i.e. things that appear in varying quantities in your system can be modeled with it. Typical contents of defined types are:
* Files in a `conf.d` style directory
* Entries in an easily parseable file such as `/etc/hosts`
* Apache virtual hosts
* Schemas in a database
* Rules in a firewall

*The class' singleton nature is valuable because you cannot have clashes in the form of multiple resource declarations, which is something that the defined type does not prevent*

Note that you cannot have two defined types with the same name.

## Design patterns

Say you need a class that ensures that the following conditions are met:
* The firewalling software is installed and configured with a default ruleset
* The malware detection software is installed 
* Cron jobs run the scanners at set intervals
* The mailing subsystem is configured to make sure the cron jobs can deliver their output

You can have two different approaches to designing a class this big:
* Do a *monolithic* implementation that contains all of the required resources to get things to work
* Do a *composite* implementation comprised of a single, `collecting` class that includes multiple classes, each with their own distinct purpose (and a small resource body). This is how Puppet modules are commonly done.

Note that classes can include other classes. In fact, a class body can have almost any manifest (resource declarations, include statements, defined types).

Here's how our class would look using the monolithic approach:
```
class monolithic_security {
  package { [ 'iptables', 'rkhunter', 'postfix' ]:
    ensure => 'installed';
  }
  cron { 'run-rkhunter':
    ...
  }
  file { '/etc/init.d/iptables-firewall':
    source => ...
    mode => 755
  }
  file { '/etc/postfix/main.cf':
    ensure => 'file',
    content => ...
  }
  service { [ 'postfix', 'iptables-firewall' ]:
    ensure => 'running',
    enable => true
  }
}
```

With the composite approach, we would instead have:
```
class divided_security {
  include iptables_firewall
  include rkhunter
  include postfix
}
```

## Writing component classes

Component classes wrap a specific resource. For example, you can declare a `netcat` class as such:
```
class netcat {
  package { 'netcat':
    ensure => 'installed'
  }
}
```
This is useful for when you might have multiple classes which use the same resource. For example, assume Classes A and B both use Resource C. Then if you include A and B in a manifest, your manifest will fail to compile because Puppet will see Resource C being declared twice. However, if you create a component class, Class C, that wraps Resource C, you can include it in A and B and your manifest will compile without any issues (because classes are singleton, so Class C, and consequently, Resource C, is created only once).

## Using defined types as resource wrappers

When wrapping a resource with a defined type, you end up with a variation on the respective resource type.  Specifically, the manifest can contain an arbitrary number of instances of the defined type, each wrapping a distinct resource. Remember, though, that the names of the resources declared in the body of the defined type must be dynamically created, using e.g. the `name` variable.

For example, you can use a defined type to make it easy to declare files whose contents are contained in Puppet modules:
```
define module_file(String $module) {
  file { $title:
    source => "puppet:///modules/${module}/${title}"
  }
}
```

Wrappers like `module_file` should also support all attributes of the wrapped resource type. You can have it accept all `file` attributes as part of its arguments, as shown below:
```
define module_file(
  String $module,
  Optional[String] $mode = undef
) {
  if $mode != undef {
    File { mode => $mode }
  }
  file { $title:
    source => "puppet:///modules/${module}/${title}"
  }
}
```

The syntax `<Type> { <property/parameter> => value> .. }` is a convenient way to indicate that, within the current scope, the value of the resource `<Type>`'s property/parameter is set to the specified default `<value>` if an overriding value is not provided. In our example, we have done this for the `File` resource to set a default mode.

## Using defined types as resource multiplexers

This is just some examples of different use cases of defined types. One is below:
```
define user_with_key(
  String $key,
  Optional[String] $uid = undef,
  String $group = 'users'
) {
  user { $title:
    ensure => present
    gid => $group,
    uid => $uid,
    managehome => true,
  }
  ssh_authorized_key { "key for ${title}":
    ensure => present,
    user => $title,
    type => 'rsa',
    key => $key,
  }
}
```

where the above defines a user account and also their corresponding ssh key. You can also use defined types as macros.

Some of the later sections talk about using defined types as iterators, and also introduce the iterator feature found in Puppet 4 and 5 (which is pretty much using Ruby's enumerator stuff). They also talk about combining component classes and defined types. Pretty much ways to write clean Puppet code, which is a lot like writing clean code in any programming language.

## Ordering and events among classes

The encapsulation given to you by Puppet classes makes it a bit difficult to maintain event handling between the resources declared inside that Puppet class and any resources outside of the Puppet class. The example they gave in the book was a background process that did some work on Apache log files subscribing to the config. files in the Apache class (so that it can restart if these files change). The process has no way of knowing what config. files the Apache class modifies (because the class encapsulates these details) so it cannot hardcode its subscription (even if it could, things would fail if these config. files change).

Puppet gives a clean way to subscribe to/notify any resources inside a class, by just subscriting to/notifying the class itself. For example,
```
file { '/var/lib/apache2/sample-module/data01.bin':
  source => '...',
  notify => Class['apache'],
}
service { 'apache-logwatch':
  enable => true,
  subscribe => Class['apache'],
}
```

where things work as you expect, i.e. The event hooks will trigger if *any* resource in the corresponding class is modified. You can do something similar with defined types.

Note that, unfortunately, the `subscribe` and `notify` stuff does not work with nested classes but it does work with defined types (inside classes, or inside other defined types).

## The anchor pattern

This is a hacky way to go around this limitation, What you basically do is for classes that you know might need event-based interactions, you invoke the following syntax:
```
anchor { '<containing_class_name>::begin':
  notify => Class['<class_name>']
}
include <class_name>
anchor { '<containing_class_name>::end':
  require => Class['<class_name>']
}
```

This works b/c `anchor` is itself a resource. The notation creates the following graph:
```
anchor['<containing_class_name>::begin'] => <class_name> => anchor['<containing_class_name>::end']
```

So when I do `before => Class['<containing_class_name>']` in some resource, that resource will be created before the first anchor resource, which will be created before the nested class `<class_name>`, so that I ensure that my resource is created before my nested class `<class_name>`.

Note that the anchor class is part of the `stdlib` module, it is not a part of the Puppet core.

## The contain function

Much more elegant. The syntax, for example, is as shown below:
```
class apache {
  contain apache::service
  contain apache::package
  contain apache::config
}
```

And here is the official documentation for the `contain` statement:
```
A contained class will not be applied before the containing class is begun, and will be
finished before the containing class is finished."
```

Note we can also contain classes in defined types as well.

## Making classes more flexible through parameters

Classes can also have parameters. For example,
```
class apache::config(Integer $max_clients=100) {
  file { '/etc/apache2/conf.d/max_clients.conf':
    content => "MaxClients ${max_clients}\n",
  }
}
```

And here is how this class would be declared:
```
class { 'apache::config':
  max_clients => 120,
}
```

The problem with parameterizing classes is that you lose their singleton characteristic.

Note you can still `include` classes, the parameters would just adopt their default values. You cannot invoke the resource-like declaration of a class (e.g. `class { 'name': }`) more than once (b/c it doesn't make sense to bind different values to a class' parameters in different locations throughout the manifest).

You also cannot have a resource declaration after an `include`, b/c the `include` implicitly declares the class resource, but with default parameters.

## Summary

Classes and defined types are the essential tools to create reusable Puppet code. Classes hold resources that must not be repeated in a manifest, while the define can manage a distinct set of adapted resources upon every invocation.
