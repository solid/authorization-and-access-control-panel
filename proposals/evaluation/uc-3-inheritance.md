# Access Inheritance Use Cases

This is part 3 of the [implementation specific use cases comparison](./use-cases.md).

See also: [UCR 2.3 Collection resource inherited access](https://solid.github.io/authorization-panel/authorization-ucr/#uc-inheritance)


## 1. Read access to a group on a collection of resources

### Setup

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

The access control `</weekly-status/.acp#authorization>` applies to all resources contained by `</weekly-status/>` and via policy `</acp/research#p1>` enables read access for all agents matched by `</acp/research#m1>`:

```turtle
# Resource: </weekly-status/.acp>
<#authorization>
  a acp:AccessControl ;
  acp:apply </acp/research#p1> ;
  acp:applyMembers </acp/research#p1> . # applies the policy to all resources contained by </weekly-status/>
```

Todo:  what is the ACR for 
`</weekly-status/2021-04-28/report.md>` and what information is in it?
   
### WAC

Bob and Alice are members of the research group `</groups/research#g1>`:

```turtle
# Resource: </groups/research>
<#g1>
    a                vcard:Group ;
    vcard:hasMember  ex:Bob ;
    vcard:hasMember  ex:Alice .
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

Bob wants to give Carol read/write access but only to the collection `</weekly-status/2021-04-28/>`.

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

The access control `</weekly-status/2021-04-28/.acp#authorization>` applies to all resources contained by `</weekly-status/2021-04-28/>` and via policy `</acp/research#p2>` enables read access for all agents matched by `</acp/research#m2>`:

```turtle
# Resource: </weekly-status/2021-04-28/.acp>
<#authorization>
  a acp:AccessControl ;
  acp:apply </acp/research#p1>, </acp/research#p2> ;
  acp:applyMembers </acp/research#p1>, </acp/research#p2> . # applies the policy to all resources contained by </weekly-status/2021-04-28/>
```

### WAC

To change the access rules to the `</weekly-status/2021-04-28/>` collection Bob must edit `</weekly-status/2021-04-28/.acl>`. But doing so will disable the default access control rule located in `</weekly-status/.acl>` . So the content of the PUT must contain the default rule and the new ones.

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

We could also use WAC's effective access control resource discovery mechanism and augment the content of `</weekly-status/.acl>`.

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

### WAC+ac:imports

[WAC+:imports](https://github.com/solid/authorization-panel/issues/210) works just as WAC does above with `:default` working as shown. The advantage of `ac:imports` is that the resource  `</weekly-status/2021-04-28/.ac>` need only be set to contain:

```Turtle
# Resource: </weekly-status/2021-04-28/.ac>
<> ac:imports <../.acl> .

<#new-authorization>
  a acl:Authorization ;
  acl:agent ex:Carol ;
  acl:default <.> ;
  acl:mode acl:Read, acl:Write .
```

The advantage of this is that it removes the needed duplication of the `</weekly-status/.acl#authorization>` rule, so that if that needs to be edited it can be done so in one place.