# Access Inheritance Use Cases

This is part 3 of the [implementation specific use cases comparison](./use-cases.md).

See also: [UCR 2.3 Collection resource inherited access](https://solid.github.io/authorization-panel/authorization-ucr/#uc-inheritance)


## Setup

The Weekly status collection is an `ldp:BasicContainer`, which contains other `ldp:BasicContainers`, one for each weekly meeting, into which further documents can be placed. 

```turtle
# Resource: </weekly-status/>
<.>
  a ldp:BasicContainer ;
  ldp:contains <2021-04-28/>, <2021-05-05/>, <2021-05-12/> .
```

We have the following hierarchy of resources (shown in more detail in the UCR):

```
</weekly-status/>
</weekly-status/2021-04-28/>
</weekly-status/2021-04-28/report.md>
</weekly-status/2021-05-05/>
</weekly-status/2021-05-05/report.md>
</weekly-status/2021-05-05/diagram.jpg>
</weekly-status/2021-05-12/>
```


## 1. Read access to a group on a collection of resources

We want to enable read access to all resources contained in `</weekly-status/>` for a group of people (`ex:Alice` & `ex:Bob`).

### ACP

Bob and Alice are part of the agent matcher `</acp/research#m1>`:

```turtle
# Resource: </acp/research>
<#m1>
  a acp:AgentMatcher ;
  acp:agent ex:Bob, ex:Alice .
```

The research policy `</acp/research#p1>` gives read access to all agents matched by `</acp/research#m1>`:

```turtle
# Resource: </acp/research>
<#p1>
  a acp:Policy ;
  acp:anyOf <#m1> ;
  acp:allow acl:Read .
```

The access control `</weekly-status/.acp#ac1>` applies to all resources contained by `</weekly-status/>` and via policy `</acp/research#p1>` enables read access for all agents matched by `</acp/research#m1>`:

```turtle
# Resource: </weekly-status/.acp>
<#ac1>
  a acp:AccessControl ;
  acp:apply </acp/research#p1> ; # applies the policy to the container resource </weekly-status/>
  acp:applyMembers </acp/research#p1> . # applies the policy to all resources contained by </weekly-status/>
```

Todo:  what is the ACR for 
`</weekly-status/2021-04-28/report.md>` and what information is in it?
   
### WAC

Bob and Alice are members of the research group `</groups/research#g1>`:

```turtle
# Resource: </groups/research>
<#g1>
  a vcard:Group ;
  vcard:hasMember ex:Bob ;
  vcard:hasMember ex:Alice .
```

The acl enabling read access to all resources contained by `</weekly-status/>` for all members of group `</groups/research#g1>` is:

```turtle
# Resource: </weekly-status/.acl>
<#authorization>
  a acl:Authorization ;
  acl:agentGroup </groups/research#g1> ;
  acl:default <.> ;
  acl:mode acl:Read .
```


## 2. Changing permissions to a subcollection

Bob wants to give Carol read/write access to the "2021-04-28" weekly status, and nothing more; that is, to the collection `</weekly-status/2021-04-28/>` and every resource it contains, but not to any other resource in the collection `</weekly-status/>`.

(What example from the UCR does this correspond to best?)

### ACP 

Carol is part of the agent matcher `</acp/research#m2>`:

```turtle
# Resource: </acp/research>
<#m2>
  a acp:AgentMatcher ;
  acp:agent ex:Carol .
```

The research policy `</acp/research#p2>` gives read access to all agents matched by `</acp/research#m2>`:

```turtle
# Resource: </acp/research>
<#p2>
  a acp:Policy ;
  acp:anyOf <#m2> ;
  acp:allow acl:Read, acl:Write .
```

The access control `</weekly-status/2021-04-28/.acp#ac1>` applies to all resources contained by `</weekly-status/2021-04-28/>` and, via policy `</acp/research#p2>`, enables read access for all agents matched by `</acp/research#m2>`:

```turtle
# Resource: </weekly-status/2021-04-28/.acp>
<#ac1>
  a acp:AccessControl ;
  acp:apply </acp/research#p1>, </acp/research#p2> ;
  acp:applyMembers </acp/research#p1>, </acp/research#p2> . # applies the policy to all resources contained by </weekly-status/2021-04-28/>
```

### WAC

To give Carol read and write access to the `</weekly-status/2021-04-28/>` collection and its content, Bob must create a new effective ACL resource, `</weekly-status/2021-04-28/.acl>`, which must express all permissions that are to govern access to `</weekly-status/2021-04-28/>`. In other words, to maintain the access permissions previously defined in `</weekly-status/.acl>`, Bob will need to include an authorization defining read access for the research group, along with an authorization defining read and write access for Carol, in the new `</weekly-status/2021-04-28/.acl>`.

```Turtle
# Resource: </weekly-status/2021-04-28/.acl>
<#authorization>
  a acl:Authorization ;
  acl:agentGroup </groups/research#g1> ;
  acl:default <.> ;
  acl:mode acl:Read .

<#new-authorization>
  a acl:Authorization ;
  acl:agent ex:Carol ;
  acl:default <.> ;
  acl:mode acl:Read, acl:Write .
```

### WAC+ relaxing acl:default

According to the [ACL ontology definition as of July 2021](https://github.com/solid/authorization-panel/pull/216#discussion_r665338497), the `acl:default` predicate is only effective in statements where the current container is the object; that is, the resource `</weekly-status/.acl>`, which is the direct effective access control list of `</weekly-status/>`, can only target that directory in statements using `acl:default`.

However, if WAC's use of `acl:default` were to be relaxed as described in [issue 191](https://github.com/solid/authorization-panel/issues/191), then one could rely on the effective access control resource discovery mechanism and augment the content of `</weekly-status/.acl>`:

```Turtle
# Resource: </weekly-status/.acl>
<#authorization>
  a acl:Authorization ;
  acl:agentGroup </groups/research#g1> ;
  acl:default <.> ;
  acl:mode acl:Read .

<#new-authorization>
  a acl:Authorization ;
  acl:agent ex:Carol ;
  acl:default <./2021-04-28/> ;
  acl:mode acl:Read, acl:Write .
```

See also: https://github.com/solid/authorization-panel/pull/216#discussion_r665230245


### WAC+ ac:imports

[WAC+:imports](https://github.com/solid/authorization-panel/issues/210) works just as WAC does above with `:default` working as shown. The advantage of `ac:imports` is that the resource  `</weekly-status/2021-04-28/.ac>` need only be set to contain:

```Turtle
# Resource: </weekly-status/2021-04-28/.ac>
<>
  ac:imports <../.acl> .

<#new-authorization>
  a acl:Authorization ;
  acl:agent ex:Carol ;
  acl:default <.> ;
  acl:mode acl:Read, acl:Write .
```

In other words, this removes the need to duplicate the `</weekly-status/.acl#authorization>` rule, so any future edits to that rule need only be done in one place.
