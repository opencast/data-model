---
sidebar_position: 1
---

# ACL (access control list)

ACLs control access to Opencast entities.
An ACL is simply a list of `role` + `action` pairs.
An entry gives users that have that particular `role` the permission to perform the specified `action` on the entity.
Both, `role` and `action` have the [type `Label`](./types).<sup>(1?)</sup>

There are two special actions recognized by Opencast.
Other actions can be used for custom purposes by external applications.
- `read`: generally, gives read access to an entity
- `write`: generally, gives write access to an entity (changing or deleting it)

*Impl note*: `read` and `write` roles should likely be stored in a way that allows for fast filtering, e.g. in a `read_roles` DB column that has a DB index.


---

:::danger[Open questions]

- (1?) Is it fine to restrict roles and actions like that? Or can we restrict it even more?

:::
