Subject: API Integration Instructions and Postman Collection

Dear [Recipient's Name],

I hope this email finds you well.

Please find attached the Postman collection file for your reference. Below, I've outlined the steps to call the API and provided example URLs and screenshots for your convenience.

Gateway URL: [gateway url]

Implement URL: [implement url]

1. Get Payee List
URL: [implement url]/getPayeeList
Description: Supports SAML3 token
Screenshot: [Insert screenshot here]
2. Add Contact
URL: [implement url]/addContact
Description: Only supports IB2B
Note: The add account and add contact API URLs are the same. We determine the action based on the presence of contactId in the contactAddressList and financialAddressList within the request body. If it is an add contact request, a contactId will be generated after adding.
Screenshot: [Insert screenshot here]
3. Add Account
URL: [implement url]/addAccount
Description: Only supports IB2B
Screenshot: [Insert screenshot here]
If you have any questions or need further assistance, please don't hesitate to contact me.
