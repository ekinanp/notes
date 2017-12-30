## Puppet Basics: Manifests, Resources, Parameters and Properties

Puppet is driven by manifests, the equivalent of scripts or programs, that are written in Puppet's Domain-Specific Language (DSL). It comes with its own background service.

To control system processes, boot options, and software installation, Puppet needs to be run with `root` privileges (it often manages OS-level facilities).

Puppet resources are `idempotent`, meaning every resource first compares the actual (system) with the desired (Puppet) state, and only initiates the action if there is a difference (configuration drift).

Resources are the building blocks of manifests. They each have a type (e.g. `notify`, `service`) and a name or title (e.g. `Hello, world!` and `puppet`). Resources are unique to a manifest, and can be referenced by a combination of its type and name, e.g. `Service["puppet"]`. Finally, a resource also contains a list of zero or more attributes, where an attribute is a key-value pair such as `enable => false`.

Attribute names are part of the puppet resource type. They come in two flavors:
* `parameters` describe things that help figure out what Puppet needs to do to make the resource a reality (e.g. `provider` for packages). For some parameters, such as the `provider`, Puppet can make educated guesses on what should be used for it.
* `properties` are things that are innate to the resource itself, such as the `ensure` property, the `uid` for a user account, the `gid` for a file, etc. They are kind of like that resource's settings, its desired state. Puppet only takes action on property values.

For each property, Puppet performs the following tasks:
* Test whether the resource is in sync with the target state
* If the resource is not in sync, trigger a sync action

A property is in sync when the state of that specific property of the resource matches what's in the manifest. In `puppet_service.pp`, the `ensure` property is in sync if the `puppet` service is not running; the `enable` property is in sync if the service provider is not configured to launch Puppet at system start.

One way to differentiate between `parameters` and `properties` is that `properties` can be out of sync, while `parameters` cannot.

## Interpreting output of the puppet apply command 

There are two key parts to `puppet apply`:
* Compile the manifest into a `catalog`, which is just the comprehensive, internal representation of the manifest.
* Apply the catalog on the machine. The catalog is how Puppet evaluates and syncs resources based on the catalog's contents. 

For example,

```
enis:ch1_writing_your_first_manifests enis.inan$ puppet apply -e'service { "puppet": enable => true, }'
WARN: Unresolved specs during Gem::Specification.reset:
  hiera (< 4, >= 3.2.1)
  gettext (>= 3.0.2)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
Notice: Compiled catalog for enis.inan-c02m48dgfd57 in environment production in 0.31 seconds
Warning: The /System/Library/LaunchDaemons/com.apple.jetsamproperties.Mac.plist plist does not contain a 'label' key; Puppet is skipping it
Error: Unable to write the file /var/db/com.apple.xpc.launchd/disabled.plist. #<IOError: File /var/db/com.apple.xpc.launchd/disabled.plist not writable!>
Notice: /Stage[main]/Main/Service[puppet]/enable: enable changed 'false' to 'true'
Notice: Applied catalog in 0.26 seconds
```

The above command's output instructed Puppet to do another change on the Puppet service (which is enabling it when the machine is booted). Here is the analysis of its log message:
* The `Notice:` keyword at the beginning of the line represents the log level. Other levels include `Warning`, `Error`, and `Debug`.
* The property that changed (`enable`) is referenced with a whole path, which is `/Stage[main]/Main/Service[puppet]/enable`. Properties are generally of the form `/<environment?>/<class>/<resource>/<property>`. `/Stage[main]` is beyond the scope of the book (I'm guessing it's the environment?). Note that the default class is `Main`, while remember that a resource is uniquely referenced as `Type[title]`.
* The rest of the log line is the type of change Puppet saw fit to apply (for example, `created` to indicate that the resource is newly added to the managed system; `changed <old_val> to <new_val>` representing the property being in-sync with its value in the catalog, etc.

## Dry testing your manifest

Use the `--noop` option.

## Using variables

Variable assignment works like in most scripting languages. Variable names are always prefixed with the `$` sign, for example:
* `$download_server = 'img2.example.net'`
* `$url = "https://${download_server}/pkg/example_source.tar.gz"`

Note that in Puppet, variables are immutable.

## Variable types

Puppet's type system will be explained in great detail in Chapter 7, but the common types (integers, strings, hashes, arrays) work like they do in many other languages. Some examples are:
```
$a_bool = true
$a_string = 'This is a string value'
$an_array = [ 'This', 'forms', 'an', 'array' ]
$a_hash = {
  'subject'   => 'Hashes',
  'predicate' => 'are written',
  'object'    => 'like this',
  'note'      => 'not actual grammar!',
  'also note' => [ 'nesting is', { 'allowed' => ' of course' } ],
}
```

Examples of accessing the value:
```
$x = $a_string
$y = $an_array[1]
$z = $a_hash['object']
```

Note that resource names/titles can also be a variable reference:
```
package { $apache_package:
  ensure => 'installed'
}
```

You can also pass arrays to declare a whole set of resources in one statement, for example:
```
$packages = [
  'apache2',
  'libapache2-mod-php5',
  'libapache2-mod-passenger',
]

package { $packages:
  ensure => 'installed'
}
```

This declaration will declare three package resources, `apache2`, `libapache2-mod-php5`, `libapache2-mod-passenger`, and ensure that they are all installed on the target system.` Intuitively, array names/titles indicate resources with the same attributes. 

## Data types

Puppet has core data types and abstract data types. Core data types are the most commonly used types of data such as string or integer, while abstract data types give you more sophisticated type validation, such as optional or variant.

## Adding control structures in manifests

The most common is the `if/else` block. For example:
```
if 'mail_lda' in $needed_services {
  service { 'dovecot': enable => true }
} else {
  service { 'dovecot': enable => false }
}
```

The `case` statement:
```
case $role {
  ‘imap_server’: {
    package { ‘dovecot’: ensure => installed, }
    service { ‘dovecot’: ensure => running, }
  }
  /_webservers$/: {
    service { [‘apache’, ‘ssh’]: ensure => running, }
  }
  default: {
    service { ‘ssh’: ensure => running, }
  }
}
```

You can also switch to type-specific code depending on variable datatypes, for example:
```
case $role {
  Array: {
    include $role[0]
  }
  String: {
    include $role
  }
  default: {
    notify { 'This nodes $role variable is neither an
    Array nor a String':}
  }
}
```

A variation of the `case` statement is the selector, which is an expression. For example,
```
package { 'dovecot':
  ensure => $role ? {
    'imap_server' => 'installed',
    /desktop$/    => 'purged',
    default       => 'removed',
  },
}
```

And like the case statement, the selector can also return results depending on the datatype:
```
package { 'dovecot':
  ensure => $role ? {
    Boolean => 'installed',
    String => 'purged',
    default => 'removed',
  },
}
```

## Controlling the order of execution

A manifest is not a script or a program. The Puppet DSL is a tool that models the system state as a set of resources, including files, packages, and cron jobs. It is declarative, meaning the manifest declares a set of resources that are expected to have certain properties. The resources are put into a catalog that Puppet then tries to build a path that goes through all the declared resources. The compiler parses manifests in order, but the configurer applies resources in a different way.

Thus, manifests should always describe what you expect to be the end result. Puppet decides what needs to be done to get there.

To show this distinction, consider the following example:
```
package { 'haproxy':
  ensure => 'installed',
}
file {'/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => 'root',
  group => 'root',
  mode => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
service { 'haproxy':
  ensure => 'running',
}
```

Here, Puppet will make sure we have the following state:
* The `HAproxy` package is installed
* The `haproxy.cfg` file is created with the specified content, which is in the file `puppet:///modules/haproxy/etc/haproxy/haproxy.cfg`
* `HAproxy` is started.

We ordered our above manifest that way because we need to:
* Ensure that the package is installed for the configuration file to exist
* Ensure that the configuration file is configured correctly before starting the service, as otherwise the service will start with the default settings. Note we also need the package itself installed.

However, the ordering in our manifest is not necessarily how Puppet will sync these relevant resources, because `it` does not know of the above specified dependencies.

## Declaring dependencies

Easiest way is resource chaining, such as below:
```
package { 'haproxy':
  ensure => 'installed',
}
->
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => 'root',
  group => 'root',
  mode => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
->
service {'haproxy':
  ensure => 'running',
}
```

The better way is by declaring the dependencies through special meataparameters, where metaparameters are parameters that can be used with any resource type. These are `require` and `before`. Both take one or more references to a declared resource as their value (remember, resources are referenced as `Type['title']`). With the `HAproxy` manifest, we can order it using the `require` parameter as follows:
```
package { 'haproxy':
  ensure => 'installed',
}
file {'/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => 'root',
  group => 'root',
  mode => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
}
service {'haproxy':
  ensure => 'running',
  require => File['/etc/haproxy/haproxy.cfg'],
}
```

There is also a `before` parameter, which indicates that the specified resource should be set-up `before` the specified resource. This is good when you have to declare a bunch of configuration files that a service might depend on. Some more examples of dependency ordering:
```
file { '/etc/apache2/apache2.conf':
  ensure => file,
  before => Service['apache2'],
}
file { '/etc/apache2/httpd.conf':
  ensure => file,
  before => Service['apache2'],
}
service { 'apache2':
  ensure => running,
  enable => true,
}
```
```
if $os_family == 'Debian' {
  file { '/etc/apt/preferences.d/example.net.prefs':
    content => '...',
    before => Package['apache2'],
  }
}
package { 'apache2':
  ensure => 'installed',
}
```

## Error propragation

Explicitly declaring dependencies also allows Puppet to "short-circuit" when applying the manifest. Specifically, if resource A depends on resource B and B fails to finish successfully, then resource A will automatically fail as well.

## Avoiding circular dependencies

Dependencies cannot form circles (i.e. the resulting DAG can't have cycles). Cycles can be explicit (where they can be deduced by examining `before` and `require` parameters) or implicit (caused by a hidden dependency that is not declared via. `before` or `require`). An example of an implicit cycle is:
```
file { '/etc/haproxy':
  ensure => 'directory',
  owner => 'root',
  group => 'root',
  mode => '0644',
}
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => 'root',
  group => 'root',
  mode => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
service { 'haproxy':
  ensure => 'running',
  require => File['/etc/haproxy/haproxy.cfg'],
  before => File['/etc/haproxy'],
}
```

where the DAG is something like:
   '/etc/haproxy/haproxy.cfg' => 'haproxy' => '/etc/haproxy' => (implicit) '/etc/haproxy/haproxy.cfg'

Some implicit dependencies include:
* If a directory and a file inside the directory are declared, Puppet will create the directory first and then the file.
* If a user and his/her primary group is declared, Puppet will first create the group and then the user.
* If a file and the owner (user) are declared, Puppet will first create the user and then the file.

## Implementing resource interaction

Resources can be configured to react to events via. the `subscribe` parameter, and to create events via. the `notify` parameter where here, "events" generally occur when a resource was found to be out of sync with its state in the catalog. A common, practical use of the event system is to reload service configurations (e.g. if a config. file changes, then a service is restarted).

Examples of the usage of `subscribe` and `notify`:
```
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => ‘root’,
  group => ‘root’
  mode => ‘0644’
  source => ‘puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
}
service { 'haproxy':
  ensure => 'running',
  subscribe => File['/etc/haproxy/haproxy.cfg'],
}
```
```
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner => ‘root’,
  group => ‘root’
  mode => ‘0644’
  source => ‘puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
  notify => Service['haproxy'],
}
service { 'haproxy':
  ensure => 'running',
}
```

Note that the `subscribe` and `notify` metaparameters offer additional semantics:
* The resource that subscribes to another resource implicitly requires it.
* The resource that notifies another resource is implicitly placed before it in the dependency graph

So `subscribe` ~ `require`, `notify` ~ `before`. You can also use the chaining syntax for signaling via. `~>`.

## Examining Puppet core resource types

These are just an overview of the key Puppet resource types. They are not really relevant to the fundamentals here, since one can look up their documentation online. Thus, no notes here.
