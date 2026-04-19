curl --request POST \
--url http://localhost:8000/api/login_check \
--header 'Content-Type: application/json' \
--data '{
"username": "{role}@bigproject.com",
"password": "Password123@"
}'

You can replace {role} by : student/admin/superadmin/recruiter/guest/hr/teacher


Then you can use the bearer token to execute the other curl commands

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/konstantine-garozashvili) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
