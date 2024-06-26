As discussed today regarding the GMT project, task-9999 add account, the request body for financial-account is missing the contact address object. Both papi and sapi financial-account requests lack this contact address object.

Since the contact address is a part of contact, we can use financial-contact to edit it.

The proposed logic is as follows:

If the upstream request body doesn’t include a contact ID, we will assume there is no existing payee and create a new one for the user.
If the upstream request body includes a contact ID, we will edit the existing payee’s contact address and add a new account.
Can I book some time with you to confirm this logic? We will also discuss the HK FCL during this meeting.
