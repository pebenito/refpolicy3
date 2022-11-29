# SELinux Reference Policy 3

This is pre-alpha quality.  It is using the Cascade language which is also
in development.  As such, syntax, structure, and API may change at any time.
This is developing in advance of the compiler development, so you should
not expect this to compile.

**Do not attempt to run this on any system that you care about.**

## Contributing

Please send any discussion to the refpolicy mail list.  If you would like to
contribute, pull requests on GitHub are strongly preferred, but patches on
the refpolicy mail list are also accepted.

## Development Notes

### Multi-call Lines

Lines that have multiple calls are to handle parent directory access:

```text
init_script_t.runtime.rw(); runtime_t.list(); var_t.list();
```

We are looking at a Cascade language feature to better handle parent
directory access.  The calls are on a single line so the parent
directory access is easier to find when it is time to drop them.

### Domain transitions

We are looking at Cascade language feature to handle file descriptor,
object, and role access across domain transitions.  The goal is to
remove any need to specify these in explicit lines.

For example, to do `iptables-save > /root/rules.txt` requires these
rules:

```text
domain iptables_t ... {
    ...
    user_home_t.append_inherited();
}

domain sysadm_t ... {
    iptables_t.domtrans();
}
```

Our goal is that each domain has a list of stdin/stdout access and the
compiler will accumulate this access across domain trasitions, so the
`iptables_t` lines won't be necessary.

### Huge page access

Access to hugetlbfs_t dirs/files needs to be transitioned to
private types, e.g. foo_t.hugepage.
