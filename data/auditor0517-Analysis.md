## My findings
I've worked on this protocol for around 15 hours and found 2 highs and 5 mediums. I believe the high risks contain serious impacts and most mediums are related to DoS.

## My Approach
After understanding the protocol by adding comments as many as possible, I tried to find any vulns by comparing it with other governance protocols.

## Architecture recommendations
I think the current architecture is good and don't have any recommendations.

## Codebase quality analysis
I think it's good but not so perfect.

Especially, I've found a high risk related to the access control and I think it should be detected during the test.

Also, it seems to contain several DoS opportunities to be mitigated.

## Centralization risks
As the initial roles(BOOTSTRAP_ROLE) are set during the initialization, all roles might be controlled by the admin.

It means the admin can control all funds by executing the actions.

Anyway, I don't think it's critical as many other protocols contain centralization risks.

### Time spent:
15 hours