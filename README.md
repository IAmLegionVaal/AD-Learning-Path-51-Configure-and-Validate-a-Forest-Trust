# AD Learning Path 51 — Configure and Validate a Forest Trust

## Goal
Create cross-forest DNS resolution and a controlled trust between `corp.lab` and `partner.lab`, then prove that only an explicitly authorized partner identity can authenticate to and access one dedicated CORP lab resource.

## Prerequisites
- Both forests healthy
- Routed lab connectivity
- DNS and time working in both forests
- Trust direction documented
- Separate authorized administrators
- A dedicated CORP test server and resource
- A dedicated PARTNER test group

## Steps
1. In DNS Manager for `corp.lab`, create a conditional forwarder for `partner.lab` that points to the PARTNER DNS servers.
2. In `partner.lab`, create a conditional forwarder for `corp.lab` that points to the CORP DNS servers.
3. Test host and SRV record resolution in both directions.
4. Open **Active Directory Domains and Trusts** on the forest-root domain.
5. Create only the one-way or two-way forest trust required by the approved lab scenario.
6. Select **Selective authentication** rather than forest-wide authentication.
7. Validate the trust from each forest.
8. Create a normal PARTNER test group and add only the approved test user.
9. On the dedicated CORP target computer object, grant that PARTNER group the **Allowed to authenticate** right.
10. Grant the same group only the required NTFS/share/application permission on the dedicated test resource.
11. Verify the approved user can authenticate to and access the intended resource.
12. Verify the same user cannot authenticate to an unrelated CORP server and cannot access unrelated resources.
13. Verify a PARTNER user outside the approved group is denied.

> Selective authentication is incomplete until the foreign principal has the **Allowed to authenticate** right on the specific target computer object. Resource permissions alone do not grant that right.

## Validation

```powershell
$CorpDomain = 'corp.lab'
$PartnerDomain = 'partner.lab'

Resolve-DnsName -Type SRV "_ldap._tcp.dc._msdcs.$PartnerDomain"
Resolve-DnsName -Type SRV "_ldap._tcp.dc._msdcs.$CorpDomain"

$Trust = Get-ADTrust -Filter * -Server $CorpDomain |
    Where-Object Name -eq $PartnerDomain

if (-not $Trust) {
    throw "Trust to $PartnerDomain was not found."
}

$Trust |
    Select-Object Name,Direction,ForestTransitive,SelectiveAuthentication

if (-not $Trust.ForestTransitive) {
    throw 'The trust is not configured as a forest trust.'
}

if (-not $Trust.SelectiveAuthentication) {
    throw 'Selective authentication is not enabled. Do not continue with access testing.'
}
```

Validate the target computer object's ACL through **Active Directory Users and Computers** with **Advanced Features** enabled. Confirm that only the intended foreign group has **Allowed to authenticate** on the dedicated target.

Perform and record these three tests:

| Test | Expected result |
|---|---|
| Approved PARTNER user to approved CORP target/resource | Allowed |
| Approved PARTNER user to unrelated CORP target/resource | Denied |
| Unapproved PARTNER user to approved CORP target/resource | Denied |

## Evidence
Under `evidence/`, store conditional-forwarder settings, DNS queries, trust direction, the `SelectiveAuthentication=True` result, sanitized target-computer ACL evidence, group/resource assignment, all three access tests, errors/remediation, and final pass/fail status. Do not record trust credentials.

## Troubleshooting
- The remote forest is not found: repair conditional forwarding and routing first.
- Validation fails: verify DNS, time, trust direction and firewall connectivity.
- Resource permission is correct but logon is denied: verify **Allowed to authenticate** on the target computer object.
- Access is broader than intended: confirm selective authentication is enabled and remove broad resource or computer-object permissions.

## Security notes
A trust provides an authentication path, not automatic access. Use the narrowest direction, selective authentication, a dedicated foreign group, explicit target-computer authentication rights and least-privilege resource permissions.

## Cleanup
1. Remove the foreign group from the test resource ACL.
2. Remove its **Allowed to authenticate** right from the target computer object.
3. Remove disposable test users/groups.
4. Remove the trust and conditional forwarders only after both forest administrators confirm the rollback plan.

## Next activity
`AD-Learning-Path-52-Enable-the-Active-Directory-Recycle-Bin`
