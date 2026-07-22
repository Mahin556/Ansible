This is an **NFS export rule** — it defines how a directory is shared over the network via NFS (Network File System). It lives in `/etc/exports` on the NFS server.

## Breaking it down piece by piece

```
/share   *(rw,sync,no_subtree_check)
  ↑       ↑  ↑    ↑    ↑
  │       │  │    │    └── option 3
  │       │  │    └─────── option 2
  │       │  └──────────── option 1
  │       └──────────────── who can access
  └──────────────────────── what is being shared
```

### `/share`
The directory on the server being exported (shared). Any client that mounts this will see the contents of `/share` on the server.

### `*`
**Who can access it.** The `*` is a wildcard meaning **any host, any IP address, anywhere** can mount this share. This is the most permissive setting possible.

You can restrict this:
```
/share  192.168.1.0/24(rw,sync,no_subtree_check)   # only this subnet
/share  192.168.1.100(rw,sync,no_subtree_check)     # only one specific IP
/share  client.example.com(rw,sync,no_subtree_check) # only this hostname
```

### `rw`
**Read-Write** access. Clients can both read AND write files.

The alternative is `ro` (read-only):
```
/share  *(ro,sync,no_subtree_check)   # clients can only read, never write
```

### `sync`
The server **waits for all writes to be committed to disk** before replying to the client that the write succeeded. Slower but safer — if the server crashes mid-write, data is not lost.

The alternative `async` replies immediately without waiting for disk commit — faster but risks data corruption on crash.

### `no_subtree_check`
When a client requests a file, NFS normally checks the entire directory tree to confirm the file is still inside the exported directory (subtree check). This can cause problems when files are renamed while open.

`no_subtree_check` disables this verification — **recommended in most cases** because it improves reliability and is now the default on most modern NFS setups.

---

## Complete options reference

| Option | Opposite | Meaning |
|---|---|---|
| `rw` | `ro` | Read-write vs read-only |
| `sync` | `async` | Wait for disk commit vs reply immediately |
| `no_subtree_check` | `subtree_check` | Skip tree verification (recommended) |
| `no_root_squash` | `root_squash` | Allow remote root to act as root (dangerous) |
| `root_squash` | `no_root_squash` | Map remote root to anonymous user (safe default) |
| `all_squash` | — | Map ALL users to anonymous (most restrictive) |
| `anonuid=1000` | — | Set which local UID anonymous maps to |
| `anongid=1000` | — | Set which local GID anonymous maps to |

---

## After editing `/etc/exports`, apply changes:

```bash
# Reload exports without restarting NFS:
exportfs -ra

# Verify what is currently exported:
exportfs -v
# /share    <world>(rw,wdelay,no_subtree_check,no_root_squash)

# Check from client side what a server is exporting:
showmount -e <server-ip>
# Export list for 192.168.1.10:
# /share *
```

---

## Security concern with this specific rule

```
/share *(rw,sync,no_subtree_check)
```

The `*` combined with `rw` means **the entire internet can mount and write to /share** if your firewall allows port 2049 (NFS). This is almost never what you want in production.

Minimum recommended hardening:
```bash
# 1. Restrict to your network only:
/share 192.168.1.0/24(rw,sync,no_subtree_check)

# 2. Enable root_squash (already the default but be explicit):
/share 192.168.1.0/24(rw,sync,no_subtree_check,root_squash)

# 3. Block NFS ports at firewall for everything except trusted hosts:
ufw allow from 192.168.1.0/24 to any port 2049
ufw deny 2049
```