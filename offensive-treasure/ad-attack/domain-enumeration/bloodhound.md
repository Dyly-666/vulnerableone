---
description: 'Extra: https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/'
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/domain-enumeration/bloodhound
---

# Bloodhound

## Start Neo4j

Open a web browser and navigate to **http://localhost:7474/**

```powershell
C:\Tools\neo4j\bin>neo4j.bat console
2021-05-11 10:03:21.143+0000 INFO  Starting...
2021-05-11 10:03:28.065+0000 INFO  ======== Neo4j 4.2.3 ========
2021-05-11 10:03:32.143+0000 INFO  Performing postInitialization step for component 'security-users' with version 2 and status CURRENT
2021-05-11 10:03:32.143+0000 INFO  Updating the initial password in component 'security-users'
2021-05-11 10:03:33.128+0000 INFO  Bolt enabled on localhost:7687.
2021-05-11 10:03:36.096+0000 INFO  Remote interface available at http://localhost:7474/
2021-05-11 10:03:36.096+0000 INFO  Started.
```

## SharpHound.exe

SharpHound has a number of different collection methods (all documented on the repository):

* **Default** - Performs group membership collection, domain trust collection, local group collection, session collection, ACL collection, object property collection, and SPN target collection
* **Group** - Performs group membership collection
* **LocalAdmin** - Performs local admin collection
* **RDP** - Performs Remote Desktop Users collection
* **DCOM** - Performs Distributed COM Users collection
* **PSRemote** - Performs Remote Management Users collection
* **GPOLocalGroup** - Performs local admin collection using Group Policy Objects
* **Session** - Performs session collection
* **ComputerOnly** - Performs local admin, RDP, DCOM and session collection
* **LoggedOn** - Performs privileged session collection (requires admin rights on target systems)
* **Trusts** - Performs domain trust enumeration
* **ACL** - Performs collection of ACLs
* **Container** - Performs collection of Containers
* **DcOnly** - Performs collection using LDAP only. Includes Group, Trusts, ACL, ObjectProps, Container, and GPOLocalGroup.
* **ObjectProps** - Performs Object Properties collection for properties such as LastLogon or PwdLastSet
* **All** - Performs all Collection Methods except GPOLocalGroup.

```powershell
C:\> SharpHound.exe -c DcOnly
C:\> SharpHound.exe -c DcOnly -d vulnableone.local
C:\> SharpHound.exe --CollectionMethod All,GPOLocalGroup --Domain vulnableone.local
C:\> SharpHound.exe --CollectionMethod All --Domain vulnableone.local --LdapUsername khan.chanthou --LdapPassword Password123 --ZipFileName output.zip
```

## SharpHound.ps1

```powershell
Invoke-BloodHound -CollectionMethod All
```

## Bloodhound Query

### Service Principal Name (SPN)

```cypher
MATCH (u:User {hasspn:true}) RETURN u
```

### Shortest Paths from Kerberoastable Users

```cypher
MATCH (u:User {hasspn:true}), (c:Computer), p=shortestPath((u)-[*1..]->(c)) RETURN p
```

### Unconstrained Delegation

```cypher
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c
```

### AllowedToDelegate to other computers

```cypher
MATCH (c:Computer), (t:Computer), p=((c)-[:AllowedToDelegate]->(t)) RETURN p
```

### ASREP Roasting

```cypher
MATCH (u:User {dontreqpreauth:true}) RETURN u
```

### Constrained Delegation

```cypher
MATCH (c:Computer), (t:Computer), p=((c)-[:AllowedToDelegate]->(t)) RETURN p
```

### GPO Query

```cypher
MATCH (gr:Group), (gp:GPO), p=((gr)-[:GenericWrite]->(gp)) RETURN p
```

### Discretionary Access Control Lists

```cypher
MATCH (g1:Group), (g2:Group), p=((g1)-[:GenericAll]->(g2)) RETURN p
```

```cypher
MATCH (g1:Group {name:"IT_Support"}), (g2:Group), p=((g1)-[:GenericAll]->(g2)) RETURN p
```

### Potential MS SQL Admins

```cypher
MATCH p=(u:User)-[:SQLAdmin]->(c:Computer) RETURN p
```

### LAPS

```cypher
MATCH (c:Computer {haslaps: true}) RETURN c
```

```cypher
MATCH p=(g:Group)-[:ReadLAPSPassword]->(c:Computer) RETURN p
```
