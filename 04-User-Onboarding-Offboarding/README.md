# 04 — User Onboarding & Offboarding

This section is the business-lifecycle layer of the lab.

## Onboarding workflow

1. Create the employee identity in Active Directory.
2. Place the account in the correct OU and security groups.
3. Join or assign the endpoint.
4. Apply Group Policy.
5. Synchronise identity to Microsoft Entra ID.
6. Enrol/manage the endpoint with Intune.
7. Assign the correct MedSecure application role.
8. Verify MFA, SSO and least-privilege application access.
9. Document completion.

## Offboarding workflow

1. Disable the user identity.
2. Revoke active sessions and cloud access.
3. Remove security-group and application-role assignments.
4. Recover, wipe or reassign the managed endpoint as appropriate.
5. Verify that MedSecure and organisational access has been removed.
6. Document the completed offboarding process.

> **Current evidence status:** the identity, GPO, hybrid identity, Intune and application-access components are already demonstrated elsewhere in this repository. A dedicated end-to-end offboarding test is still to be completed, so this section intentionally does not claim that workflow as finished evidence yet.
