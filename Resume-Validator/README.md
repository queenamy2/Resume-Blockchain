# Professional Credentials Verification Contract

## Overview

A decentralized smart contract system built on Stacks blockchain for verifying and managing professional credentials including education degrees, employment history, and skill certifications. Organizations can issue verifications, and individuals maintain verified profiles that employers and recruiters can trust for hiring decisions.

## Features

- Decentralized credential verification system
- Organization registration and verification
- User profile management
- Three types of credentials: Education, Employment, and Skills
- Credential verification and revocation mechanisms
- Read-only functions for credential validation
- Comprehensive access control system

## Contract Architecture

### Data Storage

The contract uses five primary data maps:

1. **registered-organizations**: Stores organization details and authorization
2. **user-profiles**: Stores individual user profile information
3. **education-records**: Stores education credentials
4. **employment-records**: Stores employment history
5. **skill-records**: Stores skill and certification credentials

### Error Codes

- `ERR-OWNER-ONLY (u100)`: Operation requires contract owner
- `ERR-NOT-FOUND (u101)`: Requested resource not found
- `ERR-UNAUTHORIZED-ACCESS (u102)`: Caller not authorized
- `ERR-ALREADY-EXISTS (u103)`: Resource already exists
- `ERR-INVALID-DATA (u104)`: Invalid data provided
- `ERR-CREDENTIAL-EXPIRED (u105)`: Credential has expired
- `ERR-INVALID-INPUT (u106)`: Invalid input parameters

## Key Functions

### Organization Management

#### register-organization
Registers a new organization that can issue credential verifications.

**Parameters:**
- `org-id`: Organization identifier (string-ascii 64)
- `name`: Organization name (string-ascii 100)
- `domain`: Organization domain (string-ascii 64)

**Access:** Public

#### verify-organization
Verifies an organization, allowing them to issue credential verifications.

**Parameters:**
- `org-id`: Organization identifier

**Access:** Contract owner only

#### update-organization
Updates organization information.

**Parameters:**
- `org-id`: Organization identifier
- `name`: Updated organization name
- `domain`: Updated organization domain

**Access:** Organization's authorized principal

### User Profile Management

#### register-profile
Creates or updates a user profile.

**Parameters:**
- `name`: User's full name (string-ascii 100)
- `email`: Email address (string-ascii 100)
- `profile-uri`: Optional metadata URI (string-utf8 256)

**Access:** User (tx-sender)

### Credential Management

#### add-education-credential
Adds an education credential to user's profile.

**Parameters:**
- `institution-id`: Issuing institution identifier
- `degree`: Degree title
- `field-of-study`: Field of study
- `start-date`: Education start date (timestamp)
- `end-date`: Education end date (timestamp)
- `metadata-uri`: Optional metadata URI

**Requirements:**
- User must have a profile
- Start date must be before or equal to end date

#### add-employment-credential
Adds an employment credential to user's profile.

**Parameters:**
- `organization-id`: Employer organization identifier
- `title`: Job title
- `description`: Job description (string-utf8 500)
- `start-date`: Employment start date
- `end-date`: Optional employment end date (none for current employment)
- `metadata-uri`: Optional metadata URI

**Requirements:**
- User must have a profile
- If end date provided, must be after start date

#### add-skill-credential
Adds a skill or certification credential to user's profile.

**Parameters:**
- `skill-name`: Name of skill or certification
- `issuer`: Optional issuer organization identifier
- `issue-date`: Certification issue date
- `expiry-date`: Optional expiry date
- `metadata-uri`: Optional metadata URI

**Requirements:**
- User must have a profile
- If issuer provided, organization must exist
- If expiry date provided, must be after issue date

### Credential Verification

#### verify-education-credential
Verifies an education credential.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `institution-id`: Verifying institution identifier

**Access:** Contract owner or authorized organization principal

**Requirements:**
- Organization must be verified
- Credential must be issued by the verifying organization

#### verify-employment-credential
Verifies an employment credential.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `organization-id`: Verifying organization identifier

**Access:** Contract owner or authorized organization principal

**Requirements:**
- Organization must be verified
- Credential must be issued by the verifying organization

#### verify-skill-credential
Verifies a skill credential.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `org-id`: Verifying organization identifier

**Access:** Contract owner or authorized organization principal

**Requirements:**
- Organization must be verified
- Credential must not be expired
- Credential must be issued by the verifying organization (if issuer specified)

### Credential Revocation

#### revoke-education-verification
Revokes an education credential verification.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `institution-id`: Institution identifier

**Access:** Contract owner or issuing organization

#### revoke-employment-verification
Revokes an employment credential verification.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `organization-id`: Organization identifier

**Access:** Contract owner or issuing organization

#### revoke-skill-verification
Revokes a skill credential verification.

**Parameters:**
- `profile-address`: Address of profile holder
- `credential-id`: Credential identifier
- `org-id`: Organization identifier

**Access:** Contract owner or issuing organization

## Read-Only Functions

### get-profile
Retrieves profile information for a given address.

**Parameters:**
- `address`: User's principal address

**Returns:** User profile data or none

### get-organization
Retrieves organization information by organization ID.

**Parameters:**
- `org-id`: Organization identifier

**Returns:** Organization data or none

### get-education-credential
Retrieves education credential details.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Education credential data or none

### get-employment-credential
Retrieves employment credential details.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Employment credential data or none

### get-skill-credential
Retrieves skill credential details.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Skill credential data or none

### is-education-credential-valid
Checks if an education credential is currently valid.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Boolean (true if verified)

### is-employment-credential-valid
Checks if an employment credential is currently valid.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Boolean (true if verified)

### is-skill-credential-valid
Checks if a skill credential is currently valid and not expired.

**Parameters:**
- `profile-address`: Profile holder's address
- `credential-id`: Credential identifier

**Returns:** Boolean (true if verified and not expired)

### is-verified-organization
Checks if an organization is verified and can issue verifications.

**Parameters:**
- `org-id`: Organization identifier

**Returns:** Boolean (true if verified)

## Usage Flow

1. **Organization Onboarding**
   - Organization calls `register-organization` to register
   - Contract owner calls `verify-organization` to verify the organization

2. **User Profile Creation**
   - User calls `register-profile` to create their profile

3. **Adding Credentials**
   - User calls `add-education-credential`, `add-employment-credential`, or `add-skill-credential` to add credentials
   - Credentials start as unverified

4. **Credential Verification**
   - Verified organizations call verification functions for credentials they issued
   - Credentials become verified and trusted

5. **Credential Validation**
   - Anyone can use read-only functions to check credential validity
   - Employers and recruiters can verify credentials on-chain

## Security Features

- Access control ensures only authorized parties can verify credentials
- Organizations must be verified by contract owner before issuing verifications
- Credential verifications can only be issued by the organization that issued the credential
- Revocation mechanism allows organizations to remove verifications
- Input validation on all parameters
- Expiry date checking for skill credentials