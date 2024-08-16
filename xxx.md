Subject: Switching to IAM Role for RDS Authentication

Hey [Team/Everyone],

To reduce the number of times DB passwords are accessed and minimize exposure risks (like password leaks or unauthorized access), we’ve set up a new role, xxx2, and enabled IAM role-based access for our RDS instances. You can now use tokens to log in instead of traditional passwords.

In the future, the xxx1 role will also have IAM role-based access enabled. Once this is in place, password-based logins will no longer work for that role. The tokens we use are temporary and valid for [token validity period], so you’ll need to generate a new one each time. I’ve put together a quick guide below on how to generate the token and set up your local environment.

Also, based on our discussion with [XXX], we’ll be using service accounts moving forward to obtain the necessary permissions for our DB profile role. The service account will also use the same method to generate short-lived tokens.

Feel free to reach out if you have any questions!
