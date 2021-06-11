# decWAF

work in progress

Run through the postman collection to deploy 2 different tenant configurations on a BIG-IP via AS3.

Tenant 'sre05-devsec'
deploys 2 VIPS with waf policies being prepopulated to the BIG-IP. You can use 'Import Polcy' from Postman step 2 or reference any existing policy on you BIG-IP.

Tenant 'arcadia_team'
deploys 2 VIP with a WAF policy being refrenced 