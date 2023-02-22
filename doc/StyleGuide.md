# Reference Policy Style Guide

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and updated in
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

## Source File Organization

The policy source files (*.cas) SHOULD be organized in the following
top-level sections:

1. License
2. Booleans and tunables
3. APIs
4. Virtuals
5. Domains and resources
6. Extends
7. Modules

### License Section

The license MUST be specified as the first line in the file.  It MUST be
specified using the Software Package Data Exchange (SPDX) license identifier,
for example,

```// SPDX-License-Identifier: GPL-2.0```

### Booleans and Tunables Section

This section consists of Boolean and tunable definitions.  It SHOULD
be sorted in order of the Boolean or tunable name.
It MAY be sorted further by having Booleans before tunables.

### APIs Section

This section consists API definitions.  It SHOULD be sorted in
alphabetical order by API name.

### Virtuals Section

This section consists of virtual domain and resource definitions.  It
SHOULD be sorted in alphabetical order by virtual name.

### Domains and Resources Section

This section consists of domain and resource definitions.  It
SHOULD be sorted in alphabetical order by domain/resource name,
ignoring the `_t` suffix.

### Extends Section

This section consists of `extend`s for domains/resources in other modules.
It SHOULD be sorted in alphabetical order by domain/resource name.

### Module Section

If a source file consists of one module, it SHOULD NOT have any
module definitions, as the compiler will implicitly make all file contents
into a single module in this case.

A source file MAY create multiple modules from the items within. If done,
there SHOULD be a top-level module that has the same name as the file.
This top-level module MAY contain all other modules defined in the file.
The section SHOULD be sorted in alphabetical order by module name.

## Object Organization

### Booleans and Tunables

Boolean and tunable blocks MUST have one document string line at a minimum.

### APIs

APIs SHOULD follow the style described in the functions
section below.

### Virtuals

Virtuals SHOULD follow the style described in the domains
and resources section below.

### Domains and Resources (Types)

Domain/resource blocks MUST have one document string line at a minimum.

These SHOULD be organized into the following sections:

1. Nested declarations
2. Labeling
3. Rules
4. Functions

#### Nested Declarations

This section consists of nested domain and resource definitions
and `extend`s .  It SHOULD be sorted in alphabetical order by
domain/resource name.

#### Labeling

This section consists of labeling statments, such as `file_context`.

#### Rules

This section consists of rules and function calls. These SHOULD be
sorted in the following order:

1. "Self" rules: Rules that have `self` self as the target.
2. "This" rules: Rules that reference a nested type or function
   call using `this`.
3. Remaining rules/function calls on other objects.
4. Conditional blocks
5. Optional blocks

For both #2 and #3, rules/function calls with the same target SHOULD be grouped.  Each of these groups SHOULD be sorted in alphabetical order by the
target name.

Exception: filetrans rules should be grouped with the new type (2nd
parameter). For example, the following line should be grouped with `this.tmp` and `foo_runtime_t` rules, respectively:

```
tmp_t.filetrans(this, this.tmp, file);
runtime_t.filetrans(this, foo_runtime_t, file);
```

Within conditional and optional blocks, the rule styles applies.

#### Functions

This section consists of function definitions and their implementations.
Functions SHOULD be sorted by one or more of the following methods:

* By similarity, e.g. signal functions, shm functions, sem functions, etc.
* Increasing access, e.g. getattr, read, rw, manage.
* A grouping that is logical in the context of the program.
* Alphabetical order.

Function implementations MUST have at least one line of documentation strings.
They MAY follow rule style described above.

### Extends

The `extend` block organization follows the style rules for whichever
type of object is being extended.

### Modules

Module blocks MAY be sorted in alphabetical order by object name, ignoring any
suffixes, such as `_t`.

## Example

```
// SPDX-License-Identifier: GPL-2.0

let mydomain_lockdown = false;
/// Set to true to lock out mytool privileged access.

resource general_conf_t inherits configfile {
    /// A general configuration file.
    file_context("/etc/info(/.*)?", any);
}

domain mydomain_t inherits daemon {
    /// The mytool service.
    /// This program does foo and bar.

    //
    // Declarations
    //
    resource conf inherits configfile {
        /// mytool configuration files.
        file_context("/etc/mytool(/.*)?", any);
    }

    @alias(mydomain_exec_t)
    extend exec {
        file_context("/usr/sbin/mytool", file);
    }

    resource tmp inherits tmpfile {
        /// mytool temporary files.
    }

    //
    // Rules
    //
    allow(this, self, capability, dac_override);

    this.conf.read();

    this.tmp.manage();
    tmp_t.filetrans(this, this.tmp, [ dir file ]);

    bin_t.exec();

    etc_t.read();
    // function calls and raw rules to same targets
    // are grouped together.
    dontaudit(this, etc_t, file, audit_access);

    network.tcp_client();

    network_port.tcp_connect();

    if (!mydomain_lockdown) {
        allow(this, self, capability, sys_admin);
    }

    optional {
        init_t.dbus_send();
        systemd_dbus_t.client();
    }

    //
    // Functions
    //
    fn notify_ready(domain source) {
        /// Notify mytool to begin processing.
        // SIGHUP
        allow(source, this, process, signal);
    }

    fn unix_stream_connect(source domain) {
        /// Connect to this daemon over a unix stream socket.
        allow(source, this, unix_stream_socket, [ getattr connectto ]);
        this.tmp<common_named_socket>.rw(source);
    }
}

//
// Modules
//
module mymodule_base {
    resource general_conf_t;
}

module mymodule_daemon {
    domain mymodule_t;
}

module mymodule {
    module mymodule_base;
    module mymodule_daemon;
}
```
