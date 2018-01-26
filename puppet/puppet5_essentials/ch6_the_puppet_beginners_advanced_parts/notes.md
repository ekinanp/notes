## Prelude

This presents some useful techniques to make it easier to write Puppet code for beginners. These techniques are:
* Building dynamic configuration files 
* Managing file snippets
* Using virtual resources
* Cross-node configuration with exported resources
* Setting defaults for resource parameters
* Avoiding antipatterns

## Building dynamic configuration files

This part of the chapter talks about templates, which are ways to create "similar" looking configuration files. Already know how templates work, so can just reference it when I need to.

Book talks about some things to keep in mind, performance wise, when using templates.

## Managing file snippets

This section talks about how to manage configuration files where you're not able to manage the whole file, or the file is constructed from different subclasses.

Puppet gives you several ways to do this:
* Single line
* Single entry in a section
* Building from multiple snippets
* Other resource types

This section introduces several custom resource types like the `file_line`, `ini_setting`, and `concat` resources to make stuff like inserting a line in a file, modifying an ini configuration file, and creating a configuration file from snippets of multiple classes,  easier to do.

## Using virtual resources

Recall that resources must be unique -- any resource is declared at most once in a manifest. This can be hard to manage when you have a ton of modules that each share all kinds of resources -- for example, it is entirely possible that a `package` resource is declared in one module and also in another module.

You can do component classes, e.g.
```
class yumrepos::team_ninja_stable {
  yumrepo { 'team_ninja_stable':
    ensure => present,
    ...
  }
}
```

Virtual resources let you declare resources that are not yet realized -- the idea is that you need to explicitly `realize` the resource in the manifest itself. For example, for the yumrepo stuff we can do:
```
class yumrepos::all {
  @yumrepo { 'tem_ninja_stable':
    ensure => present,
    tag => 'stable',
  }
  @yumrepo { 'team_wizard_experimantel':
    ensure => present,
    tag => 'experimental',
  }
}
```

The `@` prefix marks the given resource as virtual. Now we can realize the virtual resource via., for example, the following code (by including the `yumrepos::all` class):
```
realize(Yumrepo['team_ninja_stable'])
realize(Yumrepo['team_wizard_experimental'])
package { 'doombunnies':
  ensure => installed,
  require => [
    Yumrepo['team_ninja_stable'],
    Yumrepo['team_wizard_experimental'],
  ],
}
```

Note that if another manifest has `realize(Yumrepo['team_ninja_stable'])`, things will still work fine because that resource will only be realized once. For example, the following code would work:
```
@Type { 'title':
  ensure => present
}

realize(Type['title'])
realize(Type['title'])
```

## Realizing resources more flexibly using collectors

Another way to realize virtual resources is via. the `collector`, whose syntax is of the form
```
Yumrepo<| title == 'team_ninja_stable' |>
```

^ which really says to realize all *virtual* `Yumrepo` resources with the title `team_ninja_stable`.

As another example,
```
User<| |>
```
^ This expression says to realize all virtual `User` resources.

You can tag resources too, via. the `tag` metaparameter. By default, resources are tagged with the name of the declaring class, the containing module, and some other useful meta information. With tags, you can do something like:
```
User<| tag == '<tag-name>' |>
```

to realize User resources with the tag of `<tag-name>`. As another example,
```
@user { 'felix':
  ensure => present,
  groups => [ 'power', 'sys' ],
}
User<| groups == 'sys' |>
```

which realizes all `User` resources in the group `sys`. Note that Virtual Resources do not leak into other manifests.

## Cross-node configuration with exported resources

You can use Puppet to export common resources across nodes. For example, imagine that the manifest of `node A` contains one or more purely virtual resources. Other nodes like `B` or `C` can import some or all of these resources so that those resources become part of the catalog of those remote nodes.

The way to do this is as follows:
```
@@file { 'my-app-psk':
  ensure => file,
  path => '/etc/my-app/psk',
  content => 'nwNFgzsn9n3sDfnFANfoinaAEF',
  tag => 'cluster02',
}
```

importing manifests collect these resources with an expression:
```
File <<| tag == 'cluster02' |>>
```

You have to configure the master to support exported resources.

SKIPPED THE REST OF THIS CHAPTER -- TOO TIRED TO READ
