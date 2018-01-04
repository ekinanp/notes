Note that the package from Puppet Inc. bundles all required software components and installs them to `/opt/puppetlabs`.

```
puppet apply <manifest-file>
```

Apply the specified manifest-file on the target machine. You can pass in the `--noop` option to make Puppet refrain from taking any action on unsynced resources (to ensure that nothing breaks on your system). It will instead give you a preview of what will happen if the manifest is applied without the `--noop` switch.

```
puppet resource <resource-type> <resource-name>
```

Read the existing system state of the resource with type `<resource-type>` and name/title `<resource-name>`.

```
puppet <agent|master> --configprint all
```

Displays all available settings and their values on the master or the agent.

```
puppet config set
```

Lets you set specific configuration entries.

```
puppet cert --list
```

Shows all of the unsigned certificates that are available.

```
puppet cert --sign <agent_name>
```

Signs the certificate for the specified agent.

```
puppet cert revoke <agent>
```

Revokes the `<agent>'s` certificate (decommissions it).

```
puppet module install <module-name>
```

This installs the module in the active environment.
