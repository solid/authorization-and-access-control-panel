# Resource Access Use Cases

This is part 1 of the [implementation specific use cases comparison](./use-cases.md).

See also: [UCR 2.1 Resource access](https://solid.github.io/authorization-panel/authorization-ucr/#uc-basic)


## Setup

We have the following hierarchy of resources:

```
</resume>
</recommendations>
```


## 1. Change permissions

Alice can modify access to the `</resume>` and `</recommendations>` resources.

### ACP

The access controls `</resume.acp#ac1>` and `</recommendations.acp#ac1>` apply respectively to the `</resume>` and `</recommendations>` resources and via policy `</.acp#p1>` enable read and write over themselves for Alice who is matched by the agent matcher `</.acp#m1>`:

```turtle
# Resource: </resume.acp>
<#ac1>
  a acp:AccessControl ;
  acp:access </.acp#p1> .
```

```turtle
# Resource: </recommendations.acp>
<#ac1>
  a acp:AccessControl ;
  acp:access </.acp#p1> .
```

```turtle
# Resource: </.acp>
<#p1>
  acp:anyOf <#m1> ;
  acp:allow acl:Read, acl:Write .

<#m1>
  acp:agent ex:Alice .
```

Note: `acp:access` points to policies applied to the access control, which means the permissions applied to an access control are granular; in other words, could be read-only, for example.

### WAC

```turtle
ex:AccessControl1
  a acl:Authorization ;
  acl:agent ex:Alice ;
  acl:accessTo ex:Resource1 ;
  acl:mode acl:Control .
```

Note: `acl:Control` allows read-write operations on the ACL resource.


## 3. Read-write access

Bob can read and modify the `</resume>` resource.

### ACP

The access control `</resume.acp#ac1>` is modified to enable via policy `</resume.acp#p1>` read and write access for Bob who is matched by the agent matcher `</resume.acp#m1>`:

```turtle
# Resource: </resume.acp>
<#ac1>
  a acp:AccessControl ;
  acp:access </.acp#p1> ;
  acp:apply <#p1> .

<#p1>
  a acp:Policy ;
  acp:anyOf <#m1> ;
  acp:allow acl:Read, acl:Write .

<#m1>
  a acp:AgentMatcher ;
  acp:agent ex:Bob .
```


### WAC

```turtle
ex:AccessControl1
  a acl:Authorization ;
  acl:agent ex:Bob ;
  acl:accessTo ex:Resource1 ;
  acl:mode acl:Read, acl:Write .
```


## 8. Group access

A group of people can Read multiple resources.

### ACP

A group of people is defined in matcher `</.acp#m1>`:

```turtle
# Resource: </.acp>
<#m1>
  a acp:AgentMatcher ;
  acp:agent ex:Alice, ex:Bob .
```

A policy that allows read is defined in policy `</.acp#p1>`:

```turtle
# Resource: </.acp>
<#p1>
  a acp:Policy ;
  acp:anyOf <#m1> ;
  acp:allow acl:Read .
```

The access controls `</resume.acp#ac1>` and `</recommendations.acp#ac1>` respectively apply to the `</resume>` and `</recommendations>` resources and via policy `</.acp#p1>` enable read for everyone matched by the agent matcher `</.acp#m1>`:

```turtle
# Resource: </resume.acp>
<#ac1>
  a acp:AccessControl ;
  acp:apply </.acp#p1> .
```

```turtle
# Resource: </recommendations.acp>
<#ac1>
  a acp:AccessControl ;
  acp:apply </.acp#p1> .
```

Note: Here the group of people: Alice & Bob; is defined in `</.acp#m1>` which can be reused. In this specific case, since we want to allow read access to the group on both resources, `</.acp#p1>` is being reused.

## WAC

```turtle
ex:AccessControl1
  a acl:Authorization ;
  acl:agentGroup ex:Group1 ;
  acl:accessTo ex:Resource1 ;
  acl:mode acl:Read .

ex:AccessControl2
  a acl:Authorization ;
  acl:agentGroup ex:Group1 ;
  acl:accessTo ex:Resource2 ;
  acl:mode acl:Read .

ex:Group1
  vcard:hasMember ex:Alice, ex:Bob .
```
