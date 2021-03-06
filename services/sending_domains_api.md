# Group Sending Domains

**Note:** The Sending Domains API has reduced functionality in SparkPost Elite.

A sending domain is a domain that is used to indicate who an email is from via the "From:" header. Using a custom sending domain enables you to control what recipients see as the From value in their email clients. DNS records can be configured for a sending domain, which allows recipient mail servers to authenticate your messages. The Sending Domains API provides the means to create, list, retrieve, update, and verify a custom sending domain.

## Sending Domains in SparkPost Elite

In SparkPost Elite, the Sending Domains API can only be used to associate a tracking domain with a sending domain. None of the other attributes are currently used in SparkPost Elite.

It is currently optional to register a sending domain with the Sending Domains API in SparkPost Elite. Sending domains are not currently verified in SparkPost Elite, and it is currently possible to send from an unverified sending domain.

## Sending Domain Attributes

**Note:** "dkim" is currently ignored in SparkPost Elite. "status" will not currently affect whether messages can be sent from a sending domain in SparkPost Elite.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|domain    |string  |Name of the sending domain   | yes |The domain name will be used as the "From:" header address in the email.|
|tracking_domain | string | Associated tracking domain | no | example: "click.example1.com" |
|status | JSON object| JSON object containing status details, including whether this domain's ownership has been verified  | no | Read only. For a full description, see the Status Attributes.|
|dkim | JSON object| JSON object in which DKIM key configuration is defined|no| For a full description, see the DKIM Attributes.|

### DKIM Attributes

**Note:** "dkim" is currently ignored in SparkPost Elite. DKIM keypairs are currently configured by the Operations team. DKIM keys that are specified via the API are not currently used.

DKIM uses a pair of public and private keys to authenticate your emails. The DKIM key configuration is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|private | string | DKIM private key | yes | The private key will be used to create the DKIM Signature.|
|public | string |DKIM public key  | yes | The public key will be retrieved from DNS of the sending domain.|
|selector | string |DomainKey selector | yes | The DomainKey selector will be used to indicate the DKIM public key location.|
|headers | string| Header fields to be included in the DKIM signature |no | Header fields are separated by a colon.  Example: `"from:to:subject:date"`|

### Status Attributes

**Note:** SparkPost Elite does not currently verify sending domains. The verification status of a sending domain will not affect sending. It is currently possible to send from an unverified domain in SparkPost Elite.

Detailed status for this sending domain is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Default   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|ownership_verified | boolean | Whether domain ownership has been verified |false |Read only. This field will return "true" if any of dkim_status, spf_status, abuse_at_status, or postmaster_at_status are "true".|
|dkim_status | string | Verification status of DKIM configuration |unverified|Read only. Valid values are "unverified", "pending", "invalid" or "valid".|
|spf_status | string | Verification status of SPF configuration |unverified |Read only. Valid values are "unverified", "pending", "invalid" or "valid".|
|abuse_at_status | string | Verification status of abuse@ mailbox |unverified |Read only. Valid values are "unverified", "pending", "invalid" or "valid".|
|postmaster_at_status | string | Verification status of postmaster@ mailbox |unverified |Read only. Valid values are "unverified", "pending", "invalid" or "valid".|
|compliance_status | string | Compliance status | | Valid values are "pending", "valid", or "blocked".|

### Verify Attributes

These are the valid request options for verifying a Sending Domain:

| Field         | Type     | Description                           | Required  | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|dkim_verify | boolean | Request verification of DKIM record | no | |
|spf_verify | boolean | Request verification of SPF record | no | |
|postmaster_at_verify | boolean | Request an email with a verification link to be sent to the sending domain's postmaster@ mailbox. | no | SparkPost.com only |
|abuse_at_verify | boolean | Request an email with a verification link to be sent to the sending domain's abuse@ mailbox. | no | SparkPost.com only |
|postmaster_at_token | string | A token retrieved from the verification link contained in the postmaster@ verification email. | no | SparkPost.com only |
|abuse_at_token | string | A token retrieved from the verification link contained in the abuse@ verification email. | no | SparkPost.com only |

### DNS Attributes

| Field         | Type     | Description                           |
|------------------------|:-:       |---------------------------------------|
|dkim_record | string | DNS DKIM record for the registered Sending Domain |
|spf_record | string | DNS SPF record for the registered Sending Domain |
|dkim_error | string | Error message describing reason for DKIM verification failure |
|spf_error | string | Error message describing reason for SPF verification failure |

## Create and List [/sending-domains]

### Create a Sending Domain [POST]

Create a sending domain by providing a **sending domain object** as the POST request body.

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
            "domain" : "example1.com",
            "tracking_domain" : "click.example1.com",
            "dkim" : {  "private" : "MIICXgIBAAKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQABAoGBAITb3BCRPBi5lGhHdn+1RgC7cjUQEbSb4eFHm+ULRwQ0UIPWHwiVWtptZ09usHq989fKp1g/PfcNzm8c78uTS6gCxfECweFCRK6EdO6cCCr1cfWvmBdSjzYhODUdQeyWZi2ozqd0FhGWoV4VHseh4iLj36DzleTLtOZj3FhAo1WJAkEA68T+KkGeDyWwvttYtuSiQCCTrXYAWTQnkIUxduCp7Ap6tVeIDn3TaXTj74UbEgaNgLhjG4bX//fdeDW6PaK9YwJBAM6xJmwHLPMgwNVjiz3u/6fhY3kaZTWcxtMkXCjh1QE82KzDwqyrCg7EFjTtFysSHCAZxXZMcivGl4TZLHnydJUCQQCx16+M+mAatuiCnvxlQUMuMiSTNK6Amzm45u9v53nlZeY3weYMYFdHdfe1pebMiwrT7MI9clKebz6svYJVmdtXAkApDAc8VuR3WB7TgdRKNWdyGJGfoD1PO1ZE4iinOcoKV+IT1UCY99Kkgg6C7j62n/8T5OpRBvd5eBPpHxP1F9BNAkEA5Nf2VO9lcTetksHdIeKK+F7sio6UZn0Rv7iUo3ALrN1D1cGfWIh2dj3ko1iSreyNVSwGW0ePP27qDmU+u6/Y1g==",
                "public" : "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                "selector" : "brisbane",
                "headers" : "from:to:subject:date"
            }
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results" : {
                "message" : "Successfully Created domain.",
                "domain"  : "example1.com"

            }
        }

+ Response 400 (application/json)

      { 
        "errors": [
          {
            "message": "invalid params",
            "description": "Tracking domain '(domain)' is not a registered tracking domain",
            "code": "1200"
          }
        ]
      }

+ Response 422 (application/json)

    {
      "errors" : [
        {
          "message": "invalid data format/type",
          "description": "Error validating domain name syntax for domain: '(domain)'",
          "code": "1300"
        }
      ]
    }

### List all Sending Domains [GET]

List an overview of all sending domains in the system.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json; charset=utf-8)

        {
            "results" : [
                {
                    "domain": "example1.com",
                    "tracking_domain": "click.example1.com"
                },
                {
                    "domain": "example2.com"
                }
            ]
        }

## Retrieve, Update, and Delete [/sending-domains/{domain}]

### Retrieve a Sending Domain [GET]

Retrieve a sending domain by specifying its domain name in the URI path.  The response includes details about its DKIM key configuration.

+ Parameters
  + domain (required, string, `example1.com`) ... Name of the domain

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "tracking_domain": "click.example1.com",
                "status": {
                    "ownership_verified": false,
                    "spf_status": "pending",
                    "abuse_at_status": "pending",
                    "dkim_status": "pending",
                    "compliance_status": "pending",
                    "postmaster_at_status": "pending"
                },
                "dkim": {
                    "headers": "from:to:subject:date",
                    "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                    "selector": "hello_selector"
                }
            }
        }


### Update a Sending Domain [PUT]

Update the attributes of an existing sending domain by specifying its domain name in the URI path and use a **sending domain object** as the PUT request body.

If a tracking domain is specified, it will replace any currently specified tracking domain.  If the supplied tracking domain is a blank string, it will clear any currently specified tracking domain. Note that if a tracking domain is not specified, any currently specified tracking domain will remain intact.

If a dkim object is provided in the update request, it must contain all relevant fields whether they are being changed or not.  The new dkim object will completely overwrite the existing one.

+ Parameters
    + domain (required, string, `example1.com`) ... Name of the domain

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
            "tracking_domain": "click.example1.com",
            "dkim" : {  "private" : "MIICXgIBAAKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQABAoGBAITb3BCRPBi5lGhHdn+1RgC7cjUQEbSb4eFHm+ULRwQ0UIPWHwiVWtptZ09usHq989fKp1g/PfcNzm8c78uTS6gCxfECweFCRK6EdO6cCCr1cfWvmBdSjzYhODUdQeyWZi2ozqd0FhGWoV4VHseh4iLj36DzleTLtOZj3FhAo1WJAkEA68T+KkGeDyWwvttYtuSiQCCTrXYAWTQnkIUxduCp7Ap6tVeIDn3TaXTj74UbEgaNgLhjG4bX//fdeDW6PaK9YwJBAM6xJmwHLPMgwNVjiz3u/6fhY3kaZTWcxtMkXCjh1QE82KzDwqyrCg7EFjTtFysSHCAZxXZMcivGl4TZLHnydJUCQQCx16+M+mAatuiCnvxlQUMuMiSTNK6Amzm45u9v53nlZeY3weYMYFdHdfe1pebMiwrT7MI9clKebz6svYJVmdtXAkApDAc8VuR3WB7TgdRKNWdyGJGfoD1PO1ZE4iinOcoKV+IT1UCY99Kkgg6C7j62n/8T5OpRBvd5eBPpHxP1F9BNAkEA5Nf2VO9lcTetksHdIeKK+F7sio6UZn0Rv7iUo3ALrN1D1cGfWIh/Y1g==",
                "public" : "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                "selector" : "hello_selector",
                "headers" : "from:to:subject:date"
            }
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results" : {
                "message" : "Successfully Updated Domain.",
                "domain" : "example1.com"
            }
        }

+ Response 400 (application/json)

      {
        "errors": [
          {
            "message": "invalid params",
            "description": "Tracking domain '(domain)' is not a registered tracking domain",
            "code": "1200"
          }
        ]
      }

+ Response 422 (application/json)

    {
      "errors" : [
        {
          "message": "invalid data format/type",
          "description": "Error validating domain name syntax for domain: '(domain)'",
          "code": "1300"
        }
      ]
    }

### Delete a Sending Domain [DELETE]

Delete an existing sending domain.

**Warning:** Before deleting a sending domain please ensure you are no longer using it. After deleting a sending domain, any new transmissions that use it will result in a rejection. This includes any transmissions that are in progress, scheduled for the future, or use a stored template referencing the sending domain.

+ Parameters
  + domain (required, string, `example1.com`) ... Name of the domain

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 204

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Domain 'wrong.domain' does not exist"
                }
              ]
            }


## Verify [/sending-domains/{domain_name}/verify]

### Verify a Sending Domain [POST]

**Note:** While it is possible to call this endpoint in SparkPost Elite, the verification status of a sending domain will not affect sending. It is currently possible to send from an unverified domain in SparkPost Elite.

The verify resource operates differently depending on the provided request fields:
  * Including the fields "dkim_verify" and/or "spf_verify" in the request initiates a check against the associated DNS record type for the specified sending domain.
  * Including the fields "postmaster_at_verify" and/or "abuse_at_verify" in the request results in an email sent to the specified sending domain's postmaster@ and/or abuse@ mailbox where a verification link can be clicked.
  * Including the fields "postmaster_at_token" and/or "abuse_at_token" in the request initiates a check of the provided token(s) against the stored token(s) for the specified sending domain.

The domain's "status" object is returned on success.

+ Request Verify DKIM and SPF (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "dkim_verify" : true,
            "spf_verify"  : true
        }
        ```


+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": true,
                "spf_status": "valid",
                "dns": {
                    "dkim_record": "k=rsa; h=sha256; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                    "spf_record": "v=spf1 include:sparkpostmail.com ~all"
                },
                "compliance_status": "pending",
                "dkim_status": "valid",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified"
            }
        }

+ Request Initiate postmaster@ email (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "postmaster_at_verify" : true
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": false,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified"
            }
        }

+ Request Verify postmaster@ correct token (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "postmaster_at_token" : "rcayptmrczdnrnqfsxyrzljmtsxvjzxb"
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": true,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "valid"
            }
        }

+ Request Verify abuse@ incorrect token (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "abuse_at_token" : "AN_INCORRECT_OR_EXPIRED_TOKEN"
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": false,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified"
            }
        }

+ Request Unable to process abuse@ request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "abuse_at_verify" : true
        }
        ```

+ Response 400 (application/json; charset=utf-8)

        {
           "errors": [
              {
                 "message": "Failed to generate message",
                 "description": "Failed to generate verification email",
                 "code": "1901"
              }
           ]
        }
