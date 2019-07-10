# Minishift

[Minishift](https://github.com/minishift/minishift) helps run OpenShift locally by running a single-node OpenShift cluster inside a VM.

See [installation instructions](https://docs.okd.io/latest/minishift/getting-started/installing.html) and then the [quick start guide](https://docs.okd.io/latest/minishift/getting-started/quickstart.html). 

## minishift commands

```
minishift start
```

When the VM starts, minishift may update the dependent products like openshift, or the `oc` command line interface. The oc binary is located in the `~/.minishift/cache/oc/v3.11.0`

Minishift maintains a configuration file in $MINISHIFT_HOME/config/config.json. This file can be used to set commonly-used command line flags persistently for individual profiles.

### Configuration

To view all persistent configuration values in the global configuration file, you can use:
```
minishift config view --global
```

The Minishift VM is exposed to the host system with a host-only IP address that can be obtained with:

```
minishift ip
```

To interact with Minishift vm:

```
minishift ssh -- docker ps
```

```
minishift logs
```

### Accessing the Web Console:

```
minishift console --url
```

