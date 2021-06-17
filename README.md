# decWAF

This lab is based on [the guidance for NGINX daclarative NAP configuration](https://github.com/MattDierick/Nginx-App-Protect-Policy/tree/master/policies).

The policy structure used is only an example and can be modyfied to meet your team's responsibilities.
![](https://github.com/dfs5/decWAF/blob/3c451d4c3e85781422237060f5338a5c37119a98/images/Screenshot_2021-06-16_at_17.28.42.png) 

This setup requires BIG-IP release 16.x and the [latest AS3 RPM](https://github.com/F5Networks/f5-appsvcs-extension/releases) package.

Usefull links:
- [AS3 documentation](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)
- [AS3 best practises](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/best-practices.html)
- [AS3 schema for Visual Code Studio or other editiors to compose AS3 declarations](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/validate.html#validate)
- [BIG-IP declarative WAF documentation](https://techdocs.f5.com/en-us/bigip-15-1-0/big-ip-declarative-security-policy/declarative-policy-getting-started.html#concept-4035)
- [BIG-IP Declarative WAF v16.0 reference and schema](https://clouddocs.f5.com/products/waf-declarative-policy/v16_0.html)

Run through the postman collection to deploy 2 different tenant configurations on a BIG-IP via AS3. Verify that the referenced profiles (e.g. "serverTLS": { "bigip": "/Common/dfs5.net_clientssl" }) are preconfigured on the BIG-IP. For the configuration of ASM log profile 'ELK_log' follow this [devcentral article](https://devcentral.f5.com/s/articles/Implementing-BIG-IP-WAF-logging-and-visibility-with-ELK). The dashboards provided by ELK can be used to get DevOps teams access to ASM logs for policy monitoring and tuning.

Variable being used in the Postman collection:

- "bigip_mgmt" Management IP address
- "bigip_username"
- "bigip_password"
- "bigip_token" used for token authentication onced token has been optained through basic auth 
- "bigip_token_timeout" can be extend up to 36000
- "policy_name" for the VIP declaration
- "policy2_name" for second VIP
- "tenant_name" is the partition on the BIG-IP where declartation gets deployed
- "link_to_policy" where the external policy declaration yaml file is hosted, e.g. GitLab
- "policy_key" defines the policy from which learning suggestions has to be exported

### Tenant 'sre05-devsec'
deploys 2 VIPS with waf policies (sre05_devsec_1dvwa.json and sre05_devsec_2dvwa.json) being prepopulated to the BIG-IP. You can use 'Import Polcy' from Postman step 2 or reference any existing policy on you BIG-IP.

### Tenant 'arcadia_team'
deploys 1 VIP with a WAF policy (arcadia_api_sec.json) being refrenced from external repository and an openAPI swagger file (openapi3-arcadia.yaml) referenced within the policy. The policy is in transparent mode and learning is enabled. Once client traffic is passing the policy learning suggestions will be generated.

## Working with learning suggestions.
In step 4 of the Postman collection learning suggestions can be accessed by the 'Export suggestions' REST call. In order to select the right policy for export use the policy id. The policy id can be found under 'List Policies' from step 2.

e.g.: 

        {
            "kind": "tm:asm:policies:policystate",
            "selfLink": "https://localhost/mgmt/tm/asm/policies/aZ4GDs7ADe8bSXeg_L_OdQ?ver=16.0.1",
            "name": "arcadia_api",
            "id": "aZ4GDs7ADe8bSXeg_L_OdQ"
        },

- Copy and paste this id into the {{policy_id}} variable and run the call. 
- Next run 'Export Suggestions' to get the suggestions listed in json format. If you want to filter on suggestions that have reached a specific level (e.g. >90%) run the 'Filter suggestions for export' POST call and set the score level to match your requirements (e.g.: "filter": "score gt 90")
- Now you can copy&paste them into the main policy declaration 'arcadia_api_sec.json' under the 'modifications' section. 
- Finaly re-apply the entire 'arcadia api sec pol' declaration to apply the learning suggestions to the policy. You can varify the modifications in the BIG-IP UI.

In order to speed up the learning process adjust the whitelist section n the 'arcadia_api_sec.json' policy to match your client IP.

        "whitelist-ips": [
			{
				"ipAddress": "10.24.0.0", 
				"ipMask": "255.255.0.0",
				"trustedByPolicyBuilder": true
			}
		]

If you want to disable signatures generating false positives you can use the modification section of the 'arcadia_api_sec.json' policy. 

        {
        "modifications": [
            {
                "entityChanges": {
                    "enabled": false
                },
                "entity": {
                    "signatureId": 200001834
                },
                "entityType": "signature",
                "action": "add-or-update"
            },
            {
                "entityChanges": {
                    "enabled": false
                },
                "entity": {
                    "signatureId": 200004461
                },
                "entityType": "signature",
                "action": "add-or-update"
            }
        ]
        }
