## Putting it all together - collecting system information with Facter

Puppet uses Facter to get details about the specific machine on which it is run (for agents, this means getting info. about the agent's machine). For example, a Puppet manifest that needs to do something based on the # of processes on the machine might be able to do something like:
```
if $::processors['count'] > 4 { ... }
```

Facter variables are called `facts`, and `processors` is one such fact. All fact names are available in the manifests as variables. Fact values are gathered by the agent and sent to the master, which will use these facts to compile a catalog. Note that the "::" prefix is preferred b/c it shows you are referencing a variable from Facter -- Facter variables are put into the master's top scope.

## Extending Facter with custom facts

Adding custom facts is straightforward. The general syntax is (in a ruby file):
```
Facter.add(:<factname>) do
  // Code to calculate the fact
  // The fact's value is whatever is returned here
end
```

The fact can be referenced as `$::<factname>` in a Puppet manifest.

If you want to confine custom facts to machines that have a specific set of satisfying facts, you can do something like:
```
Facter.add(:<factname>) do
  confine :<confining_fact> => <values>
  setcode do
    // Code here to calculate the fact
  end
end
```

You can also use other facts via. the syntax `Facter.value(:<other_fact>)` in the custom fact's ruby code.

## Simplifying things using external facts

You can also use external facts if you do not want to write Ruby code. An external fact has the following distinctions for regular custom facts:
* External facts are produced by standalone executables or files with static data, which the agent must find in `/etc/puppetlabs/facter/facts.d/`
* The data is not just a string value, but an arbitrary number of `key=value` pairs.

Note the key/value pairs can be in YAML or JSON format. You can also use your own implementation (e.g. like a shell script), it doesn't matter so long as its output is in the right format (which is only ini).

## Understanding the type system

The agent performs its work in discreet `transactions`. A transaction is started under any of the following circumstances:
* The background agent process activates and checks in on the master
* An agent process is started with the `--onetime` or `--test` options
* A local manifest is compiled using `puppet apply`

The transaction always passes several stages, which are:
1. Gathering fact values to form the actual catalog request 
2. Receiving the compiled catalog from the master
3. Prefecthing of current resource states
4. Validation of the catalog's content
5. Synchronization of the system with the `property` values from the catalog.

The resource types become important during compilation and then through the rest of the agent transaction.

## The resource type's life cycle on the agent side

Once compilation has passed, the master hands out the catalog and the agent enters the catalog validation phase. Each resource type defines some Ruby methods that ensure the passed values make sense. This can happen on two levels of granularity:
* Each attribute validating its input value. For example, the `ssh_authorized_key` resource type will fail validation if its `key` value contains a whitespace character.
* The resource as a whole being checked for consistency. In the `cron` type, for example, Puppet makes sure that the time fields make sense together.

During this phase, input values are also transformed to more suitable internal representations (like an AST) -- this is called a `munge` action. Examples include removing leading and trailing whitespace from string values, the conversion of array values to a string format (e.g. comma-separated lists, or a colon for search paths).

Next is the prefetching phase. For some resources, the agent can get an internal list of resource instances present on that system (these are referred to as enumerable types). The `package` type is a good example of this, b/c Puppet can just invoke the appropriate package manager to get a list of all the available packages on the system.

Finally, the agent begins walking through its internal graph of interdependent resources where each resource is brought in sync. if necessary. This happens separately for each individual property.

The `ensure` property is a notable exception. It's expected to manage all other properties on its own when a resource is changed from `absent` to `present` (resource is getting newly created).

Each attributes are handled independently, defining its own methods for the different phases. There are a lot of hooks, which allows for a lot of flexibility for the resource type author to add to the model.

Note that the entire validation process is performed by the agent, not by the master.

## Command execution control with providers

Puppet providers are the concrete implementations that bring a specific resource to life. A good example is package management, which has a lot of different providers (`yum`, `emerge`, `zypper`, `apt`, etc.). Most tools give you the same set of operations -- installing and uninstalling a package, and updating a package to a specific version. Some features are unique to specific tools. Some tools might not have some features (e.g. `dpkg` you cannot update anything).

Due to this diversity, Puppet providers can opt for `features`, which are resource-type specific. All providers for a type can support one or more of the same group of features. For the `package` type, we have features like `versionable`, `purgeable`, `holdable`, etc. For example
```
package { 'haproxy':
  ensure => 'purged'
}
```
The above package resource will not make sense on `rpm`, because `rpm` has no notion of a `purged` state -- the `purgeable` feature is missing from the `rpm` provider. Trying to use an unsupported feature will usually produce an error message.

## Summarizing types and providers

Providers implement the actual management actions that Puppet is supposed to perform. They map all necessary sync. steps to commands and system actions. You can confine providers to work with agents that meet a particular fact (e.g. package providers to work with specific OSes), require that the agent machine has certain commands present in it that the provider needs to do its work, etc. At a high-level, providers are Puppet's roadmap for implementing a specific Type, the Type is like the abstract base class.

