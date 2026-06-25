# AD Learning Path 51 — Configure and Validate a Forest Trust

## Goal
Create cross-forest DNS resolution and a controlled trust between `corp.lab` and `partner.lab`, then prove that only an explicitly assigned test resource is accessible.

## Prerequisites
- Both forests healthy
- Routed lab connectivity
- DNS and time working in both forests
- Trust direction documented
- Separate authorized administrators

## Steps
1. In DNS Manager for `corp.lab`, create a conditional forwarder for `partner.lab` that points to `PDC01`.
2. In `partner.lab`, create a conditional forwarder for `corp.lab` that points to the CORP DNS servers.
3. Test host and SRV record resolution in both directions.
4. Open **Active Directory Domains and Trusts** on the forest-root domain.
5. Create a forest trust with only the direction required by the lab scenario.
6. Validate the trust from each forest.
7. Create normal test groups in both forests.
8. Assign the partner test group to one dedicated CORP lab resource.
9. Verify the intended access succeeds and unrelated resources remain unavailable.

## Validation
```powershell
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.partner.lab
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.corp.lab
Get-ADTrust -Filter * -Server corp.lab |
    Select-Object Name,Direction,ForestTransitive,SelectiveAuthentication
```
Use the Domains and Trusts console to run the trust validation from both sides.

## Evidence
Under `evidence/`, store conditional-forwarder settings, DNS queries, trust properties, validation screenshots, group/resource assignment, intended-access test, unrelated-access test, errors/remediation, and final pass/fail status. Do not record trust credentials.

## Troubleshooting
- The remote forest is not found: repair conditional forwarding and routing first.
- Validation fails: verify DNS, time, trust direction, and firewall connectivity.
- Resource access fails: check group membership and the target resource permissions.

## Security notes
A trust provides an authentication path, not automatic access. Use the narrowest direction and explicit resource permissions.

## Cleanup
Remove test access and groups, then remove the trust and forwarders only after both forests are ready.

## Next activity
`AD-Learning-Path-52-Enable-the-Active-Directory-Recycle-Bin`
