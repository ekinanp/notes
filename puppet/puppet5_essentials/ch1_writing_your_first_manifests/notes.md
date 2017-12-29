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
* The property that changed (`enable`) is referenced with a whole path, which is `/Stage[main]/Main/Service[puppet]/enable`. Properties are generally of the form `/<environment?>/<class>/<resource>/<property>`. `/Stage[main]` is beyond the scope of the book (I'm guessing it's the environment?). Note that the default class is `Main`, while remember that a resource is uniquely referenced as `Type[name]`.
* The rest of the log line is the type of change Puppet saw fit to apply (for example, `created` to indicate that the resource is newly added to the managed system; `changed <old_val> to <new_val>` representing the property being in-sync with its value in the catalog, etc.

## Dry testing your manifest

Use the `--noop` option.

## Using variables

Variable assignment works like in most scripting languages. Variable names are always prefixed with the `$` sign, for example:
* `$download_server = 'img2.example.net'`
* `$url = "https://${download_server}/pkg/example_source.tar.gz"`


