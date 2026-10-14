# couchdb

[CouchDB](https://couchdb.apache.org) template for Runway. Document database
with an HTTP/JSON API, the Fauxton web UI, and multi-master replication.
Syncs with PouchDB in the browser.

Single node with persistence on a volume. CouchDB 3.5.3.

## Deploy

WIP

## Inputs

| Input | Default | Notes |
| --- | --- | --- |
| `username` | `admin` | Server admin for the API and Fauxton |

## Connecting

This one **is** routed: it gets a public HTTPS URL, so apps, PouchDB, and the
web UI can reach it. Every request except `/_up` needs a login.

The credentials are in the app config:

```sh
runway app config ls --app my-couch
```

- Web UI: `https://my-couch.example.com/_utils/`, log in with `<username>`
  and `COUCHDB_PASSWORD`.
- API:

  ```sh
  curl -u <username>:<COUCHDB_PASSWORD> https://my-couch.example.com/_all_dbs
  ```

New databases are admin-only. Set `_security` on a database to give other
users access, and do not ship the admin password to browsers.

## Notes

- **Config is rebuilt on every start.** `/opt/couchdb/etc` is not on the
  volume, so settings changed in Fauxton or over `/_node/_local/_config` are
  lost on restart. This includes the admin password — change
  `COUCHDB_PASSWORD` in the app config instead. It applies on the next
  start.
- **The template wraps the start command.** It writes
  `local.d/00-runway.ini` (single-node mode and a fixed `COUCHDB_UUID`), then
  runs the image entrypoint, which adds the admin and `COUCHDB_SECRET`.
- `single_node = true` creates `_users` and `_replicator` on first start.
- Do not change `COUCHDB_UUID`: replication checkpoints are tied to it, and
  every replication would start over.
- No lockdown: the start script writes to `/opt/couchdb/etc`.
- The `-nouveau` image variant (full-text search) needs a separate Nouveau
  server and is not used here.

## License

CouchDB is Apache-2.0. This template is maintained by Runway.
