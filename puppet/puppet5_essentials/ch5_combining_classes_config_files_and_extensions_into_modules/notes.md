## Prelude

Modules are a way of organizing classes and defines, and of sharing common code. They're software libraries for Puppet manifests and plugins. Open source modules are hosted on the Puppet Forge.

## The contents of Puppet's modules

A module is a higher-order organizational unit. It bundles up classes and defined types contributing to a common, IT management system (e.g. system aspects or a piece of software). Manifests are not all organized through modules; most modules also include files and file templates. You can also have several different kinds of Puppet plugins in a module.

## Parts of a module

*Manifests* are the most important part, b/c they contain the core functionality. They consist of classes and defined types, all sharing a namespace rooted at the module name. For example, an `ntp` module will have classes and defines that start with the `ntp::` prefix.

Many modules contain *files* that can be synced to the agent's filesystem -- this is generally used for config. files or snippets. For example,
```
file { ‘/etc/ntp.conf’:
  source => ‘puppet:///modules/ntp/ntp.conf’,
}
```

Sometimes, you may need to tweak parameters in these files so node manifests can declare custom config. settings. The tool to do this are templates.

Modules may also contain *custom facts*, custom native types and providers, and  *parser functions* (aka *custom functions*). The latter are actual functions that can be used in your own manifests.

## Module structure

Each of the above mentioned components need to be located in specific filesystem locations for the master to find them. Each module forms a directory tree, with the root named after the module itself. For example, the `ntp` module is stored in a directory called `ntp/`.

All manifests are stored in a subdirectory called `manifests/`. Each class and defined type has its own respective file. For example, the `ntp::package` class is in `manifests/package.pp`; the defined type called `ntp::monitoring::nagios` is in `manifests/monitoring/nagios.pp`. So really, `::` is like the directory separator here.

The `manifests/init.pp` is a default manifest location that is looked up for any definition from the module in question. Generally, it should hold only one class that is named after the module (e.g., the `ntp` class). Manifests can then just use `include ntp` to get the core functionality of the module.

Files and templates must be placed in `files/` and `templates/`. Static files should always be addressed through URLS, e.g.
```
puppet:///modules/<module-name>/<path>
```
with its absolute path being:
```
.../modules/<module-name>/<files>/<path>
```

All plugins are in the `lib` directory. You can have:
* Custom facts in `lib/facter/`
* Parser functions in `lib/puppet/parser/functions`
* Puppet 4 API functions in `lib/puppet/functions/`
* Custom types and providers in `lib/puppet/type/` and `lib/puppet/provider/`, respectively.

So a module tree can look something like:
```
/etc/puppetlabs/code/environments/production/modules/<module-name>
  templates/
  files/
    subdir1/ # puppet:///modules/<module-name>/subdir1/<filename>
    subdir2/ # puppet:///modules/<module-name>/subdir2/<filename>
  manifests/
    init.pp # class <module-name> is here
    params.pp # class <module-name>::params is here
    config/
      detail.pp # <module_name>::config::detail is defined here
      basics.pp # <module_name>::config::basics is defined here
  lib
    facter/ # contains .rb files with custom facts
    puppet/
      functions # contains .rb files with Puppet 4 functions
      parser/
        functions $ contains .rb files with parser functions
        type/ # contains .rb files with custom types
        provider/ # contains .rb files with custom providers
```

## Managing environments

The *environment* groups and contains modules. It consists of:
* One or more site manifest files
* A `modules` directory with your modules inside
* An optional `environment.conf` configuration file

When a master compiles the manifest for a node, it uses exactly one environment. It will always start in `manifests/*.pp`, which forms the environment's site manifest. Here is an example `environment` directory:
```
/opt/puppetlabs/code/environments/
  production/
    environment.conf
    manifests/
      site.pp
      nodes.pp
    modules/
      my_app/
      ntp/
```

The `environment.conf` file can customize the environment. Puppet usually uses `site.pp` and any other file in the `manifests` directory.

`site.pp` includes node classification with classes from the modules. Puppet looks for modules in the `modules` subdirectory of the active environment.

## Configuring environment locations

Puppet uses the `production` environment by default. This and other environments are located in `/opt/puppetlabs/code/environments` or `/etc/puppetlabs/code/environments`. You can specify another default by modifying the `environmentpath` option in `puppet.conf`.

## Obtaining and installing modules

The Puppet Forge is a good site for sharing and obtaining modules. `stdlib` is a really good place to look at how modules are organized, and to also start with some basic Puppet exercises (re-implementing some of the functionality that it provides).

## Module best practices

All manifest code should be in modules, with a few exceptions:
* The `node` blocks
* The `include` statements for very select classes that should be omnipresent
* Declarations of helpful variables that should have the same availability as the Facter facts in your manifests (useful "global" variables)

## Putting everything in modules

Consider the following class
```
include ntp::server::component::watchdog
```

Here's how Puppet does its compilation.

1. It finds the `ntp` module by checking all configured module locations of the active environment (path names in the `modulepath` setting).
2. It will then try and read the `ntp/manifests/server/component/watchdog.pp` to find the class definition. If it cannot, it will try `ntp/manifests/init.pp`.

Note that technically, you can stuff all of a module's manifests in its `init.pp` file, but you lose the advantages of a structured tree that the module manifest offers.

## Avoid generalization

One important point is that modules should be named after a specific piece of the overall system that is being managed (e.g. `apache`, `ssh`, `nagios`, `nginx`, etc.). Modules like `utilities` or `helpers` should be avoided -- these are probably better served as parser functions, custom types, or custom providers.

## Testing your modules

The `--noop` option is the most important tool for testing any changes made to your modules. It works for `puppet agent` and `puppet apply`.

Remember though that with a master, there are other agents connected to it that are likely polling for an updated catalog. These agents will run anyways.

## Safe testing with environments

Besides `production`, there should be at least one testing environment. The test environment should be a copy of the production data, so manifest changes should be prepared there first. You can then test your changes on agents via.
```
puppet agent --test --noop --environment testing
```

Note that some small mistakes cannot be detected via. `--noop`, so it is a good idea to run the code at least once.

There are also more formal ways of testing modules:

* `rspec-puppet` lets module authors implement unit tests based on `rspec`.
* Acceptance testing through `beaker`.

## Building a component module

The rest of the chapter walks you through creating an example module step by step. I can refer to this section whenever if I get stuck in the future.

## Replacing a defined type with a native type

Here are the steps you must go through:
* Naming your type
* Creating the resource type's interface
* Designing sensible parameter hooks
* Using resource names
* Adding a provider
* Declaring management commands
* Implementing the basic functionality
* Allowing the provider to prefetch existing resources
* Making the type robust during provisioning

## Naming your type

Module namespacing does not occur for custom types like it does for defined types. So you should not call the native implementation of `packt_cacti::device` just `device` -- it could clash with another notion of `device` that another module might have. The best choice for this first resource type is thus `cacti_device`. Its implementation must be in `packt_cacti/lib/puppet/type/cacti_device.rb`.

At a high-level, here's what the code looks like. First you declare the type:
```
Puppet::Type.newtype(:cacti_device) do
  @doc = <<-EOD
  Manages Cacti devices.
  EOD
end
```

Then you create the interface, which (from what I see) is defining all of the parameters for that type. For example, the code below defines the `ip` parameter for the resource:
```
newparam(:ip) do
  desc "The IP address of the device."
  isrequired
  validate do |value|
    begin
      IPAddr.new(value)
    rescue ArgumentError
      fail "'#{value}' is not a valid IP address"
    end
  end
  munge do |value|
    value.downcase
  end
end
```
Recall that `munge` just takes the input value for the parameter and transforms it into a more suitable form.

Every resource must have a `name` parameter that is the resource title. You can specify that a different parameter use the resource title. For example,
```
newparam(:name) do
  desc "The name of the device."
  #isnamevar # → commented because automatically assumed
end
```

`isnamevar` indicates that the parameter takes in the value of the resource title. This will be set by default for a parameter with the name `name`.

Along with some other stuff, the custom resource type can be used inside a manifest:
```
cacti_device { 'eth0':
  ensure => present,
  ip => $::ipaddress,
  ping_method => 'icmp',
}
```

And this code will get compiled into a catalog. However, the agent will produce an error b/c there is no provider available for this resource.

## Adding a provider 

The above section showed how to create a resource type that's ready for action. However, there is no provider to do the actual work of inspecting the system and performing the necessary synchronization. 

The name of the provider should reflect the management approach that it implements. For the Cacti example in the book, the provider relies on the Cacti CLI. For a Package resource, as another example, each package provider has its own `.rb` file, e.g. `yum.rb`, `zypper.rb`, etc.

Here is the skeleton structure (in `packt_cacti/lib/puppet/provider/cacti_device/cli.rb`):
```
Puppet::Type.type(:cacti_device).provide(
  :cli,
  :parent => Puppet::Provider
  ) do
end
```
You can see the following:
* The `type` that the provider acts on is referenced
* The name of the provider `:cli` is referenced
* `:parent` indicates the base class of the given provider -- `Puppet::Provider` is the default, which can be overwritten with your own, custom base class.

The `commands` method in providers binds executables to Ruby functions, e.g.:
```
commands :php => ‘php’
commands :add_device => ‘/usr/share/cacti/cli/add_device.php’
commands :add_graphs => ‘/usr/share/cacti/cli/add_graphs.php’
commands :rm_device => ‘/usr/share/cacti/cli/remove_device.php’
```

Declaring commands serves two purposes:
* You can call the commands through a generated method
* The provider will mark itself as `valid` only if all the commands are found on the given system.

Note for our example, `php` will not be invoked directly. But it is required to invoke the `.php` scripts. Thus, we include it as a `command` only to validate that it is present on the target system. The rest of the commands are syntatic sugar to declare and reference the relevant commands as ruby functions.

The command methods themselves return the output displayed to the console when running (not sure if it is just `stdout`, or `stdout` mixed with `stderr`).

## Implementing the basic functionality

The basic functionality of the provider can now be implemented in three instance methods. These three methods are the default that the `ensure` property expects to be available (remember that the `ensurable` shortcut was used in the type's code). 

The first method creates a resource if it does not exist yet.
```
def create
  args = []
  args << "--description=#{resource[:name]}"
  args << "--ip=#{resource[:ip]}"
  args << "--ping_method=#{resource[:ping_method]}"
  add_device(*args)
end
```

Note that there is no need to quote parameter values as they would be on the command-line -- Puppet already takes care of this behavior for you.

Here is how to destroy the entity:
```
def destroy
  rm_device("--device-id=#{@property_hash[:id]}")
end
```

Note that each resource gets its own provider instance, and all of its munged property and parameter values are contained in its `property_hash` variable.

Here is the final provider method that implements the `ensure` property:
```
def exists?
  self.class.instances.find do |provider|
    provider.name == resource[:name]
  end
end
```

## Allowing the provider to prefetch existing resources

The `instances` method above implements the prefetching of system resources during provider initialization. It is added to the provider itself. Not all resources are suitable for prefetching (e.g. it would be impractical to do this for files!) so their providers do not have an `instances` method implemented. Here's what the code would look like for Cacti devices:
```
def self.instances
  return @instances ||= add_graphs(“--list-hosts”).
  split(“\n”).
  drop(1).
  collect do |line|
    fields = line.split(/\t/, 4)
    Puppet.debug “prefetching cacti_device #{fields[3]}
    “ +
    “with ID #{fields[0]}”
    new(:ensure => :present,
    :name => fields[3],
    :id => fields[0])
  end
end
```

Note that we use `||=` so that we only prefetch once (it assigns only if the value is undefined).

We see that the method just returns an instance for each of the resources that are found on the system.

Note that Puppet requires another method to perform proper prefetching. The mass-fetching via. `instances` supplies the agent with a list of *provider instances* found on the system. From the master, however, the agent received a list of *resource type* instances. Puppet, unfortunately, has not yet mapped the resources (type instances) to the providers. The `prefetch` method makes this happen:
```
def self.prefetch(resources)
  instances.each do |provider|
    if res = resources[provider.name]
      res.provider = provider
    end
  end
end
```

^ What we do here is link up the resource type instance's provider to the provider instance itself.

All of the above stuff is really just Ruby-implementation of the concept that `resource type instance => provider instance`.

We can use our new resource type as follows:
```
node "agent" {
  include cacti
  cacti_device { "Puppet test agent (Debian 7)":
    ensure => present,
    ip => $::ipaddress,
  }
}
```

Note that a native type does `not` default `ensure` to `present` -- its value must be explicitly supplied.

## Make the type robust during provisioning

The previous section created the `cacti_device` resource type and a provider for it. However, it is not self-sufficient -- `cacti_device` requires the system to have the `cacti` package installed for it to do its work. To enforce this at the native type level, you can add the following to the `cacti_device` type:
```
autorequire :package do
  catalog.resource(:package, 'cacti')
end
```

## Enhancing Puppet's system knowledge through facts

Here is an example of how to create a custom fact that can be used with the `cacti` module. Remember that these belong in the `lib/facter` subtree of a module.

Here's a custom fact, `packt_cacti/lib/facter/cacti_graph_templates.rb` that will work:
```
Facter.add(:cacti_graph_templates) do
  setcode do
    cmd = ‘/usr/share/cacti/cli/add_graphs.php’
    Facter::Core::Execution.exec(“#{cmd} --list-graphtemplates”).
    split(“\n”).
    drop(1).
    collect do |line|
      line.split(/\t/)[1]
    end
  end
end
```

Note that this created list can be accessed in manifests via. the `$::cacti_graph_templates` variable.

## Refining the interface of your module through custom functions

Frequently, custom functions in Puppet are used for input validation. For example, the `stdlib` module comes with `validate_X` (e.g. `validate_bool`) functions for a lot of basic data types.

For example, here's a routine that validates (and munges) a passed-in IP address to a cacti device (so you can possibly remove it from the type itself):

```
module Puppet::Parser::Functions
  require ‘ipaddr’
  newfunction(:cacti_canonical_ip, :type => :rvalue) do |args|
    ip = args[0]
    begin
      IPAddr.new(ip)
    rescue ArgumentError
      raise “#{@resource.ref}: invalid IP address ‘#{ip}’”
    end
    ip.downcase
  end
end
```

and an example use-case:
```
define packt_cacti::device($ip) {
  $cli = ‘/usr/share/cacti/cli’
  $c_ip = cacti_canonical_ip(${ip})
  $options = “--description=‘${name}’ --ip=‘${c_ip}’”
  exec { “add-cacti-device-${name}”:
    command => “${cli}/add_device.php ${options}”,
    require => Class[cacti],
  }
}
```

Note that this is different for Puppet 4 -- info. about the differences are in Chapter 6.

## Making your module portable across platforms

The `Cacti` module created above is unfortunately specific to the Debian package.

If you want to get it to work for another platform, e.g. RHEL, you might need to do different stuff. One thing is that the `cli_path` variable is different.

You can pre-compute OS-specific parameters via. an additional `params` class, e.g.:
```
# …/packt_cacti/manifests/params.pp
class packt_cacti::params {
  case $osfamily {
    ‘Debian’: {
      $cli_path = ‘/usr/share/cacti/cli’
    }
    ‘RedHat’: {
      $cli_path = ‘/var/lib/cacti/cli’
    }
    default: {
      fail “the cacti module does not yet support the ${osfamily} platform”
    }
  }
}
```

We can access the variable as follows:
```
class packt_cacti::install {
  include pack_cacti::params
  file { ‘remove_device.php’:
    ensure => file,
    path => “${packt_cacti::params::cli_path}/remove_device.php’,
    source => ‘puppet:///modules/packt_cacti/cli/remove_device.php’,
    mode => ‘0755’,
  }
}
```

Notice that the `include pack_cacti::params` statement invokes the computation of the `cli_path` variable, which can then be referenced elsewhere in the `install` class declaration.

If you need additional, run-time calculation of parameters, add more variables to the `params` class.


The rest of the chapter talks about The Forge.

## Summary

Puppet development should be done in modules, with each module serving a very specific purpose.

Modules can contain manifests, files, Puppet plugins (custom resource types and providers, parser functions, or facts).
