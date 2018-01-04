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


