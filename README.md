[Google Apps (G Suite)](https://developers.google.com/identity/protocols/OpenIDConnect), [Microsoft Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code), and [GitHub](https://developer.github.com/apps/building-oauth-apps/authorization-options-for-oauth-apps/) authentication for [CloudFront](https://aws.amazon.com/cloudfront/) using [Lambda@Edge](http://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html). The primary use case for `cloudfront-auth` is to serve private S3 content over HTTPS without running a proxy server to authenticate requests.

## Description
Upon successful authentication, a cookie ( named `TOKEN`) with the value of a newly created and signed JWT is set and the user redirected back to the originally requested path. This JWT cookie is checked for validity (signature, expiration date, audience and matching hosted domain) upon each request and `cloudfront-auth` will redirect the user to their OAuth2 provider's login when necessary.

## Usage
1. If your CloudFront distribution is pointed at an S3 bucket, [configure origin access identity](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html#private-content-creating-oai-console) so S3 objects can be stored with private permissions
1. Clone or download this repo
1. Create an OAuth 2.0 client with your provider.
1. Lambda@Edge does not support environment variables, so a config file is necessary to be included in the Lambda upload ZIP. Execute `make` and enter your OAuth2 provider's necessary credentials when prompted (below).  If RSA public/private keys do not exist under the names id_rsa/id_rsa.pub respectively, new ones will automatically be generated.  
   1. Google: Client ID, Client Secret, Redirect URI, Hosted Domain, Token Age (seconds)
   1. Microsoft: Tenant, Client ID (App ID), Client Secret, Redirect URI, Token Age (seconds)
   1. GitHub: Client ID, Client Secret, Redirect URI, Token Age (seconds), Organization
1. Following the initial credentials, you may be asked about AuthZ options.
   1. Hosted Domain (*Google*)
      1. Only check that verified email ends with hosted domain. Leave 'Enter email lookup URL' blank.
   1. HTTP Email Lookup (*Google*)
      1. Check that verified email exists in JSON array from given site. Enter email lookup URL when prompted.
   1. Google Groups Lookup (*Google*)
      1. Check that verified email exists in one of given Google Groups.  Download your service account JSON file and rename to 'google-groups-authz.json' (example below).  Place it in the root directory and add the key 'cloudfront_authz_groups' whose value is a JSON array of groups to check.  The Makefile will detect this file exists and will not prompte you to enter an email lookup URL.
   1. Organization Membership Lookup (*GitHub*)
      1. Verify the user that logged in is a member of the given organization.
1. Upload the resulting `cloudfront-auth.zip` to your Lambda function.
1. Configure CloudFront to use the Lambda function upon **viewer request**:

   1. Edit CloudFront distribution behavior![alt text](https://lh3.googleusercontent.com/T4b26lGh3yu4SSxXAG3Vb63iuWxTXkqgFTiXNp5i-NCGQ6AgH_Lal5CYse6gZJOpjSK8xKi9kuF8niPKbqjbrTFYDB7n6ZNv-mANWytL_zatFwDamFQZ_1RnDnEAGkXfrKONRNfJh6w8qjLHKuCk1JWnqsIWYnIr44J2j6wFKceasggPxnh8IfhC869-Pz3GRC6AvURWLOVoQWZI5tp7NQ6U4NGZ-dI-bEjOSTqx96PEnlbIY4r-Js76SgbKI_94aow5eMXmhbGFcsheUIZ5jRXJ6NT9Z3SpPEw0tvJwqDEs5UyM8xva_Ghb33EsV3bfDzZbaKoCXk3diKnBCV5BTpfx8szaiOxiqHZY8wfFEZfkeZi-sZECSAECcnXcIWVEGId52vjtQmNi0krfwcAUSHzkEMB3E3jHMH2fd8q3Pp8YO5w1A2wgAE_SDVuT6JRS-i1vFoRx-OkfSpNI4kdY7Uh4MxvP6fR_hNVPCxilM9y0D_S8ln7MWAPE_7V3RkV214SObk_PoU4dW3u67PD1BUfD8kR96Kf6UV8s5IhM61ks9u1PvbFj822y51CWAhTRe02tcwPdB9Km0jbYXYgzkPFkzPXCYCKeTLCg0m2m4HAUS5SL7P3ftYN98FyOdYYrbtmYiJtwatH6gjwfyX6ENc2rDMa4A8Q=w1684-h586-no "Edit behavior")
   1. Point to Lambda function on viewer request (bottom of edit page)![alt text](https://lh3.googleusercontent.com/9YGTDMxX-9q_3GhW-w9ORcWejG3ZoQUBhviVb3_Dr1iCuvbmvSHM0WXLZ5UrlvUzkuDcfBtJJMqF5C7kWdJuG5P2abOiBNhLoxTF41oQqOzyWofio6TCTW_56SjjaMCzDyocusbx9GzOaJNHAWIIvDXByLwfHCaWQf7VcGdBx4WnwKwvq5_08Pv2G2JIkznTRzSrpd6KbMpkSUT7H3dOO-mZbPEl6NKvmIJ0iAW834R4KSx0gHEtzTLYu6FPN0oWHkQwGHh2x4kmBaSp1WyxaE98okVe3QMZ_bYPt2NDVSQHuPcd3mOQAjJBNnyBoq5zgJYe5r5AdSbyIJ7bfJDthUcqk_ZL67DJ39_NkFrdyJN2A5n5Iunn2axtN7vMlsi54WxfcQFpxTs3x_2QPRYGEaYUnjuLVpS7ZdlDgp3-46pUqEISCOAVb5wMU2lY4KFEdEiSOccKcvjuyK25GxvDvGkZTR5xP6DRm8A6uOmQbOEEL5M9OMB0_OS5pMW_DWAnXeqwHSLZk42Wc58YyJlLSZ0WBnFPvAHoEuV2N-mYL6NhKSoLBEK_HM6TyEH03SolS6baVyTH_cPSDwya-N7EQtnyM1aL3WKaKv6V_ETTH3g8zOB-EydUbjpEEPyUJrjqFsrHNQieeksEGIWe0gqX93r7FpxiLXk=w1528-h298-no "Point to Lambda function on viewer request")

## Examples
### google-groups-authz.json
```
{
  "type": "service_account",
  "project_id": "example",
  "private_key_id": "h54h8t1eg65s1d6fg1re81r651g",
  "private_key": "-----BEGIN PRIVATE KEY-----\ndh54et5aa4rg5d4fht5e4h5d4fg5sdf54h5sh65s1651h51s\n-----END PRIVATE KEY-----\n",
  "client_email": "cloudfront-google-authz@example.iam.gserviceaccount.com",
  "client_id": "452521516513132321315",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/cloudfront-google-authz%40example.iam.gserviceaccount.com",
  "cloudfront_authz_groups": [ "foo@example.com", "bar@example.com" ]
}

```
### HTTP Email Lookup Site
```
["foo@gmail.com","bar@gmail.com"]
```

## Build Requirements
 - [npm](https://www.npmjs.com/) ^5.6.0
 - [node](https://nodejs.org/en/) ^7.10.0
 - [openssl](https://www.openssl.org)

## Contributing
All contributions are welcome. Please create an issue in order open up communication with the community.

When implementing a new flow or using an already implemented flow, be sure to follow the same style used in `build.js`.  The config.json file should have an object for each request made.  For example, `openid.index.js` converts config.AUTH_REQUEST and config.TOKEN_REQUEST to querystrings for simplified requests (after adding dynamic variables such as state or nonce). For implementations that are not generic (most), endpoints are hardcoded in to the config (or discovery documents).
